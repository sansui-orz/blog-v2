# 应用缓存与PWA

[tag]:web|cache|pwa
[create]:2020-08-07

应用缓存（AppCache）将在chrome85以上删除，在Firefox44以上弃用，Safari在2018年弃用。

至于被弃用的原因是“AppCache无法部分更新，且有些用例会导致安全性以及严重的可用性问题”。

而现在最新版的浏览器基本都已经弃用了`AppCache`，改为采用`Service Worker`管理页面缓存。

AppCache与Service Worker是互斥的，任何用了Service Worker的页面将禁用`AppCache`。所以要从`AppCache`迁移到`Service Worker`务必将所有缓存文件迁移过去，没有被包含在`Service Worker`也是无法使用`AppCache`的。

## 使用service worker

关于service worker的使用网上有很多[文章](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps/Offline_Service_workers)了，我就不再赘述。

使用service worker实现pwa网站是很简单的，它难就难在如何管理你的更新，让你的更新能更快的触达用户。(关于更新这块还能单独再拎出来讲很多，这里先不展开，后面再补)

这里主要讲讲service worker使用中需要注意的事情。

1. 修改了静态资源，必须要更新service worker脚本，才能使其更新。

2. 资源要使用完整的路径

我写了一个简单的可使用的demo可以参考[传送门](https://sansui-orz.github.io/blog/demos/pwa/)

## 添加到主屏幕

将pwa项目添加到主屏幕有几个条件:

1. 网站需要使用https协议(可以借用github进行调试)

2. html头部需要链接正确的manifest文件，例`<link ref="manifest" href="./manifest.json" />`

3. 合适的图标可显示在主屏上（这里我之前设置了一个128\*128的图标导致无法添加到屏幕，后面设置了144\*144就可以了，原因成谜，参考[Why is my 'add to home screen' Web App Install Banner not showing up in my web app](https://stackoverflow.com/questions/43003424/why-is-my-add-to-home-screen-web-app-install-banner-not-showing-up-in-my-web-a)采用的解法。

4. 网站需要使用service worker

### manifest文件格式

```json
{
  "name": "pwa2 Home Screen App",
  "short_name": "测试pwa",
  "desription": "测试pwa添加到桌面",
  "icons": [
    {
      "src": "icons/icon_x_128.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "icons/icon_x_128.png",
      "sizes": "128x128",
      "type": "image/png"
    }
  ],
  "start_url": "/blog/demos/pwa2/index.html",
  "display": "fullscreen",
  "theme_color": "#B12A34",
  "background_color": "#B12A34"
}
```

可以打开[在线访问添加到主屏幕（需要翻墙）](https://sansui-orz.github.io/blog/demos/pwa2/)查看线上效果，建议使用手机版chrome,或者华为自带浏览器，这两个浏览器我自测是可以弹出“添加至桌面的弹窗的”，其他浏览器未知。

但是添加到主屏幕还是有一个很严重的问题“不能保证可以顺利的添加到屏幕”。

主要有两点原因：

1. 如果用户没有打开手机浏览器“允许创建快捷方式”的权限，那么将不会弹出添加框（**主动添加也无法添加**），并且在我的华为手机上该权限是默认关闭的（其他手机不太确定）。

2. 如果用户添加到桌面之后再删除，**之后再打开网站也不会弹出添加框**（需要主动添加）【注：经了解，似乎浏览器会在一定时间后再次弹出，这个时间是三个月？】。

3. 如果用户无响应，则会每隔1.5天可重新弹一次（未验证）。

这两个场景都很影响用户添加应用到桌面的行为。如果使用强引导去引导用户添加，则会加大用户的使用成本，且对于体验很不友好。所以这种方式目前在我看来只适用于强依赖性的站点，例如组织内部网站，或是有手段能够强制用户添加到桌面的应用（比如某些学校会要求家长装一些指定教育类APP），基于这种强制性手段添加才能万无一失，至于要求游客这么做完全就是难为人了。

## 使用通知api

[通知api](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)的使用十分简单，使用只需要两步:

1. 请求用户授权

2. 新建通知

### 请求用户授权

Notification的requestPermission方法只能通过用户行为调用，比如click, 且该方法会返回一个promise, 返回值为`denied, granted, default`三种，需要判断用户允许直接判断`result === 'granted'`。

```js
btn.onclick = function() {
  // 获取通知权限
  Notification.requestPermission().then(function(result) {
    /** result
      * denied 用户拒绝了通知
      * granted 用户允许通知
      * default 不知道用户的选择，行为与denied一样
    */
    if (result === 'granted') {
      notice();
    }
  });
};
```

### 发起通知

Notification的实例可以监听点击时间，关闭事件等，当通知关闭时最好将事件回调清除，避免浏览器无法回收导致内存泄漏。

```js
function notice() {
  const notification = new Notification('通知标题' + Math.floor(Math.random() * 100), {
    body: '通知体',
    icon: '/demos/notice-push/imgs/icon_x_128.png'
  });
  notification.onclick = function(e) {
    notification.close(); // 主动关闭
  };

  notification.onshow = function() {};

  notification.onerror = function() {};

  notification.onclose = function() {};
}
```

具体代码可以查看[notificate-demo](../demos/notice-push/notice.html), 或者查看[效果](https://sansui-orz.github.io/blog/demos/notice-push/notice.html)

## 推送

推送要比通知复杂很多，因为推送需要结合service worker。且目前推送api仍然处于初级阶段。

【注：推送的数据太大可能会导致推送失败，尽量数据大小在1K以内】

（推送待补充）

## 参考资料

[Application Cache 就是个坑](http://zoomzhao.github.io/2012/11/11/application-cache-is-a-douchebag/)

[使用应用缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache)

[应用程序缓存](https://www.chromestatus.com/feature/6192449487634432)

[Preparing for AppCache removal](https://web.dev/appcache-removal/)

[渐进式 Web 应用（PWA）](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps)

[通过 Service workers 让 PWA 离线工作](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps/Offline_Service_workers)

[添加到主屏幕](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps/%E6%B7%BB%E5%8A%A0%E5%88%B0%E4%B8%BB%E5%B1%8F%E5%B9%95)

[Why is my 'add to home screen' Web App Install Banner not showing up in my web app](https://stackoverflow.com/questions/43003424/why-is-my-add-to-home-screen-web-app-install-banner-not-showing-up-in-my-web-a)

[谨慎处理 Service Worker 的更新](https://juejin.im/post/6844903792522035208)

[有关 Service Worker 更新的两点改进](https://github.com/lavas-project/lavas/issues/212)

[notification](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)
