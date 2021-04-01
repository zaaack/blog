title: windows下配置vagrant的笔记
permalink: vagrant-in-windows/
tags: [vagrant]
date: 2016-06-05 09:32:53
---
# windows下配置vagrant的笔记(已转deepinlinux)
>本文档主要记录windows环境下按照下面两篇文档搭建shire-in-vagrant时所遇到的坑
>1. shire-in-vagrant文档:<http://theoden.intra.douban.com:45068/siv/tutorial.html>
>2. 搭建shire服务:<http://code.dapps.douban.com/shire/blob/master/SETUP.md>

## 1. 下载虚拟镜像
如果自动安装软件包失败（表现为很多软件在命令行上找不到，比如git，python。。），那么还是放弃使用默认的在线安装吧，可以手动下载box文件进行配置，在这些网站可以找到很多box，豆瓣的是Ubuntu 12.04.5 LTS 64位，也直接找导师要box文件(强烈推荐)：
https://atlas.hashicorp.com/boxes/search
http://www.vagrantbox.es/

可能会用到的命令
```sh
vagrant box add <box name> <path to box>
vagrant init
vagrant up
```

## 2. 文件夹同步

>我们需要在主机上写代码，并且在主机提交，而虚拟机的作用仅仅是模拟服务器环境用来查看效果的，需要特别注意的是本文档最终的实现方案在同步文件的过程中是会修改文件的权限的，因此是绝对不能在同步的一头修改文件，另一头commit文件的！！！会产生很严重的后果！！

### [vagrant官网上windows无法使用或者有坑的方案](http://docs.vagrantup.com/v2/synced-folders/index.html)：
* NFS（官方推荐）是不支持windows的，[vagrant官网](http://docs.vagrantup.com/v2/synced-folders/nfs.html)已经详细说明
* virtualbox共享文件夹 我没能在虚拟机中安装virtualbox插件，据说此种方案效率低下，而且同步文件夹内不支持linux的软链接（ln命令），这 意味着如果项目中的python代码用到了symlink()函数会报错。。（本人在尝试下一个方案时在此坑折腾良久）
* smb 最容易成功的方案，but，同上，在同步文件夹内也不支持linux的软链接（ln命令），这 意味着如果项目中的python代码用到了symlink()函数会报错，如果不使用linux的软链接，这是最好的方案

### 可行且兼容性最强的方案
#### RSync
RSync是一个ssh同步工具，也就是直接通过ssh协议将本机某个文件夹的文件同步到服务器上。。不过可惜的是这种同步是单向的，貌似也有双向同步的工具,如[unison](http://www.oschina.net/p/unison)，暂时不需要就没研究了。。

据vagrant说是可以直接集成的，但是我在win8.1 64位下没有配置成功。现在采用的方案是用node监听文件夹变化，然后调用外部命令进行同步

##### 1. 首先配置Vagrantfile，如果遇到问题请将中文注释删掉：
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
  config.vm.box = "package"

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

  #这样就可以配置smb方式的同步了，速度应该比virtualbox自带的快，第一个参数是本机目录，第二个参数是虚拟机目录。
  # config.vm.synced_folder "../code/market", "/home/vagrant/market", type: "smb"

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




##### 2. 手动配置rsync同步
**我的目录结构**

```
E:\versioncontrol\doubangit
|-code
   |-share
   |-.scripts
      |-cwrsync.bat
      |-exclude.txt //同步排除文件
      |-watch.js  //node实现方案，由于性能问题已作废
      |-watch.py  //python方案，效率较高
      |-devkey
      |-devkey.pub
   |-setup.bat  //up and watch sync
   |-halt.bat   //关机
   |-reload.bat //重启
   |-suspend.bat //休眠（可以保持服务器不关闭）
   |-watch.bat //监听文件改动并同步
|-shire-in-vagrant
```

* 安装
rsync是linux下的软件，可以通过cygwin运行,也可以通过msys运行。不过网上已经有打包好的[cwRsync](http://www.rsync.net/resources/howto/windows_rsync.html)下载。解压到没有空格没有中文的目录下，将X:\path\to\cwRsync\bin添加到环境变量中，就算安装完成了。

* 测试运行

*1. 生成ssh-key。*

我需要同步的文件夹叫做share，在E:\versioncontrol\doubangit\code目录下，那么先在该目录打开命令行，运行`ssh-keygen`，输入要保存的地址:`E:\versioncontrol\doubangit\code\devkey`。然后在code目录下可以看到devkey（私钥文件）和devkey.pub（公钥文件）两个文件。将devkey.pub 上传到虚拟机的/home/vagrant/.ssh/上，运行以下命令将公钥拷贝到authorized_keys：

```sh
cd ~/.ssh
cat devkey.pub>>authorized_keys
```

*2. 在命令行上运行以下命令*

```bat
rsync -avz --delete --chmod Du=rwx,Dog=rx,Fu=rwx,Fgo=rx --chown vagrant:vagrant --rsync-path "sudo rsync" -e "./ssh -p 2222 -l vagrant -i /cygdrive/e/versioncontrol/doubangit/code/.scripts/devkey" /cygdrive/e/versioncontrol/doubangit/code/share/* vagrant@127.0.0.1:/home/vagrant/share
```

参数解释：大多数可以查看官方文档和命令行-help，这里只说几个踩到坑的。
由于rsync同步过去生成的文件是没有权限的，用户和用户组默认是root，会造成极大的不便，虽然可以配置保留原来的权限，但是我却没有成功。所以需要通过下面的参数进行指定。
`--chmod Du=rwx,Dog=rx,Fu=rwx,Fgo=rx 用于指定文件权限。`
`--chown vagrant:vagrant 指定文件的用户:用户组`

在此处可能会遇到一个很坑爹的bug，就是--usermap \*:vagrant找不到命令的bug，几经周折才发现原来是apt-get的rsync版本太老导致的，需要手动进行安装。先到http://pkgs.repoforge.org/rsync/下载最新版的rsync的rpm包，必须和你的虚拟机平台配套。由于ubuntu不能直接安装rpm包，所以需要使用alien进行转换：

1. 先安装。`apt-get install alien`
2. 转换。`sudo alien xxxx.rpm #将rpm转换位deb，完成后会生成一个同名的xxxx.deb`
3. 安装。`sudo dpkg -i xxxx.deb`


**注意**：项目太多时会严重拖慢同步速度，每次要等上六七秒实在是不能忍啊，在我不懈的努力之下终于找到了原因，主要是.git和node_modules目录里面文件太多导致的，因此解决方法有两个：
1. rsync的同步命令中需要增加一个排除文件列表的参数(--exclude-from)，用来过滤掉这些文件。
.scripts目录下新建cwrsync.bat文件保存这些命令。
2. 增加一个命令行参数用于指定需要同步的文件夹，如果全部同步则参数为 .

```bat
@echo off
D:
cd D:\cwRsync\bin
SET HOME=%HOMEDRIVE%%HOMEPATH%
SET BASE_DIR=/cygdrive/e/versioncontrol/doubangit/code
SET SYNC_DIR=%1
@echo on
rsync -avc --exclude-from=%BASE_DIR%/.scripts/exclude.txt --chmod Du=rwx,Dog=rx,Fu=rwx,Fgo=rx --chown vagrant:vagrant --rsync-path "sudo rsync" -e "./ssh -p 2222 -l vagrant -i %BASE_DIR%/.scripts/devkey" %BASE_DIR%/share/%SYNC_DIR%/* vagrant@127.0.0.1:/home/vagrant/share/%SYNC_DIR%/
```

就可以进行下一步了。

*3. 用python监听文件变动并调用同步命令*

本来我是用nodejs写的，奈何nodejs的官方api不能递归监听文件夹，而第三方实现占用资源太多，所有这里采用python的watchdog包进行监听，调用系统底层api效率高很多。
首先需要用pip安装watchdog包
`pip install watchdog`
然后编写watch.py, 监听文件夹变动就调用cwrsync.bat进行同步

```py
#coding=utf-8
import sys
import os
import re
import time
import logging
import thread
import subprocess
from subprocess import Popen, PIPE
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

isSyncing=False
# start=time.time()

def rsync():
    global isSyncing
    # global start
    # end=time.time()
    # if isSyncing or end-start<3:
    if isSyncing:
        thread.exit_thread()
        return
    isSyncing=True
    # start=end
    st=time.time()
    # out=os.popen(sys.path[0]+'/cwrsync.bat').readlines()
    # for line in out:
    #     print line
    p=Popen(sys.path[0]+'/cwrsync.bat '+sys.argv[1], stdout=PIPE, stderr=subprocess.STDOUT)
    while True:
        line = p.stdout.readline()
        if not line:
            break
        print line.strip()
    isSyncing=False
    print '''=====================
    rsync finished in %s seconds!''' % int(time.time()-st)
    thread.exit_thread()


class EventHandler(FileSystemEventHandler):

    def __init__(self):
        pass

    def on_any_event(self, event):
        if event.event_type == 'deleted': #modified, moved,created
            return
        r = re.search('\.git([\\\\/]|$)',event.src_path)
        if r != None:
            return
        print '+++++++++++++++'
        print 'event:%s %s' % (event.event_type,event.src_path)
        thread.start_new_thread(rsync,())


if __name__ == "__main__":
    watchFolder=os.path.dirname(sys.argv[0])+'/../share/'+sys.argv[1]
    print 'start watching folder "'+watchFolder+'"...'
    event_handler = EventHandler()

    observer = Observer()
    observer.schedule(event_handler, watchFolder, recursive=True)
    observer.start()
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

```

*4. vagrant up并运行watchpy*

然后编写总的setup.bat将vagrant up和监听同步整合在一起

```bat
E:
cd E:/versioncontrol/doubangit/shire-in-vagrant
vagrant up
echo $1
python ../code/.scripts/watch.py $1
```


## 后记
其实在windows上有人开发了一个插件用于支持nfsd，但是这是一种buggy way，效率及其低下，而且经常造成vagrant崩溃（至少我这里是这样），不过还是记录一下，尤其vagrant装插件那个trick很有用。。

## nfs in windows
https://github.com/winnfsd/vagrant-winnfsd

## vagrant插件安装失败方法
cp -r gems/vagrant-winnfsd-1.1.0/ /d/HashiCorp/vagrant/embedded/gems/gems/
cp -r specifications/vagrant-winnfsd-1.1.0.gemspec /d/HashiCorp/vagrant/embedded/gems/specifications
cp -r doc/vagrant-winnfsd-1.1.0 /d/HashiCorp/vagrant/embedded/gems/doc
cp cache/vagrant-winnfsd-1.1.0.gem /d/HashiCorp/vagrant/embedded/gems/cache


sudo cp -r gems/vagrant-parallels-1.4.2/ /opt/vagrant/embedded/gems/gems/
sudo cp -r specifications/vagrant-parallels-1.4.2.gemspec /opt/vagrant/embedded/gems/specifications
sudo cp -r doc/vagrant-parallels-1.4.2 /opt/vagrant/embedded/gems/doc
sudo cp cache/vagrant-parallels-1.4.2.gem /opt/vagrant/embedded/gems/cache

## /vagrant is not a directory
https://github.com/mitchellh/vagrant/issues/5933

## windows provision大量报错
http://theoden.intra.douban.com:45068/siv/trouble-shooting.html#windows-autocrlf

## 脚本下载
[vagrant-in-windows.zip](http://pan.baidu.com/s/1qYgTM0W#path=%252Fshare)
