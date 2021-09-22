# React使用useReducer实现全局状态管理(20行代码)

[tag]:react|hook|store
[create]:2020-09-22

![示例](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/reducer-demo.gif)
[最终效果]

最近终于空出时间来打算仔细学习一下Hooks。打开文档的时候就想到一个问题“如何用Hooks实现一个全局状态管理“，之前似乎听到过同事说过有这么一回事，当时只觉得有些惊奇，原来hooks已经可以实现全局状态管理了吗？

但是出于对hooks的不了解，所以当时也没多想。而当点开hooks的文档时，自然而然的想要探究一下，究竟是怎么实现的。所以就给自己定了一个学习hooks的小目标 -- “尝试使用hooks实现一个全局状态管理库”。

首先入我法眼的是`useReducer`API, 心想还真是巧了，`reducer`？跟`redux`里面的`reducer`肯定是一个东西，看来`hooks`能够实现全局状态管理是真的。

当然，事实上我猜对了一丢丢，实现全局状态管理确实需要用到`useReducer`，但是却不够。

`useReducer`仅仅是`hooks`中提供的一个简单的方法，仅仅提数据与函数方法绑定的能力，它并不足以写一个全局状态管理库。

## `useReducer`的用法

`useReducer`的用法很简单, 这里直接使用官方示例（只是因为懒）。

```jsx
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

如果日常中用过redux进行开发的人应该很容易就看懂其中的逻辑，将处理函数与初始值传入`useReducer`方法，会返回`state`和`dispatch`。而通过调用`dispatch`，将触发处理函数，对数据进行加工。

一开始看到这里，就觉得“妥了”，直接将`useReducer`提到顶层，将整个应用包裹，然后将state和`dispatch`下发到需要的组件，不就是一个很简单的全局状态管理了吗？

当然不行，`useReducer`只能在函数组件内部调用，并且只有当前调用的函数组件能够有数据变更对应视图也改变的能力。所以这是行不通的。

然后自然的产生第二个思路，将数据提取出来，只维护一份数据，然后每个组件都使用这份数据调用`useReducer`，这样就实现多个组件操作一份数据。

## 单独使用useReducer实现（行不通）

![useReducer示意图](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/useReducer.drawio.png!trans_webp)

上图是具体的示意图，当页面1和页面2使用同一份数据(`globalData`)和同一个数据处理函数(`function reducer`)时，当页面1修改了数据，那么页面2使用时拿到的肯定也是最新的那份数据。这样就做到了页面共享数据的效果了。

但是这里很明显有个漏洞，那就是页面间的数据同步生效了，但是同页面的组件如何通信呢？

比如我在页面1的组件A里修改了一个变量a，组件A的状态更改了，并且`globalData`也更改了，但是我在组件B内也用到了变量a，因为组件A和组件B是两个不同的`state`(因为各自使用了`useReducer`生成，它们之间互不相干)，就导致组件B的状态无法更新。

或许这里可以使用订阅的方式去更新每个组件，但是这并不是我们期望的，并且消耗大还麻烦。所以这个方式是行不通的。

在这里，我们又回到了原点，总结一下我们到底遇到了什么问题？

**总的来说，我们缺少一个唯一的全局的双向数据绑定的这么一个机制，让我们仅声明一份数据，然后这份数据每次变更都能够响应给引用了该数据的组件。**

但是我们真的没有这种手段吗？很显然并不是，我们还有`context`啊！

## 加入context

我们先规划一下大概的架构应该怎么样：

1. 由于`context`需要使用`provider`包裹组件，所以先要提供一个包裹组件。

2. 由于是全局状态组件，需要将数据存储(`globalData`)和处理函数(`reducer`)提取出来，从外部提供。

3. 需要提供`connect`和`dispatch` API，让使用者能够全局获取和修改数据。

总的来说，我们只需要提供这几点的服务，一个简单的全局状态管理组件就完成了。

![reducer示意图](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/quanjuzhuangtaiguanli.drawio.png!trans_webp)

如上示意图，首先我们实现一下`Provider`组件：

(由于我是用ts进行开发的，所以全部都是采用ts语法写的，就不改了)

```tsx
let _dispatch = null;

const context = React.createContext({});

export default function Provider(props: {
  children: JSX.Element;
  innitalState: any;
  reducer: any;
}) {
  const [state, dispatch] = useReducer(props.reducer, props.innitalState);
  _dispatch = dispatch;
  return (<context.Provider value={state}>
    {props.children}
  </context.Provider>);
};
```

上面代码中，首先声明了一个变量`_dispatch`，这个变量的作用则是保存`useReducer`生成的`dispatch`变量，因为需要将修改数据的能力暴露给其他组件，所以需要将其保存到`Provider`组件外部，这里后面会在暴露出去的dispatch函数中使用到。

接着我们创建了一个`context`变量。这里为什么又将创建放在了组件外部呢？因为后面需要在`connect`函数内用到，如果放在`Provider`组件内创建的话，那么`content`的创建只能在组件初始化的时候调用，而`connect`的调用是在这之前的，所以要将`context`的创建提前。

然后就是我们的`Provider`组件的主体了。可以看到主要有三个`props`，第一个`children`自然就不用多说了，第二个`innitalState`参数表示的是全局状态的初始值，第三个就是处理函数了。在组件内部，使用`useReducer`创建响应数据以及 `dispatch`回调。

然后实现第二部分 -- `connect`.

```tsx
export function connect<P>(mapStore: (state: any) => any): (Component: React.ComponentClass<P> | React.FunctionComponent<P>) => React.FunctionComponent<P> {
  return function(Component: React.ComponentClass<P> | React.FunctionComponent<P>): React.FunctionComponent<P>{
    return function connectComponent(props: P & { children?: ReactNode }) {
      const c = useContext(context);
      const _state = mapStore(c);
      return <Component {...props} _context={_state}></Component>;
    }
  }
}
```

根据ts的类型注解也基本可以看明白，`connect`函数返回一个匿名函数，然后匿名函数要求传入 一个`ComponentClass`类组件，或者是一个`FunctionComponent`函数组件。接着匿名函数会返回一个函数组件，在这个函数组件中，我们使用`connect`函数的入参过滤一遍全局状态数据，再将过滤后的数据传递给最终的组件的`_context`变量。（这里为什么不使用...将参数散列开呢？因为这里设计成了可以是一个基本数据类型，不一定就是对象的传递）。

最终的调用如下：

```tsx
export default connect<InterfaceComponentProps>((store) => {
    return {
        a: store.a,
        b: store.b
    }
})(ClassComponent | FunctionComponent)
```

其中在具体的UI组件中，可以通过`_context`变量获取传入的全局状态：

```js
const _context = this.props._context;
console.log(_context.a, _context.b);
```

这个实现主要是参照`redux`的写法，因为这样写确实能够解决问题，主要解决的问题就是过滤全局状态，因为全局数据太多，而具体到某个组件，只会用到其中一小部分的内容，所以过滤出需要的内容，更加方便做优化。

然后是`dispatch`方法：

```tsx
export const dispatch = (params) => {
  if (_dispatch) {
    _dispatch(params);
  }
};
```

`dispatch`方法是最简单的，我们只需要兼容一下`_dispatch`是否存在就可以了。（因为当你没有使用`Provider`组件时，是没有`_dispatch`方法的，这个方法在上面的初始值为`null`, 只有调用了`useReducer`方法后才将真实的`dispatch`赋值给它）

到这里我们的全局状态管理组件就写完了。

我们提供了`Provider`组件用来接收初始值(`inintalState`)和处理函数(`reducer`)，以及提供`context`。

我们提供了`connect`组件，用来给具体的组件引用全局状态。

还提供了`dispatch`函数触发数据变更。

没错，我们只完成了这三件事。

## 具体使用

那么我们就实验一下，首先创建`initalState`数据，以及`reducer`处理函数，这里我们选择写一个简单的`todolist`：

```ts
export const task = {
  all: 0,
  completed: 0,
  uncomplete: 0,
  tasks: [],
};

export function reducer(state = task, action) {
  switch(action.type) {
    case 'add':
      state.all += 1;
      state.uncomplete += 1;
      state.tasks.push({ description: action.value, completed: false });
      return {...state};
    case 'complete':
      state.completed += 1;
      state.uncomplete -= 1;
      state.tasks[action.value].completed = true;
      return {...state};
    case 'edit':
      state.tasks[action.index].description = action.value;
      return {...state};
    default:
      console.warn('Can\'t find type. -- ' + action.type);
      return state;
  }
}
```

只记录了任务列表，所有任务数，已完成任务数，未完成任务数。并且在处理函数内提供“新建，完成，编辑”三个状态。

使用`Provider`：

```tsx
import React from 'react';
import ReactDOM from 'react-dom';
import Router from './router';
import Provider from './stores/useGlobalReducer';
import { reducer, task } from './stores/tasks';

export default ReactDOM.render(
  <Provider innitalState={task} reducer={reducer}>
    <Router />
  </Provider>, document.querySelector('#app'));
```

在`todoList`组件内使用`connect`和`dispatch`:

```tsx
import React from 'react';
import { dispatch, connect } from '../../stores/useGlobalReducer';
import { Link } from 'react-router-dom';

interface Iprops {
  _context?: {
    tasks: Array<{ description: string; completed: boolean; }>;
  };
}

function TodoList(props: Iprops) {
  const context = props._context;
  return (
    <ul>
      {context.tasks.map((item, index) => {
        return (<li key={index}>{item.description} -- <span onClick={() => {
          !item.completed && dispatch({ type: 'complete', value: index }); // 这里使用了dispatch
        }}>{item.completed ? '已完成' : '未完成'}</span> <Link to={"/detail/" + index}>编辑</Link></li>)
      })}
    </ul>
  );
}

export default connect<Iprops>((state) => { // connect
  return { tasks: state.tasks };
})(TodoList);
```

就这样，简单的调用方式。

1. 创建初始状态值(`inintalState`)与处理函数(`reducer`)。

2. 将初始状态值与处理函数传入`Provider`组件，并包裹在项目最外部。

3. 在具体需要引用全局状态的组件内部使用`connect`链接组件与全局状态。

4. 在数据变更时，使用`dispatch`触发。

具体效果可以[点击查看](https://sansui-orz.github.io/blog/demos/hooks/dist/index.html), 需要翻墙才能访问。或者可以将代码拷贝下来[查看](https://github.com/sansui-orz/blog/tree/master/demos/hooks)

[源码](https://github.com/sansui-orz/blog/tree/master/demos/hooks)

## 缺点

自己实现的简陋版自然是不完善的，必然后很多场景下用起来比较痛苦。而最大的一个缺点是当数据变更时，所有的引用组件都会被更新，即使它们的`props`并没有变化。有一个不完善的解决方案是自己手动判断`props`是否变化进而控制组件的更新。但是当组件的 `props`太多时就无法这么做了。

~~其他目前还没发现太大的缺点。~~

还有就是还不支持多store，但是这个问题要解决并不是很困难，只需要将多store再组合成一个总store，变更时遍历执行所有的reducer，理论上就解决了。当然，实际上我太懒了没去尝试，但是思路应该是没问题的。

## 总结

总的来说，对于这个实现还是比较满意的，毕竟真正核心代码不到30行，去除一些if语句框以及ts的类型，甚至不到20行。

并且在这其中涉及到比较多的知识点：

1. 高阶函数。

2. useReducer hook。

3. context && useContext hook。

4. React.memo && useMemo 管理函数组件更新。

5. ts中的泛型的使用。

6. 代码组织。

当然这个全局状态管理比较简陋，但是我认为在一些小型项目中使用是完全没有问题的。毕竟只需二十行代码就可以抛弃掉臃肿的`reduc | mobx`，只要 `react`版本够得到，就可以开箱即用。忍不住真香。
