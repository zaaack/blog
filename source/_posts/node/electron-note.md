title: "electron笔记"
slug: "electron-note"
date: "2016-08-20 08:26"
---
## electron 使用笔记总结

### 渲染进程和后台进程

渲染进程就是浏览器进程，后台进程就是命令行调用时的没有界面的进程。

后台进程通过 `new BrowserWindow` 可以创建一个渲染进程，由于两者是不同进程，因此不能共享内存，只能通过ipc传输序列化数据。


### 踩坑记录

前段时间在做 [ELaunch](https://github.com/zaaack/ELaunch), 踩了不少坑，记录一下。

#### 编译native 包
npm install 时会对包含native代码的包进行编译，但是是根据本地安装的node版本编译的，而不是electron中包含的node， 因此electron需要重新编译一次：

官方文档： http://electron.atom.io/docs/tutorial/using-native-node-modules/

```sh
npm install --save-dev electron-rebuild

# Every time you run "npm install", run this:
./node_modules/.bin/electron-rebuild

# On Windows if you have trouble, try:
.\node_modules\.bin\electron-rebuild.cmd
```

#### mac 下隐藏窗口不会自动将焦点交还给原先窗口

类似iterm2/alfred快捷键呼出的效果 这是mac下独有的bug，windows和linux下默认就是需要的。

在mac下其实已经有解决办法了， 就是

```sh

## 隐藏
mainWin.hide()
app.hide && app.hide()

## 显示
mainWin.show()
app.show && app.show()
```

issue: https://github.com/electron/electron/issues/6669

#### mac下不能使用复制黏贴快捷键

需要手动创建菜单，添加快捷键

issue: https://github.com/moose-team/friends/issues/123#issuecomment-106843964
