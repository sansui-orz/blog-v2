# animationend & transitionend

[tag]:记录|css|js|animation
[create]:2020-08-03

之前一直不知道原来js可以监听动画结束与变换结束，所以一直采用的是setTimeout去做动画结束之后的逻辑。

## animationend

### animationend用法

用法很简单，只需要给animation的元素添加事件监听就可以了。

```javascript
document.querySelector('.demo').addEventListener('animationend', function(e) {
  // to do ...
}, false);
```

### 坑

如上例，如果.demo中有子元素也拥有animation变换的话，那么同样会触发回调，所以在回调需要判断一下触发该事件的元素`event.target.className`或者其他什么可以判断身份的api就ok.

## transitionend

### transitionend用法

```javascript
document.querySelector('.demo').addEventListener('transitionend', function(e) {
  // to do ...
}, false);
```

### 另一个坑

如果你的transition设置成多个属性，或者all, 当你多个属性进行变换时，会多次触发transitionend事件，即每个属性变化之后触发一次。

解决方式就是通过`event.propertyName`判断变换的属性名。
