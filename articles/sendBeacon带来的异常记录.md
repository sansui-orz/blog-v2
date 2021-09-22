# sendBeacon带来的异常记录

[tag]:bug记录|js
[create]:2020-01-13

## 场景描述

在一个列表类型的项目打点中，遇到了一个问题。使用sendBeacon发送几十条请求后，移动端抓包出现抓不到数据请求的情况。

1. 初步怀疑代码异常：

接着进行代理断点调试，在代码中进行断点，发现代码运行无异常，确实是sendBeacon调用成功，但是就是无法发送数据。

结论：排除了代码出错的可能。

2. 怀疑抓包工具限制了单域名请求数：

通过更换抓包工具抓包，以及刷新页面后请求发送正常，以及更换其他浏览器，发现并无异常。

结论：排除这个可能性。

3. 怀疑客户端限制了单个域名下请求数：

更换为xhr进行请求的发送，并无异常。表示并没有对单域名请求数进行限制。

结论：排除这个可能性。

4. sendBeacon api异常：

然后将问题定位到sendBeacon本身api的异常上，通过查w3c与MDN文档，并没有发现对请求次数有限制。且在qq浏览器与pc端chrome也正常。

结论：排除这个可能性。

5. 浏览器本身对sendBeacon的实现有问题：

最后怀疑是浏览器本身对于sendBeacon的实现上与规范有差异，采用对比的方法测试了多款移动浏览器。 最终发现内部几个浏览器均出现sendBeacon发送异常，而chrome, qq浏览器并没有这个情况。

将问题定位到浏览器实现上之后，向内核同学反馈了该情况。

结论：sendBeacon在chrome低版本中，会对单个页面内通过sendBeacon发送的请求进行累积，累积发送数据达到64k（准确值是: 64 * 1024 = 65536）的时候，将不能在发送请求。这个bug在内核xx以上的版本上修复了这个问题。

得到内核同学的这个结论后，查了相关文档

查了之后发现chrome的内核代码中确实有这么个限制

<https://chromium.googlesource.com/chromium/blink/+/master/Source/modules/beacon/NavigatorBeacon.cpp#79>

<https://chromium.googlesource.com/chromium/blink/+/master/Source/core/frame/Settings.in#290>

<https://chromium.googlesource.com/chromium/blink/+/master/Source/core/loader/BeaconLoader.cpp#81>

相关的sendBeacon文档中也有这么几段描述：

<https://w3c.github.io/beacon/>

> The sendBeacon() method is not intended to provide background synchronization or transfer capabilities. The user agent restricts the maximum accepted payload size to ensure that beacon requests are able to complete quickly and in a timely manner.
> Compared to the alternatives, the sendBeacon() API does apply two restrictions: there is no callback method, and the payload size can be restricted by the user agent. Otherwise, the sendBeacon() API is not subject to any additional restrictions. The user agent ought not skip or throttle processing of sendBeacon() calls, as they can contain critical application state, events, and analytics data. Similarly, the user agent ought not disable sendBeacon() when in "private browsing" or equivalent mode, both to avoid breaking the application and to avoid leaking that the user is in such mode.

**很明显是我们对于sendBeacon的运用有误：**

sendBeacon适用于当页面卸载的时候发送一些打点请求，或者向服务器发送一些记录用户信息的内容，目的是用以解决当页面卸载时，通过xhr发送异步请求可能丢失的问题。

并且sendBeacon对于是否成功将请求插入到待发送队列是有返回值的，成功为true, 失败为false。如果在非页面卸载的场景下使用sendBeacon，应该注意是否成功将请求插入，充分考虑失败的情况。而在之前，我们并没有注意到这一点。

且尽管sendBeacon有返回值以判断是否成功调用，但对于该请求是否成功发送，以及发送时机，响应结果都无法得知，使用该api是应充分考虑到这一点。

所以这个bug的出现主要原因是因为我们对于sendBeacon的滥用，在平时打点时应该主要还是使用xhr，只有当页面卸载时需要发起一些请求时才使用sendBeacon。

反思：

使用一些新的api时我们不应该贪图它的方便或是新颖。而更多的应该从其场景出发，这个api适用的场景是什么，是否有什么限制，使用是否合理，需要注意什么，当这个api出现问题是否可以做兼容处理。

注意：

有一个点需要注意，在翻sendBeacon的文档时发现页面的unload事件并不可靠，在项目中不应该过度的依赖这个事件，而visibilitychange事件始终是可靠的。

这是因为当在移动端的时候，浏览器进入后台后，系统可能会终止该进程。所以当此时不进入浏览器而直接将浏览器退出，就会出现unload事件没有触发的情况。

而visibilitychange却始终触发。

参考文档：

<https://chromium.googlesource.com/chromium/blink/+/master/Source/modules/beacon/NavigatorBeacon.cpp#79>
<https://chromium.googlesource.com/chromium/blink/+/master/Source/core/frame/Settings.in#290>
<https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon>
<https://w3c.github.io/beacon/>
<https://web.dev/disallow-synchronous-xhr/>
<https://github.com/w3c/beacon/pull/39>
<https://stackoverflow.com/questions/28989640/navigator-sendbeacon-data-size-limits>

测试代码:

```js
var url = 'https://play.google.com/log?asds=ddecss';
var n = 65536;

function sendBeaconBySize(size) {
    var data = new Array(size).join('X');
    var result = navigator.sendBeacon(url, data);
    if (result) {
        console.log('发送成功', size);
    } else {
        console.log('发送失败', size);
    }
}
```
