title: rem 布局
permalink: rem-layout/
tags: [css3, css]
date: "2017-02-10 9:00"
---

```scss
/**
* /js/lib/flexrem.js
 *
 */

$inital-rem-size: 100px !default;
$design-width: 375px !default;


html {
  font-size: $inital-rem-size / $design-width * 320px;
}

@each $width in (320px, 360px, 400px, 414px, 480px, 500px, 650px) {
  @media only screen and (min-width: #{$width}) {
    html {
      font-size: $inital-rem-size / $design-width * $width;
    }
  }
}


@function px2rem($px) {
  @if unitless($px) { // 不管带不带单位都可以
    $px: $px * 1px;
  }
  @return $px / $inital-rem-size * 1rem;
}

```


```js
/**
 *
 * 根据屏幕宽度自动设置html的fontSize, 使rem的大小自适应
 * usage:
 * dui.flexrem({
 *            //设计稿宽度
 *         		designWidth: <number>, initial: 375
 *         		//rem 初始像素值，initial: 100
 *         		//html的fontSize = (opts.initRem/opts.designWidth*winWidth)+'px'
 *         		initRem: <number>,
 *            //用来计算的最大的 winWidth, initial: 640
 *         		maxWinWidth: <number>
 *            //用来计算的最小的 winWidth, initial: 320
 *            minWinWidth: <number>,
 * })
 *
 * 由于android webview 偶尔会在 document.load 之后识别 viewport 导致 window.innerWidth 发生变化, 这段 js 需要在 window.onload 后运行，因此还需要配合 css 使加载前页面也能正确显示
 *
 *
 * $inital-rem-size: 100px; // 默认值
 * $design-width: 375px;// 默认值
 * @import '/css/card/widgets/_flexrem.scss';
 *
 * width: @px2rem(12)rem; => 12/100 = 0.12rem
 *
 */

!(function(win, doc) {

  var dui = window.dui || (window.dui = {}),
    timer = null,
    firstCalled = true,
    defaults = {
      designWidth: 375,
      initRem: 100,
      maxWinWidth: 640,
      minWinWidth: 320,
    },
    evts = 'onorientationchange' in win ? ['orientationchange', 'resize'] : ['resize'];

  if (dui.flexrem !== undefined) {
    return console.error('flexrem.js shouldn\'t be called twice!!')
  }
    /**
     * auto set html's fontSize to make rem flexable
     * @param  {object} opts
     *         {
     *            //设计稿宽度
     *         		designWidth: <number>, initial: 375
     *         		//rem 初始像素值，initial: 100
     *         		//html的fontSize = (opts.initRem/opts.designWidth*winWidth)+'px'
     *         		initRem: <number>,
     *            //用来计算的最大的 winWidth, initial: 960
     *         		maxWinWidth: <number>
     *         }
     * @return
     */
  dui.flexrem = function(opts) {
    if(!firstCalled) console.error('flexrem() can only be called once!');
    firstCalled =false
    var options = Object.assign({}, defaults, opts)
    for (var i = 0; i < evts.length; i++) {
      win.addEventListener(evts[i], function() {
        clearTimeout(timer);
        timer = setTimeout(setFontSize, 300);
      }, false);
    }

    win.addEventListener("pageshow", function(e) {
      if (e.persisted) {
        clearTimeout(timer);
        timer = setTimeout(setFontSize, 300);
      }
    }, false);
    // 初始化
    setFontSize();

    function setFontSize() {
      var winWidth = window.innerWidth || window.clientWidth;
      winWidth = Math.min(winWidth, options.maxWinWidth)
      winWidth = Math.max(winWidth, options.minWinWidth)
      var size = (winWidth / options.designWidth) * options.initRem;
      doc.documentElement.style.fontSize = size + 'px';
    }
  }


  function toType(obj) {
    return ({}).toString.call(obj).match(/\s([a-zA-Z]+)/)[1].toLowerCase();
  }

  if (typeof Object.assign != 'function') {
  Object.assign = function (target, varArgs) { // .length of function is 2
    'use strict';
    if (target == null) { // TypeError if undefined or null
      throw new TypeError('Cannot convert undefined or null to object');
    }

    var to = Object(target);

    for (var index = 1; index < arguments.length; index++) {
      var nextSource = arguments[index];

      if (nextSource != null) { // Skip over if undefined or null
        for (var nextKey in nextSource) {
          // Avoid bugs when hasOwnProperty is shadowed
          if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
            to[nextKey] = nextSource[nextKey];
          }
        }
      }
    }
    return to;
  };
}



}(window, document));

```
