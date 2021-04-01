title: linux下配置vagrant的笔记
permalink: vagrant-in-linux/
tags: [vagrant,linux]
date: 2016-06-17 09:32:53
---
# linux配置vagrant

>https://www.vagrantup.com/docs/

## 常用命令

### 从本地box导入
```sh
vagrant box add <box name> <path to box> #导入box文件
vagrant init #初始化一个Vagrantfile配置文件，若已有则跳过此步
vagrant up #启动vagrant
```

### 配置共享

linux和osx都支持nfs共享，但是我用的deepinlinux(基于debian)默认没有nfs包，因此需要先手动安装
```sh
sudo apt-get install nfs-kernel-server
```
然后配置Vagrantfile文件
```rb
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  #这里必须和vagrant box add <box name> <path to box>的box name一致
  config.vm.box = "shire"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.host_name = "vagrant"
# vagrant默认是通过端口转发和虚拟机通信，也就是虚拟机可以像主机一样用127.0.0.1访问。不过这种方式无法在外部连接端口很多的软件，比如mysql。
  config.vm.network :forwarded_port, guest: 9010, host: 9010
  config.vm.network :forwarded_port, guest: 9015, host: 9015
  config.vm.network :forwarded_port, guest: 9200, host: 9200

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  #这种方式可以用主机上的mysql管理软件连接虚拟机的mysql
 config.vm.network "private_network", ip: "192.168.33.10"

 # config.vm.network "private_network", type: "dhcp"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  #config.vm.synced_folder "../vagrant", "/home/vagrant"

  # config.vm.synced_folder "../code/market", "/home/vagrant/market", type: "smb"
  #这样就可以配置nfs方式的同步了，第一个参数是本机目录，第二个参数是虚拟机目录。
  config.vm.synced_folder "../share/", "/home/vagrant/share/", type: "nfs",  nfs_udp: false


  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = 'true'


  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
    #如果嫌虚拟机太慢可以多分配些资源
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.cpus=4
  end
end
```

## 常用命令介绍

```sh
vagrant status #当前虚拟机的状况
vagrant global-status #全局虚拟机的状态
vagrant box list #查看本地已添加的box
vagrant up [id] #启动虚拟机
vagrant suspend [id] #休眠虚拟机
vagrant resume [id] #恢复虚拟机
vagrant reload [id] [--provision] #重新载入配置文件,需要重启,[--provision]表示同时根据配置文件更新虚拟机内的依赖环境
vagrant ssh [id] #进入虚拟环境
vagrant halt [id] #关机
vagrant provision [id] #在一个正在运行的虚拟机中更新依赖
```
