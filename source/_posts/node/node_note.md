title: node/npm笔记
permalink: node-or-npm-note/
date: 2016-06-05 09:32:53
tags: [node]
---

## 安装包
```sh
# 不安装devDependencies
npm i --production
# 如果可以的就使用本地全局的依赖link到当前模块目录下
npm i --link

# 安装时更新package.json
# 保存到dependencies
npm i --save,-S <pacakge>
# 保存到devDependencies
npm i --save-dev,-D <pacakge>
```

## 发布应用

需要注意的是发布应用必须使用官方源，不能使用淘宝源
```sh
# 链接到本地环境变量目录下
npm link

# 注册
npm adduser

# 登陆
npm login

## 发布, 目录下
npm publish

## 不发布
npm unpublish <package>@<version>

```

忽略文件，`.npmignore`, 同`.gitignore`
