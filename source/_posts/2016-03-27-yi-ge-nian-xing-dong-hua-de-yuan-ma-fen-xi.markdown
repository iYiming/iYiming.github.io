---
layout: post
title: "一个粘性动画的源码分析"
date: 2016-03-27 10:54:09 +0800
comments: true
categories: iOS
---

周四(2016-03-24)下午在公司做了一次技术分享，现整理成博客的形式。  

分享的内容是一款动画，动画源码下载地址、效果及 Keynote 下载地址如下：

源码下载地址：  

[https://github.com/KittenYang/GooeyTabbar](https://github.com/KittenYang/GooeyTabbar) 

Keynote 下载地址：

[http://pan.baidu.com/s/1qYUUKaw](http://pan.baidu.com/s/1qYUUKaw)  

源码效果：  

![animation_gooeytabbar/01.gif](/images/animation_gooeytabbar/01.gif)    


##0.什么是 CADisplayLink ？

####名词解释

> A CADisplayLink object is a timer object that allows your application to synchronize its drawing to the refresh rate of the display.  
> 
>——By Apple
>   
>CADisplayLink 对象是一个定时器对象，能够让你的应用程序随着屏幕刷新频率同步绘制。

####如何使用 CADisplayLink ？

```
var displayLink: CADisplayLink? // 声明一个 CADisplayLink 对象

/**
配置 CADisplayLink 对象
*/
func configDisplayLink() {
  displayLink = CADisplayLink(target: self, selector: "handleDisplayLink:")
  displayLink?.addToRunLoop(NSRunLoop.currentRunLoop(), forMode: NSDefaultRunLoopMode)
}
    
/**
执行 CADisplayLink 指定的方法
    
- parameter displayLink: 即上面初始化的 displayLink
*/
func handleDisplayLink(displayLink: CADisplayLink){
    //这里做些处理事情

}
    
/**
  关闭 DisplayLink
*/
func stopDisplayLink(){
    displayLink?.invalidate() //从 Runloop 中移除
}
```

用代码的方式解释一下: 

1. ``CADisplayLink`` 对象指定了一个选择器 ``handleDisplayLink:``。  

2. 将 ``CADisplayLink`` 对象添加至 ``NSRunLoop.currentRunLoop()`` 。

3. 屏幕刷新完毕后，会执行上面指定的选择器 ``handleDisplayLink:``（iOS 设备的屏幕刷新频率( FPS )是 60Hz，因此 ``CADisplayLink`` 的 ``selector`` 默认调用周期是每秒 60 次）。

####CADisplayLink VS NSTimer  

看完上面的介绍，肯定会问 ``CADisplayLink`` 和 ``NSTimer`` 有什么区别？

1、原理不同  

>``CADisplayLink`` 是一个能让我们以和屏幕刷新率同步的频率将特定的内容画到屏幕上的定时器类。``CADisplayLink`` 以特定模式注册到 Runloop 后，每当屏幕显示内容刷新结束的时候，Runloop 就会向 ``CADisplayLink`` 指定的 target 发送一次指定的 selector 消息， ``CADisplayLink`` 类对应的 selector 就会被调用一次。
``NSTimer`` 以指定的模式注册到 Runloop 后，每当设定的周期时间到达后，Runloop会向指定的 target 发送一次指定的 selector 消息。

2、周期设置方式不同  

>iOS设备的屏幕刷新频率(FPS)是60Hz，因此 ``CADisplayLink`` 的 selector 默认调用周期是每秒 60 次，这个周期可以通过 ``frameInterval`` 属性设置，``CADisplayLink`` 的 selector 每秒调用次数 ``=60/frameInterval``。比如当 ``frameInterval`` 设为 2，每秒调用就变成 30 次。因此，``CADisplayLink`` 周期的设置方式略显不便。  
>
``NSTimer`` 的 selector 调用周期可以在初始化时直接设定，相对就灵活的多。

3、精确度不同  

>iOS 设备的屏幕刷新频率是固定的，``CADisplayLink`` 在正常情况下会在每次刷新结束都被调用，精确度相当高。
``NSTimer`` 的精确度就显得低了点，比如 ``NSTimer`` 的触发时间到的时候，Runloop 如果在忙于别的调用，触发时间就会推迟到下一个 Runloop 周期。更有甚者，在 OS X 10.9 以后为了尽量避免在 ``NSTimer`` 触发时间到了而去中断当前处理的任务，NSTimer新增了``tolerance`` 属性，让用户可以设置可以容忍的触发的时间范围。  

4、使用场合
>从原理上不难看出，``CADisplayLink`` 使用场合相对专一，适合做界面的不停重绘，比如视频播放的时候需要不停地获取下一帧用于界面渲染。
``NSTimer`` 的使用范围要广泛的多，各种需要单次或者循环定时处理的任务都可以使用。

请参考：

[http://blog.csdn.net/wzzvictory/article/details/22417181](http://blog.csdn.net/wzzvictory/article/details/22417181)  

##1.如何画曲线 ？

效果展示：

![animation_gooeytabbar/02.gif](/images/animation_gooeytabbar/02.gif)  

p0,p2 是普通点，p1 是控制点。 

代码实现：  

```
override func drawRect(rect: CGRect) {
  // Drawing code
  let p0 = CGPoint(x: 50, y: 50) // 点 p0 
  let p1 = CGPoint(x: 65, y: 25) // 点 p1
  let p2 = CGPoint(x: 100, y: 50) // 点 p2
  
  let path = UIBezierPath() // 创建 贝塞尔曲线路径 对象
  path.moveToPoint(p0) // 将画笔移至 点 p0
  path.addQuadCurveToPoint(p2, controlPoint: p1) // 为 p2添加二维曲线控制点为p1
  
  let context = UIGraphicsGetCurrentContext() // 当前绘制上下文
  CGContextAddPath(context, path.CGPath) // 将创建的贝塞尔曲线路径对象添加至上下文中
  UIColor.redColor().set() // 设置绘制颜色
  CGContextStrokePath(context) // 绘制
}
```

##2.如何使用弹性动画 ？

```
/**
弹性动画
        
- parameter duration:               动画时长
- parameter delay:                  延迟时长
- parameter usingSpringWithDamping: 阻尼系数
- parameter initialSpringVelocity:  初始弹性速度
- parameter options:                动画选项 如:EaseInOut
- parameter animations:             动画 Block
- parameter completion:             动画完成 Block
*/
@available(iOS 7.0, *)
public class func animateWithDuration(
duration: NSTimeInterval, 
delay: NSTimeInterval, 
usingSpringWithDamping dampingRatio: CGFloat, 
initialSpringVelocity velocity: CGFloat, 
options: UIViewAnimationOptions, 
animations: () -> Void, 
completion: ((Bool) -> Void)?
)
```

上面的注释已经写得很清楚，需要注意的是 ``usingSpringWithDamping`` 参数，即阻尼系数，数值范围 ``0 <= n <= 1``,系数越大弹性越大。  

更多详细介绍，请参考：  

[https://www.renfei.org/blog/ios-8-spring-animation.html](https://www.renfei.org/blog/ios-8-spring-animation.html)  

##3.粘性动画原理

效果展示：

![animation_gooeytabbar/03.gif](/images/animation_gooeytabbar/03.gif)   

代码实现： 

```
override func drawRect(rect: CGRect) {
  // Drawing code
  let viewWidth = self.frame.width // view 的宽度
  let viewHeight = self.frame.height // view 的高度

  UIColor.redColor().set()
  let context = UIGraphicsGetCurrentContext()
  CGContextMoveToPoint(context, 0, viewHeight) // 左下角点
  CGContextAddLineToPoint(context, viewWidth, viewHeight) // 右下角点
  CGContextAddLineToPoint(context, viewWidth, 120) // 右上角点
  CGContextAddQuadCurveToPoint(context, viewWidth/2.0, 120 + CGFloat(progress), 0, 120) // 左上角 控制点
  CGContextClosePath(context) //将开始点和结束点连接起来
  CGContextFillPath(context) // 填充
}
```

##4.源码分析

详见[分享链接](http://pan.baidu.com/s/1qYUUKaw)里的源码注释。

##5.源码中的几个问题

####内存引用问题

源码 TabbarMenu.swift 中有这么一段代码：

```
private var animateButton : AnimatedButton? // 动画按钮

//动画那个按钮 即那两条横线
animateButton = AnimatedButton(frame: CGRect(x: 0, y: 200, width: 50, height: 30))
self.view.addSubview(animateButton!)
animateButton!.didTapped = { (button) -> () in
  self.triggerAction() // 按钮点击触发方法
}
```

这里面存在着一个保留环的问题：  

1. TabbarMenu 实例 ``self`` 引用了 ``animationButton``。  
2. ``animationButton`` 的 block ``didTapped`` 又引用了
``self.triggerAction()``。

如何解决这个问题？在 Objective-C 中我们使用 ``__weak``,在 Swift 中呢？我们需要添加 ``[unowned self]``。实现如下：

```
//动画那个按钮 即那两条横线
animateButton = AnimatedButton(frame: CGRect(x: 0, y: 200, width: 50, height: 30))
self.view.addSubview(animateButton!)
animateButton!.didTapped = { [unowned self](button) -> () in
  self.triggerAction() // 按钮点击触发方法
}
```

关于 Swift 保留环，如何处理的问题，请参考：  

[Block retain cycles in Swift?](http://stackoverflow.com/questions/24042949/block-retain-cycles-in-swift)  

####私有方法问题  

源码 AnimatedButton.swift 中有这么一段代码：

```
self.addTarget(self, action: #selector(AnimatedButton.animate), forControlEvents: .TouchUpInside)  

@objc private func animate() {
  ...
}
```

``@objc`` 是什么鬼？

如果去掉 ``@objc``,在 Xcode 7.3 中会直接报错(Xcode 老版本 运行点击按钮会出现崩溃)：  

![animation_gooeytabbar/04.png](/images/animation_gooeytabbar/04.png)

[经查证](http://stackoverflow.com/questions/25056278/swift-access-control-with-target-selectors)，原因如下：

> Declarations marked private are not exposed to the Objective-C runtime if not otherwise annotated. IB outlets, IB actions, and Core Data managed properties remain exposed to Objective-C whatever their access level. If you need a private method or property to be callable from Objective-C (such as for an older API that uses a selector-based callback), add the @objc attribute to the declaration explicitly.

大意为:

在 Swift 中,如果将方法或者属性标记为 ``private``,将不会给 Objective-C Runtime 暴漏，如果想从 Objective-C（一些老的 API，像基于选择器的回调），必须明确添加 ``@objc`` 标识。

####修改高斯模糊视图 alpha 问题

在运行源码的过程中，我们看到控制台打印出如下警告：

>2016-03-27 10:29:28.879 GooeyTabbar[54798:1868007] <UIVisualEffectView 0x7fcc9bd06fe0> is being asked to animate its opacity. This will cause the effect to appear broken until opacity returns to 1.

不应该修改 ``UIVisualEffectView`` 的 ``opacity`` 的属性。应做如下修改：

```
let overlay = UIVisualEffectView()
// Put it somewhere, give it a frame...
UIView.animateWithDuration(0.5) {
    overlay.effect = UIBlurEffect(style: .Light)
}
```

请参考：

[http://stackoverflow.com/questions/29307827/how-does-one-fade-a-uivisualeffectview-and-or-uiblureffect-in-and-out](http://stackoverflow.com/questions/29307827/how-does-one-fade-a-uivisualeffectview-and-or-uiblureffect-in-and-out)  

##6.分享的心得体会

####0.如果一件事情你从来没做过，身边又没有合适的老师，怎么办 ？  

 **从网上找相关专业最好的东西来模仿**。

 我找的是苹果 WWDC keynote 视频：  
 
 [What is New in Swift](https://developer.apple.com/videos/play/wwdc2015-106/)    
 
####1.如何进行设计 ？  
 
  现在才发现设计是如此重要，做程序猿也要文艺点。由于原先就意识到这个问题，读过《写给  大家看的设计书》这本书，如何对齐、如何重复、如何配色等等。这些对我启发很大。每个像素都值得深究。  
  
####2.如何配图 ？

   Keynote 中最重要的一点，就是好的配图。配图能激发听者的兴趣。因为原先写博客的原因，积累了一些有意思的配图。平常发现有意思的图，就应该收藏下来，说不定哪天就能派上用场。  
   
####3.一些新的发现

   通过学习 Keynote，我找到了一些好的资料，比如许岑老师的教程以及如何使用 Keynote 做动效等。把每件小事做好，都是一件不简单的事情。  

####4.不足

口才。终于明白，提前排练的重要性（连乔布斯也要在演讲前都要反复排练）。自己没有排练过，导致语次颠倒，逻辑稍有混乱。

##7.图片版

![images/1.jpeg](/images/animation_gooeytabbar/1.jpeg)

![images/2.jpeg](/images/animation_gooeytabbar/2.jpeg)

![images/3.jpeg](/images/animation_gooeytabbar/3.jpeg)

![images/4.jpeg](/images/animation_gooeytabbar/4.jpeg)

![images/5.jpeg](/images/animation_gooeytabbar/5.jpeg)

![images/6.jpeg](/images/animation_gooeytabbar/6.jpeg)

![images/7.jpeg](/images/animation_gooeytabbar/7.jpeg)
