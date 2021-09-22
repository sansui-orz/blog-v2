# React同构探索

[tag]:react|ssr|node
[create]:2020-07-17

[代码](../demos/react-ssr)

## 技术栈

koa + react + typescript (未集成全局状态管理（redux或mobx), 未处理css（sass,less))

## 遇到的难点

1. 服务端代码与浏览器端代码运行环境不一致，需要区分编译
2. 模块热更替（同样服务器端更新以及前端代码更新不一致）

## 第一点，服务端代码与浏览器端代码编译环境不一致问题

一般来说，浏览器端代码需要编译成es5，而服务器端代码则不一定（可能用ts编译成es5/es6/es7）怎么选还是看node版本支不支持。

而且在用上ts之后，这块配置会更加的玄幻一些（因为有些问题完全让人摸不着头发）

这里记录一个问题，在用了ts之后，两个node端文件同时声明同一个变量会报错。

```javascript
// app/controller/a.ts
const Koa = require('koa');
// ...
```

```javascript
// app/controller/b.ts
const Koa = require('koa');
// ...
```

如上代码，会报Koa声明了两次的错误。百度google无果后，改成`import * as Koa from 'koa'`后解决。

目测估计是`tsconfig.json`错误。

解决该问题之后发现react文件与node端文件公用一份`tsconfig.json`不兼容。

在借鉴已有的项目（老项目）之后决定效仿前辈，前端代码单独拎一个`tsconfig.json`。

这样就解决了两端代码配置问题。

## 第二点，模块热更替

这块的坑更多一点。

### 心路历程1 -- 使用webpack-dev-server

一开始使用打算使用`webpack-dev-server`做前端代码的模块热更替（事实上百度很多这个解决方案），但是这个方案只适合于客户端渲染（csr)，因为它会开启一个端口服务（也就是项目预览的那个端口)。

但是同构不一样，同构使用的是node服务开启的端口，这样就无法做到热更新了。所以就放弃了这个方案。

### 心路历程2 -- 使用webpack的watch配置

既然放弃了`webpack-dev-server`，就考虑是否有可以监听文件变更，然后自动打包的库呢。

然后就找到两种方案，一种是使用gulp去构建脚本，第二种则是使用webpack自带的watch配置。既然已经用webpack构建了，自然还是使用webpack的watch配置了。

[配置](https://www.webpackjs.com/configuration/watch/)watch为true, 相应的配置watchOptions之后，可以轻松实现监听文件变动并打包的功能。

但是这并不完美，因为它和我们平时csr的项目不一样，无法做到热更新，在构建完之后还需要我们手动刷新页面。所以就抛弃了这个watch的方案。

### 心路历程3 -- webpack-dev-middleware && webpack-hot-middleware

然后百度一下知道可以使用`webpack-dev-middleware`和`webpack-hot-middleware`这两个库（这两个库的出镜率很高），但是折腾了挺久发现它跟koa不兼容，还好去搜了一下npm，找到两个兼容库`koa-webpack-dev-middleware`与`koa-webpack-hot-middleware`。

看这名字就知道是为koa而生的。尽管这样想，但是用起来还是要了一条老命。

折腾了很久还是没有成功开启热更新，找不到打包之后的文件在哪里，也没有文档，readme上解释的也不清楚，甚至去看了一下源码，于是无补，正在焦头烂额之际，找到一个开箱即用的库`koa-webpack`。

### 心路历程4 -- hoa-webpack

找到了`koa-webpack`这个库，这个库的底层封装了`webpack-dev-middleware`与`webpack-hot-client`。但是实际上配置是差不多的，只不过是将两个调用集成到了一起，然后配置也合并到了一起。

所以没有搞定`webpack-dev-middleware`就相当于也没有搞定`koa-webpack`。同样遇到了打包之后文件不知道在哪里的问题，但是好在它的打包后将文件写入磁盘的配置是有效的（我单独使用`webpack-dev-middleware`时是无效的，并没有在磁盘看到文件），所以我设置了`devMiddleware.writeToDisk`为true。

天可怜见，此时终于有热更新了，但是这里的`koa-webpack`不支持接收非对象的webpack对象，对此只能使用两个`koa-webpack`实例去构建脚本了（分别是打包出来的浏览器端跑的代码与服务器端跑的代码）

### 心路历程5 -- nodemon

搞好了前端代码的热更替，就要考虑服务端的热更新应该怎么做，于是又是一顿百度。发现`nodemon`这个库能够大概的实现我们需要的能力。

它可以监听文件变动从而重启node服务。并且操作简单，并不需要配置，仅需要在运行服务端代码时将`ts-node index.ts`换成`nodemon index.ts`, 它就可以自动监听文件，并重启服务。

### 心路历程6 -- 探索更完美的解决方案

致此热更新能力大概完成，但是始终不完美，比如后端热更新仍然需要刷新页面，又比如仅改变node端代码触发了后端重启，此时会重新跑一遍webpack构建，这样费时且低效。

原本计划使用pm2或许可以做到开启两个进程，一个进程跑服务端代码，一个进程跑`koa-webpack`，但是这里又涉及到两个进程间如何使用同一个app实例的问题，且重启实例之后，前端热更新模块如何处理的问题。

所以最后还是觉得这个方案并不靠谱。

或许可以使用代理的方式，在测试环境时开启两个koa实例，一个跑后端逻辑一个跑前端代码热更新中间件，然后再将打包后的静态资源代理到后端实例端口。
