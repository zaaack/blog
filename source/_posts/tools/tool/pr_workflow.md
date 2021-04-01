title: pull request 工作流基础
permalink: pr-workflow/
tags: [git]
date: 2016-06-05 09:32:53
---

## 1.fork代码

点击项目主页的fork按钮，你懂的

## 2.在本地创建dev分支进行开发

```sh
git clone <your focked project>
git branch dev #新建分支dev
git checkout dev #切换分支为dev
```

后面两句可以合并,等价于:
```sh
git clone <your fork>
git checkout -b dev #新建并切换到分支dev
```

## 3.写代码和提交

当你在你的分支上写了很多代码后,可以随时提交到自己fork的远程仓库里,默认是origin,用于保存当前的工作状态:
```sh
git add -A #添加全部,也可以只添加某个文件夹或者修改的文件: git add <glob pattern>
git commit -m <message>
git push origin dev
```

如果想更新最新的代码,你需要使用以下代码添加上游仓库:
```sh
git remote add upstream <上游仓库地址>
```

然后通过下面命令查看:
```sh
git remote -v
```

当你觉得自己的部分已经写好了,可以提交到上游仓库了,需要先将自己的commit合并到
```sh
git rebase -i <commit> #合并commit,<http://blog.csdn.net/zmyde2010/article/details/8603810>
git fetch upstream master
git rebase upstream/master #重置提交顺序，并检查是否有冲突，将自己的提交放在master的最上方
 git push origin dev -f #强制更新到fork版本库的master分支
# 最后提一个pull request将自己fork的版本库的dev分支提交给主项目的master分支，自己的master分支只有在别人给你贡献代码时才有用,别人fork了你fork的项目，然后往你的master分支提pull request，然后再将你的dev分支和别人的分支合并到你fork的版本库的master分支
```
也可以使用`pull --rebase`:

```sh
git rebase -i <commit>
git pull --rebase upstream master
git push origin dev -f
```

写好代码后，最好先用`git status`查看当前项目的状态,
然后将需要提交的文件用`git add`添加到版本库中
然后再运行`git commit -m <message>`进行提交

## 4.同步上游版本库

fork相当于在服务器上clone整个项目，如果要保持fork的代码最新，需要先clone到本地，然后添加上游项目的地址，再进行更新。

比如一个项目的地址是：http://code.dapps.douban.com/market.git
我fork后的地址为：http://code.dapps.douban.com/yangzhen/market.git

然后需要运行的命令行：
```sh
# clone到本地
git clone http://code.dapps.douban.com/yangzhen/market.git
# 添加上游版本库的源
git remote add upstream http://code.dapps.douban.com/market.git
```

之后就可以写代码和在本地commit了。
git commit 有用的参数
-m --message (必选) 提交的备注
-A --all 提交前add所有改动的代码
-i --include 在提交前add特定的文件
-o --only 只提交特定的某个文件
--short 简洁显示
--amend 只更新上次的提交（相当于将本次提交合并到上次提交）

在每次pull request前做如下操作，即可实现和上游版本库同步

```bat
git remote update upstream
git checkout {branch name} #切换分支，这一步必不可少!

git rebase upstream/{branch name}
```
其中rebase的作用是合并分支代码,
checkout和rebase 会把你的分支里的每个提交(commit)取消掉，并且把它们临时 保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把"mywork"分支更新 为最新的"origin"分支，最后把保存的这些补丁应用到"mywork"分支上。（具体可见 http://blog.csdn.net/hudashi/article/details/7664631 ）


## 5.修正rebase冲突

请看这个博客：http://www.cnblogs.com/sinojelly/archive/2011/08/07/2130172.html


## 6.撤销上次rebase的修改
实际上，在rebase之前的提交会以ORIG_HEAD之名存留。如果rebase之后无法复原到原先的状态，可以用git reset --hard ORIG_HEAD复原到rebase之前的状态
```sh
git reset --hard ORIG_HEAD
```


## 7.从别人那里pull代码

```sh
git pull --rebase <remote> <branch>
```

## 8.更多的git内容
[git](../git-base/)
