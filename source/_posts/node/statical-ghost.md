title: 怎样使用statical-ghost静态博客生成软件? # these are yaml config, notice the space !
permalink: how-to-use-statical-ghost-to-generate-blog/
tags: [node, statical-ghost]
date: 2015-12-29 20:56:38
---

又一个静态博客生成器, 基于node, 使用ghost主题。现在已经有很多静态博客生成器了，最常见的是基于ruby的jekyll, orgpress ，奈何我对ruby不熟,或者没有ruby环境，或者安装麻烦, 或者太过复杂，其实我都没用过，但是造轮子总是可以找到无数的理由。。

 node下有大名鼎鼎的hexo，但是比较复杂，而且官方主题数量有限，而ghost作为wordpress的劲敌，进过数年的发展已经比较稳定，并且积累了相当数量的主题，不需要担心主题的问题，而且功能简单，方便自定义。总之，由于种种原因，这个博客生成器便诞生了。第一次写node工具，走了些弯路，node异步文件读写踩了不少坑，诞生总算基本完成了，希望大家多多指教。

# 安装

```sh
npm i -g statical-ghost
```

# 特性

* 迅速，由于使用node的多进程技术，生成上千篇文章只需几秒。
* 容易使用, 只需记住几条简单的命令即可。
* 基于ghost主题，大量的免费主题可供挑选(<http://marketplace.ghost.org/themes/free/>)。


# 使用

## 初始化

首先

```sh
sg init # or sg i
```

然后，你会得到一个如下的文件夹结构

```
  <current directory>
    |-posts   # 你的markdown文章(以.md结尾), 支持多级目录
    |-public # 生成博客所在目录
    |-tmp # 缓存文件夹
    |-themes # 主题目录
```

这个命令不仅生成了文件夹，还自动生成了一份默认配置文件和一篇示例文章， 甚至下载了一个默认ghost主题。

也就是说这个命令之后就已经可以进入下一步进行生成博客了。当然你也可以尝试在`/posts`目录下写一些文章。

## generate

这一步进行生成博客。
```sh
sg generate # or sg g
```

## server

当你看到`/public`目录下产生了很多文件之后， 就可以使用这个命令

```sh
sg server # or sg s
```

来启动一个本地静态服务器了, 这个命令也可以通过监听文件, 主题或配置改变自动生成博客。
现在点击 <http://127.0.0.1:8080> 来看你的博客吧 !!!

## 查看帮助

```sh
sg -h
```
然后就可以看到如下的帮助信息:
```
Usage: sg [command] [options]


  Commands:

    generate|g             generate blog
    init|i                 initialize blog structure in current directory
    clean|c                clean temp dir
    deploy|d               deploy to server, use config.deploy in config.yaml as command
    generateAndDeploy|gd   generate and deploy
    server|s               start a static server, which would auto generate when posts changed

  another static glog generator by ghost theme

  Options:

    -h, --help           output usage information
    -V, --version        output the version number
    -c, --config <path>  set config path. default is ./config.yaml
    -p, --port           the port of local server
```

## config file
配置文件为工作目录下的`config.yaml`文件，使用yaml书写。

# TODO
