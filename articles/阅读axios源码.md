# 阅读axios.js源码

[tag]:记录|js|源码
[create]:2019-10-28

记录在阅读axios源码时发现的一些亮点：

1. 看入口文件axios.js，可以注意到一个axios实例并不是通过new一个实例来生成的，而是通过axios自身的一个方法，叫`axios.create`

在内部实现中，axios本身是一个函数，跟Axios.prototype.request是同一个函数，但是是通过bind方法产生的一个新的函数，然后将Axios.prototype内的所有属性继承到axios中, 这样就产生了一个新的axios实例，并且axios本身还可以被当作函数直接调用，我想这就是为什么它不适用new关键字直接生成一个实例的原因。但是同时又有点很没必要，这样做的话它们之间并没有继承关系。

2. 在core/Axios.js中，请求前，响应后的中间件处理很有意思。

它在实例化的时候会在当前实例挂载一个interceptors属性，里面有两个属性值，一个是request(用来存放请求前中间件)，一个是response(用来存放请求后中间件)

然后它的中间件的结构也挺有意思，它每组中间件需要两个函数组成，一个then函数，一个catch函数。

然后当发起请求的时候，会将进行序列化，序列化过程类似于这样:

```javascript
class Axios {
  constructor() {
    ...
    this.interceptors = {
      request: [],
      response: []
    };
  }

  request(...) {
    ...
    var promise = Promise.resolve(config); // 先将config的结构抛出，然后经过层层中间件的修改，最终得到完整的内容
    // 序列化中间件, dispatchRequest该方法就是发起请求的方法
    var chain = [dispatchRequest, undefined];
    /* this.interceptor.request的结构大概是这样
    * [{ thenFun: fn, catchFun: fn }, { thenFun: fn, catchFun: fn }]
    */
    this.interceptor.request.forEach(({ thenFun, catchFun }) => {
      chain.unshift(thenFun, catchFun); // 将方法放到数组前面
    });
    this.interceptor.response.forEach(({ thenFun, catchFun }) => {
      chain.push(thenFun, catchFun); // 将方法放到数组后面
    });

    /* 然后遍历这个数组，这时候这个数组的结构就很规范
    * 由于请求前处理插入到了数组前面，中间是请求方法，后面是响应后的处理
    * 所以直接用promise将这一串数组给串联起来，就是一个完整的流程
    */
    while (chain.length) {
      promise = promise.then(chain.shift(), chain.shift());
    }
  }
}
```

其他一些内容就比较中规中矩，没有太大的亮点。其中数据的格式化与校验做的比较详细，以及浏览器端与node端的请求区分也挺有意思。
