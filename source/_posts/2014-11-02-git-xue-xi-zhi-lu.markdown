---
layout: post
title: "Git 学习之路"
date: 2014-11-02 23:46:31 +0800
comments: true
categories: IT
---

本文章只是简单把自己常用的Git命令罗列一下，便于查找。本文主要参考 [GitHub Git Cheat Sheet](https://training.github.com/kit/downloads/github-git-cheat-sheet.pdf)。如果你想系统的学习Git，你可以阅读[官方中文文档](http://git-scm.com/doc),可以从这里下载[中文Pro Git epub下载地址](http://pan.baidu.com/s/1c0xUk3e)。后续学到内容，会在本文章里持续更新。  

##0.安装Git

[各平台版本Git下载](http://git-scm.com/downloads) 

如果想使用Git图形化操作(推荐使用命令行工具)，不仅包括全部常用仓库操作功能，还能自动更新命令行Git版本  

[Windows版下载地址](https://windows.github.com)   

[Mac版下载地址](https://mac.github.com)  

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
0.存在有修改的文件  
1.将修改的文件添加到缓冲区：git add命令  
2.正式提交文件：git commit命令  

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
####4.合并某一分支到当前分支
```
git merge 其他分支名称
```
####5.删除某一分支
```
git branch -d 要删除的分支名称
```
####6.删除远程某一分支
```
git push origin :要删除的分支名称
````