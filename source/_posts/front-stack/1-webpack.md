title: webpack简单配置-从0搭建一个完整的前端开发技术栈(1)
permalink: setup-your-front-end-stack-from-zero-1-webpack/
tags: [fe-stack, webpack]
date: 2016-07-31 11:56:36
---

为了能使用最新的[es6语法](http://es6.ruanyifeng.com/)和[sass语法]()，我们需要使用webpack进行编译。

## 安装

```sh
npm i -D webpack node-sass sass-loader css-loader style-loader autoprefixer postcss-loader url-loader file-loader extract-text-webpack-plugin babel-loader babel-core babel-preset-react babel-preset-es2015 babel-runtime babel-plugin-transform-runtime webpack-dev-server cross-env
```

一下子安装这么多包，对于初学者很容易会感觉很难，其实只需要多看看文档，理清架构就能理解各个包是干啥用的了。

首先我们要知道webpack的基本结构，然后再来慢慢讲解这些包的作用。

## 配置webpack

在项目的根目录下新建 webpack.config.js 文件

```js
var path = require('path');
var ExtractTextPlugin = require("extract-text-webpack-plugin");
var webpack = require("webpack");
var autoprefixer = require('autoprefixer');

module.exports = {
  entry: {
    app: './app.js',
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/',
    filename: '[name].js'
  },
  module: {
    loaders: [
      {
        test: /\.(png|jpg|jpeg|gif|ttf|eot|woff|woff2|svg)$/,
        loader: 'url'
      }, {
        test: /\.(css|scss)$/,
        loader: ExtractTextPlugin.extract('style',
                  'css!postcss-loader!sass-loader')
      }, {
        test: /\.js[x]{0,1}$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'react'],
          plugins: ['transform-runtime']
        }
      }
    ]
  },
  postcss:[ autoprefixer({ browsers: ["last 2 version", "Explorer >= 9"] }) ],
  plugins: [
    new ExtractTextPlugin("[name].css"),
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify(process.env.NODE_ENV || 'development')
      }
    })
  ],
  devServer: {
    contentBase: './',
    publicPath: '/'
  }
}
```
然后我们一步步讲解每一个配置的含义：
先将webpack配置文件的大纲抽取出来：

```js

module.exports = {
  entry: { //需要打包的入口文件，这个文件中会通过依赖而关联整个项目

  },
  output: { //打包后输出的文件

  },
  module: {    //模块，每一个文件就是一个模块
    loaders: [ //模块的加载器，通过什么方式加载这些文件

    ]
  },
  plugins: { //webpack插件

  },
  devServer: {//webpack内置了一个开发用的服务器webpack-dev-server的配置，可以实现热更新等功能

  }
}
```

然后一步步看我们到底配置了啥：
```js
var path = require('path');
var ExtractTextPlugin = require("extract-text-webpack-plugin");
var webpack = require("webpack");
var autoprefixer = require('autoprefixer');

module.exports = {
  // 需要打包的文件配置
  entry: {
    app: './app.js', //通过key value的形式配置了需要打包的文件
  },

  //输出文件配置
  output: {
    path: path.resolve(__dirname, 'dist'), //输出的目录，我们是配置为当前目录下的dist目录
    publicPath: '/static/', //发布后的服务器或cdn上的路径, 配置这个后webpack-dev-server会自动将html中引用的部署路径自动路由到本地的开发路径上
    filename: '[name].bundle.js',// 输出的文件名，[name]就是entry的key
  },

  //模块加载器
  module: {
    loaders: [ //加载器数组
      {
        test: /\.(png|jpg|jpeg|gif|ttf|eot|woff|woff2|svg)$/,//用来匹配文件的正则
        loader: 'url?limit=10000' //加载器的名称，此处为url-loader,`?`后面可以添加loader的参数， 具体得参考loader的github主页。
      }, {
        test: /\.(css|scss)$/,
        loader: ExtractTextPlugin.extract('style',
                  'css!postcss-loader!sass-loader') //使用ExtractTextPlugin,将样式抽出到单独的文件中，webpack默认是构建html的style标签; 多个loader可以通过!连接起来，相当于管道一样，最后面的loader先传入文件，然后再传出给前面的loader
      }, {
        test: /\.js[x]{0,1}$/,
        exclude: /(node_modules|bower_components)/,
        // loader也可以使用使用数组进行配置, loaders:['babel','...']
        // 参数可以用querystring: 'babel?presets[]=es2015&presets[]=react'
        // 或query字段： loader: 'babel', query: {presets: ['es2015', 'react']}
        // 或参数传json:
        // 'babel?{presets:["es2015", "react"]}'
        loaders:['babel?{presets: ["es2015", "react"], plugins: ["transform-runtime"]}'] //transform-runtime主要用来避免babel-core/polyfill 污染全局变量，使用babel-runtime, 具体请上github
      }
    ]
  },
  postcss:[ autoprefixer({ browsers: ["last 2 version", "Explorer >= 9"] }) ], //postcss-loader 的配置，这里我们主要是使用autoprefixer

  //webpack 插件配置
  plugins: [
    new ExtractTextPlugin("[name].css"), //抽取样式到单独的 文件中，文件名称则为[name].css
    new webpack.DefinePlugin({ //定义变量,这些变量会在build的时候执行，可以给不同的命令传入不同的env，这样就能实现服务端与本地的配置不同了。
      'process.env': {
        NODE_ENV: JSON.stringify(process.env.NODE_ENV || 'development')
      }
    })
  ],

  //webpack-dev-server配置 http://webpack.github.io/docs/webpack-dev-server.html#api
  devServer: {
    contentBase: ''//index.html的本地路径，默认是当前目录
  }
}
```



## 开发配置

以上只是最基础的打包配置，在开发时我们还希望能监听源码自动实时编译打包，自动更新模块免刷新等，这些功能使用webpack-dev-server 就能轻松实现。但是如果需要使用我们自己的后端环境时，就只能准备两份配置文件，不过由于webpack的配置可以以js的文件方式存在，因此我们只需要assign一下就行了。

```js
var webpackConfig = require('./webpack.config.js')
module.exports = Object.assign({}, webpackConfig, {
  debug: true,
  devtool: 'eval' //开发时加快编译速度
})
```


更多配置细节可以阅读文档： http://webpack.github.io/docs/configuration.html

## npm scripts
然后在npm scripts里面配置开发和构建命令, 其中cross-env可以跨平台的配置NODE_ENV

```json
{
  "scripts": {
    "start": "webpack-dev-server --hot --inline",
    "build": "cross-env NODE_ENV=production webpack -p --progress --profile --colors",
  }
}

```


## 用到的包简介：

### 独立包
* webpack: webpack
* webpack-dev-server: webpack开发服务
* cross-env: 跨平台环境变量

### webpack loaders
* node-sass sass-loader: 编译sass，sass-loader依赖node-sass
* css-loader: 加载css，可以使用module参数实现css模块化
* style-loader: 样式加载器，加载css到style标签
* autoprefixer postcss-loader: 自动添加浏览器前缀
* url-loader file-loader 以url的形式加载图片等文件，url-loader依赖file-loader, 不过比file-loader强的是可以使用limit参数将一定大小内的文件转成base64
* babel-loader babel-core babel-preset-react babel-preset-es2015 babel-runtime babel-plugin-transform-runtime: babel 相关

### webpack plugins
* extract-text-webpack-plugin 抽取文本为单独文件的插件
