# 使用typescript写一个npm组件库

[tag]:typescript|npm|lib
[create]:2020-07-29

## 前言

在现阶段的前端日常工作中，npm包以及typescript很多人都不陌生了，但是估计使用ts写过npm包的人还是不是太多的。我这两天刚好在捣鼓这个，遇到一些坑，东拼西凑总算趟过。

由于日常技术栈是react.js，所以写了一个react + ts的组件库。

## 简单介绍一下目录结构

├── examples （示例文件夹，本地开发时webpack的入口也放在这里）
├── lib （输出文件夹，编译后的文件放在这个文件夹）
├── src （组件文件夹，组件源文件放在这个文件夹，examples也是直接引用这个文件夹内的文件）
├── .babelrc （babel配置文件，这里编译使用的是babel）
├── .gitignore
├── .npmignore （npm配置忽略文件，打包上传时需要忽略哪些内容）
├── tsconfig.json
├── webpack.config.json
└── package.json

主要的文件与文件夹作用我都标出来了，其他文件如果你是使用ts+react开发的话你应该也知道了它的用法。

这里我使用了webpack进行做开发环境的编译，而在打包时则是使用的babel进行编译。

### 关键文件配置

关于组件的编写就不赘述了，相信大家都会，关键是简单介绍一下我遇到的问题。

```javascript
// package.json
{
  // ...
  "scripts": {
    "start": "webpack-dev-server --config webpack.config.js",
    "build": "babel src -d lib --extensions '.tsx' --plugins transform-class-properties"
  }
  // ...
}
```

这里开发的时候使用webpack而编译的时候使用babel是因为我使用webpack打包之后，在其他项目引用会抱webpack渲染出来的是字符串的错误，百度没有找到解决方案就使用了 tsc 去编译 tsx的代码，tsc编译出来的代码看上去是没问题的，但是引用时会报没有`_interopRequireDefault`的错误。然后找了一圈也还是没有发现解决方案。最后才采用babel-cli去编译。

`src -d lib` 的意思是编译src下的文件到lib文件夹。

`--extensions '.tsx'`的意思是设置可tsx后缀的文件。

`--plugins transform-class-properties`是使用抓换类到原型的插件。

```javascript
// .babelrc
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-typescript",
    "@babel/preset-react"
  ]
}
```

注意这里需要配置介个babel的presets设置。

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: "./examples/index.tsx",
  resolve: {
    extensions: [ '.tsx', '.ts', '.js', '.jsx' ]
  },
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              fallback: require.resolve('file-loader'),
            }
          }
        ]
      },
      {
        test: /\.tsx?$/,
        exclude: /node_modules/,
        use: 'ts-loader',
      },
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: [ 'style-loader', 'css-loader' ],
      },
    ]
  },
  devServer: {
    hot: true,
    port: 8001,
    open: true,
    host: "0.0.0.0",
  },
  plugins: [new HtmlWebpackPlugin({
    template: './examples/index.html'
  })],
};
```

webpack配置只有在开发环境的时候才会用到，所以只是简单配置了能跑起来就ok。

```javascript
// tsconfig.json
{
  "compilerOptions": {
    "jsx": "react",
    "target": "es5",
    "module": "esnext",
    "sourceMap": true,
    "lib": [
      "es6",
      "dom",
      "es2017",
      "esnext"
    ],
    "allowJs": true,
    "declaration": true,
    "outDir": "./lib",
    "rootDir": "./",
    "removeComments": true,
    "importHelpers": true,
    "isolatedModules": true,
    "strict": true,
    "noImplicitAny": false,
    "strictFunctionTypes": false,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": false,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "baseUrl": "./",
    "types" : ["react", "react-dom", "@types/node", "@types/webpack-env"]
  },
  "include": [
    "./src/**.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

同样ts配置只在开发的时候用到。

这样配置就完成了，开发完直接`npm run build` 然后 `npm publish`发布就行了，至于`npm login`和其他步骤，可以去看其他教程。

也可以直接看我写的npm包[react-infinite-auto-scroll](https://github.com/sansui-orz/react-infinite-auto-scroller)
