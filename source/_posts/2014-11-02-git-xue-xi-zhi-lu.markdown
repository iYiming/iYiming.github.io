---
layout: post
title: "Git 学习之路"
date: 2014-11-02 23:46:31 +0800
comments: true
categories: IT
---

本文章只是简单把自己常用的Git命令罗列一下，便于查找。本文主要参考 [GitHub Git Cheat Sheet](https://training.github.com/kit/downloads/github-git-cheat-sheet.pdf)。如果你想系统的学习Git，你可以阅读[官方中文文档](http://git-scm.com/doc),可以从这里下载[中文Pro Git epub下载地址](http://pan.baidu.com/s/1c0xUk3e)。后续学到内容，会在本文章里更新。  

##0.安装Git

[各平台版本Git下载](http://git-scm.com/downloads) 

Mac OS X、Windows、Linux、Solaris版本，根据系统下载相应的安装器，安装即可。

##1.配置Git   

####0.设置提交者名称

```
git config --global user.name "填写提交者名称"  
```

####1.设置提交者邮箱

```
git config --global user.email "填写邮箱地址"  
```

####2.设置显示命令行颜色  

```
git config --global color.ui auto
```

上面的是配置全局的，下面命令配置当前文件夹下的 Git

```
git config user.name "填写提交者名称" 
```

##2.创建仓库  
####0.创建本地仓库（XXX代表项目名称）  

```
git init XXX  
```
如果本地想绑定远程仓库，使用下方命令：  

```
git remote add origin https://github.com/XXX.git
```

####1.克隆远程仓库（下载一个项目和他的全部版本历史记录）  

```
git clone https://github.com/XXX.git
```

##3.管理改变

Git 主要的不同点在于，有一个叫做缓冲区的概念。主要存在三个步骤：  

0. 存在有修改的文件  
1. 将修改的文件添加到缓冲区：git add命令  
2. 正式提交文件：git commit命令  

####0.查看改变的文件名  

```
git status
```

####1.查看文件改变的具体内容  
```
git diff
```
####2.将文件添加到缓冲区  

```
git add 文件名
```
####3.查看缓冲区中和最后一次提交版本的差异的内容  

```
git diff --staged
```
####4.移除缓冲区中的文件并保留文件的修改内容

```
git reset 文件名
```
####5.提交文件，加入到版本控制

```
git commit -m "这里填写提交的文本内容"
```
##4.分支管理

Git中有一个重要的概念叫做branch（分支），仓库里的不同分支相当于不同的分组。  

####0.查看所有分支

```
git branch
```
####1.创建新分支

```
git branch 新分支名称
```

####2.创建远程分支（远程分支就是将本地分支push到服务器上）

```
git push origin 本地分支名称
```
####3.切换到某一分支 

```
git checkout 分支名称
```
####4.检出远程某一分支

```
git checkout -b dev origin/dev
```

```
git checkout --track origin/dev
```

####5.合并某一分支到当前分支

```
git merge 其他分支名称
```

####6.删除某一分支

```
git branch -d 要删除的分支名称
```
####7.删除远程某一分支  

```
git push origin :要删除的分支名称
```
##4.stash

####0.暂存当前修改

```
git stash
```

####1.取出最近暂存

```
git stash apply
```

####2.获取暂存列表
```
git stash list
```

####3.使用某个暂存

```
git stash apply stash@{num}
```

```
git stash pop stash@{num}
```
####4.删除暂存

```
git stash drop stash@{num}
```

####5.清空暂存

```
git stash clear
```

##5.Tag管理  

git跟其它版本控制系统一样，可以打标签(tag), 作用是标记一个点为一个版本号，如0.1.3, v0.1.7, ver_0.1.3.在程序开发到一个阶段后，我们需要打个标签，发布一个版本，标记的作用显而易见。

下面介绍一下打标签，分享标签，移除标签的操作命令。

####0.打标签  

```
git tag -a 0.1.3 -m "Release version 0.1.3"
```

详解：git tag 是命令  
-a 0.1.3是增加名为0.1.3的标签  
-m 后面跟着的是标签的注释  

打标签的操作发生在我们commit修改到本地仓库之后。完整的例子:
```
git add .

git commit -m "fixed some bugs"

git tag -a 0.1.3 -m "Release version 0.1.3"
```

####1.提交标签到远程服务器上

```
git push origin --tags
```

####2.删除标签的命令

```
git tag -d 0.1.3
```
####3.删除远端服务器的标签

```
git push origin :refs/tags/0.1.3
```
####4.检出标签

```
git checkout tags/0.1.3
```

参考文章  

[git tag 操作教程](http://www.360doc.com/content/11/1115/16/2036337_164561697.shtml)