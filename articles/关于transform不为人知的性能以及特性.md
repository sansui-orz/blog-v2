# 关于transform不为人知的性能以及特性

[tag]:记录|css|transform
[create]:2020-07-03

## 背景：

在轮播图组件中使用到了left+transform去定位每个块，而当我这样做的时候却发现，它在手机浏览器上显示效果不佳，会出现闪跳，卡顿的情况。

发现这个问题后，通过将left的定位改为transform，统一用transform来定位之后，就解决了这个问题。

在这中间，我发现transform这个css属性里面的料还是挺多的。

1. 当transform下各个属性的组合顺序不一致时，其表现形式也不一样。

例:

```css
.class1 {
  transform: scale(0.5) translateX(100px);
}

.class2 {
  transform: translateX(100px) scale(0.5);
}
```

上面这两个属性乍看之下似乎是一样的，但是其实他们的表现形式并不一样：

class1是先将元素缩小一倍，然后再移动100px

class2则是先移动100px，然后再缩小一倍

似乎这两个逻辑也没问题，但是请看下图:

![tansform](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200623182410.jpg!trans_webp)

从这个图中可以分析出浏览器绘制时的流程：

1. class1缩小，因为transform-origin默认是在元素中间，所以此时缩小后视觉上class1左边应该距离父元素左边 100 * 0.5 / 2 = 25px
2. class2缩小之后再往右边平移100px, 需要注意的是，这里的平移是基于自身平移的，所以平移的实际距离为100 * 0.5 = 50px, 此时距离父元素左边框距离为75px;
3. class2先移动，所以直接右移100px
4. 移动后缩小，所以最后是100px + 100 * 0.5 / 2 = 125px;

可能下图看上去会更加明了：
![transform2](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200623184149.jpg!trans_webp)

## 所以

以上说明了，transform属性的顺序，会前者影响后者的

2. 这里又可以引申出一个问题，即是，transform是否会影响到除自身外到其他元素：

例子:

![transform3](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200628140447.jpg!trans_webp)

## 最终

答案是，transform并不会改变其在文档流中的布局

2.1 既然transform不会影响文档流的布局，那如果a标签使用了transform，这时候它的实际可点击区域是在文档流里的原位置，还是在转换后的位置呢？

由于截图不太方便，所以图片就不放出来了。答案是，a标签的实际位置是transform后的位置，这应该也是合理的位置。

3. 最后总结一下就是，transform，本身的属性会前后影响。并且无论transform的节点怎么变换，其影响也仅仅是在它本身的，并不会对父子&兄弟节点造成影响。

也是由于它这个仅作用于自身的特性，使得它在动画上的性能比left，width突出。

使用它进行节点的大小变换，位置移动，并不会造成文档数的布局变换。大大减少了渲染线程的开销。

所以在实际应用中，如果需要实现动画，请尽可能的选择使用tansform/opacity此类不影响文档树的api。

## 参考资料：

http://sy-tang.github.io/2014/05/14/CSS%20animations%20and%20transitions%20performance-%20looking%20inside%20the%20browser/

http://zencode.in/14.CSS%E5%8A%A8%E7%94%BB%E7%9A%84%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.html
