# event.composedPath事件记录

[tag]:记录|js|composedPath
[create]:2019-12-27

获取事件的路径，即具体元素到window对象的路径

例如:

```html
<html>
    <body>
        <button>click</button>
    </body>
</html>
```

```javascript
document.querySelector('button').onclick = function(e) {
    console.log(e); // [button, body, html, window]
};
```

兼容性: chrome > 53, ie不支持, safari > 10, firefox > 52
