# webpack手动构建Vue开发环境

[tag]:webpack|vue|config
[create]:2018-11-06

*当然，还是建议照着官方文档来，这篇文章基本上当作是学习笔记，有错误欢迎支持，后续也是会持续更新的*`

[中文官方起步教程](https://www.webpackjs.com/guides/)

废话不多说开始吧！

新建一个文件夹，名字就叫做demo1吧

然后进入文件夹打开命令行进行项目构建

**输入: `npm init -y`**

解释一下-y的意思就是跳过提问阶段，直接生成一个新的 package.json 文件。

接着安装一下依赖，先大概想好会需要什么依赖

由于是vue项目构建，所以当然需要vue, vue-router, 请求这里就用大家都爱用的axios
注意这三个依赖是需要在打包后的文件使用的，所以安装的时候需要加上`--save`

**执行: `npm install --save vue vue-router axios`**

然后由于会使用到es2015的语法，所以babel是要的
**执行: `npm install --save-dev babel-core babel-loader babel-preset-env babel-preset-es2015 babel-preset-stage-3`**

注意这里不使用`--save`，因为这里只是在开发与构建的时候使用的

注意经过测试Bebel-loader@8.x.x的版本会报错。所以请安装7.x.x，通过`npm install babel-loader@7`

当然webpack要加，这是我们的主角

**执行: `npm install --save-dev webpack webpack-cli html-webpack-plugin webpack-dev-server webpack-merge uglifyjs-webpack-plugin vue-template-compiler`**

解释一下这些都是做什么的

- webpack: 当然是主力
- webpack-cli: 命令工具，由于是看着文档来的，文档中使用的是webpack-cli来运行webpack，所以就用这个把。而实际vue项目中用的应该是cross-env
- html-webpack-plugin: webpack的html模版插件，用来生成html模版的
- webpack-dev-server: 开发热更替插件，就是当你改了代码然后帮你刷新的那个
- webpack-merge: 这个插件用来合并webpack的配置。由于后面会将开发环境与生产环境的配置分开来，所以先安装这个
- uglifyjs-webpack-plugin: 代码压缩插件

ok, 现在最后还剩下各种loader了

**执行: `npm install --save-dev css-loader style-loader file-loader vue-loader`**

这些loader的作用就是用来加载各种不同的资源以便于构建打包的；

然后到此我们的依赖就装的差不多了，剩下的一点就在项目进行中添加吧！

将这些依赖装完之后呢，我们需要先创建几个文件定一下文档结构：

```sh
|- /dist
|- /src
|  |- App.vue
|  |- index.js
|
|- .babelrc
|- index.html
|- webpack.common.js
|- webpack.dev.js
|- webpack.pro.js
|- package.json
```

可以看到我们将webpack配置拆分为三个文件，这是因为将开发环境与生产环境的配置分开来了。
而避免重复又将通用的配置提取到webpack.common.js里面。

首先先处理一下webpack.common.js。让我们一步一步来。

生产环境和开发环境通用的内容有很多，首先entry入口配置，output出口配置, module的rules配置

接下来在webpack.common.js写入以下内容:

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
    entry: './src/index.js', // 入口文件地址
    output: {
        filename: '[name].[hash].js', // 输出的文件名
        path: path.resolve(__dirname, 'dist') // 输出路径
    },
    plugins: [
        new HtmlWebpackPlugin({ // 生成构建html的模版
            template: './index.html' // 指定模版路径
        }),
        new VueLoaderPlugin() // vue解析所需要的插件
    ],
    module: {
        rules: [
            {
                test: /\.css$/, // 指定loader解析css文件
                use: ['style-loader', 'css-loader']
            },
            {
                test: /\.vue$/, // 指定loader解析vue文件
                use: 'vue-loader'
            },
            {
                test: /\.js$/, // 指定babel进行js转码
                use: [{
                        loader: 'babel-loader',
                        options: {
                            presets: ['es2015']
                        }
                    }],
                    exclude: /node_modules/
            },
            { // 指定引用文件loader
                test: /\.(png|jpg|gif|svg)$/,
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]?[hash]'
                }
            }
        ]
    }
}
```

如上，我们基本已经配置完了webpack的通用配置了，接下来就分别针对不同的环境进行一下特殊的配置

webpack.dev.js

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development', // 指定当前环境，支持参数development, production, none会对应开启一些优化
    devtool: 'inline-source-map', // 加入资源映射，方便在开发阶段进行调试
    devServer: { // 开启开发服务器
        contentBase: './dist'
    }
});
```

webpack.pro.js

```javascript
const webpack = require('webpack');
const merge = require('webpack-merge');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'production',
    plugins: [
        new UglifyJSPlugin({ // 启用代码压缩
            sourceMap: true
        }),
        new webpack.DefinePlugin({ // 指定当前环境，防止某些库会用到这个参数
            'process.env.NODE_ENV': JSON.stringify('production')
        })
    ]
})
```

到这里，webpack的基本配置就先这样了。优化就在后面再做。

接下来处理一下我们的入口文件，目前的入口文件还是空的

index.js

```javascript
import Vue from 'vue';
import App from './App.vue';

new Vue({
    el: '#app',
    render: h => h(App)
});
```

App.vue

```js
<template>
    <div>{{hello}}</div>
</template>
<script>
export default {
    name: 'app',
    data() {
        return {
            hello: '你好的vue和webpack'
        };
    }
}
</script>
```

到这里基本内容就完成了，那么如果运行呢

我们需要在package.json里面添加几条命令

package.json

```json
{
  "scripts": {
    "build": "webpack --config webpack.pro.js",
    "start": "webpack-dev-server --open --config webpack.dev.js"
  },
}
```

然后在命令行输入```npm start```即可运行。 输入```npm run build```打包

让我们先试一下有没有问题, 反正我这里已经是可以运行了。

如果报错了，需要认证的看错误信息，其实很多错误自己是完全可以解决的了。还要善于百度，必应，谷歌。一般我查错误都是先百度，然后如果没有找到解决方法再必应，最终没办法就只好谷歌了。没办法，谁叫英语没好好学呢。

接下来开始优化我们的配置。首先如果你修改一下index.js。然后打包，就会发现dist目录下会有两个main文件。这不是我们想要的效果，我们始终只想在dist下存在一个main文件。

所以我们要在每次打包之前先清除掉dist文件夹里面的文件。很巧的是刚好有个插件可以实现

安装`npm install --save-dev clean-webpack-plugin`

然后打包只有在生产环境的时候，所以我们将这个插件的使用放到webpack.pro.js里

修改后的webpack.pro.js

```js
...
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = merge(common, {
    ...
    plugins: [
        new CleanWebpackPlugin(['dist/*']),
        ...
    ]
});
```

注意这里给clean-webpack-plugin传入的参数为['dist/*']，与官方指南的不同，因为穷人家是在window下操作的，mac下估计是['dist']（我也不知道的啦哈哈哈）

好的，现在打包-修改-打包。就发现dist里面只剩下一份代码。nice~接着我们来第二步优化

一般来说项目中很多框架从初始化项目结构的时候起，内容就不会变的。比如说，你初始化项目的时候用的vue的版本是2.5.0那么估计以后也是一直使用这个版本的，所以这部分代码基本不会改变，我们应该把它抽离出来做一个单独的文件，这样用户加载的时候就可以尽可能的使用缓存了。不然每次修改了代码然后重新打一个包出来，用户又要加载这个巨大包。

那么，动手吧！
修改后的webpack.common.js长以下那样

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  entry: {
    main: './src/index.js',
    vendor: [
      'vue'
    ]
  }, // 入口文件地址
  output: {
    filename: '[name].[hash].js', // 输出的文件名
    path: path.resolve(__dirname, 'dist') // 输出路径
  },
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          name: 'vue',
          chunks: 'initial',
          minChunks: 2
        }
      }
    }
  },
  plugins: [
    new HtmlWebpackPlugin({ // 生成构建html的模版
      template: './index.html' // 指定模版路径
    }),
    new VueLoaderPlugin() // vue解析所需要的插件
  ],
  module: {
    rules: [
      {
        test: /\.css$/, // 指定loader解析css文件
        use: ['style-loader', 'css-loader']
      }, {
        test: /\.vue$/, // 指定loader解析vue文件
        use: 'vue-loader'
      }, {
        test: /\.js$/, // 指定babel进行js转码
        use: [{
          loader: 'babel-loader',
          options: {
            presets: ['es2015']
          }
        }],
        exclude: /node_modules/
      }, { // 指定引用文件loader
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'file-loader',
        options: {
          name: '[name].[ext]?[hash]'
        }
      }
    ]
  }
};
```

先看看效果，先记下刚刚打包的main文件大小为67kb, 然后`npm run build`

看看打包之后：main文件大小为4kb，vendor文件大小为64kb。很明显vue已经被我们分离出来了。

但是还有个问题。如果你修改了index.js后再打包，你会发现文件名变了。因为我们之前的文件名有包含一个随机hash值。而修改了index.js的话，打包后的hash值就会变化。所以如果想要将vue缓存在用户客户端的话，那么就要让它的名字不要变化，这里我们可以用到webpack的另外一个插件HashedModuleIdsPlugin来控制hash

我们来修改一下webpack.pro.js

```js
const webpack = require('webpack');
const merge = require('webpack-merge');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const common = require('./webpack.common.js');
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = merge(common, {
  mode: 'production',
  plugins: [
    new CleanWebpackPlugin(['dist/*']),
    new webpack.HashedModuleIdsPlugin(), // <--- 注意增加了这一段。
    new UglifyJSPlugin({ // 启用代码压缩
      sourceMap: true
    }),
    new webpack.DefinePlugin({ // 指定当前环境，防止某些库会用到这个参数
      'process.env.NODE_ENV': JSON.stringify('production')
    })
  ]
});
```

并将webpack.common.js里面的输出配置里的hash改成chunkhash

```js
output: {
    filename: '[name].[chunkhash].js',
    path: path.resolve(__dirname, 'dist')
}
```

这样子我们再打包的时候就会发现不管怎么更改Index.js输出的vendor文件名已经不再变化了。

那么这样的话我们的项目架构就差不多搭建搞了。 当然有很多不完善，但是我会继续努力继续学习的。

顺便追加记录一下配置sass环境

首先需要先增加`npm install --save-dev node-sass sass-loader`

然后改一下webpack.common.js里面的rules的配置，如下:

```js
rules: [
    ...
    {
        test: /\.scss$/,
        use: ['vue-style-loader', 'css-loader', 'sass-loader']
    },
    {
        test: /\.sass$/,
        use: ['vue-style-loader', 'css-loader', 'sass-loader?indentedSyntax']
    },
    {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
            loaders: {
                'scss': ['vue-style-loader', 'css-loader', 'sass-loader'],
                'sass': ['vue-style-loader', 'css-loader', 'sass-loader?indentedSyntax']
            }
        }
    }
]
```

然后就可以开心快乐的使用sass啦，注意在vue文件里面需要为style指定lang="scss"

[完整代码](https://github.com/sansui-orz/blog/tree/master/demos/demo1)
