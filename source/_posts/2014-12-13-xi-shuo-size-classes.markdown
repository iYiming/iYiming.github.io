---
layout: post
title: "细说 Size Classes"
date: 2014-12-13 15:11:02 +0800
comments: true
categories: iOS
---

本来想细说说Size Classes，没想到苹果给开发者封装的这么简单。

Size Classes是iOS8的一个新特性，在一个Xib文件上适配所有iOS屏幕，一想就是一件很酷的事情。Size Classes 分为 紧凑 (Compact) 、任意 (Any) 、 正常 (Regular)。

iOS设备分为如下：

![Size Classes]({{ root_uri }}/images/xishuo_classclasses/xishuo_classclasses_01.png)  

如果记不下来的话，也不需要记，可以通过下图红色区域知道，哪种设备到底使用啥样的Size Classes：

![Size Classes]({{ root_uri }}/images/xishuo_classclasses/xishuo_classclasses_02.png)  

##实战
实现一个关于一个介绍页面

iPhone 竖屏下，如下图：

![Size Classes]({{ root_uri }}/images/xishuo_classclasses/xishuo_classclasses_03.png)  

iPhone 横屏下，如下图：

![Size Classes]({{ root_uri }}/images/xishuo_classclasses/xishuo_classclasses_04.png)  

##代码
[https://github.com/iYiming/YMSizeClasses](https://github.com/iYiming/YMSizeClasses)

##注意事项
哪些约束需要在当前下红色框安装（install），如果没有当前Size Classes，点击下方绿框图。

![Size Classes]({{ root_uri }}/images/xishuo_classclasses/xishuo_classclasses_05.png)  


##参考文章  
[初探 iOS8 中的 Size Class](http://blog.csdn.net/yongyinmg/article/details/39315829)  
[Xcode 6中自动布局Auto Layout和Size Classes的使用](http://blog.csdn.net/lihuiqwertyuiop/article/details/39271887)  
[iOS8 Day-by-Day :: Day 7 :: Adaptive Layout and UITraitCollection](http://www.shinobicontrols.com/blog/posts/2014/07/28/ios8-day-by-day-day-7-adaptive-layout-and-uitraitcollection)  

