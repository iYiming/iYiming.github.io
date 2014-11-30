---
layout: post
title: "细说 Storyboard"
date: 2014-11-29 18:35:39 +0800
comments: true
categories: iOS
---

Storyboard是iOS应用程序的用户界面的可视化表示，展示屏幕的内容和场景和不同场景之间的连接(Segue)。一个故事板由一个或多个场景组成，场景间通过Segue对象连接起来。Segue对象表示两个控制器之间的转换关系(push,modal,custom)。通过Storyboard也可以在控制器之间互相传送数据。  

![Layer Subclass]({{ root_uri }}/images/xishuo_storyboard/xishuo_storyboard_01.png)  

总的来说Storyboard中有以下东西:  
0.Scene(场景)  
1.初始化:用来指示Storyboard中的第一个场景  
2.Segue:用来连接场景，展示场景转变  有三种类型：push,modal,custom
3.Dock:每个场景都有个Dock,包含控制器、First Respnder，Exit。Dock主要用来在视图和控制器之间创建Action和Outlet。  

##0.Scene

对应单独的控制器和控制器的视图。在iPhone中。每一个场景对应整个的内容。在iPad中，多个场景可以同时出现在屏幕上。

##1.Segue对象

管理者两个场景之间的转变，可以设置Segue来改变场景的转变的类型(push,modal,custom)。也可以通过继承Segue对象来实现自定义转变。  
使用 ``prepareForSegue:sender:``方法在控制器之间传送数据。当Segue触发时就来调用这个方法。该方法可以让让你在下一个控制器出现在屏幕前，自定义设置他。  
控制器之间的转变通常发生在某些事件下，比如一个按钮点击时。但是也可以用代码通过调用控制器的 ``performSegueWithIdentity:sender:`` 来控制转变.

##2.实战
为了详细说明Storyboard中所包含内容，做个Demo详细介绍下：  
注明：本篇只是简单介绍Storyboard的基本用法，暂未讨论Autolayout、Size Classes。以后会对Autolayout、Size Classes分别写篇文章单独介绍。  

[代码下载地址](https://github.com/iYiming/YMStoryboard)  

参考文章：  
[官网Storyboard教程1](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/Storyboard.html)  
[官网Storyboard教程2](https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOS/SecondTutorial.html)  
[Storyboard的简单使用](http://my.oschina.net/plumsoft/blog/53886)  
[Storyboard使用心得](http://blog.csdn.net/quanqinyang/article/details/17137759)  