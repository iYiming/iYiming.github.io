---
layout: post
title: "POP 动画"
date: 2015-06-28 18:09:39 +0800
comments: true
categories: iOS
---

前段时间看了些，动画方面的文章，有一个很不错的教程：[iOS-Core-Animation-Advanced-Techniques](https://github.com/AttackOnDobby/iOS-Core-Animation-Advanced-Techniques)，看会这个动画方面基本上差不多了，剩下的就是举一反三了。

疏于原先没有太过关注Facebook的[POP动画引擎](https://github.com/facebook/pop),做的效果这么好，于是就研究了下，找到了些几篇很好的文章：

POP使用指南：  

[http://geeklu.com/2014/05/facebook-pop-usage/](http://geeklu.com/2014/05/facebook-pop-usage/)

很好的POP示例教程:

[https://github.com/schneiderandre/popping](https://github.com/schneiderandre/popping)  
[https://github.com/kejinlu/facebook-pop-sample](https://github.com/kejinlu/facebook-pop-sample)

上面都是Objective-C写的，况且Swift出来已经一年了，由于公司项目一直使用OC，Swift也得加强学习，故以后做Demo就用Swift了。

OK，那就仿Popping,用Swift写一个POP动画：SwiftPopping。效果如下：

![POP](/images/POP/POP1.gif)

##0.桥接

POP动画引擎是用Objective-C写的，想要用在Swift项目中，需要用到桥接。  

开始桥接项目：前提是创建了SwiftPopping项目并且添加了POP。

第一步：创建桥接文件：SwiftPopping-Bridging-Header.h  
第二步：在SwiftPopping-Bridging-Header.h文件中写入：```#import <pop/POP.h>```  
第三步：详见下图：

![POP](/images/POP/POP2.jpg)

需要注意的地方就是1中 ```SwiftPopping-Bridging-Header.h``` 所在项目的位置。

OK,编译运行，成功！

如果想在Objective-C项目中调用Swift代码，可以参考下面的文章：  

[http://www.shimingwei.com/iOS/OC_Use_Swift.html](http://www.shimingwei.com/iOS/OC_Use_Swift.html)

##1.创建按钮FlatButton

由于我们需要自定义UIButton的一些效果（点进去缩放、出来还原效果），故创建一个FlatButton，代码如下：

```
import UIKit

class FlatButton: UIButton {
    //标题居上下左右间距
    override var titleEdgeInsets: UIEdgeInsets{
        get{
            return UIEdgeInsetsMake(4.0,28.0,4.0,28.0)
        }
        set{
            self.titleEdgeInsets = newValue
        }
    }

    class func button() -> FlatButton{
        var button =  buttonWithType(UIButtonType.Custom) as! FlatButton
        return button
    }

    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()//设置按钮
    }

    required init(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }

    //内容大小
    override func intrinsicContentSize() -> CGSize {
        var size = super.intrinsicContentSize()

        return CGSizeMake(size.width + self.titleEdgeInsets.left + self.titleEdgeInsets.right,
                size.height + self.titleEdgeInsets.top + self.titleEdgeInsets.bottom)
    }

    /**
    设置按钮
    */
    func setup(){
        self.backgroundColor = self.tintColor;
        self.layer.cornerRadius = 4.0
        self.setTitleColor(UIColor.whiteColor(), forState: UIControlState.Normal)
        self.titleLabel?.font = UIFont(name: "Avenir-Medium", size: 22)

        //添加事件
        self.addTarget(self, action: "scaleToSmall", forControlEvents: UIControlEvents.TouchDown | UIControlEvents.TouchDragEnter)
        self.addTarget(self, action: "scaleAnimation", forControlEvents: UIControlEvents.TouchUpInside)
        self.addTarget(self, action: "scaleToDefault", forControlEvents: UIControlEvents.TouchDragExit)
    }

    //缩小
    func scaleToSmall(){
        var scaleAnimation = POPBasicAnimation(propertyNamed: kPOPLayerScaleXY)
        scaleAnimation.toValue = NSValue(CGPoint:CGPointMake(0.95, 0.95))

        self.layer.pop_addAnimation(scaleAnimation, forKey: "scaleToSmallAnimation")
    }

    //还原
    func scaleToDefault(){
        var scaleAnimation = POPBasicAnimation(propertyNamed: kPOPLayerScaleXY)
        scaleAnimation.toValue = NSValue(CGPoint:CGPointMake(1, 1))
        
        self.layer.pop_addAnimation(scaleAnimation, forKey: "scaleToDefaultAnimation")
    }

    //动画
    func scaleAnimation(){
        var scaleAnimation = POPSpringAnimation(propertyNamed: kPOPLayerScaleXY)
        scaleAnimation.toValue = NSValue(CGPoint: CGPointMake(1.0, 1.0))
        scaleAnimation.velocity = NSValue(CGPoint:CGPointMake(3.0, 3.0))
        scaleAnimation.springBounciness = 18.0
        
        self.layer.pop_addAnimation(scaleAnimation, forKey: "scaleAnimation")
    }
}

```

##2.创建ButtonViewController

用于实现我们Gif图的效果。

```
import UIKit

class ButtonViewController: UIViewController {
    var flatButton = FlatButton.button()//按钮
    var errorLabel = UILabel()//错误标签
    var activityIndicatorView = UIActivityIndicatorView(activityIndicatorStyle: UIActivityIndicatorViewStyle.Gray)//导航栏右侧Item

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.

        addButton()//添加按钮
        addLabel()//添加标签
        addActivityIndicatorView()//添加活动指示视图
    }

    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)

        errorLabel.layer.transform = CATransform3DMakeScale(0.5, 0.5, 1)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    /**
    添加按钮
    */
    func addButton(){
        flatButton.backgroundColor = UIColor(red: 52/255.0, green: 152/255.0, blue: 219/255.0, alpha: 1.0)
        flatButton.setTitle("Log in", forState: UIControlState.Normal)
        flatButton.setTranslatesAutoresizingMaskIntoConstraints(false)
        flatButton.addTarget(self, action: "touchUpInside:", forControlEvents: UIControlEvents.TouchUpInside)
        view.addSubview(flatButton)

        //添加约束
        self.view.addConstraint(NSLayoutConstraint(item: flatButton, attribute: NSLayoutAttribute.CenterX, relatedBy: NSLayoutRelation.Equal, toItem: view, attribute: NSLayoutAttribute.CenterX, multiplier: 1, constant: 0.0))

        self.view.addConstraint(NSLayoutConstraint(item: flatButton, attribute: NSLayoutAttribute.CenterY, relatedBy: NSLayoutRelation.Equal, toItem: view, attribute: NSLayoutAttribute.CenterY, multiplier: 1, constant: 0.0))
    }

    /**
    添加标签
    */
    func addLabel(){
        errorLabel.font = UIFont(name: "Avenir-Light", size: 18)
        errorLabel.textColor = UIColor(red: 231/255.0, green: 76/255.0, blue: 60/255.0, alpha: 1.0)
        errorLabel.setTranslatesAutoresizingMaskIntoConstraints(false)
        errorLabel.text = "Just a serious login error."
        errorLabel.textAlignment = NSTextAlignment.Center
        view.insertSubview(errorLabel, belowSubview: flatButton)

        //添加约束
        self.view.addConstraint(NSLayoutConstraint(item: errorLabel, attribute: NSLayoutAttribute.CenterX, relatedBy: NSLayoutRelation.Equal, toItem: flatButton, attribute: NSLayoutAttribute.CenterX, multiplier: 1, constant: 0.0))

        self.view.addConstraint(NSLayoutConstraint(item: errorLabel, attribute: NSLayoutAttribute.CenterY, relatedBy: NSLayoutRelation.Equal, toItem: flatButton, attribute: NSLayoutAttribute.CenterY, multiplier: 1, constant: 0.0))

        errorLabel.layer.transform = CATransform3DMakeScale(0.5, 0.5, 1)
    }

    /**
    添加活动指示图
    */
    func addActivityIndicatorView(){
        activityIndicatorView.hidesWhenStopped = true
        self.navigationItem.rightBarButtonItem = UIBarButtonItem(customView: activityIndicatorView)
    }

    /**
    按钮点击时

    :param: button 按钮
    */
    func touchUpInside(button: FlatButton){
        button.userInteractionEnabled = false
        activityIndicatorView.startAnimating()
        hiddenErrorLabel()

        let delayTime = dispatch_time(DISPATCH_TIME_NOW,
        Int64(1 * Double(NSEC_PER_SEC)))
        dispatch_after(delayTime, dispatch_get_main_queue()) {
            self.shakeFlatButton()
            self.showErrorLabel()
            self.activityIndicatorView.stopAnimating()
        }
    }

    /**
    摇晃按钮
    */
    func shakeFlatButton(){
        var shakeSpringAnimation = POPSpringAnimation(propertyNamed: kPOPLayerPositionX)
        shakeSpringAnimation.velocity = NSNumber(float: 2000.0)
        shakeSpringAnimation.springBounciness = 20.0
        shakeSpringAnimation.completionBlock = {(animation, finished) in
            self.flatButton.userInteractionEnabled = true
        }

        flatButton.layer.pop_addAnimation(shakeSpringAnimation, forKey: "shakeSpringAnimation")
    }

    /**
    显示错误提示标签
    */
    func showErrorLabel(){
        //缩放
        var scaleSpringAnimation = POPSpringAnimation(propertyNamed: kPOPLayerScaleXY)
        scaleSpringAnimation.toValue = NSValue(CGPoint: CGPointMake(1, 1))
        scaleSpringAnimation.velocity = NSValue(CGPoint: CGPointMake(3, 3))
        scaleSpringAnimation.springBounciness = 20.0

        errorLabel.layer.pop_addAnimation(scaleSpringAnimation, forKey: "scaleSpringAnimation")

        //position Y
        var positionSpringAnimation = POPSpringAnimation(propertyNamed: kPOPLayerPositionY)
        positionSpringAnimation.toValue = NSNumber(float: Float(flatButton.layer.position.y + flatButton.intrinsicContentSize().height))
        positionSpringAnimation.springBounciness = 20.0

        errorLabel.layer.pop_addAnimation(positionSpringAnimation, forKey: "positionSpringAnimation")
    }

    /**
    隐藏错误提示标签
    */
    func hiddenErrorLabel(){
        //缩放
        var scaleSpringAnimation = POPSpringAnimation(propertyNamed: kPOPLayerScaleXY)
        scaleSpringAnimation.toValue = NSValue(CGPoint: CGPointMake(0.5, 0.5))
        scaleSpringAnimation.velocity = NSValue(CGPoint: CGPointMake(3, 3))
        scaleSpringAnimation.springBounciness = 20.0

        errorLabel.layer.pop_addAnimation(scaleSpringAnimation, forKey: "scaleSpringAnimation")

        //position Y
        var positionSpringAnimation = POPSpringAnimation(propertyNamed: kPOPLayerPositionY)
        positionSpringAnimation.toValue = NSNumber(float: Float(flatButton.layer.position.y))
        positionSpringAnimation.springBounciness = 20.0

        errorLabel.layer.pop_addAnimation(positionSpringAnimation, forKey: "positionSpringAnimation")
    }
}
```


