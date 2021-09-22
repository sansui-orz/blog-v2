# position:fixed失效问题记录

[tag]:bug记录|css
[create]:2019-12-27

## 问题描述

position: fixed遇到父元素含有transform属性时，会变成position: absolute的效果

## 解决方法

除了移除父元素的transform属性之外，暂无其他方法

***根据查询资料，以下情况都会出现position:fixed失效***

- 祖先元素transform不为none
- 祖先元素perspective不为none
- 祖先元素拥有will-change属性
- 还有不同的环境会造成该问题，但是上面三个与环境无关
