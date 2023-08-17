# 一个useReducer使用示例

[tag]:react|useReducer
[create]:2023-01-22

在一开始看Hooks的时候，我确实不太理解`useReducer`应该如何使用，似乎`useState`已经足够我平时开发使用，而在实际编码后才能逐渐理解`useReducer`所应付的场景，这里碰到一个很棒的例子，也刚好借此写下自己的理解。

考虑如下代码，我们假设有一个书单组件，用例是允许用户扫描查找书籍，由于查找服务的速率限制，不能立刻就查找到，因此用上socket。整体逻辑很简单，但是我们如果没有经过深思熟虑，很容易就会写出类似下面这个逻辑。在组件挂载的时候链接socket，同时处理业务逻辑，并且在卸载的时候将socket关闭。
```js
const BookEntryList = props => {
  const [pending, setPending] = useState(0);
  const [booksJustSaved, setBooksJustSaved] = useState([]);

  useEffect(() => {
    const ws = new WebSocket(webSocketAddress("/bookEntryWS"));

    ws.onmessage = ({ data }) => {
      let packet = JSON.parse(data);
      if (packet._messageType == "initial") {
        setPending(packet.pending);
      } else if (packet._messageType == "bookAdded") {
        setPending(pending - 1 || 0);
        setBooksJustSaved([packet, ...booksJustSaved]);
      } else if (packet._messageType == "pendingBookAdded") {
        setPending(+pending + 1 || 0);
      } else if (packet._messageType == "bookLookupFailed") {
        setPending(pending - 1 || 0);
        setBooksJustSaved([
          {
            _id: "" + new Date(),
            title: `Failed lookup for ${packet.isbn}`,
            success: false
          },
          ...booksJustSaved
        ]);
      }
    };
    return () => {
      try {
        ws.close();
      } catch (e) {}
    };
  }, []);

  //...
};
```

## 问题

上面代码很简单，问题也很明显。我们使用了state变量pending但是没有将其设为依赖，由于闭包的原因，代码中的pending将始终为0。
当然，我们有多种方法可以解决这个问题:
1. 将pending列为依赖项，但是这样一来每当pending变化的时候我们都将重新链接socket。
2. 使用函数式的setPending更新状态，简单看来这不失为一个好方法。
3. 使用useRef始终获取最新的pending值，这个也可行。

当然还有更多的解决方案，但是我们先今只讨论Hooks相关的功能，上面三个方案基本上都可以解决闭包的问题，但是第一个由于会导致socket反复重新链接首先排除。第二个方案乍一看上去也解决了关键问题，但是如果不仅仅是设置值的时候需要引用到该state，该代码又将变得再度变得棘手。第三个方案倒是解决的比较彻底，但是一旦引用的state变多，这将会增加很多冗余的代码，使组件变得复杂难以维护。

## 使用useReducer

那我们看看使用`useReducer`的解决方案如何:

```js
function scanReducer(state, [type, payload]) {
  switch (type) {
    case "initial":
      return { ...state, pending: payload.pending };
    case "pendingBookAdded":
      return { ...state, pending: state.pending + 1 };
    case "bookAdded":
      return {
        ...state,
        pending: state.pending - 1,
        booksSaved: [payload, ...state.booksSaved]
      };
    case "bookLookupFailed":
      return {
        ...state,
        pending: state.pending - 1,
        booksSaved: [
          {
            _id: "" + new Date(),
            title: `Failed lookup for ${payload.isbn}`,
            success: false
          },
          ...state.booksSaved
        ]
      };
  }
  return state;
}
const initialState = { pending: 0, booksSaved: [] };

const BookEntryList = props => {
  const [state, dispatch] = useReducer(scanReducer, initialState);

  useEffect(() => {
    const ws = new WebSocket(webSocketAddress("/bookEntryWS"));

    ws.onmessage = ({ data }) => {
      let packet = JSON.parse(data);
      dispatch([packet._messageType, packet]);
    };
    return () => {
      try {
        ws.close();
      } catch (e) {}
    };
  }, []);

  //...
};
```

虽然整体代码行数有所增加，但是我们的整体逻辑得到了简化，useEffect主体更加简单，可阅读性也更高。没有闭包带来的state引用问题，同时整体更易测试。将读取与写入分开，我们的useEffect本身只关心调度动作。

其中还有一个小技巧，在diapatch中船驶入的数据是一个长度为2的数组，第一个值为操作的类型，第二个是具体的数据。相比于只传一个包含了操作类型的对象，这种数组的传参方式使操作类型与实际数据彻底区分开，也更易于理解。

## 参考:
[state-and-use-reducer](https://adamrackis.dev/blog/state-and-use-reducer)
[useReducer docs](https://react.dev/reference/react/useReducer#)