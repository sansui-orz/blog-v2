# leetcode算法学习

## 字符串

### 外观数列

```text
给定一个正整数 n ，输出外观数列的第 n 项。

「外观数列」是一个整数序列，从数字 1 开始，序列中的每一项都是对前一项的描述。

你可以将其视作是由递归公式定义的数字字符串序列：

countAndSay(1) = "1"
countAndSay(n) 是对 countAndSay(n-1) 的描述，然后转换成另一个数字字符串。
前五项如下：

1.     1
2.     11
3.     21
4.     1211
5.     111221
第一项是数字 1
描述前一项，这个数是 1 即 “ 一 个 1 ”，记作 "11"
描述前一项，这个数是 11 即 “ 二 个 1 ” ，记作 "21"
描述前一项，这个数是 21 即 “ 一 个 2 + 一 个 1 ” ，记作 "1211"
描述前一项，这个数是 1211 即 “ 一 个 1 + 一 个 2 + 二 个 1 ” ，记作 "111221"
要 描述 一个数字字符串，首先要将字符串分割为 最小 数量的组，每个组都由连续的最多 相同字符 组成。然后对于每个组，先描述字符的数量，然后描述字符，形成一个描述组。要将描述转换为数字字符串，先将每组中的字符数量用数字替换，再将所有描述组连接起来。
```

***解决方法***

就是写个while循环中，按照每步执行，最后得到结果

```js
/**
 * @param {number} n
 * @return {string}
 */
var countAndSay = function (n) {
  var f = [1];
  n = n - 1;
  while (n > 0) {
    var _f = [];
    var current = f[0];
    var count = 1;
    for (let i = 1; i < f.length; i++) {
      if (current === f[i]) {
        count++;
      } else {
        _f.push(count, current);
        count = 1;
        current = f[i];
      }
    }
    _f.push(count, current);
    f = _f;
    n--;
  }
  return f.join('');
};
```

### 最长公共前缀

> 编写一个函数来查找字符串数组中的最长公共前缀。
> 如果不存在公共前缀，返回空字符串 ""。

***解决方法***

一开始想的比较多，但是最后没想到什么优雅的解法，就直接按照循环的来，没想到写完之后，时间超越47%，空间超越90%，似乎确实没有什么特别优雅的解法啊。

```js
/**
 * @param {string[]} strs
 * @return {string}
 */
var longestCommonPrefix = function(strs) {
    var i = 0;
    if (strs.length === 1) return strs[0];
    while (true) {
        for (let j = 0; j < strs.length - 1; j++) {
            if (!strs[j][i] || strs[j][i] !== strs[j+1][i]) {
                return strs[0].substr(0, i);
            }
        }
        i++;
    }
    return strs[0].substr(0, i);
};
```

## 链表

### 删除链表中的节点

> 请编写一个函数，使其可以删除某个链表中给定的（非末尾）节点。传入函数的唯一参数为 要被删除的节点 。
> 例： head = [4,5,1,9], node = 5， 其中只输入链表中的一个节点5，需要从链表中删除这个节点

***解决方法***

在确认只输入一个节点的时候，其实事情还挺明了的，就是由着链将整个链条向前移动一位，这样就起到了删除节点的作用了。

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} node
 * @return {void} Do not return anything, modify node in-place instead.
 */
var deleteNode = function(node) {
    while(node.next) {
        node.val = node.next.val;
        if (!node.next.next) {
            node.next = null;
        } else {
            node = node.next;
        }
    }
};
```

只是没想到的是，时间击败98%，空间击败89%。（虽然leetcode的这个计算一点都不准，还是有点开心的）

好吧，虽然很不想承认，但是我确实是又想差了，因为链表本质上是一条链状的对象指针，所以其实只需要将当前的值修改，将当前的next指向next.next，就能够起到删除节点的作用了。。。

```js
var deleteNode = function(node) {
    node.val = node.next.val, node.next = node.next.next;
};
```

一行搞定了。。