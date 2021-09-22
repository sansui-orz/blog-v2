# Element.closest

[tag]:记录|js|closest
[create]:2020-08-14

Element.closest用来获取：匹配特定选择器且离当前元素最近的祖先元素（也可以是当前元素本身）。如果匹配不到，则返回 null。

这个方法很像`Element.matches`，所以一下就想到了可以直接应用到事件代理上面。

直接通过`event.target.closest('.a .b')?.getAttribute('data-index')`这种形式，获取到挂在dom上的属性。

但是closest的兼容性并不好。

![closest兼容性](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200814113645.jpg!trans_webp)
