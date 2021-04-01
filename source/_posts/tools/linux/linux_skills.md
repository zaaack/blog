title: linux技巧汇总
permalink: linux-skills/
tags: [linux]
date: 2016-05-26 09:29:42
---

>一些linux技巧的备忘记录

### linux全局代理
#### proxychains(通常不好用)
/etc/proxychains.conf
```sh
socks5  127.0.0.1 1080
proxychains wget http://www.youtube.com -r
```
#### cow(<https://github.com/cyfdecyf/cow>)
cow可以作为二级代理，自动学习分流.
cow可以作为二级代理，自动学习分流.
然后在~/.zshrc或~/.bashrc中配置变量 ,还可以设置ftp代理ftp_proxy
```sh
export http_proxy='http://localhost:8118'
export https_proxy='http://localhost:8118'
```

### linux系统更新命令
```sh
sudo apt-get update && sudo apt-get dist-upgrade -y
```

#### 开机启动脚本
`/etc/rc.local `写入命令 or `~/.config/autostart`写入.desktop

#### 解决ntfs挂载问题

安装nftsfix,然后：
```sh
sudo ntfsfix /dev/sda1
sudo ntfsfix /dev/sda5
sudo ntfsfix /dev/sda6
```
