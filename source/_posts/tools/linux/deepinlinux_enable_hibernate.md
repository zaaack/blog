title: 怎样开启deepinlinux/debian的休眠模式
permalink: how-to-enable-deepinlinux/debian-hiberate
tags: [linux]
date: 2016-05-26 09:22:40
---

>参考教程:<https://wiki.debian.org/Hibernation/Hibernate_Without_Swap_Partition>

# 1.开启虚拟内存
一般需要大于电脑实际使用内存，Uswsusp支持压缩，所以我电脑8g，但是还是分配4G内存
```sh
sudo dd if=/dev/zero of=/srv/swapfile bs=1M count=4096
sudo mkswap /srv/swapfile
sudo swapon /srv/swapfile
```
关闭分区可以运行
```
sudo swapoff /srv/swapfile
```
为了每次开机自动挂载swap分区，需要在 /etc/fstab中加上下面一行:
`/srv/swapfile   swap    swap    defaults        0       0`
修改/etc/fstab文件若有错误可能导致开不起机，因此需要测试下，命令含义是挂载全部分区
```sh
sudo mount -a
```

# 2.安装配置Uswsusp
```sh
sudo apt-get install uswsusp
dpkg-reconfigure -pmedium uswsusp
```
配置对话如下:
>At the "Continue without a valid swap space?" question, answer Yes.
>At the "Swap space to resume from:" prompt, select the partition where the file above was created.
>Answer the "Encrypt snapshot?" and "Show splash screen?" questions as you please.

修改/etc/defaults/grub启动脚本
```
GRUB_CMDLINE_LINUX_DEFAULT="resume=/dev/sda7 quiet"
```
其中 "/dev/sda7" 是swap file所在的分区

# 3.创建休眠脚本
我 的是在`~/bin`下，并且这个目录以及放到环境变量中了。由于deepin的锁屏命令运行后就不再执行脚本了，所以我"创新"的把这个脚本放到了后面。在休眠前最好关闭一些占用内存并且有记忆功能的应用比如chrome，IDE，虚拟机(有bug，休眠一般会造成正在运行的虚拟机崩溃)等。
susp.sh
```sh
sudo swapoff /srv/swapfile
sudo swapon  /srv/swapfile #清空虚拟内存
sudo /usr/sbin/s2disk
dde-lock # 锁定
```
