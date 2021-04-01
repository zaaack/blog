title: redux学习笔记(1)
permalink: redux-note-1/
date: "2016-07-24 09:32:53"
tags: [redux, js]
---

>redux+react+es6已经成为现在最流行的前端开发方式之一了，然而自己却还未好好学过，真是惭愧，惭愧。。不过，千里之行始于足下，现在先从看官方教程开始，慢慢来吧。。


## Redux简介


官方介绍上说，Redux是一种用于JavaScript应用的可预测的状态管理器，它可以用于多种环境，如client，server和native，并且提供了一个可以实时的带时光机功能的调试器。

看到官网介绍，其实redux是不仅仅局限于react的，甚至不仅仅局限于客户端，服务端也是可以用的。其核心api非常简洁，看看官方README的例子基本就懂了，这意味着项目中往往还得搭配大量其他的包，然后分别去看各个包的文档。。好吧。。慢慢来吧。。

```js
import { createStore } from 'redux'//从redux中引入createStore方法

//reducer，redux的核心概念之一，只是一个简单的参数为(state, action)的函数，用于修改state
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// 创造一个store对象来管理你的项目中的state.
// API: { subscribe, dispatch, getState }.
let store = createStore(counter)

//订阅UI更新事件，可以用getState()方法获得当前状态
store.subscribe(() =>
  console.log(store.getState())
)

//分发action，唯一修改状态state的方法
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1

```

## 查看全部文章

[点击查看全部文章](/tags/redux)

## 参考文档

* redux中文文档：http://cn.redux.js.org/docs/
* redux-totorial, 一个很简短的教程，可以让你领略 Flux 和 Redux 思想的精髓：https://github.com/react-guide/redux-tutorial-cn
* http://div.io/topic/1309
