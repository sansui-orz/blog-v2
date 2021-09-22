# generator详解

[tag]:generator|es6
[create]:2021-03-21

相信大家对generator都不是很陌生了，在现在的前端开发中，即使你对它并不熟悉，但是大概率你是使用过它的（即使你使用的是async/await或者promise，最终可能也是通过babel等工具转成generator，当然可能你并没有发现这一点）

## 简单的用法

generator的用法非常简单，但是其中涉及到的知识却非常的有意思（没错，我觉得实在是太有趣了）。很多在async/await或promise中不是很方便实现的功能，在generator看来都是小意思。

```js
function *foo() {
  const s = 10 * (yield 10);
  return s;
}

const f = foo()
f.next(); // { value: 10, done: false }
f.next(5); // { value: 50, done: true }
```

可以看到上面的代码，这是一个很简单的关于generator函数的使用实例。一切看起来似乎非常正常，但是仔细分析函数的运行你会察觉到一点有意思的东西 -- 它的输入与输出。

在foo函数中，我们在代码运行到赋值语句时，通过`yield`关键字暂停了函数的运行，并且输出数值10，在恢复执行这个函数执行的时候我们又传入了一个数值5。（就是`f.next(5)`）

这是不是很有趣？一个函数居然拥有多次输出，并且可以在函数执行期间多次注入值。以往我们的js函数都是非常简单的，在调用的时候传入实参，然后等待函数运行结束再接受其返回值。

但是generator打破了这个规则，它允许你在函数执行期间暂停执行，然后再次输入变量去控制接下来的执行流程，最终得到不一样的执行结果，这一点非常有意思。比如下面这段实例代码：

```js
var a = 1;
var b = 2;
function *foo() {
  a++;
  yield;
  b = b * a;
  yield;
  a = b + 3;
}
function *bar() {
  b--;
  yield;
  a = 8 + b;
  yield;
  b = a * b;
}
// 该段代码节选自《你不知道的Javascript 中卷》 p241
```

考虑上面的实例代码根据不同的调用顺序，最终可以产生多少种a和b的值。我们可以简单的尝试一下：

```js
var fo = foo()
var ba = bar()
fo.next(); // a = 2, b = 2
ba.next(); // a = 2, b = 1
fo.next(); // a = 2, b = 2
ba.next(); // a = 10, b = 2
fo.next(); // a = 5, b = 2
ba.next(); // a = 10, b = 2
// a = 10, b = 2
```

看到了吧，两个generator函数同时更改a,b变量的值，根据两种调用顺序的不同排列，可以产生处不同的结果。而使用async/await或者promise，你是很难实现这种交叉执行的。（因为你很难在方式执行过程中的控制它们，而这对generator来说则很简单）

## 生成器与迭代器的概念

generator的意思是生成器，而这个生成器生成的则是一个迭代器。比如在上例中，我们执行foo函数，得到的就是一个迭代器，foo函数则是一个生成器了。

1. ”生成器 - 迭代器 - interable“ 间的关系
2. for of循环的异常结束会终止迭代器的挂起状态。
