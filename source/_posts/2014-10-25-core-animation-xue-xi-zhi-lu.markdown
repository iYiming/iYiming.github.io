---
layout: post
title: "Core Animation 学习之路"
date: 2014-10-25 08:30:40 +0800
comments: true
categories: iOS
---

Core Animation是一个图形渲染和动画的基础设施,既可以使用在Mac OS上也可以使用在iOS上。利用Core Animation，所有你需要做的就是配置一些动画参数（如起始和结束点）。Core Animation使用硬件加速，所以流畅的动画不会加重CPU的负担从而减慢你的应用程序。  
![Layer Subclass]({{ root_uri }}/images/core_animation_guid/core_animation_guid_4.png)  


##CALayer对象  

Layer对象是使用Core Animation一切东西的核心。Layers管理应用可视化内容，提供改变演示和可视化内容的选项。

####不同的Layer类提供专门的行为  

Core Animation定义了许多标准的Layer类，他们种的每一个都提供特殊的用途。CALayer是所有这些Layer类的基类。它定义了所有Layer队形必须支持的行为，是layer-backed视图默认的类型。然而，你也可以使用下图中特定的Layer类:  
![Layer Subclass]({{ root_uri }}/images/core_animation_guid/core_animation_guid_1.png)  

##提供Layer的内容  

Layer是数据对象 ，管理你应用程序提供的内容，一个Layer的内容包括一个位图（包含你想展示的可视化的数据），你可以使用以下三种方式中的一种来提供位图的内容：  
1.直接分配图片对象到layer对象的内容属性（对于从不或者很少改变的内容的layer是最好的一种技术  
2.分配一个delegate对象到这个layer，让这个delegate画出layer的内容（这种技术适用于layer 内容周期性改变，可以提供一个额外的对象，比如view）  
3.定义一个Layer子类，自己重写一个绘制方法来提供layer内容（这种技术适用于你需要创建一个layer子类 或者 你想改变layer的基础绘制行为）  

layer-backed的视图（iOS下都是）自动提供了layer的内容。  

####①使用一张图片作为Layer的内容  

因为一个layer就是一个管理位图图片的容器，所以你可以分配一张图片直接到layer的contents属性。分配一张图片到一个图层很容易 让你指定精准的你想要展示在屏幕上的图片。Layer直接使用你提供的图片内容，不会尝试重新创建这张图片的拷贝。这种行为可以减少内存消耗，在节省内存的情况下可以在多个地方使用相同的图像

你提供的图片必须是个CGImageRef类型（在 OS X v10.6或者以后，你可以直接提供NSImage类型），提供图片的时候，请注意提供图片的分辨率适配真实设备分辨率。  

```
CALayer *imageLayer = [CALayer layer];
imageLayer.backgroundColor = [UIColor redColor].CGColor;
UIImage *layerImage = [UIImage imageNamed:@"LayerImage.jpg"];
imageLayer.contents = (id)layerImage.CGImage;
imageLayer.bounds = CGRectMake(0, 0, layerImage.size.width, layerImage.size.height);
imageLayer.contentsScale = 2.0f;
imageLayer.position = CGPointMake(layerImage.size.width/2.0f, layerImage.size.height/2.0f);
[self.view.layer addSublayer:imageLayer];
```

####②使用delegate来提供layer的内容  

如果你的layer内容是动态变化的，你可以使用delegate独享来提供和更新你想要的内容。展示时，layer调取你的delegate方法来提供需要的内容  
1）如果你的delegate实现了diplayLayer：方法，这个实现响应创建一个位图和分配它到图层的content属性  
![Layer Subclass]({{ root_uri }}/images/core_animation_guid/core_animation_guid_2.png)   
2）如果你的协议实现了drawLayer：inContext：方法，Core Animation创建一个位图，创建一个graphics context来挥之位图，然后调取你的delegate方法来填充位图。所有你的协议方法会知道提供的graphics context  
![Layer Subclass]({{ root_uri }}/images/core_animation_guid/core_animation_guid_3.png)  
备注：简单的说displayLayer：方法，使用的内容是原本就有的 如图片等，而drawLayer：inContext：方法是在画布上画出一些内容。   
协议必须实现displayLayer 或者drawLayer：inContext方法，如果协议同时实现了这两个犯法，这个layer仅会调用displayer：方法。  
对于有自定义内容的layer-backed视图，你应该继续重写view的方法来实现你的绘制，layer-backed视图自动让他、他的layer的delegate 实现需要的delegate方法，你不能改变这种设置，而是应该实现你的view的drawRect：方法来绘制你的内容。  

##Layer内容动画  

Core Animation提供的基础框架使得创建一个Layer优雅的动画变得容易。使用Core Animation，仅改变一个属性你就可以创建动画，你也可以自己创建动画，明确的设置动画参数。  

####简单改变Layer属性动画  

你可以实现一个简单的动画，明确的或者不明确的。不明确的动画使用默认的时间和动画属性来完成动画，明确的动画需要自己来设置这些属性。所以简单的动画适用于你想使用默认时间、代码量少的动画。  

不明确的动画写法:  
```
theLayer.opacity = 0.0;
```
明确的动画写法:  
```
CABasicAnimation* fadeAnim = [CABasicAnimation animationWithKeyPath:@"opacity"];
fadeAnim.fromValue = [NSNumber numberWithFloat:1.0];
fadeAnim.toValue = [NSNumber numberWithFloat:0.0];
fadeAnim.duration = 1.0;
[theLayer addAnimation:fadeAnim forKey:@"opacity"];

theLayer.opacity = 0.0;//必须要写
```

####Keyframe Animation来改变Layer属性动画  
与CABasicAnimation（设置两个点 开始值、结束值）不同的是，一个CAKeyframeAnimation可以设置一系列的目标值。例如改变边框粗细动画：
```
CALayer *myLayer = [CALayer layer];
myLayer.bounds = CGRectMake(0, 0, 100, 100);
myLayer.position = CGPointMake(100, 200);

CAKeyframeAnimation* widthAnim = [CAKeyframeAnimation animationWithKeyPath:@"borderWidth"];
NSArray* widthValues = [NSArray arrayWithObjects:@1.0, @10.0, @5.0, @30.0, @0.5, @15.0, @2.0, @50.0, @0.0, nil];
widthAnim.values = widthValues;
widthAnim.calculationMode = kCAAnimationPaced;
widthAnim.duration = 10.0f;

[myLayer addAnimation:widthAnim forKey:@"BorderWidthChanges"];
[self.view.layer addSublayer:myLayer];
```
你也可以提供给路径给keyframeanimation，如 一个方块围着圆形路径旋转  
```
CALayer *myLayer = [CALayer layer];
myLayer.bounds = CGRectMake(0, 0, 100, 100);
myLayer.backgroundColor = [UIColor redColor].CGColor;

CAKeyframeAnimation *keyframeAnimation = [CAKeyframeAnimation animation];
keyframeAnimation.keyPath = @"position";
CGRect boundingRect = CGRectMake(10, 100, 300, 300);
keyframeAnimation.path = CFAutorelease(CGPathCreateWithEllipseInRect(boundingRect, NULL));//创建一个圆形路径
keyframeAnimation.duration = 4;
keyframeAnimation.repeatCount = HUGE_VALF;
keyframeAnimation.calculationMode = kCAAnimationPaced;

[myLayer addAnimation:keyframeAnimation forKey:@"EllipseKeyframeAnimation"];

[self.view.layer addSublayer:myLayer];
```
####执行一组动画  

如果你想执行对一个layer执行多个动画，那就要使用CAAnimationGroup对象,例如对一个layer既执行borderwidth改变又要改变bordercolor：   

```
// Animation 1
CAKeyframeAnimation* widthAnim = [CAKeyframeAnimation animationWithKeyPath:@"borderWidth"];
NSArray* widthValues = [NSArray arrayWithObjects:@1.0, @10.0, @5.0, @30.0, @0.5, @15.0, @2.0, @50.0, @0.0, nil];
widthAnim.values = widthValues;
widthAnim.calculationMode = kCAAnimationPaced;

// Animation 2
CAKeyframeAnimation* colorAnim = [CAKeyframeAnimation animationWithKeyPath:@"borderColor"];
NSArray* colorValues = [NSArray arrayWithObjects:(id)[UIColor greenColor].CGColor,
(id)[UIColor redColor].CGColor, (id)[UIColor blueColor].CGColor,  nil];
colorAnim.values = colorValues;
colorAnim.calculationMode = kCAAnimationPaced;

// Animation group
CAAnimationGroup* group = [CAAnimationGroup animation];
group.animations = [NSArray arrayWithObjects:colorAnim, widthAnim, nil];
group.duration = 5.0;

[myLayer addAnimation:group forKey:@"BorderChanges"];
```

####停止一个正在运行的明确动画
动画一般是完成时才停止，如果你要停止一个正在运行着的动画，你可以

调用removeAnimationForKey方法，key是addAnimation：forKey时候的key，他来指定动画唯一
如果想要layer上的所有动画，调用removeAllAnimation:方法

当你移除一个动画，Core Animation通过使用当前值来重画layer。因为当前值通常是动画的结束值，这样就会导致layer突然停止动画。如果你想让layer保持最后的动画frame，你可以使用layer对象的展示树来获取最终值，然后赋给对象的layer树

####监测动画的起始状态  

Core Animation 有两种不同的方式来通知 动画状态。  
1.使用setCompletionBlock:方法添加完成block到当前处理。当所有的动画处理完成是，这个处理执行你的完成block  
2.分配delegate到你的CAAnimation对象，实现animtaionDidStart：方法和animationDidStop：finished：协议方法  

##Core Animation类的继承关系图  

![Layer Subclass]({{ root_uri }}/images/core_animation_guid/core_animation_guid_5.png)  

参考文章：  
[苹果官方文档 Core Animation Guid](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1)  
[动画解释](http://objccn.io/issue-12-1/)  
[Core Animation 之基础介绍](http://blog.csdn.net/totogo2010/article/details/8604719)  