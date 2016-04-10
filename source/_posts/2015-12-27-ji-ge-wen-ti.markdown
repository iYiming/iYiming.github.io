---
layout: post
title: "几个问题"
date: 2015-12-27 23:34:40 +0800
comments: true
categories: 挨踢
---

![几个问题 3](/images/ji-ge-xiao-wen-ti/3.jpg)

答案比问题容易，所以得学会提出问题。这是最近看《艾龙.马斯克传》，学到的一个观点。故就把工作中遇到的问题，简单整理下。

近期问题（这些问题，有的比较小，或者比较容易，不过确实在使用它们时遇到了问题）：  
 
0.如何使用 SourceTree？  

1.Xcode 插件不能使用的问题？  

2.如何在使用 Auto Layout 的情况下，自动计算 Cell 的高度？  

3.如何使用 Instruments？  

4.如何使用 Vim？

##0.如何使用 SourceTree

问题 1：如果一个文件夹，我曾经 clone 过项目到这个文件夹。即使把这个文件夹内的文件全部删除，我再在这个文件夹 clone 项目时，提示我这个文件不为空。这个文件夹已经为空了啊？不知道这个问题如何解决。

问题图片：

![几个问题 0](/images/ji-ge-xiao-wen-ti/0.png)

我在 StackOverflow 中找到的答案，但仍没有解决我的问题：

[http://stackoverflow.com/questions/20718193/issue-with-sourcetree-while-cloning-a-github-repository](http://stackoverflow.com/questions/20718193/issue-with-sourcetree-while-cloning-a-github-repository)  


下面展示的为 将 master 分支上的代码 合并到 newfeature1 分支 上：  

![几个问题 1](/images/ji-ge-xiao-wen-ti/1.png)

##1.Xcode 插件不能使用的问题？


原先 写过一篇文章：  

[http://iyiming.me/blog/2015/07/09/xcode-cha-jian-bu-neng-shi-yong-wen-ti/](http://iyiming.me/blog/2015/07/09/xcode-cha-jian-bu-neng-shi-yong-wen-ti/)  

无意间又找到了这个文章：

[https://github.com/ForkPanda/RescueXcodePlug-ins](https://github.com/ForkPanda/RescueXcodePlug-ins)  

补充：  

- 插件安装在哪里？在终端输入下面命令：  

```
sudo open /Users/你的用户名/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins
```

- 安装完新版 Xcode 点击了 skip bundle，使用下面命令：

 注：6.4是Xcode版本，可更改，在终端运行

```
defaults delete com.apple.dt.Xcode DVTPlugInManagerNonApplePlugIns-Xcode-6.4 
```

##2.如何在使用 Autolayout 的情况下，自动计算 Cell 的高度

最近在做项目的过程中 发现了一个奇葩的问题，计算没问题，显示的却有问题。最后分析原因是 混用 frame 和 Autolayout 导致的，那么问题来了，如何动态计算 Cell 的高度呢？

有两种我们常用的方式：  

0.使用 frame 的情况下，就还是用 原始的方法吧。（不推荐，转成 Autolayout 模式吧）
 
1.使用 Autolayout 的情况下，我们可以使用下面这种方式：  

**前提：Cell 的 AutoLayout 设置正确（高度能确定）**

```
- (CGFloat) tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [self cellForIndexPath:indexPath];
    CGSize size = [cell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];
    return size.height + 1;
}
```

以前写过的一个简单 Demo：  

[https://github.com/iYiming/YMGroupMsg](https://github.com/iYiming/YMGroupMsg)  


有没有好的替代品？这里有个第三方封装库，具体请看这篇文章(附源码)：

[http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/](http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/)  

这个会单独写篇文章，分析源码。

##3.如何使用 Instruments

Google 搜索 iOS Instruments ，排行第一的结果是这篇文章：

[https://segmentfault.com/a/1190000000387082](https://segmentfault.com/a/1190000000387082)  

后续会有一篇独立文章介绍 如何使用 Instruments。

##4.如何使用 Vim

关于 Vim，这篇文章再好不过了：

[简明 Vim 练级攻略](http://coolshell.cn/articles/5426.html)   

还有一个速查卡：

[给程序员的 VIM 速查卡](http://coolshell.cn/articles/5479.html)    
