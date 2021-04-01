title: 解决linux配置openvpn客户端不能push dns的问题
permalink: linux-openvpn/
tags: [linux]
date: 2016-07-08 09:32:53
---
在openvpn的配置文件**.conf中添加下面两行
```sh
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```
然后需要手动安装包:
`sudo apt-get install openresolv`
or
`sudo apt-get install resolvconf`

由于在linux下很多软件都可以修改dns配置文件（`/etc/resolv.conf`)，为了避免冲突一般使用`openresolv`和`resolvconf`来管理dns配置。但是若遇到两者都不能自动修改`/etc/resolv.conf`文件的问题，可以手动编辑`/etc/openvpn/update-resolv-conf`文件来实现修改dns：

 https://gist.github.com/zaaack/7ab3d00c688115c7ad80d44d2ffbd121
