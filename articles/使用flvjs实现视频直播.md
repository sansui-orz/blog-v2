# html5视频直播

[tag]:video|flv
[create]:2020-08-11

参考[使用flv.js与video.js实现播放视频直播(简教程)](https://www.jianshu.com/p/d9c66d7d1653)文章，采用flv.js与video.js构建html5直播。

## 目前web直播技术

- m3u8(HLS)

- FLV（需要在前端转码）

- WebRTC

- rtmp(仅支持flash播放)

- DASH

## 兼容性

### m3u8

m3u8在移动端以及safari中兼容表现良好，而在pc端基本上不兼容，需要在pc端进行兼容处理（如使用video.js)

特点：直播时移高，部分浏览器可原生支持播放

### FlV

目前使用flv格式的视频直播需要借助`Media Source Extensions`api对视频进行解码，可以使用video.js或者flv.js这类库进行视频播放，实测在pc以及安卓进行播放都没有问题。

特点：直播时移小，一定要借助转码才能播放

### WebRTC

该api还在研究，只是听说它可以用作直播，但是我从MDN了解到的是它被设计来是用作在线视频连线，即在线会议这种场景的使用，还不确定是否能用于视频直播。

### rtmp

仅支持flash播放，但是从查资料过程中似乎见到“video.js可以编译rtmp格式的视频用于html5播放”的说法，待考证。

特点：直播时移小

## 搭建简单直播体验

### 安装livego

[livego](https://github.com/gwuhaolin/livego/blob/master/README_cn.md)是一个简单高效的直播服务器。

1. 直接下载编译好的[二进制文件](https://github.com/gwuhaolin/livego/releases), 然后在命令行中执行

2. 从Docker启动：`docker run -p 1935:1935 -p 7001:7001 -p 7002:7002 -p 8090:8090 -d gwuhaolin/livego`

3. 从源码编译

- 下载源码 `git clone https://github.com/gwuhaolin/livego.git`
- 到livego目录中执行`go build` (每安装go的先去[安装](https://golang.org/dl/))

我这里采用的是第三种，直接从源码编译，编译完之后在命令行中输入`./livego`就可以跑起来了。

![livego](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200813150908.jpg!trans_webp)

### 使用ffmpeg进行推流

[ffmpeg](https://ffmpeg.org/about.html)是领先的多媒体框架，能够解码，编码， 转码，mux，demux，流，过滤和播放人类和机器创建的几乎所有内容。

安装ffmpeg，在mac中推荐使用brew进行安装`brew install ffmpeg`, 简单又方便。

由于我电脑上没有摄像头，所以这里需要准备一个输入文件: [demo.flv](https://s3plus.meituan.net/v1/mss_7e425c4d9dcb4bb4918bbfa2779e6de1/mpack/default/demo.flv) --- 这个文件链接直接使用[这篇文章内提供的](https://github.com/gwuhaolin/livego/blob/master/README_cn.md)

准备好ffmpeg与demo.flv之后，开启推流。

首先获取一个房间的channelKey。

打开浏览器，访问<http://localhost:8090/control/get?room=movie>, 输入如下：

![channelKey](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200813151706.jpg!trans_webp)

其中`rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk`就是channelKey。

推流：`ffmpeg -re -i demo.flv -c copy -f flv rtmp://localhost:1935/{appname}/{channelkey}`, appname默认是live, channelkey我们上面已经拿到了。

正确跑起来之后是下面这个效果:

![ffmpeg](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200813154217.jpg!trans_webp)

### 使用flv.js进行flv流直播

[flv](https://github.com/Bilibili/flv.js)是bilibili开发的进行flv转码的js库。

新建一个空白html页面。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>html5在线直播</title>
</head>
<body>
  <script src="https://cdn.bootcdn.net/ajax/libs/flv.js/1.5.0/flv.min.js"></script>
  <video id="videoElement"></video>
  <script>
    var videoElement = document.getElementById('videoElement');
    var flvPlayer = flvjs.createPlayer({
        type: 'flv',
        isLive: true,
        url: 'http://localhost:7001/live/movie.flv'
    });
    flvPlayer.attachMediaElement(videoElement);
    flvPlayer.load();
    videoElement.onclick = function() {
      if (flvjs.isSupported()) {
          flvPlayer.play();
      }
    };
  </script>
</body>
</html>
```

开启一个静态资源服务器访问页面，这里我使用[http-server](https://www.npmjs.com/package/http-server)

打开页面后点击播放，就可以看到直播视频已经在播放了。

![live](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200813154957.jpg!trans_webp)

### 使用m3u8/hls播放

由于hls在safiri跟移动端天然支持，所以可以不使用任何库直接调用，livego开启的直播服务器同样支持hls格式的视频源。

简单改一下页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>html5在线直播</title>
</head>
<body>
  <video id="videoElement" preload="auto" controls>
    <source src="http://localhost:7002/live/movie.m3u8" type="application/x-mpegURL">
  </video>
</body>
</html>
```

然后直接使用safari打开后效果如下：

![safari](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200813163633.jpg!trans_webp)

## flv源码流程分析

对flv.js源码与整体架构感兴趣的可以看看我简单整理出来的flv调用流程图, 这里我看的是1.5.0的版本

![flv调用流程图](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/flvjs.jpg!trans_webp)

其实整个flv.js最核心的方法也就是视频的编解码，其他的多用点心研究一下还是挺好理解的。

做个简单的模块讲解:

- browser.js表示的是调用者的js代码。

- flv包含flv.js对外暴露的接口。

- flv-player主要是flv的表层实现。

- mse-controller对于Media Source的具体实现与控制。

- MediaSource表示MediaSource的实例。

- transmuxer是对于web worker事件的转发，（如果不支持web worker则直接与transmuxer Controller进行通信。

- transmuxer controller对转码流程控制。

- flv-demuxer对flv视频格式进行解码。

- mp4-remux对解码后的数据进行mp4格式编码。

- io-controller对接transmuxer controller与服务端返回数据，进行数据的吞吐操作。

- FetchStreamLoader则是对于请求方法的具体实现与兼容，其提供了fetch/websocket/xhr等多种loader。

以上模块分析并不一定准确，毕竟我也没有太仔细的深入研究，主要也是我对于视频解码与编码这块并没有一点基础，所以也就没有深入。只是希望能够帮助到下一个想要研究一下flv实现的人。

## 参考资料

[使用flv.js与video.js实现播放视频直播(简教程)](https://www.jianshu.com/p/d9c66d7d1653)

[livego README](https://github.com/gwuhaolin/livego/blob/master/README_cn.md)

[使用flv.js做直播](https://github.com/gwuhaolin/blog/issues/3)

[Web直播，你需要先知道这些](https://github.com/imweb/blog/issues/2)

[H5直播实现方案](https://github.com/Tiramisupxl/blog/issues/1)

[WebRTC直播技术(一)-初探WebRTC](https://imweb.io/topic/5930541b7720c3b21fa5c303)

[WebRTC API](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)

[直播原理与web直播实战](https://juejin.im/post/6844903582790057997)
