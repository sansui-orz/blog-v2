# 纯css实现loading动画

[tag]:css|loading|html
[create]:2019-12-13

![效果](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200805151032.jpg!trans_webp)

[源码](../demos/loading.html)

代码:

```html
<div class="box">
    <div class="loading">
        <div class="p1">
            <div class="c1"></div>
        </div>
        <div class="p2">
            <div class="c11"></div>
        </div>
    </div>
</div>
```

```css
:root {
  --time: 2s;
  --size: 300px;
  --border: 80px;
  --bordercolor: #ff9c38;
}

.box {
  display: inline-block;
  overflow: hidden;
  border-radius: 50%;
  margin: 40px auto;
  transform: scale(0.15);
  padding: 4px;
}

.loading {
  background: white;
  border-radius: 50%;
  width: var(--size);
  height: var(--size);
  border: var(--border) solid var(--bordercolor);
  position: relative;
  overflow: visible;
}

.loading::before,
.loading::after {
  content: ' ';
  width: var(--border);
  height: var(--border);
  position: absolute;
  bottom: calc(0px - var(--border));
  left: 50%;
  background: var(--bordercolor);
  margin-left: calc(0px - var(--border) / 2);
  z-index: 3;
  border-radius: 50%;
  transform-origin: center calc(0px - var(--size) / 2);
}

.loading::before {
  animation: ball1 var(--time) linear 0s infinite;
}

.loading::after {
  animation: ball2 var(--time) linear 0s infinite;
}

.p1,
.p2 {
  width: calc(var(--border) + var(--size) / 2 + 4px);
  height: calc(var(--border) * 2 + var(--size) + 10px);
  position: absolute;
  top: calc(-5px - var(--border));
  overflow: hidden;
}

.p1 {
  left: calc(-2px - var(--border));
}

.p2 {
  right: calc(-2px - var(--border));
}

.c1,
.c11 {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  /* background: rgba(255, 255, 255, 0.5); */
    background: rgba(255, 255, 255, 1);
}

.c1 {
  transform-origin: right center;
  animation: loading1 var(--time) linear 0s infinite;
}

.c11 {
  transform-origin: left center;
  animation: loading11 var(--time) linear 0s infinite;
}

@keyframes loading1 {
  from {
    transform: rotate(0deg);
  }

  25% {
    transform: rotate(0deg);
  }

  48% {
    transform: rotate(-180deg);
  }

  75% {
    transform: rotate(-180deg);
  }

  98% {
    transform: rotate(-360deg);
  }

  to {
    transform: rotate(-360deg);
  }
}

@keyframes loading11 {
  from {
    transform: rotate(0deg);
  }

  2% {
    transform: rotate(0deg);
  }

  25% {
    transform: rotate(-180deg);
  }

  48% {
    transform: rotate(-180deg);
  }

  52% {
    transform: rotate(-180deg);
  }

  75%{
    transform: rotate(-360deg);
  }

  to{
    transform: rotate(-360deg);
  }
}

@keyframes ball1 {
  from {
    transform: rotate(0deg);
  }

  2% {
    transform: rotate(0deg);
  }

  48% {
    transform: rotate(-360deg);
  }

  to {
    transform: rotate(-360deg);
  }
}

@keyframes ball2 {
  from {
    transform: rotate(0deg);
  }

  52% {
    transform: rotate(0deg);
  }

  98% {
    transform: rotate(-360deg);
  }

  to {
    transform: rotate(-360deg);
  }
}
```

1. 采用css变量

2. loading动画两头采用圆形伪类元素实现
