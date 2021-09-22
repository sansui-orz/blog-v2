# 大整数运算

[tag]:js|math
[create]:2020-07-30

在js中，number类型的精度问题一直以来被人所诟病，也一直是面试官们喜欢的诘问材料。

## 精度不准的原因

js中Number的实现遵循 IEEE 754 标准，使用 64 位固定长度来表示，也就是标准的 double 双精度浮点数（相关的还有float 32位单精度）。

这么做的原因是因为它节省存储空间。

注意以下实现方法都是基于传入参数为字符串正整数的前提下，如果需要处理负数或者小数需要在方法之外做一层适配。

## 加法

```javascript
function addition(n1, n2, ...others) {
  const n1Arr = n1.split('').reverse(); // 将字符串分割成数组，并反向，反向的目的在于循环时可以按照手动计算的习惯，从右往左算
  const n2Arr = n2.split('').reverse();
  const len = Math.max(n1Arr.length, n2Arr.length); // 取出数组中最长的长度, 要根据最长的长度循环
  const resolve = [];
  let pre = 0; // 记录相加后大于等于10时进一位数。
  for (let i = 0; i < len; i++) {
    const r = Number(n1Arr[i] || 0) + Number(n2Arr[i] || 0) + pre;
    pre = 0;
    resolve[i] = r % 10;
    if (r / 10 >= 1) {
      pre = 1;
      if (i === len - 1) { // 如果最后面的计算超过10，需要手动添加到数组
        resolve.push(1);
      }
    }
  }
  const result = resolve.reverse().join('');
  if (others && others.length > 0) { // 如果超过一个数相加，则递归调用
    return addition(result, ...others);
  }
  return result;
}
```

## 减法

```javascript
function subtraction(n1, n2) {
  const n1Arr = n1.split('').reverse();
  const n2Arr = n2.split('').reverse();
  let dir = 0; // 先计算一下两个值哪个比较大，因为用大的减去小的比较方便计算
  if (n1Arr.length > n2Arr.length) {
    // 左边的数组长
    dir = 1;
  } else if (n1Arr.length < n2Arr.length) {
    // 右边的数组长
    dir = -1;
  } else { // 两个数组一样长
    for (let i = n1Arr.length - 1; i >= 0; i--) {
      if (Number(n1Arr[i]) > Number(n2Arr[i])) { // 长度一样则判断每一位数的大小，注意此处要从数组尾部开始算，因为前面把数组倒过来了
        dir = 1;
      } else if (Number(n2Arr[i]) > Number(n1Arr[i])) {
        dir = -1;
      }
    }
    if (dir === 0) dir = 1;
  }
  const resolve = [];
  const p1 = dir === 1 ? n1Arr : n2Arr;
  const p2 = dir === 1 ? n2Arr : n1Arr;
  let _reduce = 0; // 记录相减时不够减需要从前一位取值
  for (let i = 0; i < p1.length; i++) {
    const v1 = Number(p1[i]) + _reduce;
    const v2 = Number(p2[i] || 0);
    resolve[i] = v1 < v2 ? (v1 + 10 - v2) : (v1 - v2);
    _reduce = 0;
    if (v1 < v2) { // 如果不够减，就取前面一位
      _reduce = -1;
    }
  }
  return (dir === -1 ? '-' : '') + resolve.reverse().join(''); // 注意需要把负号补上
}
```

## 乘法

```javascript
function multiplication(n1, n2) {
  const n1Arr = n1.split('').reverse();
  const n2Arr = n2.split('').reverse();
  const resolve = [];
  // 模拟手动计算的乘法方式，分别将两个值的各个位置逐个相乘
  for (let i = 0; i < n2Arr.length; i++) {
    let a = 0;
    resolve[i] = [];
    for (let j = 0; j < n1Arr.length; j++) {
      const r = Number(n1Arr[j]) * Number(n2Arr[i]) + a;
      a = 0;
      resolve[i][j] = r % 10;
      if (r >= 10) {
        a = Math.floor(r / 10);
      }
      if (j === n1Arr.length - 1 && a !== 0) {
        resolve[i][j + 1] = 1;
      }
    }
  }
  const rr = [];
  for (let x = 0; x < resolve.length; x++) { // 逐个相乘之后的乘积再错位相加
    rr[x] = resolve[x].reverse().join('');
    rr[x] = rr[x].padEnd(rr[x].length + x, '0');
  }
  return addition(...rr); // 再将乘积相加
}
```

## 除法

除法稍微有些复杂，待补充。。。

## 参考链接

[ECMAScript中的Number Type与 IEEE 754-2008](https://juejin.im/post/6844903834356023303)

[关于大数除法](https://www.cnblogs.com/fightformylife/p/4022058.html)
