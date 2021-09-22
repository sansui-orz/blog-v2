## lodash源码阅读

[tag]:lodash|源码
[create]:2019-11-07

> 在工作的间隙，多读一些优秀的代码。发现亮点便记录下来。

- [x] isBoolean
  它判断的时候使用的是全等判断(===)，然后还考虑到Boolean实例的情况（Boolean实例是一个对象，使用全等的时候不发生隐式转换，所以即不等于true也不等于false）

- [x] iObjectLike
  满足typeof等于object且不等于null

- [x] isObject
  满足不等于null, typeof等于object或者function

- [x] isFunction
  满足typeof等于[object Function]或[object AsyncFunction]或[object GeneratorFunction]或[object Proxy]

- [x] isArrayLike
  是否是类数组，或数组, 满足[], html集合, 字符串， （arguments应该也是类数组）

- [x] isEmpty
  1. 如果是类数组，判断数组长度，长度0为空
  2. 如果是map或者set，判断size, size0为空
  3. 如果入参是一个原型链(prototype)，则用Object.keys判断其长度
  4. 否则当作一个对象来判断, for in循环，如果hasOwnProperty则不为空
    4.1 如果是function, 直接for in并不会进入循环
  5. 以上条件都不满足，则为空

- [x] isNull
  直接全等判断

- [ ] isNative
  判断是否是原生方法, 首先是isObject其次通过正则匹配（还没搞懂匹配原理）

- [x] isString
  typeof 是string或者是String的实例

- [x] isNil
  这个函数用来判断入参是否等于(==)null
  null / undefined / ?

- [x] isNumber
  满足typeof等于number，或者是number的实例对象

- [x] isUndefined
  是否全等于undefined

- [x] chunk
  将一个数组分成指定长度的多个数组，第一个参数为原数组，第二个为指定的子数组的长度

- [x] toInteger
 这个函数是将小数，或非数字转换成整数，对于Infinity也可以转换成在js中的数值最大值

- [x] toFinite
  将无限大或无限小的数转成一个固定的最大值最小值，如果入参非最大值或最小值，则返回其自身或0

- [x] toNumber
  将传入参数转换成数字, 其关注了对象的valueOf，二进制数，八进制数

- [x] defer
  使用setTimeout将传入方法进行异步调用

- [x] clone
  调用底层baseClone, 浅拷贝

- [ ] baseClone
  底层方法，简单看了下应该是个浅拷贝，兼容比较多，所以导致比较复杂，暂时放着

- [x] forEach | each
  传入对象或数组，第二个参数为遍历方法。

- [x] before
  第一个参数为可以调用的次数，第二个参数为该调用的方法。超出调用次数后再次调用会返回最后调用的结果 (还没搞懂它的应用场景)

- [x] divide
  求商，就是divide(6, 4)返回6/4 = 1.5。就只是做了一些兼容，如`divide(5. undefined) => 5`

- [x] union
  扁平化数组，并且去重
  union([2, 3], [1, 2]) // => [2, 3, 1]

- [x] eq
  比较两个参数是否全等，这里比较特别的是对NaN做了比较，即`param1 !== param1 && param2 !== param2`也是成立全等的

- [x] delay // (func, wait, ...args)
  和上面的defer是一摸一样的，将传入函数做setTimeout的异步调用

- [x] has // (obj, key)
  底层直接调用hasOwnProperty,只是多了个object==null的兼容判断

- [x] create // (prototype, properties)
  底层调用Object.create。只是做了些兼容，第二个参数为新实例对象需要合并的属性

- [x] escape // string => string
  对传入string进行简单字符转换，只能转换`&<>"'`

- [x] escapeRegExp: (string) => string
  Escapes the `RegExp` special characters "^", "$", "\", ".", "*", "+", "?", "(", ")", "[", "]", "{", "}", and "|" in `string`

- [x] cloneDeep // 深拷贝

- [x] add
  顾名思义，就是传入两个参数，进行相加的操作，对传入数进行了安全的转换，如add(1, undefined) => 1

- [x] countBy: (Array | Object, (item) => boolean) => Object<{ propname: number }>
  遍历传入的数组或对象，计算出指定的值各有多少，并返回最终的计算值
  例子：
  ```javascript
  const users = [
    { 'user': 'barney', 'active': true },
    { 'user': 'betty', 'active': true },
    { 'user': 'fred', 'active': false }
  ]

  countBy(users, value => value.active);
  // => { 'true': 2, 'false': 1 }
  ```
- []