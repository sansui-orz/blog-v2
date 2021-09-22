# Hash路由以及History路由的实现原理以及优缺点

[tag]:hash|history|router
[create]:2019-10-08

## Hash路由的实现原理

通过变更location.hash来进行路由的改变，通过window.onHashChange来监听hash的变化，再根据hash判断需要显示什么页面

## History路由的实现原理

通过HTML5提供的History API来实现URL的变化。

其中主要的两个API:

```javascript
history.pushState();
history.replaceState();
```

这两个API可以在不刷新页面的情况下，操作浏览器的历史记录。

例：

```javascript
window.history.pushState(null, null, path);
window.history.replaceState(null, null, path);
```

其他常用API:

```javascript
window.history.back(); // 后退
window.history.forward(); // 前进
window.history.go(2); // 跳转到指定页面，当前为0
window.history.length // 历史堆栈中页面的数量
```

监听history的路由变化:

```javascript
window.addEventListener(‘pop state’, function() {
  var currentState = history.state;
});

// 或者
window.onpopstate = function() {}
```

## Hash路由的优缺点

优点:

- 兼容性好，基本上浏览器都支持（我还没遇见不支持的目前， ie8以上）

- 不需要服务端额外的设置和开销

缺点:

- 服务器无法准确捕获路由的信息

- 对于需要锚点功能的需求会与路由冲突（即a标签内#id，进行页面内定位）

- 一定要hash不一样才会触发hashchange

## History路由的优缺点

优点:

- 后端可以准确的获取到url的数据

- 相同的url也可以触发popstate

缺点:

- 兼容性不如hash

- 需要后端支持

## history之pushState，replaceState详解

### pushState接收三个参数(状态对象，标题，url)

- 状态对象: 一个与指定网址相关的状态对象，popstate触发时，该对象会传入回调函数, 如果不需要这个对象，直接传null就可以了

- 标题：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填null。

- url: 新的网址，必须与当前页面处于同一个域。浏览器地址栏将显示这个网址。

```javascript
window.history.pushState({title: '首页1'}, '首页2', '/index');
window.addEventlister('popstate', function(e) { // 或者window.onpopstate
    const state = e.state;
    document.title = state.title; // 首页1
  });
```

> 注意 pushState 绝对不会触发hashchange事件，即使仅仅新的url的hash与旧的url的hash不一样

## replaceState

> replaceState的传参与pushState一样，唯一不同的是replaceState不会产生新的历史记录，而是直接替换掉当前记录

未完待续

## 参考文档

<https://developer.mozilla.org/zh-CN/docs/Web/API/History_API>
<https://router.vuejs.org/zh/guide/essentials/>
<history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90>
