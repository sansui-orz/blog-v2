# 给JS加上强类型检查

[tag]:JSDoc|类型校验
[create]:2021-05-20

通常在一些老项目中，是没有强类型检查的。这就导致了我们对这些项目不敢下手修改，畏首畏尾，生怕一点改动引起项目的运行异常。尤其是这类项目基本上你都不会是第一任开发者，对其中的功能可能都了解的不全面。

那么对于这种项目其实我们可以逐步的对其进行改造，给其加上类型，一步一步的将其变得更复杂（不是，是变的跟TS一样想改就改。

这就需要用到[JSDoc](https://jsdoc.zcopy.site/index.html#block-tags)类型标记。

使用JSDoc加上vs code对代码逐步声明类型，逐渐转成强类型代码，让代码逻辑更清晰有调理，同时可以养成写代码注释的好习惯，也可以使用JSDoc生成文档，实在是一举多得。

## 环境配置

其实环境的配置非常简单，只需要VS Code开启了js验证:

![js验证](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/WX20210516-232332.png!trans_webp)

这个默认就是开启的。

之后在js代码中第一行增加`// @ts-check`就成功开启了类型校验了，非常简单。

![demo-1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/WX20210516-232835.png!trans_webp)

当然，如果不想要每个js文件都去加这个注释也可以直接写一个`jsconfig.json`文件(其实`tsconfig.json`也可以，打开`allowJs`就好了，记得安装typescript依赖),然后就可以全局的开启类型校验了。但是并不建议这么做，因为对于老项目来说，一个一个的文件去更改会更为方便且稳妥，而对于新项目来说直接用ts就好啦，用了一圈下来还是觉得这个的方便性以及功能还是不如ts强大的。

## 变量类型声明

```js
// @ts-check

/** @type {string} */
var str = '123';

/** @type {number} */
var num = 123;

/** @type {{ a: number; }} */
var obj = { a: 123 };

/** @type {number[]} */
var arr = [1, 2, 3];

/** @type {() => number} */
var fn = () => 123;

/** @type {*} */
var any = 'any';

/** @type {unknown} */
var unknown;
```

## 类型断言

注意类型断言要将需要断言的变量使用`()`包裹。

```js
// @ts-check

/**
 *
 * @param {string | number} str
 * @returns {string[]}
 */
function strSplit(str) {
  return /** @type {string} */ (str).split('');
}
```

## 定义接口

```js
// @ts-check

/**
 * @typedef {{
 *  name: string;
 *  age: number;
 * }} UserInfo
 */

/** @type {UserInfo} */
var userInfo = {
  name: 'zhang san',
  age: 18
};
```

当你想要对接口的每个字段进行注释说明是，还有另一种稍微繁琐一点的方式。

```js
// @ts-check

/**
 * @typedef UserInfo
 * @property {string} name 名字
 * @property {number} age 年龄
 */

/** @type {UserInfo} */
var userInfo = {
  name: 'zhang san',
  age: 18
};
```

## 泛型

```js
// @ts-check

/**
 * @template T, N
 * @typedef UserInfo
 * @property {T} name 名字
 * @property {N} age 年龄
 */

/** @type {UserInfo<string, number>} */
var userInfo = {
  name: 'zhang san',
  age: 18
};
```

## 类型继承（子类型）

```js
// @ts-check

/**
 * @template T, N
 * @typedef UserInfo
 * @property {T} name 名字
 * @property {N} age 年龄
 *
 * @typedef { { sex: string } & UserInfo<string, number> } UserInfoSex
 *
 */

/** @type {UserInfoSex<string, number>} */
var userInfo = {
  name: 'zhang san',
  age: 18,
  sex: 'girl'
};
```

## 跨文件类型引用

在a.js中:

```js
// @ts-check

/**
 * @template T, N
 * @typedef {{ name: T, age: N }} UserInfo
 */

module.exports = {};
```

在b.js中:

```js
// @ts-check

/**
 * @typedef { { sex: string } & import('./a.js').UserInfo<string, number> } UserInfoSex
 *
 */

/** @type {UserInfoSex} */
var userInfo = {
  name: 'zhang san',
  age: 18,
  sex: 'girl'
};
```

对于在项目中大量存在的数据类型，建议还是使用`*.d.ts`类型文件，比如写在`global.d.ts`中就可以直接在全局使用，不需要每次都引入那么麻烦。当然对于作用范围仅限于一两个文件的类型还是直接写在JSDoc注释中会让代码阅读更加方便。

## 函数

```js
// @ts-check

/**
 * 建议使用这种方式定义
 * @param {string} name
 * @param {number} [age=10] 可选参数
 * @returns {string}
 */
function userInfo(name, age=10) {
  return `名字叫${name}, 年龄是${age}`;
}

/**
 * 使用这种方式定义方法时，定义参数会失效，比如下例，不传age就会报错
 * @type { (name: string, age？: number) => string }
 */
function userInfo2(name, age) { // 解决这个问题可以给age设置默认值，例如age=10
  return `名字叫${name}, 年龄是${age}`;
}
```

### 函数剩余参数

```js
// @ts-check

/**
 * @param {string} name
 * @param {number} [age=18]
 * @param {...any} other
 * @returns {string}
 */
function userInfo(name, age, ...other) {
  return `名字叫${name}, 年龄是${age}, other: ${other.join(',')}`;
}
```

### this参数声明

```js
// @ts-check

/**
 * @typedef UserInfo
 * @property {string} name 名字
 * @property {number} age 年龄
 * @property { () => void } say 方法
 */

/**
 * @typedef {{username: string;}} UserName
 */

/** @type {UserInfo} */
var userInfo = {
  name: 'zhang san',
  age: 18,
  /** @this {UserName} */
  say() {
    console.log(this.username);
  }
};
```

## 类

类的使用大部分就是使用上面说过的一些形式组合起来，只有继承和实现接口是比较特异的。

### 继承

```js
// @ts-check
import React from 'react';

/**
 * @extends {React.Component<PropsType, StateType>} 使用 extends 声明继承类型
 */
export default class List extends React.Component {
  render() {
    return <div></div>;
  }
}
```

### 类实现接口

```js
// @ts-check

/**
 * @typedef UserInfo
 * @property {string} name 名字
 * @property {number} age 年龄
 * @property {() => void} say 方法
 */

/** @implements {UserInfo} */
class ClassUserInfo {
  name = '张三';
  age = 18;
  say() {
    console.log(this.name);
  }
}

const userInfo = new ClassUserInfo();
userInfo.say();
```

## 枚举

枚举只能限制对象的值，类似如下的使用方法。

```js
/** @enum {number} */
const date = {
  day: 1,
  month: 5,
  year: 2021,
}
```

其他的一些方法作用性不高，查看完全的注解可以查看[JSDoc](https://jsdoc.zcopy.site/index.html#block-tags)，更加详细的用法建议直接看[Typescript官方文档](https://www.tslang.cn/docs/handbook/type-checking-javascript-files.html)。

## 总结

在一个新的小应用中特地去使用了一下JSDoc注释的类型约束，总体体验下来就是写的确实不如typescript方便，本来ts就已经给开发带来了额外的代码量了，所以总的来说，如果不是因为实在是老项目无法使用ts的话，不建议使用这种形式去约束js，因为对比之下ts实在优秀。即使你的项目再小，也应该用上typescript，而非JSDoc。

当然，也不是说就不能用JSDoc，无论在ts还是在js，该写注释还是要写注释，这能让看你代码的人更加了解你的意图（尤其是你的命名比较梦幻的时候），这样即使以后别人接收，连交接文档都能少写两页呢。6⃣️6⃣️6⃣️
