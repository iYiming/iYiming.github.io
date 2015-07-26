---
layout: post
title: "自定义下拉刷新（二）"
date: 2015-07-22 20:26:20 +0800
comments: true
categories: iOS
---

上篇关于下拉刷新的文章代码敲得太烂，决定重敲一遍。

实现的效果如下：

![](/images/custom_refresh_2/1.gif)

##下拉刷新原理

其实原理很简单：  

>0.UIScrollView的contentInset属性：用于保持停留  
>1.UIScrollView的contentOffset属性：获取偏移量，用于控制相应偏移量下，刷新视图如何改变


##怎样方便重用

如何获取偏移量？  

有两种方式可以获得:

>0.在含有UIScrollView（及其子类）控制器里面使用UIScrollViewDelegate，从而获取偏移量。  

>1.使用强大的KVO啊，通过监听contentOffset，很容易的就得到了偏移量。

但是如果要在多个控制器中使用下拉刷新，那么每个控制器都实现UIScrollViewDelegate或KVO，繁琐且臃肿。

使用UIScrollViewDelegate的方式肯定行不通(通过继承的方式？？像UITableViewController？我没有想到好的解决办法)，因为每个控制器都要实现。故UIScrollViewDelegate的方式被pass掉了。

我们继续考虑使用KVO，KVO监听的是ScrollView及其子类，如果每个控制器都写监听的话也是和UIScrollViewDelegate类似，但是我们可以不写啊。

滚动视图（UIScrollView及其子类）和下拉刷新视图是一块的，有刷新的地方就有下拉刷新视图。我们把下拉刷新视图添加在滚动视图上（UIScrollView及其子类），获得到刷新视图就可以获得滚动视图啦。这样我们把监听写在刷新视图里，就不用到处写了。

还有个问题，我们如何给滚动视图添加下拉刷新视图呢？有没有简单的方式给滚动视图添加下拉刷新视图。我们可以使用扩展，添加一个下拉刷新视图属性，只要我们给这个下拉刷新视图赋值，我们就把他添加到滚动视图上。这样就很简洁啦。

其实上面的这些东西是我读[MJRefresh代码](https://github.com/CoderMJLee/MJRefresh)发现的，受益匪浅。

##实现效果中的三个动画

动画是这个下拉刷新最大的亮点，也是它吸引我，我才决定仿这么一个动画(仿的是RSS阅读器[Unread](https://itunes.apple.com/cn/app/unread-rss-news-reader/id911364254?l=en&mt=8))。

###0.展开、关闭动画
原先是使用POP实现的，这次使用多次动画的方式。代码如下，

```
/**
展开动画

:param: animationLayer 动画Layer
*/
func spreadLayerAnimation(animationLayer: CAShapeLayer){
    layerWidthAnimation(animationLayer, width: 100, animationTypeStr: "Spread", animationIndexStr: "1")
}

/**
关闭动画

:param: animationLayer 关闭Layer
*/
func closeLayerAnimation(animationLayer: CAShapeLayer){
    layerWidthAnimation(animationLayer, width: 0, animationTypeStr: "Close", animationIndexStr: "1")
}

/**
宽度动画

:param: animationLayer    动画层
:param: width             宽度
:param: animationTypeStr  动画类型 展开或关闭
:param: animationIndexStr 动画索引
*/
func layerWidthAnimation(animationLayer: CAShapeLayer,width: CGFloat,animationTypeStr: String,animationIndexStr: String){
    var animationLayerStr = "shapeLayer1"
    if animationLayer.isEqual(shapeLayer1){
        animationLayerStr = "shapeLayer1"
    }

    let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)

    var animation = CABasicAnimation(keyPath: "bounds.size.width")
    animation.duration = 0.13
    animation.delegate = self
    animation.toValue = NSNumber(float: 20)
    animation.setValue(animationLayerStr, forKey: "AnimationLayer")
    animation.setValue(animationTypeStr, forKey: "AnimationType")
    animation.setValue(animationIndexStr, forKey: "AnimationIndex")
    animation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
    animationLayer.addAnimation(animation, forKey: "animation")
    animationLayer.frame = CGRect(x: (screenWidth - width)/2.0, y: animationLayer.frame.origin.y, width: CGFloat(width), height: animationLayer.frame.size.height)
}

//MARK 动画结束
override func animationDidStop(anim: CAAnimation!, finished flag: Bool) {
    let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)

    let animationLayerStr = anim.valueForKey("AnimationLayer") as! String
    let animationTypeStr = anim.valueForKey("AnimationType") as! String
    let animationIndexStr = anim.valueForKey("AnimationIndex") as! String

    var shapeLayer = CAShapeLayer()
    if animationLayerStr == "shapeLayer1"{
        shapeLayer = shapeLayer1;
    }

    var width:CGFloat = 0
    if animationIndexStr == "1"{
        if animationTypeStr == "Spread"{
            width = 20
        }else if animationTypeStr == "Close"{
            width = 20
        }
    }else if animationIndexStr == "2"{
        if animationTypeStr == "Spread"{
            width = 80
        }else if animationTypeStr == "Close"{
            width = 4
        }
    }else if animationIndexStr == "3"{
        if animationTypeStr == "Spread"{
            width = 40
        }else if animationTypeStr == "Close"{
            return
        }
    }else if animationIndexStr == "4"{
        if animationTypeStr == "Spread"{
            width = 60
        }
    }else{
        return
    }

    layerWidthAnimation(shapeLayer, width: width, animationTypeStr: animationTypeStr, animationIndexStr:String(animationIndexStr.toInt()! + 1))
}
```


###1.旋转动画

其实就是沿着一定的路径进行旋转，使用CAKeyframeAnimation的path属性按照响应的贝塞尔曲线旋转。如下图：  

/images/custom_refresh_2/2.gif

```
//设置界面
func settingUI(){
    let screenWidth = CGRectGetWidth( UIScreen.mainScreen().bounds)

    layer1.backgroundColor = UIColor.redColor().CGColor
    layer1.frame = CGRect(x: (screenWidth - 8)/2.0, y: 100, width: 8, height: 8)
    layer1.cornerRadius = 4
    view.layer.addSublayer(layer1)


    layer2.backgroundColor = UIColor.redColor().CGColor
    layer2.frame = CGRect(x: (screenWidth - 8)/2.0, y: 120, width: 8, height: 8)
    layer2.cornerRadius = 4
    view.layer.addSublayer(layer2)


    layer3.backgroundColor = UIColor.redColor().CGColor
    layer3.frame = CGRect(x: (screenWidth - 8)/2.0, y: 140, width: 8, height: 8)
    layer3.cornerRadius = 4
    view.layer.addSublayer(layer3)
}

@IBAction func startAnimation(sender: AnyObject) {
    let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)
    CGPoint(x:(screenWidth - 8)/2.0 + 4, y: 124.0)

    var animation1 = CAKeyframeAnimation(keyPath: "position")
    animation1.delegate = self
    animation1.duration = CFTimeInterval(0.3)
    animation1.path = UIBezierPath(arcCenter: CGPoint(x:(screenWidth - 8)/2.0 + 4, y: 124.0),radius: CGFloat(58), startAngle:CGFloat(-M_PI_2), endAngle: CGFloat(0), clockwise: true).CGPath
    animation1.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
    layer1.addAnimation(animation1, forKey: "groupAnimation")
    CATransaction.begin()
    CATransaction.setDisableActions(true)
    layer1.position = CGPoint(x: (screenWidth - 8)/2.0 + 58 + 4, y: 124)
    CATransaction.commit()

    var animation3 = CAKeyframeAnimation(keyPath: "position")
    animation3.delegate = self
    animation3.duration = CFTimeInterval(0.3)
    animation3.path = UIBezierPath(arcCenter: CGPoint(x:(screenWidth - 8)/2.0 + 4, y: 124.0),radius: CGFloat(58), startAngle:CGFloat(M_PI_2), endAngle: CGFloat(M_PI), clockwise: true).CGPath
    animation3.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
    layer3.addAnimation(animation3, forKey: "groupAnimation")
    CATransaction.begin()
    CATransaction.setDisableActions(true)
    layer3.position = CGPoint(x: (screenWidth - 8)/2.0 - 58 + 4, y: 124)
    CATransaction.commit()
}
```


###2.按顺序放大、改变颜色动画

有三个Layer，第一个Layer动画（放大、改变颜色）结束后（使用animationDidStop协议方法），开始第二个动画然后依次类推。如下图：

![](/images/custom_refresh_2/3.gif)

```
var circleShapeLayer1 = CAShapeLayer()
var circleShapeLayer2 = CAShapeLayer()
var circleShapeLayer3 = CAShapeLayer()

/**
设置界面
*/
func settingUI(){
    var shapeWidth:CGFloat = 10
    let bezierCGPath =  UIBezierPath(ovalInRect: CGRect(x: 0, y: 0, width: shapeWidth, height: shapeWidth)).CGPath

    let fillCGColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor

    let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)
    var marginLeft = CGFloat((screenWidth - 70)/2.0)

    circleShapeLayer1.path = bezierCGPath
    circleShapeLayer1.fillColor = fillCGColor

    circleShapeLayer1.frame = CGRect(x: marginLeft, y: 100, width: shapeWidth, height: shapeWidth)
    view.layer.addSublayer(circleShapeLayer1)

    circleShapeLayer2.path = bezierCGPath
    circleShapeLayer2.fillColor = fillCGColor
    circleShapeLayer2.frame = CGRect(x: marginLeft + 30, y: 100, width: shapeWidth, height: shapeWidth)
    view.layer.addSublayer(circleShapeLayer2)

    circleShapeLayer3.path = bezierCGPath
    circleShapeLayer3.fillColor = fillCGColor
    circleShapeLayer3.frame = CGRect(x: marginLeft + 60, y: 100, width: shapeWidth, height: shapeWidth)
    view.layer.addSublayer(circleShapeLayer3)
}

/**
添加图层动画

:param: layer       图层
:param: layerTagStr 图层Tag
*/
func addLayerAnimation(layer: CAShapeLayer,layerTagStr: String){
    var scaleKeyframeAnimation = CAKeyframeAnimation(keyPath: "transform")

    var t1 = CATransform3DMakeScale(1.0, 1.0, 0)
    var t2 = CATransform3DMakeScale(1.5, 1.5, 0)
    var t3 = CATransform3DMakeScale(1.0, 1.0, 0)

    scaleKeyframeAnimation.values = [NSValue(CATransform3D:t1),NSValue(CATransform3D:t2),NSValue(CATransform3D:t3)]
    scaleKeyframeAnimation.keyTimes = [0,0.5,1]

    let normalCGColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
    var highlightedColor = UIColor(red: 60/255.0, green: 60/255.0, blue: 60/255.0, alpha: 1.0).CGColor
    var colorKeyframeAnimation = CAKeyframeAnimation(keyPath: "fillColor")
    colorKeyframeAnimation.values = [normalCGColor,highlightedColor,normalCGColor]
    colorKeyframeAnimation.keyTimes = [0,0.5,1]

    var groupAnimation = CAAnimationGroup()
    groupAnimation.duration = CFTimeInterval(0.35)
    groupAnimation.animations = [scaleKeyframeAnimation,colorKeyframeAnimation]
    groupAnimation.removedOnCompletion = true
    groupAnimation.setValue(layerTagStr, forKey: "LayerTag")
    groupAnimation.delegate = self

    layer.addAnimation(groupAnimation, forKey: "scaleKeyframeAnimation")
}

//开始动画
@IBAction func startAnimationButtonClicked(sender: AnyObject) {
    addLayerAnimation(circleShapeLayer1,layerTagStr: "1")
}

//动画结束协议
override func animationDidStop(anim: CAAnimation!, finished flag: Bool) {
    var  layerTagStr = anim.valueForKey("LayerTag") as! String

    if layerTagStr == "1"{
        addLayerAnimation(circleShapeLayer2,layerTagStr: "2")
    }else if layerTagStr == "2"{
        addLayerAnimation(circleShapeLayer3,layerTagStr: "3")
    }else if layerTagStr == "3"{
        addLayerAnimation(circleShapeLayer1,layerTagStr: "1")
    }
}

```

代码已传至GitHub：[https://github.com/iYiming/YMRefreshView](https://github.com/iYiming/YMRefreshView)