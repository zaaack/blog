title: ios上safari div滚动时默认未开启平滑滚动
permalink: ios-safari-scroll/
date: 2016-08-16 09:32:53
tags: [bug, ios]
---

在ios上将一个div设置成可以滚动，它的效果和安卓以及电脑上都是不一样的，感觉没有那种弹性，原因是ios safari在非window的滚动都默认没有开启平滑滚动，需要给滚动的div设置一个属性就可以了：

```css
.scroll-div{
  -webkit-overflow-scrolling: touch;
}
```
