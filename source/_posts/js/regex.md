title: javascript的正则踩坑笔记
permalink: js-regex-note/
date: 2016-07-28 09:32:53
tags: [js,regex]
---

## replace传递函数

JavaScript的replace的第二个参数除了大家熟知的字符串外，还可以是一个函数，于是可以轻松实现比占位符($1,$2,$3...)更强大的功能

但是，他的参数是这样的

match, group1, group2, group3, ..., offset, input

不定长的group居然在中间。。好吧，这么反人类的设计我忍了。。传个数组也行啊。。

## 零宽断言x(?=y),x(?!y)不能在开头。。

看MDN，这两种语法只能用于某个字符串的后面，表示后面跟着y或不跟着y的x，但是我想表示x前面不跟着啥咋办？如果是一个字符的话可以用[^y]，多个字符我就不知道了。。
