# window.requestIdleCallback

[tag]:记录|js
[create]:2019-12-05

该方法将指定回调插入一个事件队列，该队列内的事件将在浏览器空闲的时候被调用。

## 使用方法

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)

```javascript
var handle = window.requestIdleCallback(callback[, options]);
```

- 返回值: 一个无符号长整数，可传入window.cancelIdleCallback来结束回调

- callback: 回调，会接受一个参数。参数不说

- options: 可选，具有timeout属性的对象，timeout表示最晚多少毫秒后调用，如果超时没调用，则会在下一次空闲时强制调用。

***注意***该事件队列先进先出，如果超时了的，则优先调用。这样会打乱事件顺序
