title: webpack2 -从0搭建一个完整的前端开发技术栈(6)
permalink: setup-your-front-end-stack-from-zero-6-webpack2/
tags: [fe-stack, webpack]
date: 2017-04-28 05:15:21
---

之前的框架是 full-stack 的，最近需要帮以前工作室的学妹弄一个简单的 h5 页面时才发现，自己居然手头没有一套靠谱的纯粹配置，如果从其他项目中剥离出来又实在提不起兴趣，于是下了官方的 create-react-app, 发现各种问题，比如

1. 不支持 react 热更新，只支持热刷新
2. 与 sass 整合很不方便，sass 直接编译到源码目录，干扰视线。。
3. babel，webpack 也不支持扩展；
4. ...

无法适应大型项目，和我理解中的只是用来写 demo, 或者简单页面的脚手架差不多。临时看了一下 styled-components, 发现这货比我想象的要高级，支持嵌套语法，auto-prefixer, 既然是 css-in-js,那么动态复用就完全不用担心了，更让我惊艳的是工具链的支持，在 atom 中，基本上必装的 language-babel 插件已经默认支持了，在 js 中也可以享受到强大的 css 提示。

于是在 create-react-app + styled-components 的组合下写起展示类页面来体验也挺不错的。

不过，没有一套自己常用的纯前端的脚手架总感觉心里不太安稳，而且转眼过去大半年了，webpack2, react-hot-loader v3 已经是正式版了，个人也积累了一些不错的库，于是便打算再折腾一个最新的脚手架。

作为一名合格的「前端配置工程师」，每半年~一年升级一次配置才能紧跟前端的潮流。。

先放代码：<https://github.com/zaaack/react-starter.git>

这里主要记录一下猜到的坑。。

#### react-hot-loader
毫无悬念，虽然之前有过成功配置 react-hot-loader 的经历，而且那时也是 v3，只不是是 v3-beta, 现在是 v3 正式版，但是还是在这里卡住了大部分时间...

其实官方的两个 demo 已经是非常友好的了，

* https://github.com/wkwiatek/react-hot-loader-minimal-boilerplate
* https://github.com/gaearon/react-hot-boilerplate/tree/next

webpack2 官方也有一个教程，不过需要注意的是 babel-preset-es2015 的参数配置有点老了，具体可以参考上面的 [react-hot-loader-minimal-boilerplate](https://github.com/wkwiatek/react-hot-loader-minimal-boilerplate)

* https://webpack.js.org/guides/hmr-react/

不过 react-hot-loader 还是太 hack了点，很多地方都需要加 patch，不仅仅是在 entry 中，babel presets 中也要加。开始时就是忘了加 babel presets 中的 `react-hot-loader/babel`, 导致虽然 log 正常，但是却完全没反应。没有报错提示无意是最难调试的了，只能去翻源码以及在源码中加 log。

发现之后加上去了，但是还是老样子，到底是哪里有问题呢？经过反复耐心测试才发现居然是少了 babel-polyfill, 而且还不能使用 babel-transform-runtime, 虽然之前的配置中 可以通过对 transform-runtime 加 {polyfill:false} 参数使之共存，但是这个 trick 貌似在最新版中不能用了，不过没关系，可以通过 NODE_ENV 区分啊，大不了 开发环境就不加嘛。

可是这样还是不行，不过还好在反复实践中发现
虽然 webpack 中配置了 babel, 但是还是会去读取 .bablerc 文件，在翻 babel-loader 的源码时在文档中成功找到了相关依据，改了一个配置就搞定了。。此时 react-hot-loader 所有的坑都已填满，可以放心食用了。。


#### react 测试
其实之前一直都没写过 react 测试，老实说除了一些基础 node 库之外，我还没在业务代码中写过测试呢，更不用说 UI 测试了。。所以这次就干脆把测试也加上了。React 自带了 `react-dom/test-utils`，不过 API 太过冗长，Facebook 自己都不用， 参考了下阮老师的[这篇](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html), 发现很多 demo 都跑不起来了，倒不仅仅是因为接口的变动，更坑的是 bug...bug.

比如 `renderIntoDocument` 这个方法，最新的 react 已经去掉了返回值，就算看源码自己用 callback 实现也不行，因为我就尝试了，callback 也没有返回值了！连之前 callback 可以用的 `this`也变成 null 了。。所以要么直接用 document，要么用 airbnb 的 enzyme, 考虑到写测试是一件很痛苦的事情, 而且
enzyme 的接口其实和 jquery 很类似，目测挺方便的，本着不给自己添麻烦原则，就只能牺牲一下硬盘空间了。。我的 node_modules 目录已经快撑爆了吧。。


然而，随便给 demo 代码写了点 测试，想点击一个 react-router 的 Link 实现跳转，死活没反应。在 js-dom 的官方 readme 上看见了巨大的 "不支持跳转" 的说明，好吧，这很好解决，加个 `NODE_ENV=test` 换成 `createMemoryHistory` 不就可以了，issue 中也有类似的解决方案。可以还不行！！！一样的没有任何报错，任何反应（可见良好的错误提示是多么重要）。无聊之下又开始翻源码，打 log, 可以确定事件是触发了的，于是又去看 react-router 的 Link 组件，打 log, 终于发现是 因为 Link 中为了确保是鼠标左键点击做了 `event.button === 0` 的判断，而 jsdom 中的 `MouseEvent.button === undefined`, 并且是个老 issue 了，果断再来[一发](https://github.com/tmpvar/jsdom/issues/1831)。

至于 Facebook 官方的测试框架 jest 强大的 snapshot 功能，可喜的发现 AVA 已经 内置了，使用非常方便，就不细说了，直接看代码和官方文档吧。

恩，至此所有坑已填完，可以上传 repo 了。是的，这篇文章其实不是什么分享，只是单纯的吐槽，解决这些坑其实几乎毫无价值，半年之后，可能有些修了，可能又会有新的 bug。。人生苦短，且行且珍惜，所以半年更新一下配置，又可以保持一下自己的折腾能力了。。

最后，再来安利一发自己的 react 脚手架 [react-starter](<https://github.com/zaaack/react-starter.git>)。

以后有时间再整个 koa 后台的 starter，graphql 的，ssr 的。。。
