# Object.create

[tag]:记录|js
[create]:2019-12-03

该方法创建了一个新对象，使用现有的对象来提供新创建的对象的__proto__

如下例所示，obj1作为了b的__proto__。

```javascript
var obj1 = {a: 1};
var b = Object.create(obj1);
b.a; // 1
b; // {}
obj1.a = 2;
b.a; // 2
b.__proto__ === obj1; // true
```

create方法其实还带有第二个参数，表示要添加到新创建对象的属性值, 例子:

```javascript
var a = {a: 1};
var b = Object.create(a, {
  b: {
    writable: true,
    configurable: true,
    value: 2,
  },
  c: {
    configurable: false,
    get: function() { return 10; },
    set: function(val) { console.log('set val', val); }
  }
});
b.b; // 2
b.c; // 10
b.a; // 1
```

然后Object.create还被经常用来创建空对象:

```javascript
var nullObj = Object.create(null);
var normalObj = {};
typeof normalObj.toString; // 'function'
typeof nullObj.toString; // 'undefined'
```
