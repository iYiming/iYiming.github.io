---
layout: post
title: "Auto Layout 实战"
date: 2014-12-13 11:35:25 +0800
comments: true
categories: iOS
---

介绍下项目中运用到的用Storyboard中实现复杂的UITableViewCell，原先的博客也实现了这么个功能，但是遇到的这个界面略显复杂。

效果如下图：

![Auto Layout]({{ root_uri }}/images/autolayout_combat/autolayout_combat_01.png)  


##代码下载：  
[https://github.com/iYiming/YMGroupMsg](https://github.com/iYiming/YMGroupMsg)  

##中间遇到的问题：

####0.UILabel如何多行
虽然numberOfLines已经设置为0，但是由于分别在iPhone，iPad上测试都出现不能换行现象，问题是没有设置preferredMaxLayoutWidth属性，但是不能再Xcode属性面板里设置，而需要动态的在代码里实现。  

在YMGroupMsgTextCell中设置，如下代码：  
kScreenWidth为当前设备屏幕的宽度。  
```
- (void)awakeFromNib {
// Initialization code
self.groupValueLabel.preferredMaxLayoutWidth = kScreenWidth - 30;
self.contentTextLabel.preferredMaxLayoutWidth = kScreenWidth - 30;
}
```

####1.约束冲突  
中间出现了好多错误冲突，红色的那种，出于不了解情况，出现这种错误就不知道怎么解决。但是Xcode提供了很好的解决冲突的方式，通过改变约束的优先级来解决，根据Xcode提示的解决的办法一步步弄好就OK了。

####2.Xcode出现了项目警告（不是约束警告）  

在Xcode6下开发时，使用autolayout的Storyboard出现警告：Automatic Preferred Max Layout Width is not available on iOS versions prior to 8.0

原因：xcode6下默认开启自动设置UILabel的preferredMaxLayoutWidth 的属性，而自动的特性只支持iOS8，故导致这个警告，如果想兼容iOS7则需要消除。

解决方法：设置固定的preferredMaxLayoutWidth  （勾选Explicit）。

![Auto Layout]({{ root_uri }}/images/autolayout_combat/autolayout_combat_02.png)   

参考文章：  
[http://blog.sina.com.cn/s/blog_5fde60890102v3wj.html](http://blog.sina.com.cn/s/blog_5fde60890102v3wj.html)