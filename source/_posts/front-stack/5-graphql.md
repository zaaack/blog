title: graphql学习笔记-从0搭建一个完整的前端开发技术栈(5)
permalink: setup-your-front-end-develop-stack-from-zero-5-graphql/
tags: [fe-stack]
date: 2016-12-06 17:11:57
---

本来打算放弃这个系列了，但是前端发展太快，这个系列估计可以“永无止境”了。。

最近看了下 graphql, 了解了下当前的相关技术栈的整合情况，发现 relay 已经超出了我对查询一个 api 的耐心，而开源组织的 [apollo client](http://dev.apollodata.com/) 以及一些简化的第三方 graphql 客户端都非常好用，大部分应用其实没有必要那么动辄 70多kb 的高性能客户端数据框架，平时 restful 的 api 没有字段按需返回, 缓存只用 http 协议自带的，性能也是完全满足需求的，而 relay 个人感觉为了优化而牺牲开发体验是得不偿失的，一个查询就需要新建一个文件，就算是服务端 mvc 一个文件也能写好几个查询啊，过去一行代码搞定的事情现在搞成一个文件，大大增加了前端项目的复杂度，简直不能忍。

还好 graphql 是一种协议，这意味着只要有足够多的人关注就会有层出不穷的社区项目，比如 apollo 就是其中优秀的一种，支持 redux, ssr(server-side-rendering), 可以搭配 react/angular/vanilla 使用。

不过既然是写博客记录，还是从头开始写起吧，先来看看 GraphQL。

# GraphQL 简介

简单的说，GraphQL 是一种传输协议，最大的价值在于可以通过查询语言按需返回数据，避免了 restful 传输过多不必要的字段的问题，但是由于每次请求返回的数据不一样，因此复用接口时无法使用 http 自带的缓存协议，如etag，因此需要客户端框架来处理缓存，如 relay, apollo。 这篇文章主要内容其实就是官网的 learn 部分中第一篇的中文化，毕竟官网真心写的太好了。。主要的目的是为了后面服务端和客户端配置 GraphQL 提供一些基础。

使用 GraphQL 时，大致流程是先需要服务端定义好各种类型，然后通过 "root" 对象类型来进行查询，官网上称作`entry point`(入口点)，然后客户端通过发送查询语句来得到相应的数据。这里先对查询语句有大致的了解，后续可以在实践中逐步学习服务端类型的定义。

## 查询和修改

直接看 Graphql 可能会感到难以接受，因为不知道背后的实现原理，对实际使用时并没有明确的概念。实际上GraphQL 只是一种协议而已，等到实现的时候其实并没有想像的那么神奇，服务度的每一个可以查询的接口都需要通过 schema 去定义出来。。

### 查询字段(Fields)

查询是和 json 非常类似的，只不过“只有key”部分而已，因为 value 部分等着由服务端提供啊。。
比如:

```js
var query = `{
  hero {
    name
  }
}`

// ==>
var response = {
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}

```
可以看到服务端可以返回的数据结构和查询语句非常类似。

### 参数(Arguments)

查询语句中可以传递参数，比如:

```js
var query = `{
  human(id: "1000") {
    name
    height
    unitHeight: height(unit: FOOT)
  }
}`

// =>
var response = {
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72,
      "unitHeight": 5.6430448
    }
  }
}

```
在查询语句里参数可以通过`argName: value` or `argName: $variableName`的方式传递值或者变量；
在服务端的 schema 中略有差异，通过`argName: typeName = defaultValue`的方式设置参数类型和默认值。

在这个例子中还使用了下面介绍的别名来得到同一字段的不同形式的值。

通过参数可以很容易的实现分页，客户端框架一般会提供包装的分页方法来进行优化。

### 别名(Alias)

别名用于在同一层级获取有着相同结构的不同数据，比如

```js
var query = `{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}`
// =>
var response = {
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}

```

### 片段(Fragments)
片段就是一个类型的一部分字段的集合，可以通过es6展开运算符来展开所有字段到当前的查询结构中，极大的方便了复用（感觉和类型放在一起更好理解）


```js
var query = `{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}`
// =>
var response = {
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}

```

### 变量(Variables)
虽然有了参数，但是查询语句只是一个字符串而已，还是需要有能传入动态参数的地方（虽然es6模版字符串也能做到），因此引入了变量的概念。
引入变量需要三个步骤
1. 替换查询语句中的静态值为`$variableName`
2. 声明`$variableName`为查询语句中可接受的变量
3. 将 `variableName: value` 通过另一个分开的变量键值传递给服务端，通常是 json 形式的 get 参数


```js
var query = `query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}`
var variables = {
  "episode": "JEDI"
}
// =>
var response = {
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}

```

### 操作名(Operation name)

上面例子中的查询语句很多是一种省略形式，我们来看一个完整的：

```js
var query = `query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}`

```

其中`query`是操作类型，是可以自定义的，比如定义成`mutation`, `subscription`,这里只是约定俗成的名字. `HeroNameAndFriends`是查询名字,服务端需要有对应的配置才能查询.

在 graphql 中，服务端的定义大致如下:
```js
var schema = `
type Query {
  hero(episode: $episode) {
    name
    friends {
      name
    }
    # ...
  }
}

type Mutation {
  addHero(hero: $hero)
}

schema {
  query: Query
  mutation: Mutation
}
`

```
通过 schema 定义的字段在 GraphQL 中叫做 `entry point` (入口点), 其实和其他的对象类型差不多，唯一的区别在于可以通过`query xxx { ... }`、`mutation xxx { ... }`的方式用来得到内部的查询定义。


### 指令(Directives)

通过指令可以通过变量修改查询的结构

```js

var query = `
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
`
var variables = {
  "episode": "JEDI",
  "withFriends": false
}

// =>
var response = {
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

自带有两种指令，服务端也可以自定义更多：

`@include(if: Boolean)` 参数为true则包含该字段
`@skip(if: Boolean)` 参数为true则忽略该字段

### 行内片段
片段的简写形式
```js

var query = `
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
`
```

## schema和type 的定义方式
参考：<http://graphql.org/learn/schema>



## 更多参考

<http://graphql.org/learn>
