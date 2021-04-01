title: m 站访问速度优化和 service worker 可行性的调研
permalink: m-site-performance
date: 2016-06-05 09:32:53
tags: [mobile-web, performance]
---

移动网页的性能优化其实是个很大的话题，从网络请求到页面渲染，再到 JS 性能，对应框架的优化，
几乎每一个步骤都能扩展出一个话题出来。。不过如果限定场景为 m 站的话，这个问题就相对简单许多了。
m 站有以下特点:
1. 已展示页面为主，交互较少，性能瓶颈主要集中于网络请求方面
2. 前端框架较为统一 (Zepto, React)
3. staticng 提供了完整和强大的 http 缓存支持和静态文件处理 (compress, concat...)

## 使用现有技术进行优化

### Http 缓存与 staticng

```sh
Pragma: no-cache # Http 1.0

Expires # http 1.1
Cache-Control
Last-Modified/If-Modified-Since
Etags/If-None-Match
Vary
```

其实 http 缓存 staticng 已经帮我们处理好了，我们需要做的主要是充分利用 staticng 提供的工具来发挥缓存的作用，以及减少 请求数量。
stating 主要提供了这些功能:
* `static/istatic` 接口获取添加了 hash 的 cdn 链接,
* `@import`语法合并静态文件
* `collect_css/collect_js` 收集合并页面上的内联 `js/css`

#### 减少 css/js 的数量
那么怎样引入静态文件才能最大化的减少请求数量呢?

我发现 m 站中主要有这么几种写法：

* No. 1
```html
<%dev name="more_header()">
 ## ...
 <link rel="stylesheet" href="${ static('/path/to/some.css') }">
</%dev>

<%dev name="more_footer()">
  <script src="${ static('/path/to/some.js') }" charset="utf-8"></script>
</%def>
```
这种写法就是普通的引入外联 css/js 的方式，能够利用 staticng 提供的各种 http 缓存功能，但是却多了一次请求，在移动端还有优化的空间。

* No. 2

```html
<%dev name="more_header()">
 ## ...
 <style media="screen">
   ${ istatic('/path/to/some.css') | n }
 </style>
</%dev>

<%dev name="more_footer()">
  ## ...

  <script type="text/javascript">
    ${ istatic('/path/to/some.js') | n }
  </script>
</%def>
```

这种写法比起第一种写法 虽然介绍了请求数量，但是把 css/js 都输出到 html 中，而 html 是 **不会** 缓存的，
如果 css/js 比较多的话，那么加载时对用户体验的影响还是比较明显的。

* No. 3

```html
<%dev name="more_header()">
 ## ...

 <%block filter="collect_css">
   ${ istatic('/path/to/some.css') | n }
 </%block>
</%dev>

<%dev name="more_footer()">
  ## ...

  <%block filter="collect_js">
    ${ istatic('/path/to/some.js') | n }
  </%block>
</%def>
```

这种写法使用了 staticng mixed_static 的特性，可以把页面上输出的 css/js 收集到一起，然后在页面上的占位符
(`<!-- COLLECTED CSS -->`, `<!-- COLLECTED JS -->`) 上输出，在 pre 上看起来和第二种差不多，但是实际上在线上环境，
就会自动变成 **外联** 形式的，能够充分利用 http 缓存。

![](images/2017/02/Jietu20170223-093305.jpg)

而且还有一个很实用的功能，就是 `collect_css/collect_js` 中的代码是做了去重处理的，同一个 block 在同一个页面中多次引用
只会输出一次，这在 widgets 中非常有用，能够避免重复引入代码。

[staticng/../mixed_static.py#L45](https://github.intra.douban.com/dae/staticng/blob/7078d5c664f7a8cfafdd2c01d9089ad54bd0a785/staticng_client/middlewares/mixed_static.py#L45)

看起来第三种写法应该是引入业务代码的最佳的写法，于是我先对 talion 进行了如下改造:

1. 减少公用库的请求 (url.min.js in main.js => weixin.js, react.min.js + react-dom.min.js => react-all.min.js)
2. 将 首页/广播流/电影条目页 原先 从页面输出(or 外联)的 业务逻辑相关 js/css 放到 mixed_static 中, 减少请求数量

然后再看了一下广播页的 Chrome 控制台，发现请求数量确实减少了...几个，但是还是有 60-70 个请求, 数量级并没有明显的变化。。观察请求的数量，其实大头还是在图片。

#### 优化图片请求

在前端优化图片加载的方式主要就是延迟加载了，只有在图片出现到用户视口内才加载。
其实在 m 站中是存在图片延迟加载库的，只是只有首页 feed 流和 文章详情页(日记/长评/帖子) 中用到了，可能因为这些地方的图片比较多吧。但是现在随着 m 站的不断开发和完善，很多以前相对比较轻量的页面，比如条目页等，也多了很多图片，比如这是电影《海洋奇缘》的条目页，竟然达到了 80 个请求。这些请求大部分都是图片，比如电影的截图，还有推荐位的 cover。像这些地方就比较适合在前端实现延迟加载。





#### 优化
1. 减少公用库的请求 (url.min.js in main.js => weixin.js, react.min.js + react-dom.min.js => react-all.min.js)
2. 将原先从页面输出的/外联的 业务逻辑相关 js/css 放到 mixed_static 中, 减少请求数量
3. 图片 lazy load (`/js/lib/lazify.js`)


## http 缓存优化:

其实 http 缓存 staticng 已经帮我们处理好了，大部分的时候并不需要特别

```sh
Pragma: no-cache # Http 1.0

Expires # http 1.1
Cache-Control
Last-Modified/If-Modified-Since
Etags/If-None-Match
Vary
```

合并文件，减少 http 请求次数
1. 减少公用库的请求 (url.min.js in main.js => weixin.js, react.min.js + react-dom.min.js => react-all.min.js)
2. 将原先从页面输出的/外联的 业务逻辑相关 js/css 放到 mixed_static 中, 减少请求数量
3. 图片 lazy load
https://github.intra.douban.com/dae/staticng/blob/7078d5c664f7a8cfafdd2c01d9089ad54bd0a785/staticng_client/middlewares/mixed_static.py#L45

http://www.alloyteam.com/2012/03/web-cache-2-browser-cache/
https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching

## Service Worker & PWA!

## 背景

## 缺少低级的缓存 API
Http cache
HTML5 AppCache

### 缺少一个可以长时间运行的后台进程
Web Worker
只能被一个页面访问, 和页面的生命周期绑定在一起，页面关闭了 worker 也会结束
Shared Worker
能被多个同源页面访问，但是多个页面都关闭了，worker 也会结束

http://stackoverflow.com/questions/20084348/what-happens-to-a-web-worker-if-i-close-the-page-that-created-this-web-worker

### AppCache 太反人类

* 声明式缓存，不够灵活
* 需要第二次访问才能更新，如果需要立即更新需要根据更新事件提示手动刷新
* 更新很麻烦，坑太多..
https://alistapart.com/article/application-cache-is-a-douchebag
* 已经被废弃

### Service Worker

#### 优点
* 与页面无关，可以在没有页面的情况下运行，直到浏览器在资源不足时回收
* 提供了低级的缓存接口，能拦截所有页面请求并使用 Cache API 手动控制缓存
* 可以实现 Web push

#### 限制
* 由于可以拦截所有请求，容易遭受 M-I-M 攻击, 需要 https 才能使用
* worker 需要和页面同源，但是在 worker 内部可以通过 `importScripts('<url>') ` 同步引入非同源的js
* 兼容性：除了 Chrome 其他的短期内都不会支持

![](/images/2017/02/Jietu20170221-182454.jpg)

### 相关概念

Web Worker 只能和单个页面中通信，页面关闭就结束了
Shared Worker 可以在多个页面中访问,页面关闭还在
Service Worker 可以在多个页面中访问，主要用来拦截网络请求
Cache API

### 生命周期
![](images/2017/02/sw-lifecycle.png)


https://jakearchibald.github.io/isserviceworkerready/
https://html.spec.whatwg.org/multipage/workers.html#workers
https://github.com/w3c/ServiceWorker/blob/master/explainer.md
http://jixianqianduan.com/frontend-javascript/2014/06/05/webworker-serviceworker.html
