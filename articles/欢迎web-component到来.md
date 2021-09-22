# 欢迎Web Component到来

[tag]:html|js|component
[create]:2020-08-03

web component是谷歌一直在推进的浏览器原生组件，相对于react/vue/angular等框架，它优点在于没有依赖，代码量小，且工作模式简单。

根据目前兼容性来看，已经可以适当的运用于生产环境了。未来会不会发展出不依赖任何编译能力的基于web component的mvvm或者类似的框架也说不定，我还是很看好它的。

![custom elements v1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200731180911.jpg!trans_webp)

## web component的简单使用

为了与原生html标签做区分，自定义标签的名字需要包含`-`，例如`<user-info>`不能写成`<userinfo>`。

通过customElements.define(name, constructor, options)方法定义自定义元素。

以下为一个简单的用户信息组件，仅包含用户名与用户头像:

```javascript
class UserInfo extends HTMLElement {
  constructor() {
    super();
    this.innerHTML = this.render();
  }

  render() {
    return `
      <img src="https://semantic-ui.com/images/avatar2/large/kristy.png" />
      <p class="username">程序员</p>
    `;
  }
}
window.customElements.define('user-info', UserInfo);
```

```html
<body>
  <user-info></user-info>
</body>
```

可以看到上文中直接指定`this.innerHTML = this.render()`, 此时this指向的就是`user-info`这个节点，而自定义节点与原生节点的使用是没有任何不同的，同样可以添加事件监听，同样可以改变属性等。

渲染出来的效果如下:

![渲染1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200803142536.jpg!trans_webp)

## 使用template

除了直接使用字符串类型的文档定义，还可以使用`template`标签。template标签的优势在于其相对于字符串，或者通过api创建节点更加符合前端们对于编写html的习惯，并且template内容的解析是在DOM解析的时候，解析完会驻留用于后续的自定义节点。

```html
<body>
  <template id="user-info">
    <img src="https://semantic-ui.com/images/avatar2/large/kristy.png" />
    <p class="username">程序员</p>
  </template>
  <user-info></user-info>
</body>
```

```javascript
class UserInfo extends HTMLElement {
  constructor() {
    super();
    const temp = document.getElementById('user-info');
    const content = temp.content.cloneNode(true);
    this.appendChild(content);
  }
}
window.customElements.define('user-info', UserInfo);
```

上面使用了template的代码本质上跟使用字符串的代码是一致的，只不过是它们之间的解析时机不一样。对于哪种方法更好也没有一个明确的结论，可能用react的觉得第一种好，用vue的觉得第二种好。

但是从上面两种形式已经是可以看出react/vue这类框架的影子了，如果web component兼容性好一些的话，或许以后就不需要在加载这些第三方的库了，直接原生的html+js就能快速搞定一个不大的业务。（当然大的业务需要更好的组织能力，所以用上框架还是很有必要的）。

## 使用Shadow DOM

[Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)允许将隐藏的DOM树附加到常规的DOM树上的节点上。

```javascript
class UserInfo extends HTMLElement {
  constructor() {
    super();
    const temp = document.getElementById('user-info');
    const content = temp.content.cloneNode(true);
    const shadow = this.attachShadow({ mode: 'open' });
    /**
    * mode = open表示可以在主页上下文中访问该Shadow DOM
    * mode = closed则表示不可以
    */
    shadow.appendChild(content);
  }
}
window.customElements.define('user-info', UserInfo);
```

使用了Shadow DOM之后再看dom结构是这样的

![shadow](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200803174033.jpg!trans_webp)

并且你会发现img的样式不起效了，这是因为Shadow DOM中的任何DOM都不会影响到DOM树，相应的也无法应用到外部样式（这点存疑，我在 chrome上测试确实无法使用外部样式）。

所以对于Shadow DOM来说，它的作用在于隔离自定义组件与外部环境，防止样式污染，或内容被篡改。

## 使用slot

[slot（插槽）](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots)是与`template`一起的一个特性，在vue与微信小游戏中都有类似的特性，而在react中，指的就是render props。

```html
<body>
  <template id="user-info">
    <img src="https://semantic-ui.com/images/avatar2/large/kristy.png" />
    <p class="username"><slot name="user-name">程序员</slot></p>
  </template>
  <user-info>
    <span slot="user-name">帅气程序员</span>
  </user-info>
</body>
```

## 同步class属性以及DOM属性

使用get/set就可以将class内的属性与dom的属性保持一致了。

```javascript
class UserInfo extends HTMLElement {
  get disabled() {
    return this.getAttribute('disabled');
  }

  set disabled(val) {
    if (val) {
      this.setAttribute('disabled', '');
    } else {
      this.removeAttribute('disabled');
    }
  }

  constructor() {
    super();
    const temp = document.getElementById('user-info');
    const content = temp.content.cloneNode(true);
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.appendChild(content);
  }
}
window.customElements.define('user-info', UserInfo);
```

## 生命周期

- constructor: 构造时

- connectedCallback: 元素每次插入DOM时。

- disconnectedCallback: 元素每次从DOM移除时。

- attributeChangedCallback: 指定的属性变更时。需要在`observedAttributes`指定需要监听的属性。

- adoptedCallback: 自定义元素被移入新的Document时。

```javascript
class UserInfo extends HTMLElement {
  static get observedAttributes() {
    return ['avatar', 'username'];
  }

  constructor() {
    super();
    const temp = document.getElementById('user-info');
    const content = temp.content.cloneNode(true);
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.appendChild(content);
  }

  connectedCallback() {
    console.log('connectedCallback');
  }

  disconnectedCallback() {
    console.log('disconnectedCallback');
  }

  attributeChangedCallback(attrName, oldVal, newVal) {}

  adoptedCallback() {
    console.log('adoptedCallback');
  }
}
window.customElements.define('user-info', UserInfo);
```

看这生命周期函数，有点react内味了吧。

## api

- customElements.define(tagName, constructor, options) 定义处理自定义元素的方法，options可以通过设置extends设置集成某个原生dom从而具备原生dom的基础能力。

- customElements.get(tagName) 通过标签名返回元素的构造函数。

- customElements.whenDefined(tagName) 如果元素没定义，则在定义时会触发promise回调，如果已经定义，则立即触发promise回调。（**这个应该归类到生命周期**）

## 设置未定义时的默认样式

通过设置style为:

```css
user-info:not(:defined) {
  background: red;
}
```

来设置组件为定义时的默认样式，可以直接作为骨架屏用，还是很好用的。

## 思考

其实web component的写法已经很类似与目前的主流框架了，是否基于web component可以实现react形式的组件化开发呢？

理论上来说这是可以实现的，但是需要解决几个问题：

1. html标签不能插入到文档内，而是通过字符串的形式由js返回，例如render方法返回一个字符串，这个可以用gulp动态编译将template标签编译到js内（css同理也可以这样实现) 这样做的目的是更好的拆分组件，一个组件一个js

2. 数据双向绑定, 修改数据时对应视图上的数据也需要更改，这个可以使用mobx或者自己写一个底层的数据双向绑定逻辑解决（但是性能这块需要优化）

3. 全局数据的组织形式，这个方面是基于整个架构方面的思考了。

4. 兼容性。这个也是为什么web component现如今不温不火的原因，并没有太好的解决方案。但是可以使用[poly-fill](https://github.com/webcomponents/custom-elements.git)方案解决。

暂时也就想到这些点，但是其实仔细考虑一下就会发现，理论上能想到的那些问题都有手段可以解决，更多该考虑的应该是实现成本以及是否确切需要该能力的问题。

而使用web-component还能够拥有更好的SEO，和更小的包体积。

这样看下来似乎web component完全可以开发成类似与react/vue这种mvvm前端框架。并且对于组件的划分，代码的组织是不是更好呢？

（可以将每个组件拆分成一个js，加载时再通过node后端将多个组件js合并成一个，实现动态加载组件的能力）

## 参考链接

[Web Components 入门实例教程](http://www.ruanyifeng.com/blog/2019/08/web_components.html)

[自定义元素 v1：可重用网络组件](https://developers.google.com/web/fundamentals/web-components/customelements#prestyle)

[使用 shadow DOM](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/Using_shadow_DOM)

[使用 templates and slots](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/Using_templates_and_slots)
