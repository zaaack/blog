title: git基础知识
permalink: git-base/
tags: [git]
date: 2016-06-05 09:32:53
---
基本的提交流程请参照[pr-workflow](../pr-workflow/)，这里主要讲git的版本回退和一些细节问题

## 常用命令
### git diff
git diff可以用来比较两个文件的变动，或者两个分支的变动

git diff <变动前的文件> <变动后的文件> #比较两个文件 git diff <变动前的提交> <变动后的提交> #比较两个文件

### git reset
git reset –mixed：此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息

git reset –soft：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可

git reset –hard：彻底回退到某个版本，本地的源码也会变为上一个版本的内容

以下是一些reset的示例：

```sh
# 回退所有内容到上一个版本
git reset HEAD^

# 回退a.py这个文件的版本到上一个版本
git reset HEAD^ a.py

# 向前回退到第3个版本
git reset –soft HEAD~3

# 将本地的状态回退到和远程的一样
git reset –hard origin/master

# 回退到某个版本
git reset 057d

# 回退到上一次提交的状态，按照某一次的commit完全反向的进行一次commit
git revert HEAD
```

## 常见问题
### 1. 如何回退某个文件到指定提交

```sh
git reset <commit> <path/to/file>  #恢复文件版本
git checkout <path/to/file>         #检出原来版本的文件
```

### 2. 删除本次提交（保留代码）

```sh
git reset <commit> #默认为 –mixed，保留代码，可以查看修改，重新提交
git status
```

### 3. 切换未提交的分支时如何保存原来的代码

```sh
git stash #缓存当前分支的修改，切换分支后不会丢失之前的修改。。
```

### 4. git 合并多个提交(包括message)

```sh
git log #查看提交历史
git rebase -i HEAD
# or git rebase -i <commit>
# 如果遇到冲突
# 需要先解决冲突 然后git add . 再git rebase --continue
git push origin dev -f #-f 强制提交
```

### 5. 本地过滤文件
有时需要过滤一些自己个人用的测试代码，但是又不想修改.gitignore 可以在.git/info/exclude中添加需要过滤的路径或文件。

比如

```sh
gulpfile.js/task/mobile-test.js
static/
static_resource/mobile/test

mobile/views/test/
templates/mobile/test

.ftpconfig
```

但是这种方法只能忽略未加入git版本库的文件，如果是已经加入版本库的文件，可以使用以下方法：
```sh
git update-index --assume-unchanged /path/to/file
```
如果需要取消，只需要运行以下命令就可以了
```sh
git update-index --no-assume-unchanged /path/to/file
```

### 6.只clone一个最新的commit
这种方法可以避免clone github上的项目因为太大而太慢..

git clone --depth=1 https://github.com/angular/angular-seed.git <your-project-name>

### 7. git patch

git可以将diff导出为一个文件（patch)，然后通过再应用这个文件，有两种方式


#### git diff的
```
git diff <prev_commit> <commit> > patch.diff
git apply patch.diff
```

#### git 本身的
```
git format-patch <commit1>..<commit2> #从commit1 到commit2的diff
git format-patch -1 <c1> # 单独一个commit
git format-patch <c1> # 从某commit以来的修改（不包含该commit）

git am xxx.patch
```

#### 检查是否应用成功
```
git apply --stat patch.patch # 检查patch
git apply --check patch.diff # 测试应用patch
```
## 常用概念
HEAD 指向当前提交的一个指针 HEAD^^ HEAD往前两个版本 HEAD~3 HEAD回退3个版本
