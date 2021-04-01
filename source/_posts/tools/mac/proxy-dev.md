title: mac 下代理开发环境总结
permalink: mac-proxy-dev/
tags: [mac, proxy]
date: 2017-04-06 21:00:00
---

在工作中开发，代理技术对于开发环境是不可避免的，小到入职时配置 VPN，再到开发环境中通过代理使用线上 API，通过代理拦截 App 请求，合理的代理应用对于开发来说经常会有事半功倍的作用。这里记录一下我在工作中代理的运用。

## 1. 代理服务器

我这里用的使用的是由阿里巴巴开源的基于 nodejs 的代理工具--- [anyproxy](https://github.com/alibaba/anyproxy)，优点是支持 https，使用 nodejs 作为配置文件，学习成本低，~~缺点是配置 API 看起来太老，连 promise 都不支持，大公司的开源项目不知道什么时候会挂~~刚上了官网，4.0 beta 已经开始支持 promise 了。

## 2. 手机代理软件

Android 5.0 以下可以使用 DroidProxy 直接修改 wifi 代理配置，但是 5.0 以上更新了安全策略，只能用开 VPN 的了。我这里用的是虽然界面很丑，但是好得能用的 Drony. （以上 APK 可以去 <https://apkpure.com>搜索下载）。虽然占用了 VPN，不过一般电脑上各种代理和 VPN 都是开启状态，因此手机上还是能走电脑上的代理和 VPN 的。

Drony 和 connectbot 的 ssh 端口转发冲突，导致无法代理 dns 访问电脑 vpn 的内网，localproxy chains 相关配置未测试..
