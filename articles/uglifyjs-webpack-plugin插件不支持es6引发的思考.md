# uglifyjs-webpack-plugin插件不支持es6引发的思考

[tag]:webpack|思考|js
[create]:2020-09-14

## 背景

为什么会想到这个问题？

最近在研究`web component`，发现微软不声不响的开发了一个`web component`的库 -- [fast](https://fast.design/docs/introduction)，所以自己就二话不说，操起键盘就自己从头撸了一个webpack+ts的配置，打算试探试探它。

而在打包的时候准备压缩一下代码，看看最终它打包出来有多大。结果发现了一个问题[“UglifyJS不支持ES6”](https://stackoverflow.com/questions/47439067/uglifyjs-throws-unexpected-token-keyword-const-with-node-modules)。

然后百度谷歌查解决方法，查问题出在哪，结果是基本上所有人都说是因为UgliftJS不支持ES6，只要把它换成[terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin)就好了。

显然这个答案不能让我满意，这只是提出了一个更好的替代方案。这是因为我想到，明明我的代码已经编译成了es5了，但是UgliftJS报错是不支持es6，这是因为编译的时候顺序的问题吗？

我理想中UgliftJS应该是在代码编译完之后才会跑的（实时似乎也是这样，之前用它的时候似乎也没报错）。而我看生产出来的js文件里并没有es6语法，说明转为es5语法是成功的。

这就只有一种可能了，UgliftJS的运行在编译之前，这看上去似乎有些不合理。

这激起了我的好奇心，我决定搞清楚它究竟是怎么运行的。

## ugliftJS的调用时机

webpack的插件调用都是通过调用插件暴露出来的apply方法，而apply方法主要的作用就只是将插件真正的执行挂载在某个编译的特定时机上。

一个极简的插件长下面这样:

```js
module.exports = class TestPlugin {
  apply(compiler) {
    compiler.hooks.compile.tap('TestPlugin', (compilation) => {
      console.log('----> testPlugin');
    });
  }
}
```

在webpack初始化时，就会将plugins循环，调用每一个插件的apply方法，这时不同的插件的执行就由其自己决定了。

而常用的编译钩子有以下这几个

| 钩子 | 说明 | 参数 | 类型 |
| -- | -- | -- | -- |
| afterPlugins | 启动一次新的编译 | compiler | 同步 |
| compile | 创建compilation对象之前 | compilationParams | 同步 |
| compilation | compilation对象创建完成 | compilation | 同步 |
| emit | 资源生成完成，输出之前 | compilation | 异步 |
| afterEmit | 资源输出到目录完成 | compilation | 异步 |
| done | 完成编译 | stats | 同步 |

而上面的TestPlugin插件则是挂载在compile钩子上，也就是在创建compilation对象之前。而tap则是指添加一个同步钩子，此外还有两个异步钩子，它们都是由[Tapable](https://www.npmjs.com/package/tapable)类提供:

- tap: 同步钩子

- tapAsync: 异步钩子，通过callback通知异步调用结束

- tapPromise: 异步钩子，通过promise通知异步结束

关于自定义组件这块不再深入，详情可以看看[webpack自定义插件](https://juejin.im/post/6844904095774408711)。写的比较详细了。（还有一个点是， webpack中文网上的自定义插件的教程用的还是老版本的写法，其中钩子部分已经无效了)

既然知道了插件的调用时机是由插件自己决定的，那么事情似乎就很明朗了，只要我们打开`uglifyjs-webpack-plugin`的项目，直接查看它的apply方法内将执行挂载到哪个时候。

```js
apply(compiler) {
    const buildModuleFn = ...

    const optimizeFn = ...

    const plugin = {
      name: this.constructor.name
    };
    compiler.hooks.compilation.tap(plugin, compilation => {
      if (this.options.sourceMap) {
        compilation.hooks.buildModule.tap(plugin, buildModuleFn);
      }

      const {
        mainTemplate,
        chunkTemplate
      } = compilation; // Regenerate `contenthash` for minified assets

      for (const template of [mainTemplate, chunkTemplate]) {
        template.hooks.hashForChunk.tap(plugin, hash => {
          const data = (0, _serializeJavascript.default)({
            uglifyjs: _package.default.version,
            uglifyjsOptions: this.options.uglifyOptions
          });
          hash.update('UglifyJsPlugin');
          hash.update(data);
        });
      }
      // optimizeChunkAssets钩子是在优化资源之前
      compilation.hooks.optimizeChunkAssets.tapAsync(plugin, optimizeFn.bind(this, compilation));
    });
  }
```

上面的代码是`uglifyjs-webpack-plugin`的apply方法内主要的钩子绑定逻辑。

而其中先是在compilation(compilation对象创建完成)的时候挂载了事件，此时显然代码是还未编译的，而后从compilation里面取出`mainTemplate`和`chunkTemplate`，然后再分别给它们绑上hashForChunk事件钩子，这两个钩子查文档并没有查到是什么钩子，但是这钩子是挂载`mainTemplate`和`chunkTemplate`上，似乎并不是我们要找的，先跳过。

而后面再次给optimizeChunkAssets(优化资源之前)挂载了一个异步的钩子, 此时调用的`optimizeFn`应该就是压缩代码的具体逻辑了。

所以代码的压缩是在资源优化之后触发的，这时候代码中的es6应该已经转成es5了才对，怎么会有报不支持const压缩的错误呢。

稳妥起见，先将uglify-js插件调用前的编译文本log出来看了一下（在optimizeFn直接将待压缩源码log出来），结果发现在代码中的es6语法已经转成es5了，但是node_modules中的包却还是es6。

但是明明已经加了babel-loader，理论上这部分引用也应该转成es5了呀。

## loader调查

既然已经知道插件的调用时机没有问题，那就从调查loader入手，首先想到要确定插件到底有没有调用。

而loader的原理，简单来说就是提供一个转换函数，webpack会将待编译的源码以文本的形式传入该方法，然后可以选择同步或异步的执行编译，然后将编译结果返回。

对于测试插件是否调用了只需要在待调用插件前后各安排一个自己写的插件，看看自己的插件是否有调用就可以了。

一开始是这么想的，当然也是这么做的。

写一个仅仅是校验是否有调用的插件很简单：

```js
// test-loader.js
module.exports = function(source) {
  console.log('----> test loader');
  return source;
}
```

不需要对源码做任何处理的直接返回，只要调用了该插件就会log出内容。

还需要在webpack.config.js内定义一下loader的目录, 我们写的loader放在了根目录下的`./webpack-loaders`文件夹:

```js
resolveLoader: {
  modules: ['./node_modules', './webpack-loaders']
}
```

然后简单的配置了一下loader。

```js
{ test: /\.js$/, use: ['test-loader2']},
{ test: /\.js$/,
  exclude: /(node_modules|bower_components)/,
  use: {
    loader: 'babel-loader',
    options: {
      presets: ['@babel/preset-env']
    }
  }
},
{ test: /\.ts$/, use: ['test-loader', 'ts-loader', 'test-loader2'] },
```

测试之后，发现babel-loader没有调用。然后想到，我可以直接找到babel-loader的代码里面进行log啊（真是猪油蒙了心）。

期间发现一个点，就是 **“loader的调用顺序是从右到左的”**，比如上面对于ts文件的调用顺序应该是`test-loader2` -> `ts-loader` -> `test-loader`。

直接在babel-loader里面加了log，甚至连ts-loader内也给加上了。

果然是babel-loader没有编译，为什么呢？为什么babel配置了却不会跑呢。但是ts-loader是有跑的。

## 水落石出

仔细观察了一下ts-loader的log，发现src目录下的两个ts文件都被编译了，然后打开src目录，看着仅有的两个ts文件，果然如此。

因为我的代码都是ts文件，所以跟babel-loader的匹配规则不符肯定是不会调用babel-loader的，但是明明引入的npm包是.js文件，却也没有编译。

脑中突然闪过一道闪电，不会是我排除掉了node_modules目录吧，打开webpack.config.js一看，果然`exclude: /(node_modules|bower_components)/`。

由于babel的配置是直接复制过来的，所以根本没有注意到，而且在加压缩代码插件之前，由于浏览器支持大部分的es6了，所以即使不编译也不会报错。

**所以最终的结果是，由于我的疏忽，导致babel-loader忽略了node_modules文件夹，从而导致第三方模块没有编译，引发的uglify-js因不支持es6而无法压缩。**

## 总结

从这件事上看出了我的粗心以及对于webpack不够熟悉。而从中却又学到了关于webpack插件的知识, loader的知识，觉得非常开心。

而`uglifyjs-webpack-plugin`居然不支持es6语法的压缩，这无异是非常吃亏的，毕竟浏览器都已经支持了，而工具库反而还没支持，确实是有些匪夷所思。

还有就是很多事情确实不如网上说的那样，当你有异议而又有能力去搞清楚问题所在，就去调查清楚。就像这次网上的所有答案都说是因为`uglifyjs-webpack-plugin`不支持es6语法的压缩，只需要将其换成`terser-webpack-plugin`就可以了，却没有意识到为什么要换呢？

## 参考资料

[动态数据校验之 JSON Schema(ajv)](https://juejin.im/post/6844904017487724558#heading-23)

[webpack自定义插件](https://juejin.im/post/6844904095774408711)

[webpack自定义loader](https://juejin.im/post/6844904095774408711)
