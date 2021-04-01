title: deepin技巧汇总
permalink: deepin_skills/
tags: [linux]
date: 2016-05-26 09:29:42
---

>这里主要是记录切换开发环境到deepinlinux下后遇到的一些常见问题和解决方案。deepinlinux是深度开发的一款国产Linux，曾经基于ubuntu，不过最新版的2015.2已经基于debian了，因此许多教程不再适用。其特色是包含已经购买的了的crossover，能正常运行QQ等软件，还和网易云音乐合作开发了linux版，开箱即用，非常方便。


### deepin ctrl+shift+F4切换到命令行后如何启动gui

`sudo service lightdm restart`

### deepin 启动器图标所在文件夹

` ~/.local/share/applications/`
or
`/usr/share/applications`

### 添加windows字体文件

```sh
sudo ln -s /home/z/deepin_tricks/fonts /usr/share/fonts/winfonts
cd /usr/share/fonts/winfonts
mkfontscale && mkfontdir
```

### 解决atom beautify 插件ctrl-alt-b快捷键冲突

右键搜狗输入法->设置->高级->打开fcitx设置->全局配置->高级->找到ctrl-alt-b修改未ctrl-alt-v
