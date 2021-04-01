title: tmux使用介绍
permalink: tmux-intro/
tags: [linux, tmux]
date: 2016-06-05 09:32:53
---

tmux是一个linux分屏软件，可以将ssh登陆的命令行窗口进行自由分屏，避免了开多个连接增加服务器负担。
安装命令：
```sh
apt-get install tmux
```

基本使用指令

## 1. 基本使用

命令            |功能
----------------|----------------------------------------
tmux            |进入到tmux分屏模式
 tmux kill-server   |关闭服务器
 tmux list-session  |列出所有会话
 tmux kill-session [-t <session id>]  |关闭(某个)会话
man tmux        |进入到tmux帮助文档
ctrl+b          |进入到tmux命令模式(类似vi的命令模式)

## 2. 面板管理(Panel)

>tmux最常用的功能就是把命令行窗口分屏，下面让我们看看面板的常用操作

命令            |功能
----------------|----------------------------------------
%               |将当前面板纵向分成两个
"               |将当前面板横向分成两个
o(other panel)  |切换面板
x(close panel)  |关闭当前面板
PageUp          |1.向上翻页 2.进入复制模式(Esc退出)
PageDown        |向下翻页
Ctrl+方向键	    |以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键	      |以5个单元格为单位移动边缘以调整当前面板大小

## 3. 窗口管理(Window)

>tmux还提供了模拟命令行窗口的功能，这意味着你可以新建一个命令行窗口隐藏在后台运行一些东西。。

命令                |功能
--------------------|----------------------------------------
c(create window)    |新建窗口
w(windows)          |显示所有窗口的列表
n(next window)      |下一个窗口
p(previous window)  |上一个窗口
,                   |重命名窗口
.	                  |修改当前窗口编号；相当于窗口重新排序
&                   |删除当前窗口
!                   |将当前面板跳出来作为单独一个窗口运行
Ctrl+方向键	        |以1个单元格为单位移动边缘以调整当前面板大小
Alt+方向键	          |以5个单元格为单位移动边缘以调整当前面板大小
:pane-resize -L/R/T/B <number>  |左/右/上/下移动窗口边缘

## 4. 会话管理(Session)

命令                |功能
--------------------|----------------------------------------
tmux detach(C+b,d)  |分离当前会话，可以后台运行tmux管理的命令
tmux attach         |附加会话



## 5. 复制模式

* 前缀 [ 进入复制模式
* 按 space 开始复制，移动光标选择复制区域
* 按 Enter 复制并退出copy-mode。
* 将光标移动到指定位置，按 PREIFX ] 粘贴


## 6. 启动鼠标滚轮

有时想通过鼠标滚轮来滚动 tmux 界面，只需要 在 `~/.tmux.conf` 中加入以下几行就可以了。
```sh
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= '#{pane_in_mode}' 'send-keys -M' 'select-pane -t=; copy-mode -e; send-keys -M'"
bind -n WheelDownPane select-pane -t= \; send-keys -M
setw -g mouse on
```
如果报错的话可能和 tmux 版本有关，我的版本号是 2.1
```sh
$ tmux -V
tmux 2.1
```
