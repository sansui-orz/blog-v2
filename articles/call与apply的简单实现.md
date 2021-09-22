# call与apply的简单实现

[tag]:记录|js
[create]:2020-07-30

原理是通过“谁调用函数，函数就指向谁”的原理，将方法绑定到传入的this上。

## call的实现

```javascript
Function.prototype.myCall = function() {
  const [_this, ...args] = arguments;
  _this = Object(_this) || window; // 强制将_this转成对象
  const fnName = Symbol(); // 保证不与指定对象内属性冲突，使用symbol
  _this[fnName] = this; // 将当前方法添加到指定对象上, 这样方法里的this才能指向该对象
  const result = _this[fnName](...args); // 调用
  delete _this[fnName]; // 主动删除指定this上的引用，防止内存泄漏
  return result;
}
```

## apply的实现

```javascript
Function.prototype.myApply = function() {
  const [_this, args] = arguments; // 仅仅只是入参形式不一样。
  _this = Object(_this) || window;
  const fnName = Symbol();
  _this[fnName] = this;
  const result = _this[fnName](...args);
  delete _this[fnName];
  return result;
}
```

## bind的实现

```javascript
Function.propotype.myBind = function() {
  const [_this, ...args] = arguments;
  const fn = this;
  return function(...args2) {
    // 其实里面返回函数可以直接使用call, apply实现，但是既然前面已经有实现了，直接用实现也可以。
    _this = Object(_this) || window;
    const fnName = Symbol();
    _this[fnName] = fn;
    const result = _this[fnName](...args, ...args2);
    delete _this[fnName];
    return result;
  };
}
```

需要注意，通过箭头函数绑定的this无法再更改。因为编译时会直接将函数外的this声明成变量再通过作用域链引用，如果需要更改，只能修改父级作用域的this。
