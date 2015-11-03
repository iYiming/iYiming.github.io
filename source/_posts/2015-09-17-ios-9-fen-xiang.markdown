---
layout: post
title: "iOS 9 分享"
date: 2015-09-17 13:34:04 +0800
comments: true
categories: iOS
---

公司组织分享iOS 9新特性及适配问题，故整理如下：  

本次分享，主要包含2个部分：**iOS 9新特性** 和 **细节知识点**。

关于iOS 9的新特性，该分享仅介绍如下内容：  

0. Multitasking：多任务

1. Search：搜索

2. App Thinning：App瘦身

细节知识点以[https://github.com/ChenYilong/iOS9AdaptationTips](https://github.com/ChenYilong/iOS9AdaptationTips)为主。  

##新特性

关于新特性方面，主要参考[https://developer.apple.com/ios/](https://developer.apple.com/ios/)官方文档。

###1. Multitasking(多任务)


iOS 9 中的多任务分为**3**种表现形式:  
>
>0. Slide Over (临时调出的滑动覆盖) 
>
>1. Split View (真正的同时使用两个 App 的分割视图)
>
>2. Picture in Picture (视频播放的画中画模式)   


***1.Slide Over (临时调出的滑动覆盖)***

Slide Over功能可以让用户在两个应用之间做到快速切换，该功能可在iPad Air、iPad Air 2、iPad mini 2、iPad mini 3上实现。  
>
现在，你可以在打开第二个应用的同时不离开原来的应用。这样你可以快速地浏览网页、回复短信、或是在备忘录中写点什么。然后向右滑动，你就会又返回到原来你所使用的应用。

效果如下：  

![Slide Over](/images/iOS9-Share/1.jpg)


***2.Split View (真正的同时使用两个 App 的分割视图)***

可将屏幕分成两部分，同时运行两个应用。也支持同一款应用的多界面，比如 Safari 浏览器中可并排摆放两个标签页。此功能目前只能在iPad Air 2上实现。  

>
>在你的 iPad Air 2 上使用Split View，你可以在自由操作Apps上走的更远，有两个充满活力的Apps同时活跃在一个时间里。你可以在工作草图旁打开参考照片、你可以在写一篇论文的同时在iBooks中引用一本书的内容。当所有你所需要的东西摆在你面前时，你会更加集中注意。

效果如下：  

![Slide Over](/images/iOS9-Share/2.jpg)


***3.Picture in Picture (视频播放的画中画模式)***

Picture in Picture功能：用户在看视频过程中如果想看或回邮件，视频将以悬浮窗口的形式保留在屏幕上，大小和位置可任意调整。  
>
在使用FaceTime或是观看视频时，请按Home键，你的视频就会缩小到屏幕的一个角落。打开第二个应用，即使你在使用其他应用，FaceTime中家人的笑脸依旧呈现，你的视频依旧播放。所以，在你收看你最喜欢的电视节目时，回复刚刚收到的电子邮件。

效果如下：  

![Slide Over](/images/iOS9-Share/3.jpg)

注：  

0. 因为苹果使用的是iOS 8提供的**Size Classes**来支持多任务，看来以后我们离不开Storyboard啦。

1. 对于Picture in Picture(画中画)，需要注意是：  

 **如果你使用 AVPlayerViewController 或者 AVPlayerLayer 来播放视频的话，那什么都不用做就已经支持了。但如果你之前选择的方案是 MPMoviePlayerController 或者 MPMoviePlayerViewController 的话，你可能也需要尽早迁移到 AVKit 的框架下来，因为 Media Player 将在 iOS 9 被标记为 deprecated 并不再继续维护。**


###2. Search(搜索)

关于[搜索API](https://developer.apple.com/ios/search/)

看下面的文章就够了：  

[http://www.jianshu.com/p/f9bd0404d901](http://www.jianshu.com/p/f9bd0404d901)

###3. App Thinning(App瘦身)  

iOS App Thinning表现为3个部分：

>
>0. App Slicing
>
>1. Bitcode 
>
>2. On-Demand Resources
>

***1.App Slicing***

现在在 AppStore 下载 App 问题：

>
> 0. 因为 iOS App 为了后向兼容，现在都同时包含了 32 bit 和 64 bit 两个 slice。 
>
> 1. 在图片资源方面，更是存在 1x 2x 3x 的图像（1x现在不太需要了，推荐大家使用pdf，矢量图就一套） 
> 

上面的两个问题导致现在的 App 体积剧增。在购买和下载的时候，又是把整个 App 包都下载，浪费流量等资源。于是苹果在 iOS 9 中使用 App Slicing。

使用了 App Slicing 后，节省了用户流量的同时，对于开发者来说，不需要做太多事情，只需要使用asset catalog 标记好 2x 3x 就可以了。

其他的任务，都是AppStore帮咱们处理，如下图：

![App Slicing](/images/iOS9-Share/4.png)

***2. Bitcode***

原来我们是通过 Xcode 提交二进制文件到AppStore，这样就会出现这样一个问题——当编译器更新时，我们必须重新编译一遍再提交到 AppStore 。但是出现了Bitcode后，我们就不在需要再重复这样的操作。  

Bitcode 是 LLVM 的中间码。在用户下载之前，App Store 在自动编译应用程序。   

Bitcode 支持在新项目中是默认开启的，没有特别理由的话，你也不需要将它特意关掉。  

***3.On-Demand Resources***

这个在游戏中用的比较多，听名字就能够看出，按需加载资源。在你需要的时候再下载响应的图片。  

例如：你在游戏中每个关卡的时候只下载这一关卡的资源。是不是会出现这种问题，你在路上空闲时间玩游戏，到了下一关卡，资源还没有下载，那就不能玩了。  


##细节知识点

0.如何适配iOS9：  

[https://github.com/ChenYilong/iOS9AdaptationTips](https://github.com/ChenYilong/iOS9AdaptationTips)

1.iOS 9 day by day中文翻译：  

[http://www.jianshu.com/p/3768b9c65974](http://www.jianshu.com/p/3768b9c65974)

2.开发者所需要知道的 iOS 9 SDK 新特性：  

[http://onevcat.com/2015/06/ios9-sdk/](http://onevcat.com/2015/06/ios9-sdk/)  

3.观看WWDC:  

[https://github.com/insidegui/WWDC](https://github.com/insidegui/WWDC)  

