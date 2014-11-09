---
layout: post
title: "iPhone 屏幕适配"
date: 2014-11-09 14:32:22 +0800
comments: true
categories: 
---
随着iPhone6、iPhone6 plus的发布，屏幕适配任务是每一个iOS开发者必须面对的问题。最近也在做适配，由于界面布局都是用代码写的，适配变得比较麻烦，中间也遇到了不少的问题，现在整理一下。另外，随着iOS8带来的Size Classes，以后手写界面将会成为历史。  

下方整理的是一些相关知识的文章和截图，非常感谢以下博主的分享，我就不重复造轮子了。

##0.Xcode 6新建项目的大屏幕适配  
[Launch Screen/Lauch Images](http://blog.shiqichan.com/Launch-Screen-in-iOS-7-and-8/)  

文章截图见下:

![Layer Subclass]({{ root_uri }}/images/iphone_adaptation/iphone_adaptation_01.png)  

##1.学习Auto Layout
[关于Auto Layout写的很好的一篇文章](http://vit0.com/blog/2013/12/07/iosxue-xi-zhi-autolayout/)  

文章截图见下:

![Layer Subclass]({{ root_uri }}/images/iphone_adaptation/iphone_adaptation_02.png)  

##2.手写布局第三方类库Masonry  

[Github:Masonry](https://github.com/Masonry/Masonry)  

##3.如何使用Size Classes

[苹果官方学习文档](https://developer.apple.com/library/ios/recipes/xcode_help-IB_adaptive_sizes/_index.html)

##4.其他相关文章  
[Onevcat:可视化开发，IB 的新时代](http://onevcat.com/2014/10/ib-customize-view/)

再次感谢以上博客作者的文章！
