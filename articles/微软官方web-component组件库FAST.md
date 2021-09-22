# 微软官方web-component组件库FAST

[tag]:web component|FAST
[create]:2020-09-15

由于之前熟悉了一下[web component](./欢迎web-component到来.md)，所以对于它比较感兴趣，甚至内心还有个“是否可以自己基于web component写一个前端库“的冲动。当然，碍于技术以及惰性，我觉得我是不会动手的。不过最近发现了一个web component的库，所以就去研究了一下。

FAST是基于Web组件和现代Web标准构建的技术集合。其主要包含，`@fluentui/web-components`，`@microsoft/fast-components`，`@microsoft/fast-foundation`与`@microsoft/fast-element`。

其中`@fluentui/web-components`与`@microsoft/fast-components`都是可开箱即用的组件库，两者的不同点主要在于其设计风格。`@fluentui/web-components`采用的是微软的流畅设计，也就是像windows/office/Edge这些设计模式，而`@microsoft/fast-components`则是一种行业为中心的设计系统并提供更多可自定义的配置。不过两者都实现并提供了目前主流UI框架都会提供的button,input,select等基础组件。

一个简单的使用`@microsoft/fast-components`的例子：

```html
<html>
  <head>
    <script type="module" sync src="https://unpkg.com/@microsoft/fast-components"></script>
  </head>
  <body>
    <fast-design-system-provider use-defaults>
      <!-- 按钮 -->
      <fast-button>Hello world</fast-button>
      <!-- 手风琴 -->
      <fast-accordion>
        <fast-accordion-item expanded>
          <span slot="heading">Panel one</span>
          Panel one content
        </fast-accordion-item>
        <fast-accordion-item>
          <span slot="heading">Panel two</span>
          Panel two content
        </fast-accordion-item>
        <fast-accordion-item expanded>
          <span slot="heading">Panel three</span>
          Panel three content
        </fast-accordion-item>
      </fast-accordion>
      <!-- 锚点/a标签 -->
      <fast-anchor href="https://fast.design" appearance="hypertext">FAST</fast-anchor>
      <!-- 标记 -->
      <fast-badge appearance="accent">Danger</fast-badge>
      <!-- 卡片 -->
      <fast-card>
        <h3>Card title</h3>
        <p>At purus lectus quis habitant commodo, cras. Aliquam malesuada velit a tortor. Felis orci tellus netus risus et ultricies augue aliquet.</p>
        <fast-button>Learn more</fast-button>
      </fast-card>
      <!-- 多选框 -->
      <fieldset>
        <legend>Fruits</legend>
        <fast-checkbox checked>Apple</fast-checkbox>
        <fast-checkbox checked>Banana</fast-checkbox>
        <fast-checkbox>Honeydew</fast-checkbox>
        <fast-checkbox checked>Mango</fast-checkbox>
      </fieldset>
    </fast-design-system-provider>
  </body>
</html>
```

其展现效果如下图：

![1](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200915155531.jpg!trans_webp)

对于使用这类组件库，基本都是一些中后台的管理系统，而`@fluentui/web-components`与`@microsoft/fast-components`在这方面的优势则是在于其编译简单，如上面的调用，并不需要使用webpack或者gulp进行代码转移，使用简单，但是缺点也比较明显，对于业务的特殊定制不友好且数据操作的变化比较困难（看官方实例是用原生js操作dom进行数据变更），这也是fast还不成熟的一个表现。

而`@microsoft/fast-foundation`则是一个fast的基础框架，它主要提供一个底层的模版以及数据逻辑，将顶层的样式表现交给开发者，供开发者实现自己的设计风格组件。由于对它不是很感兴趣，所以这里不做过多介绍。如果想要进一步了解，可以去它的官网了解一下 -> [链接](https://fast.design/docs/introduction)

## 自定义组件

由于日常工作中写c端的应用多一些，所以经常需要特殊定制一些组件。而fast也提供了自定义的能力，所以着重研究了一下。

让我们从一个简单的todo-list组件开始入手。

首先需要自己搭建好应用的架子，我采用的是webpack + ts。简单搭好架子安装依赖`npm i @microsoft/fast-element`（没错，就是只有一个依赖），**注意我这里使用的版本是0.17.0**

新建index.ts

```ts
const { FASTElement, customElement, html } = require('@microsoft/fast-element');

const template = html<List>`
  <div class="list">list</div>
`;

@customElement({
  name: 'todo-list',
  template,
})
export default class List extends FASTElement {
}
```

这样我们就定义了一个`todo-list`的标签，你就可以在页面中直接通过`<todo-list></todo-list>`使用。

接着我们需要生产数据，所以我们需要一个输入组件，为我们添加待办事项。

新建add.ts

```ts
const { FASTElement, customElement, observable, html } = require('@microsoft/fast-element');

const template = html<AddInput>`
  <div class="add-input">
    <input :value="${(x: AddInput) => x.value}" @input="${(x: AddInput, c) => x.inputHandle(c.event)}" />
    <button @click="${(x: AddInput) => x.clickHandle()}">添加</button>
  </div>
`;

@customElement({
  name: 'add-input',
  template,
})
export default class AddInput extends FASTElement {
  @observable value = '';

  inputHandle(e: any) {
    this.value = e.target.value;
  }

  clickHandle() {
    if (this.value) {
      this.addFn(this.value);
      this.value = '';
    }
  }
}
```

输入组件比起列表组件多了很多东西，其中在html的模版中，我们使用了箭头函数获取数据并映射到dom中，这是fast获取数据的一种方式，通过这样的回调映射我们就可以将组件类上的数据与dom模版关联，达到双向数据绑定的效果。

而需要双向绑定的数据则需要使用`observable`装饰器进行定义，这一点跟`mobx`是一样的。

最后在点击“添加”按钮的时候，调用this.addFn方法将数据结果抛出，这个方法是从外部传入的，类似于react的this.props.addFn。只是在fast可以直接从this调用。

接下来将输入组件添加到列表头部。

```ts
const { FASTElement, customElement, observable, html } = require('@microsoft/fast-element');
import AddInput from './add'

AddInput; // 注意这里需要使用一下引用，否则可能被打包时移除

interface IItem {
  name: string;
  done: boolean;
}

const template = html<List>`
  <div class="list">
    <add-input :addFn="${(x: List) => x.addHandle}"></add-input>
  </div>
`;

@customElement({
  name: 'todo-list',
  template,
})
export default class List extends FASTElement {
  @observable list: IItem[] = [];

  addHandle(val: string) {
    this.list.unshift({
      name: val,
      done: false
    });
  }
}
```

由于web组件定义后可直接通过web标签使用，所以配置了webpack将未被使用的import移除的话，会将组件的定义移除，解决方法很简单，就是引用之后将应用对象应用一遍。就像上面的AddInput;

此时我们已经可以往列表组件内添加数据了，但是我们还需要将数据展示出来，所以定义todo-item组件。

item.ts

```ts
const { FASTElement, customElement, html, when, attr } = require('@microsoft/fast-element');

const template = html<Item>`
  <div class="c-item" @click="${(x: Item) => x.clickHandle()}">
    ${(x: Item) => x.ccontent}
    ${/* 注意这里使用的when相当于if */
    when((x: Item) => x.done, html<string>`<span>☑️</span>`)}
  </div>
`;

@customElement({
  name: 'todo-item',
  template,
})
export default class Item extends FASTElement {
  @attr ccontent: string; // 注意这里的是attr装饰器

  @attr done: boolean;

  clickHandle() {
    this.toggleState(!this.done);
  }
}
```

注意这里使用了两个新的特性`when`和`attr`, 其中when表示，当第一个参数为true时，展示传入的第二个参数，相当于react的if/vue的v-if。而attr则是指对于dom节点上属性的应用，即改变了dom节点的数据，该值会对应改变，反过来该值变了dom的属性也会一起改变。

将todo-item添加到todo-list中。

```ts
const { FASTElement, customElement, observable, html, repeat } = require('@microsoft/fast-element');

const template = html<List>`
  <div class="list">
    <add-input :addFn="${(x: List) => x.addHandle}"></add-input>
    ${repeat((x: List) => x.list, html<Item, List>`
      <todo-item :ccontent="${(x: IItem) => x.name}" :done="${(x: IItem) => x.done}" :toggleState=${(_: IItem, c) => c.parent.toggleState.bind(c.parent, c.index)}></todo-item>
    `, { positioning: true })}
  </div>
`;

export class IItem {
  @observable name: string = '';
  @observable done: boolean = false;

  constructor(name: string) {
    this.name = name;
  }
}

@customElement({
  name: 'todo-list',
  template,
})
export default class List extends FASTElement {
  @observable list: IItem[] = [];

  addHandle(val: string) {
    this.list.unshift(new IItem(val));
  }

  toggleState(index: number, val: boolean) {
    this.list[index].done = val;
  }
}
```

**这里有几个点是需要注意的**：

- repeat函数相当于forEach循环，其第一个参数传入需要循环的数组，第二个参数传入循环生成的模版内容。而在其循环体中，箭头函数发生了更改，其第一个值表示该次循环的item，第二个参数则是一个聚合对象，包含以下内容:

| 属性名 | 含义 |
| -- | -- |
| event | 事件处理程序中的事件对象 |
| parent | 上层作用域上下文引用，这里指的就是todo-list组件 |
| index | 序号 |
| length | 数组长度 |
| isEven | 是否是偶数次循环 |
| isOdd | 是否是奇数次循环 |
| isFirst | ～ |
| isInMiddle | ~ |
| isLast | ~ |

由于加入这些属性的成本比较高，所以默认是不可用的，需要使用则需要在第三个参数传入`{ positioning: true }`进行开启。

- 对象引用的数据变化无法监听到。

  这点比较烦，如果只对list加了observable装饰器，那当改变list的数据的时候无法更新组件。而解法有两个，1. 每次更新list都重新生成一个新引用，2. 将需要监听变动的属性都使用observable进行监听。

  我们这里采用的是第二个方案。

- 节点属性需要加上`:`才能监听到变更，如上面todo-item组件的ccontent属性，需要像`<todo-item :ccontent="${(x) => x.name}"></todo-item>`这样传入节点。

就这样一个简单的`todo-list`组件就完成了，其中包含一个`todo-list`的父组件，和`add-input`,`todo-item`自组件。

效果如下：

![效果](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200915172456.jpg!trans_webp)

## 总结

到这里对于fast的介绍基本已经完了，由于web-component在浏览器端的支持程度还不高，且react/vue/angular当道，以至于很少有人注意到一个组件化模块化的原生DOM规范正在悄然发展。

不可否认fast还很稚嫩，很多方面做的不够好甚至一团糟，比如数据只能通过函数的形式映射到dom中，或者是dom只能通过字符串的形式定义，抑或是对于引用数据类型的深度数据变动监听不支持等。可以预见它还有很长一段路要走，但是这并不妨碍它可能成为未来的主流技术。

## 参考链接

[FAST Introduction](https://fast.design/docs/introduction)

[FAST examples](https://github.com/microsoft/fast/tree/master/examples)
