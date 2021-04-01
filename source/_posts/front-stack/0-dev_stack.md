title: 从0搭建一个完整的前端开发技术栈(0)
permalink: setup-your-front-end-stack-from-zero-0/
tags: [fe-stack]
date: 2016-07-31 11:56:36
---


## 说明

看着github上的前端项目往往会包含很多牛逼的配置文件，一直想整合一个属于自己的前端开发技术栈，将webpack, lint, flow, editconfig, test, travis等技术搭配在一起，以免自己以后的项目显得太过业余。。。

这是个很大的坑，看看[react-boilerplate](https:github.com/mxstbr/react-boilerplate)有多复杂就知道了，然而大部分时候我并不需要这么复杂，我希望能构建一个比较通用的开发模型，然后再根据项目需要适当加减，比如我想用sass，然而react-boilerplate并没有，而且使用起来也略过复杂，光npm scripts就有20多条命令，对于大部分项目来说还是略显笨重，短平快才是我的目标，当然里面很多已有的实践对于初学者还是有很大的参考价值的。

本系列会逐步记录我开发一个代替Chrome已经移除的应用启动器的Chrome应用的过程，重点是开发环境的搭建，计划采用如下技术栈：

* 编译框架：babel+flow+webpack编译js代码， scss+webpack编译css代码
* 代码风格：editconfig配置通用的编辑器配置，airbnb的eslint和stylelint
* MVVM框架：redux+react
* 测试：ava
* 持续集成：travis

后续会逐渐记录每个框架的学习过程。。

## 需求

![](http://i0.sinaimg.cn/IT/cr/2013/0221/1719482159.png)

上面是搜索框，下面是4*4的应用面板，可以随鼠标滚轮左右滚动，每页数量超出自动排到右边新页，数量不足不会自动挤满。右上角菜单点击弹出二级菜单，暂时有显示/隐藏disable的应用，显示扩展，设置（跳到设置页面），关于（调chrome api弹window窗显示）。

应用右键弹出启用/禁用/卸载/添加快捷方式到桌面等菜单

应用面版数量可以设置

数据获取：由于只有chrome扩展可以得到全部应用数据，因此采用在线（github.io）网页+扩展注入js相互通信的模式，而在线网页可以包装成chrome app，然后就可以固定到docker上了（也许这里会有权限问题，不过应该可以通过app弹出新窗口绕过）


## 查看全部文章

[点击查看全部文章](/tags/fe-stack)
