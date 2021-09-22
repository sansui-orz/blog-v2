# try_catch的作用范围与其返回值

[tag]:记录|js
[create]:2019-11-11

对于try catch应该每个人都知道，但是可能大部分人都没有详细的了解过它。

try catch的基本用法

```javascript
try {
  doSomethig();
} catch(err) {
  console.log('err', err);
} finally {
  console.log('in finally');
}
```

首先说说它的捕获范围，之前我一直以为只有在try语句的{}内的代码出错了会被catch到。但是后面生活给了我一巴掌。

```javascript
window._test = function(func) {
  func();
}
var test = function(arg) {
  window._test(() => arg());
}

function attempt(func, ...args) {
  try {
    return func(...args)
  } catch (e) {
    return e
  }
}

var t = attempt(test, 'test');
```

看上面代码，try代码块中只有一个函数调用，但是该函数调用时没有问题的， 所以按照我以前的想法，应该不会catch到什么错误。但是不是这样的，它会catch整条调用链，这条链上任何一个步骤出错了都将被catch。

接下来再看下面一段代码:

```javascript
window._test = function(func) {
  setTimeout(() => {
    func();
  }, 1000);
}
var test = function(arg) {
  window._test(() => arg());
}

function attempt(func, ...args) {
  try {
    return func(...args)
  } catch (e) {
    return e
  }
}

var t = attempt(test, 'test');
```

这段代码是无法catch到错误的，因为最终出错的是一个异步调用。那么这样就很明显可以得出一个结论了。try catch可以捕获到try代码块中产生的所有同步代码的错误，而产生的异步调用无法catch。

那么既然打开了try catch的话题，那就再探索一下.

```javascript
try {
  try {
    throw new Error('test');
  }
  finally {
    console.log('finally');
  }
} catch(err) {
  console.error('outer catch', err);
}
// finally
// out catch
```

从上代码可看出，错误将被离其作用域中最近的catch块捕获

```javascript
try {
  try {
    throw new Error("oops");
  }
  catch (ex) {
    console.error("inner", ex.message);
    throw ex;
  }
  finally {
    console.log("finally");
    return;
  }
}
catch (ex) {
  console.error("outer", ex.message);
}

// 注: 此 try catch 语句需要在 function 中运行才能作为函数的返回值, 否则直接运行会报语法错误
// Output:
// "inner" "oops"
// "finally"
```

如果从finally块中返回一个值，那么这个值将会成为整个try-catch-finally的返回值，无论是否有return语句在try和catch中。这包括在catch块里抛出的异常
