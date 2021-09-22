# transform: motrix

[tag]:记录|css
[create]:2020-07-03

css的`transform`属性中有个比较特殊的，少用的值是`matrix`（矩阵）

它有六个值，且六个值只能是数字。

假设六个值用变量表示，是如下这个样子：

```css
.test {
  transform: matrix(a, b, c, d, e, f);
}
```

它实际上对于坐标的转化公式是这样的：
![transform2](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200703163158.jpg!trans_webp)

可以知道我们转化后的坐标是：`[a * x + c * y + e, b * x + d * y + f]`

仔细想想很容易就能发现，这个公式包含了`tranfrom`的其他属性值，`translate scale rotate skew`

## translate

首先是简单的`translate`, 如果我们需要将元素向右移10px, 向下移20px。我们可以简单的用`[a * x + c * y + e, b * x + d * y + f]`知道abcdef各项的值需要怎么设置:

`[1 * x + 0 * y + 10, 0 * x + 1 * y + 20]`，假设原来坐标是`[0, 0]`。那计算出来的坐标就是`[10, 20]`这符合我们的预期。

所以这时候`transform`的值为`matrix(1, 0, 0, 1, 10, 20)`

## sacle

如果需要让宽度缩小到0.4, 高度缩小到0.6。也跟translate一样，直接套转换公式很轻易就能得出:

`[0.4 * x + 0 * y + 0, 0 * x + 0.6 * y + 0]`

所以这时候`transform`的值为`matrix(0.4, 0, 0, 0.6, 0, 0)`

## rotate

至于旋转就比较复杂了，不同于位移与缩放，这个属性需要起作用则必须需要两个值的作用。

这里用到了比较复杂的，三角函数忘记怎么算了。

反正最终推导出的结果是`matrix(cosθ, sinθ, -sinθ, cosθ, 0, 0)`。

## 参考文档

[理解CSS3 transform中的Matrix(矩阵)](https://www.zhangxinxu.com/wordpress/2012/06/css3-transform-matrix-%E7%9F%A9%E9%98%B5/)
