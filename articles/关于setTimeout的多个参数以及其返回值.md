# 关于setTimeout的多个参数以及其返回值

[tag]:记录|js|setTimeout
[create]:2019-11-11

关于setTimeout，我们，大部分人使用的时候可能都是这样:

```javascript
setTimeout(function() {
  doSomething();
}, 1000);
```

其实setTimeout可以传两个以上参数这件事估计很多人都不知道或者快要忘记了。其实setTimeout可以传无限个参数，除了第一第二个参数外，其他参数都将作为参数传入调用函数,如下:

```javascript
setTimeout(function(arg1, arg2, arg3, arg4, arg5) {
  console.log(arg1, arg2, arg3, arg4, arg5);
}, 1000, 1, 2, 3, 4, 5);
// 1, 2, 3, 4, 5
```

还有有些人会这样写setTimeout:

```javascript
var t = setTimeout(...);

componentWillUnmount() {
  if (t) clearTimeout(t);
}
```

其实这个用法本身并没有什么问题，但是就是需要搞懂，setTimeout会返回一个正整数，表示定时器的编号，就算你clearTimeout了，t本身并不会变成null或者undefined，它会始终是个正整数。所以如果确实需要清除掉t的值，需要clearTimeout后再重新复制null / undefined。

这里还需要注意的是，setTimeout / setInterval是共用一个编号池的，所以请不要编号混用以免造成不必要的麻烦。
