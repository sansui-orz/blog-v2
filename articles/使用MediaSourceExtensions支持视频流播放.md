# 使用Media Source Extensions支持视频流播放

[tag]:video|MSE|stream
[create]:2020-08-24

[Media Source Extensions](https://developer.mozilla.org/zh-CN/docs/Web/API/Media_Source_Extensions_API)提供了实现无插件且基于 Web 的流媒体的功能。

简单来说，MSE起到video标签与视频流数据之间桥梁的作用。

那么为什么需要这个桥梁呢？

因为浏览器对视频格式的支持非常有限，尤其是一些流式的视频，在MSE出现之前，这类视频格式的支持主要依靠Flash等插件，但是由于Flash臃肿，不安全等原因，现代浏览器已经逐渐不再支持，Chrome也在2020-12后停止对flash的支持。所以MSE也就应运而生。

前面也说到了，MSE在前端视频播放中起到的作用仅仅是作为VIDEO与视频数据之间的桥梁，让视频数据流能够通过MSE传递给VIDEO进行播放，可是浏览器不是不支持这类的流式视频格式吗？为什么这里又能将视频数据传递给VIDEO进行播放呢？

这里主要涉及到视频相关的较为深入一丢丢的知识了，一个视频主要有两个比较主要的因素组成，一个是它的封装格式，一个是它的编码格式。例如mp4/mov这类的叫封装格式，也就相当于将视频数据装进一个叫 mp4或mov的盒子里，其间的区别主要在于盒子的形状，容量，质量等会不一致，但是里面装的视频数据还是一样的。那么编码格式讲的就是这个盒子里装的视频数据了，其实也就相当于视频的压缩算法，现在常见的编码格式有h263/h264/h265/av1等，当然，浏览器对于这些编码格式也不是全部支持，但是大部分对于一些常见的编码是支持的，比如h264.

基于封装格式和编码格式这两点，再加上MSE，我们就可以实现在浏览器上播放大多数的视频格式。比如用h264编码的flv流式视频在Chrome上是不支持播放的，但是h264编码的mp4却可以，那么我只需要在前端拿到flv视频数据的时候将视频数据从flv这个视频盒子里拿出来，然后再放进一个新的mp4的盒子，这样浏览器是不是就可以播放了呢？答案是肯定的。

但是这时候就有个问题了，因为流式视频，流式视频，那么它传输的时候肯定也是流式传输数据的，我们怎么将这些流式的数据一点点的传递给video呢？要知道我们之前的video的使用仅仅是给它一个src就不需要我们管了。但是对于流式视频显然不能这样。这时候就要到我们的MSE登场了, MSE能够提供多个SourceBuffer（资源容器）,每当你收到一段视频数据的时候，你就可以将这部分的数据append到对应的SourceBuffer里了。

最后参考[MSE（Media Source Extensions）的一点尝试](https://blog.csdn.net/weixin_41196185/article/details/82229244), 自己搭建了简单的一个视频流播放（不包含视频转码）。

[源码地址](https://github.com/sansui-orz/blog/tree/master/demos/video-live/mse)

## 搭建服务端视频流服务

搭建一个node的视频流服务，这里为了简单，不使用任何依赖，直接终端运行`node server`跑服务。

文件名：***server.js***

```js
const http = require('http');
const fs = require('fs');

const port = 3001;

const server = http.createServer((req, res) => {
  if (/\.mp4$/.test(req.url)) {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'application/octet-stream');
    const stream = fs.createReadStream(__dirname + '/frag_bunny.mp4');
    stream.pipe(res);
  } else {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/html; charset=utf-8');
    res.end(fs.readFileSync(__dirname + '/index.html'));
  }
});

server.listen(port, () => {
  console.log('open: http://localhost:' + port);
});
```

可以看到上面的服务很简单，当访问`.mp4`后缀的路由时，流式返回视频流数据。否则返回我们的静态页面。

但是需要注意不是所有mp4文件都可以流式传输的，使用普通mp4进行流式播放会报错，需要使用[fregament mp4](https://blog.csdn.net/lyuan1314/article/details/9289827)格式的视频才行。

这里我直接使用的是[MSE（Media Source Extensions）的一点尝试](https://blog.csdn.net/weixin_41196185/article/details/82229244)内提供的一个[fragament mp4文件](https://raw.githubusercontent.com/nickdesaulniers/netfix/gh-pages/demo/frag_bunny.mp4)。 当然你也可以通过 ffmpeg自己生成你想要的视频，ffmpeg是一个很强大的视频处理工具库，市面上很多的视频相关应用都是基于它进行开发的，有兴趣的可以深入研究一下这一块。

通过

```js
res.setHeader('Content-Type', 'application/octet-stream');
const stream = fs.createReadStream(__dirname + '/frag_bunny.mp4');
stream.pipe(res);
```

简单将该mp4视频流下发。

紧接着开始写播放页面。

文件名 ***index.html***

```html
<!-- 省略 -->
<body>
  <video id="video"></video>
  <button>开始</button>
  <!-- https://blog.csdn.net/weixin_41196185/article/details/82229244 -->
  <script>
    let inited = false;
    let done = false;
    let intervalKey;
    const bufferCache = [];
    const video = document.getElementById('video');
    const mediaSource = new MediaSource();
    video.src = URL.createObjectURL(mediaSource);

    mediaSource.addEventListener('sourceopen', function() { // 实例已附加到媒体元素，准备接收或者正在接收数据 https://developers.google.com/web/fundamentals/media/mse/basics
      if (inited) return;
      inited = true;
      const sourceBuffer = mediaSource.addSourceBuffer('video/mp4;codecs="avc1.42E01E,mp4a.40.2"');
      sourceBuffer.addEventListener('updateend', function() {
        if (done && bufferCache.length === 0) {
          mediaSource.endOfStream();
        }
      });
      function intervalAppendSourceFromCache() {
        intervalKey = setInterval(() => {
          if (bufferCache.length > 0 && !sourceBuffer.updating) {
            console.log('定时器定时查询sourceBuffer状态并将缓存写入', bufferCache);
            const bufferOnHead = bufferCache.shift();
            sourceBuffer.appendBuffer(bufferOnHead);
          }
          if (bufferCache.length === 0 && done) {
            clearInterval(intervalKey);
          }
        }, 200);
      }

      intervalAppendSourceFromCache();

      fetch('./video.mp4').then(function(res) {
        console.log('查看返回直播数据流', res);
        return res.body.getReader();
      }).then(readStream);

      function readStream(resp) {
        return resp.read().then((result) => {
          console.log('将数据流读出来', result);
          done = result.done;
          if (!result.done) {
            if (sourceBuffer.updating) {
              console.log('sourceBuffer正在处理，先讲数据丢进缓存列表', bufferCache);
              bufferCache.push(result.value.buffer);
            } else {
              if (bufferCache.length > 0) {
                const bufferOnHead = bufferCache.shift();
                console.log('从缓存列表获取缓存插入到sourceBuffer', bufferOnHead);
                bufferCache.push(result.value.buffer);
                sourceBuffer.appendBuffer(bufferOnHead);
              } else {
                console.log('正常将流数据插入sourceBuffer', result.value.buffer);
                sourceBuffer.appendBuffer(result.value.buffer);
              }
            }
            return readStream(resp); // 递归调用读取resp
          } else {
            console.log('done');
          }
        });
      }
    });
    document.querySelector('button').onclick = function() {
      video.play();
    };
  </script>
</body>
<!-- 省略 -->
```

上面的数据获取以及数据插入的逻辑稍微复杂一丢丢，可以参照下面的流程图理解:

![流程图1](../demos/video-live/mse/lct.drawio.png)

需要注意在代码中获取到视频数据并不是直接将数据添加到sourceBuffer的，而是简单的维护了一个缓存数组，在使用一个定时器去循环查询sourceBuffer的状态是否是updateing的状态，这是因为如果网络很流畅的情况下，fetch的回调会短时间内大量触发，而sourceBuffer一次只能处理一段视频数据，所以其他的数据则需要等待前面的数据处理完毕之后才能够添加给sourceBuffer。

还有就是sourceopen回调的触发时机，在网上搜索了挺久都没有看到哪里有介绍sourceopen是什么时候触发回调的，最后在[Media Source Extensions](https://developers.google.com/web/fundamentals/media/mse/basics)这篇文章发现，该回调会在**实例已附加到媒体元素，准备接收或者正在接收数据**时触发。

将play事件绑定在button的click事件内则是因为chrome不允许网页自动播放，必须要在用户有主动触发的行为才行。

## 总结

就这样一个简单的视频流播放就实现了，尽管它只有简单的播放实现，但是了解到mse是怎么工作的，就对其他视频转码插件如flv.js/hls.js/dash.js等有了一个简单的认识，使用起来更加得心应手。

## 更多功能

1. 实现视频回放功能。

2. 在fetch回调到appendBuffer之间进行数据转换，以支持更多视频格式。

3. 构思一下清晰度切换应该如何实现。

## 参考资料

[Media Source Extensions](https://developers.google.com/web/fundamentals/media/mse/basics)

[MSE（Media Source Extensions）的一点尝试](https://blog.csdn.net/weixin_41196185/article/details/82229244)

[Media Source Extensions](https://developer.mozilla.org/zh-CN/docs/Web/API/Media_Source_Extensions_API)
