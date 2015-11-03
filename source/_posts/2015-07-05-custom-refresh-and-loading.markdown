---
layout: post
title: "自定义下拉刷新和上拉加载"
date: 2015-07-05 22:15:04 +0800
comments: true
categories: iOS
---

关于下拉刷新和上拉加载，项目中一直使用[MJRefresh](https://github.com/CoderMJLee/MJRefresh)（原先还用过[EGOTableViewPullRefresh](https://github.com/emreberge/EGOTableViewPullRefresh)，MJRefresh更好用些），今天就分析下如何用Swift来实现这个功能。


关于如何下拉刷新和上拉加载，认识到两点就可以了：

0.UIScrollViewDelegate

1.UIScrollView中的contentInset

直白一点说，就是在UIScrollView及其子类在下拉或者上拉到一定的偏移量时，设置contentInset，固定住UIScrollView，显示动画加载页面，数据到达之后再恢复contentInset。

UIScrollView很强大，除了上面说的，我们有时用它来显示放大缩小图片。

今天来自定义一个下拉刷新视图，并实现上拉加载功能。

前些天，一篇公众号（Pinapps,里面的文章挺好）文章里推荐了一个iPhone上的RSS阅读器[Unread](https://itunes.apple.com/cn/app/unread-rss-news-reader/id911364254?l=en&mt=8)，软件做的很棒，就把全部功能都买了下来（程序内购买）。其中里面的下拉刷新效果挺棒的，正好仿一下：  

![下拉刷新](/images/custom_refresh_and_loading/1.gif)

分析下这里面的效果：

一开始线条是隐藏的，有了偏移后，线条从原先的窄线条慢慢变成宽线条线条，其中线条的颜色和REFRESH颜色也是有变化的。为了能做出连续性的动画我们使用POP(如果不是很了解POP的话，可以看一下[我写的这篇文章](http://iyiming.me/blog/2015/06/28/pop-dong-hua/))

已传至Github（时间仓促，没有加以优化）：  

[https://github.com/iYiming/MyDemo/tree/master/Weibo](https://github.com/iYiming/MyDemo/tree/master/Weibo)

下面是代码的一些注释：

前提：添加POP动画，创建一个单视图项目，在ViewController里面添加一个UITableView。

先来实现刷新视图：即那三条红线加上REFRESH。

我们创建Swift文件，继承自UIView并命名为RefreshView：

代码如下：

```
import UIKit

@IBDesignable class RereshView: UIView {
    var topLayer: CALayer?//顶部线
    var middleLayer: CALayer?//中间线
    var bottomLayer: CALayer?//底部线

    var textLayer: CATextLayer?//REFRESH文字

    override init(frame: CGRect) {
        super.init(frame: frame)

        setup()//设置
    }

    required init(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        //fatalError("init(coder:) has not been implemented")

        setup()//设置
    }

    /**
    设置
    */
    func setup(){
        //顶部线
        topLayer = CALayer()
        topLayer!.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
        topLayer!.frame = CGRectMake(98, 0, 4, 4)
        topLayer!.cornerRadius = 2
        self.layer.addSublayer(topLayer)

        //中间线
        middleLayer = CALayer()
        middleLayer!.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
        middleLayer!.frame = CGRectMake(98, 14, 4, 4)
        middleLayer!.cornerRadius = 2
        self.layer.addSublayer(middleLayer)

        //底部线
        bottomLayer = CALayer()
        bottomLayer!.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
        bottomLayer!.frame = CGRectMake(98, 28, 4, 4)
        bottomLayer!.cornerRadius = 2
        self.layer.addSublayer(bottomLayer)

        //REFRESH
        textLayer = CATextLayer()
        textLayer!.foregroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor
        textLayer!.fontSize = 10
        textLayer!.contentsScale = UIScreen.mainScreen().scale
        textLayer!.string = "REFRESH"
        textLayer!.opacity = 0
        textLayer!.frame = CGRectMake(75, 62, 50, 20)
        self.layer.addSublayer(textLayer)
    }
}
```

注意我们在Class前面加了一个@IBDesignable，这样我们就能在Storyboard中查看我们自定义的视图（如果想详细了解的话，请看下Onevcat的[可视化开发，IB新时代](http://onevcat.com/2014/10/ib-customize-view/)）。

在ViewController.swift中我们添加了如下代码：

```
import UIKit

class ViewController: UIViewController,UITableViewDataSource,UIScrollViewDelegate {
    @IBOutlet weak var tableView: UITableView!//列表视图
    @IBOutlet weak var refreshView: RereshView!//下拉刷新视图
    @IBOutlet weak var refreshViewTopLayoutConstraint: NSLayoutConstraint!//下拉刷新居上的约束

    var showBottomLayer:Bool = true//显示刷新视图的底部线 变宽
    var hiddenBottomLayer:Bool = true//隐藏刷新视图的底部线 即变窄
    var showMiddleLayer:Bool = true//显示刷新视图的中间线 变宽
    var hiddenMiddleLayer:Bool = true//隐藏刷新视图的中间线 变窄
    var showTopLayer:Bool = true//显示刷新头部的底部线 变宽
    var hiddenTopLayer:Bool = true//隐藏刷新视图的头部线 变窄

    var refreshing = false//是否在刷新中

    var loadingAcitivityIndicatorView: UIActivityIndicatorView?//上拉加载指示菊花
    var showFooterLoadingView:Bool = false//是否显示上拉加载视图


    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.

        settingUI()//设置界面
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    //设置界面
    func settingUI(){
        let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)
        let screenHeight = CGRectGetHeight(UIScreen.mainScreen().bounds)

        loadingAcitivityIndicatorView = UIActivityIndicatorView(activityIndicatorStyle: UIActivityIndicatorViewStyle.Gray)
        loadingAcitivityIndicatorView?.hidesWhenStopped = true
        loadingAcitivityIndicatorView?.frame = CGRectMake((screenWidth - 20)/2.0,screenHeight - 40, 20, 20)
        self.view.addSubview(loadingAcitivityIndicatorView!)
    }

    /**
    添加size动画

    :param: layer   要添加动画的layer
    :param: size    动画到的size
    */
    func addSizeAnimation(layer: CALayer,size: CGSize){
        var springAnimation = POPSpringAnimation(propertyNamed: kPOPLayerSize)
        springAnimation.toValue = NSValue(CGSize: size);
        springAnimation.springBounciness = 18
        layer.pop_addAnimation(springAnimation, forKey: "layerSpringAnimation")
    }

    /**
    添加position动画

    :param: layer    要添加动画的layer
    :param: position 动画到的position
    */
    func addPositionAnimation(layer: CALayer,position: CGPoint){
        var springAnimation = POPSpringAnimation(propertyNamed: kPOPLayerPosition)
        springAnimation.toValue = NSValue(CGPoint: position);
        springAnimation.springBounciness = 18
        layer.pop_addAnimation(springAnimation, forKey: "layerSpringAnimation")
    }

    //UITableViewDataSource
    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 100
    }

    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        var cell = tableView.dequeueReusableCellWithIdentifier("Cell") as! UITableViewCell

        cell.textLabel?.text = "\(indexPath.row)"

        return cell
    }

    //UIScrollViewDelegate
    //将要开始减速时
    func scrollViewWillBeginDecelerating(scrollView: UIScrollView) {
        var contentOffsetY = scrollView.contentOffset.y

        if contentOffsetY > 0{
            return
        }

        if contentOffsetY < -52{//偏移 52 时 我们显示加载视图
            refreshing = true 
            scrollView.contentInset = UIEdgeInsetsMake(62.0, 0.0, 0.0, 0.0)
            refreshViewTopLayoutConstraint.constant = 20//刷新视图居上约束

            //移除POP动画
            refreshView.topLayer?.pop_removeAllAnimations()
            refreshView.middleLayer?.pop_removeAllAnimations()
            refreshView.bottomLayer?.pop_removeAllAnimations()

            //设定layer位置
            refreshView.topLayer?.frame = CGRectMake(121, 0, 4, 4)
            refreshView.middleLayer?.frame = CGRectMake(98, 14, 4, 4)
            refreshView.bottomLayer?.frame = CGRectMake(75, 28, 4, 4)
            refreshView.textLayer?.opacity = 0

            self.addPositionAnimation(self.refreshView.topLayer!,position: CGPointMake(121, 14))
            self.addPositionAnimation(self.refreshView.middleLayer!,position: CGPointMake(98, 14))
            self.addPositionAnimation(self.refreshView.bottomLayer!,position: CGPointMake(75, 14))


            let delayTime = dispatch_time(DISPATCH_TIME_NOW,Int64(1 * Double(NSEC_PER_SEC)))
            dispatch_after(delayTime,dispatch_get_main_queue()) {
                UIView.animateWithDuration(0.3, animations: { () -> Void in
                    scrollView.contentInset = UIEdgeInsetsMake(0.0, 0.0, 0.0, 0.0)
                    self.refreshViewTopLayoutConstraint.constant = -52
                }, completion: { (Bool) -> Void in
                    self.refreshing = false
                    self.refreshView.topLayer!.frame = CGRectMake(98, 0, 4, 4)
                    self.refreshView.middleLayer!.frame = CGRectMake(98, 14, 4, 4)
                    self.refreshView.bottomLayer!.frame = CGRectMake(98, 28, 4, 4)
                })
            }
        }
    }

    //滚动时
    func scrollViewDidScroll(scrollView: UIScrollView) {
        var contentOffsetY = scrollView.contentOffset.y

        if contentOffsetY <= 0 {
            if refreshing{
                return;
            }

        self.refreshView.hidden = false

        refreshViewTopLayoutConstraint.constant = -contentOffsetY - 52

        if contentOffsetY < -30{
            self.hiddenBottomLayer = true
            if showBottomLayer{
                showBottomLayer = false

                self.addSizeAnimation(refreshView.bottomLayer!, size: CGSizeMake(50, 4));

                refreshView.bottomLayer?.backgroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor
        }

        if contentOffsetY < -46{
            self.hiddenMiddleLayer = true
            if showMiddleLayer{
                showMiddleLayer = false

                self.addSizeAnimation(refreshView.middleLayer!, size: CGSizeMake(50, 4));

                refreshView.middleLayer?.backgroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor
            }


            if contentOffsetY < -62{
                self.hiddenTopLayer = true
                if showTopLayer{
                    showTopLayer = false

                    self.addSizeAnimation(refreshView.topLayer!, size: CGSizeMake(50, 4));

                    refreshView.topLayer?.backgroundColor = UIColor(red: 192/255.0, green: 47/255.0, blue: 46/255.0, alpha: 1.0).CGColor

                    refreshView.middleLayer?.backgroundColor = UIColor(red: 192/255.0, green: 47/255.0, blue: 46/255.0, alpha: 1.0).CGColor

                    refreshView.bottomLayer?.backgroundColor = UIColor(red: 192/255.0, green: 47/255.0, blue: 46/255.0, alpha: 1.0).CGColor

                    refreshView.textLayer?.foregroundColor = UIColor(red: 192/255.0, green: 47/255.0, blue: 46/255.0, alpha: 1.0).CGColor
                }
            }else{
                self.showTopLayer = true
                if hiddenTopLayer{
                    hiddenTopLayer = false

                    self.addSizeAnimation(refreshView.topLayer!, size: CGSizeMake(4, 4));

                    refreshView.topLayer?.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor

                    refreshView.middleLayer?.backgroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor

                    refreshView.bottomLayer?.backgroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor

                    refreshView.textLayer?.foregroundColor = UIColor(red: 64/255.0, green: 64/255.0, blue: 64/255.0, alpha: 1.0).CGColor
                }
            }

        }else{
            self.showMiddleLayer = true
            if hiddenMiddleLayer{
                hiddenMiddleLayer = false

                self.addSizeAnimation(refreshView.middleLayer!, size: CGSizeMake(4, 4));

                refreshView.middleLayer?.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
            }
        }
    }else{
            self.showBottomLayer = true
            if hiddenBottomLayer{
                hiddenBottomLayer = false

                self.addSizeAnimation(refreshView.bottomLayer!, size: CGSizeMake(4, 4));

                refreshView.bottomLayer?.backgroundColor = UIColor(red: 200/255.0, green: 200/255.0, blue: 200/255.0, alpha: 1.0).CGColor
            }
        }  

        refreshView.textLayer?.opacity = Float(-contentOffsetY/62.0)
    }else{

        self.refreshView.hidden = true

        let screenWidth = CGRectGetWidth(UIScreen.mainScreen().bounds)
    }

        if scrollView.contentOffset.y + scrollView.frame.size.height < scrollView.contentSize.height{
            loadingAcitivityIndicatorView?.stopAnimating()
            tableView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0);
            showFooterLoadingView = false
        }else{
            tableView.contentInset = UIEdgeInsetsMake(0, 0, 60, 0);
            loadingAcitivityIndicatorView?.startAnimating()
            showFooterLoadingView = true;

            //添加上拉加载数据代码
            //...
        }
    }
}
```

需要注明的是：

其中下拉刷新视图和上拉加载视图的父级视图是ViewController.view。



