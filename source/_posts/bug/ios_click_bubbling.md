title: ios上safari非默认可点击的元素的点击事件无法冒泡问题的解决
permalink: ios-safari-bubbling/
date: 2016-07-29 09:32:53
tags: [bug, ios]
---

在ios的safari上的事件系统有个很古老的bug，在较老的zepto和react上都全线中枪，直至今日也是这样。。

比如以下这两种写法很有可能在ios上无效：

```js
$(document).on('click', 'div.target', function () {
  console.log('clicked!');
})

```

```js
var App = React.createClass({
    render: function(){
      return (
        <div onClick={this.onClick}>点我</div>
      )
    },
    onClick: function () {
      console.log('jsx clicked!');
    }
  })

```

解决办法，用css在需要冒泡点击的元素上设置

```css
.clickable {
  cursor: pointer;
}

```
