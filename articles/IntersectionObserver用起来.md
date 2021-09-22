# IntersectionObserver

[tag]:IntersectionObserver|html|js
[create]:2019-12-05

## 概念

在很多场景下，我们经常会做元素曝光计算。

而js在这方面的表现比较乏力。

在以前，我们解决这类问题基本都是通过监听scroll滚动，然后动态去计算每个元素相对于视图的位置。

甚至于上个月我做项目的时候也是采用的这个方案，因为它是目前前端开发视图元素曝光最常用的方法。

然而我今天又看到一个专门为元素曝光提供的新api。

那就是 Intersection Observer

Intersection Observer是一个构造器，它可以创建并返回一个IntersectionObserver对象。

## 使用

简单的使用语法如下:

```javascript
const observer = new IntersectionObserver((entries, observer) => {}, { // 配置可选
    root: HTMLElement, // 根元素, 不提供则默认文档根元素
    rootMargin: '0px 0px 0px 0px', // 上 右 下 左
    threshold: [1], // [0,,1]阈值数组, 表示元素在露出指定百分比的时候会触发回调
});
// 监听元素
observer.observe(HTMLElement);
// 还有takeRecords, unobserve, disconnect方法，不做详细介绍
```

ok. 它的用法非常简单，接下来是我写的一个在react中的简单demo：

***list.jsx***

```javascript
import React from 'react';
import Item from './item';
import './index.css';

function getList(len) {
    let i = 0;
    let arr = [];
    while(i < len) {
        i++;
        arr.push(Math.random());
    }
    return arr;
}

export default class _IntersectionObserver extends React.Component {
    state = {
        list: getList(50)
    }

    refList = [];

    componentDidMount() {
        const observer = new IntersectionObserver((changeTargets, observer) => {
            changeTargets.forEach((item) => {
                const target = item.target;
                if (item.isIntersecting && !target.classList.contains('show')) {
                    target.classList.add('show');
                } else if (!item.isIntersecting && target.classList.contains('show')) {
                    target.classList.remove('show');
                }
            });
        }, { rootMargin: '0px 0px 0px 0px', threshold: [1]});
        this.refList.forEach(it => {
            observer.observe(it);
        });
    }

    render() {
        return (
            <div className="list">
                {
                    this.state.list.map((it, idx) => {
                        return <Item ref={({ ref }) => {
                            this.refList[idx] = ref;
                        }} key={it} index={idx} it={it} />
                    })
                }
            </div>
        );
    }
}
```

***list.css***

```css
.item {
    line-height: 100px;
    text-align: center;
    opacity: 0.2;
    font-size: 40px;
    color: white;
}

.show {
    background: #ccc !important;
    color: black;
}
```

***item.jsx***

```jsx
import React from 'react';

export default class Item extends React.Component {
    ref = null;

    render() {
        const props = this.props;
        const color = '#' + `${props.it}`.substr(2, 6);
        return <div ref={ref => this.ref = ref} className="item" style={{background: color,}}>{props.index}</div>;
    }
}
```

如果你运行起来，你就会发现，确实非常的方便，并不需要自己写多少的代码。

当然，兼容问题还是要有的。can i use表示ie不支持，chrome51以上，QQ浏览器不支持，安卓5.6以上，ios12.2以上。

这个兼容情况确实不容乐观。

不过这个方法是在线程空闲的时候才调用的，如果这样的话，它的优先级就是最低的。

那问题还是有点严重。毕竟谁无法接受突然它停止工作吧。

## 参考资料

[阮一峰老师](https://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
