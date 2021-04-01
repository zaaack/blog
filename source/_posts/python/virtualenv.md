title: virtualenv
permalink: virtualenv/
tags: [python]
date: 2016-06-05 09:32:53
---


>virtualenv是一个python虚拟运行时环境，可以将一个项目所用到的python运行时和python依赖包都打包到项目中，与本机环境隔离，类似node的node_modules

sudo pip install virtualenv #安装virtualenv到本机

cd /path/to/your/project/ #转到项目目录
virtualenv venv           #安装python运行时和pip
. venv/bin/activate       #激活venv，顺便查找按章req.txt中包含的python包
pip install -r req.txt    #安装req.txt中的包
