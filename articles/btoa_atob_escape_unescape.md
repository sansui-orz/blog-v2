# btoa与atob与escape/unescape

[tag]:记录|js|转码
[create]:2019-10-28

## btoa

该函数可以从 String 对象中创建一个 base-64 编码的 ASCII 字符串，其中字符串中的每个字符都被视为一个二进制数据字节。

例子:

```javascript
window.btoa("Hello, world");
// SGVsbG8sIHdvcmxk
```

## atob

对经过 base-64 编码的字符串进行解码。

例子:

```javascript
const base64Str = window.btoa("Hello, world");
window.atob(base64Str);
// Hello, world
```

## escape/unescape

对传入字符进行加密解密, 例如一些空格符，换行符，特殊符号，特殊文字等。