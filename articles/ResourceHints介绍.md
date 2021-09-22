# Resource Hints

[tag]:resource hints|link|html
[create]:2020-07-22

Respirce Hints规范定义了`<link>`标签的`dns-prefetch`, `preconnect`, `prefetch`, `prerender`属性值如何控制资源加载。

## dns-prefetch（dns预解析）

一个请求的完整发出应该包括"dns解析，tcp链接，（tsl验证），http请求发送"这几步，而dns-prefetch则可以先帮你将"dns解析"这部分工作提前，进而减少实际请求资源的耗时。

它的用法如下：

```html
<link rel="dns-prefetch" href="https://images.example.com">
```

当浏览器识别到dns-prefetch的标识时，尽管它不知道如何使用该域名，也会尽快解析。

![未使用dns-prefetch](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/dns-prefetch01.jpg!trans_webp)

上图为未使用dns-prefetch是的请求情况，可以看到dns解析时间占了总请求时间的一半。

![使用了dns-prefetch](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/dns-prefetch02.jpg!trans_webp)

使用了dns-prefetch后，很明显的看到请求时间变短了。

## preconnect（预链接）

预链接与dns预解析类似，都是提前进行链接的一部分。不同点在于`preconnect`不仅仅只是做了dns解析这部分，而是将整个链接建立好，但还未发请求出去。

如果预链接建立之后发送该域名下的请求，则会发现它可以直接来到发送数据这一阶段，因为前面的“DNS解析，TCP握手，包括https中的TLS验证“都已经准备完成了。

```html
<link rel="preconnect" href="https://scripts.example.com">
```

但是需要注意的是，每个浏览器都有最大连接数限制，不要超过最大连接数，一般来说，最大连接数在5～10个左右。不要因为preconnect把链接占用了，导致其他资源请求无法发出。

第二个需要注意的点是，在http 1.1中，虽然keep-alive能保持链接不被关闭，但是每个链接仅能传输一个http请求，只有在http2中只吃了多路复用的问题之后，才能达到preconnect的最佳实践效果。

![preconnect](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/preconnect.jpg!trans_webp)

可以看到，使用了preconnect之后连续发送三个请求，只有第一个命中了connect，后面两个仍然是新建一个connect。

## prefetch (预获取)

prefetch会以最低的优先级去下载资源，它下载完成的时机是无法确定的，加载的资源类型参照preload允许的类型，且没有同源限制。

一般的应用场景在预加载下一个页面（如在商品列表预加载商品详情），或者是轮播图（使用了图片懒加载的下一张轮播图）等不需要明确加载成功或失败的场景。

```html
<link rel="prefetch" href="https://example.com/news/?page=2" as="document">
```

![prefetch实践1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/prefetch01.jpg!trans_webp)

![prefetch实践2](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/prefetch02.jpg!trans_webp)

从上面两张图可以看出，当prefetch之后再次使用该资源会直接使用缓存内容，而prefetch发起请求的实际是当请求空闲的时候。

如例子中，一开始解析html文档树会加载两张图片，分别是资源5与资源6，而等这些资源优先加载了之后才会开始加载prefetch的资源。

当使用prefetch将资源加载到本地之后，对于该资源的请求都会命中`prefetch cache`。

## prerender (预渲染)

之前的都是网络的预链接或资源的预加载，而prerender则可以真正的将页面跑起来，除了没有展示出来之外，页面会渲染，js会执行。

但是如果用户没有点击进入预渲染的页面，这样做会浪费cpu浪费流量，尽管用户不知道，但是这是一个损害用户的行为，并且如果资源过大或者消耗cpu很厉害，预渲染行为将被停止。

并且预渲染的兼容并不好，仅有chrome/ie11/edge对其支持度还不错。虽然即使不支持也不会产生什么错误，但是可能会让你收集回来的性能报告不准确。

总的来说，目前prerender并不成熟。尽管如果用上它你的下一个页面会变得很快。

```html
<link rel="prerender" href="https://example.com/news/?page=2">
```

## preload（预加载）

> 注意这不在Resource Hints规范中，它有自己本身的规范，相见底部参考文档中的preload链接

相较于prefetch, preload的兼容性更好，加载优先级更高，可设置Accept请求头，受同源策略限制（但是可以设置crossorigin），可用媒体查询动态加载，且支持监听资源加载成功或失败。

```html
<link rel="preload" href="https://example.com/news/?page=2" as="document">
```

当解析到preload时，浏览器会尽快加载该资源。

![preload1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/preload1.jpg!trans_webp)

可以看到preload资源在dom解析完成前就已经发出了。（img标签资源会在dom解析是发出，所以preload应该也是遇到就发出）

利用其加载优先级高的特性，我们可以将某些急需加载的模块提前，比如重要的脚本，主图，css等。提前它们的加载时机。

需要注意的是，加载字体文件时，会卡住其他页面的渲染，导致页面白屏，直至字体加载完毕。

## 结语

可以看到对于Resource Hints运用的好的话，对于前端页面的加载性能是非常有帮助的。即使只是进行dns预解析在一个同时使用多个域名资源的项目中的优化效果仍然是非常可观的。

还需要注意的点是，这些优化手段与其他方式结合将会更加有效，比如preload加载资源时，如果同时加载十几二十个同域的资源，则会触发浏览器同域下最大并发数的限制（据我所知，chrome是6个），此时如果结合将资源部署到不同域名下的策略的话，那么就可以同时发出这么多请求，本来需要等待(n/x)时间，修改之后则只需要(n/n)的时间。

![拆分域名](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/split-domain.jpg!trans_webp)

可以看到，拆分域名之后，`cdn-brilio-net.akamaized.net`达到了最大并发数（6个）的限制，但是`hl-img.peco.uodoo.com`域名下的请求并不受影响。

## 参考链接

[Optimizing Performance With Resource Hints](https://www.smashingmagazine.com/2019/04/optimization-performance-resource-hints/)

[Resource Hints "W3C Working Draft 02 July 2019"](https://www.w3.org/TR/resource-hints/)

[preload](https://www.w3.org/TR/preload/)

[通过rel="preload"进行内容预加载](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Preloading_content)
