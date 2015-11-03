---
layout: post
title: "iOS7之转场动画"
date: 2015-05-31 19:54:24 +0800
comments: true
categories: iOS
---

iOS7添加了转场动画，让我们更加方便的切换控制器。

iOS7 SDK支持两种自定义过渡：**自定义视图控制器过渡**和**交互式视图控制器过渡**。

怎么来区分自定义视图控制器过渡和交互式试图控制器过渡呢？

**当过渡是时间的函数式，它通常是一个自定义视图控制器过渡，如果他是一个手势识别器或者其他类似事件的函数，它通常是一个交互式视图控制器过渡**。

举个例子：  
UINavigationController过渡使我们最常见的自定义视图控制器过渡。UIPageViewController过渡不是随着时间进行，它需要跟随我们的手指滑动，所以它是一种交互式视图控制器过渡。


##0.自定义视图控制器过渡

###0.使用Storyboard来实现自定义视图控制器过渡

我们可以使用连线（segue）的方式，来实现自定义视图控制器过渡，segue有好几种类型，
选择某个segue，我们可以看到下图：  

![0.png](/images/TransitionAnimation/0.png)

我们选择自定义这种方式可以看到：

![3.png](/images/TransitionAnimation/3.png)  

我们可以指定segue Class 它是一个UIStoryboardSegue子类，我们可以创建一个UIStoryboardSegue子类IYMCustomSegue，填写在这里。

在UIStoryboardSegue里有个perform方法，我们在IYMCustomSegue重写一下：

```
- (void) perfrom{
//这里实现动画
...
}
```

下面实现一个自定义push动画：

```
- (void) perform
{
UIViewController *sourceViewController = self.sourceViewController;
UIViewController *desinationViewController = self.destinationViewController;

CGRect sourceRect = sourceViewController.view.frame;
CGRect desinationViewFromRect = CGRectMake(sourceRect.origin.x + sourceRect.size.width, sourceRect.origin.y, sourceRect.size.width, sourceRect.size.height);
desinationViewController.view.frame = desinationViewFromRect;
desinationViewController.view.alpha = 0;
[sourceViewController.view.superview addSubview:desinationViewController.view];

[UIView animateWithDuration:0.3 animations:^{
sourceViewController.view.alpha = 0.5;
desinationViewController.view.alpha = 1;
desinationViewController.view.frame = sourceRect;
} completion:^(BOOL finished) {
sourceViewController.view.alpha = 1;
[desinationViewController.view removeFromSuperview];

[sourceViewController.navigationController pushViewController:desinationViewController animated:NO];
}];
}
```

###1.代码方式实现

自定义试图控制器过渡是由两个协议实现的，即UIViewControllerTransitioningDelegate 和 UIViewControllerAnimatedTransitioning。

我们从A控制器到B控制器，通常有两种方式：一种是presentViewController方式，另一种是pushViewController。

####0.使用presentViewController方式:从A控制器present到B控制器

>设置B控制器的transitionDelegate为A控制器  
>在A控制器中实现UIViewControllerTransitioningDelegate协议  
>实现UIViewControllerAnimatedTransitioning  

实现UIViewControllerTransitioningDelegate中的方法：  

```
- (id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source;

- (id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed;

```

这两个方法返回的是一个实现了UIViewControllerAnimatedTransitioning协议的类实例。通过方法名字我们就可以看到它们适用于presentViewController方式。如果自定义过渡效果只有一个视图控制器使用，视图控制器本身就可以实现UIViewControllerAnimatedTransitioning协议，否则，就需要一个单独的继承自NSObject一个单独的类来实现UIViewControllerAnimatedTransitioning协议。

这里我们就实现一个B视图控制器过渡动画，就直接在A控制器中实现UIViewControllerAnimatedTransitioning。

实现UIViewControllerTransitioningDelegate中的方法：

```
// This is used for percent driven interactive transitions, as well as for container controllers that have companion animations that might need to
// synchronize with the main animation. 
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext{
//返回动画时间
return 1.0f;
}

// This method can only  be a nop if the transition is interactive and not a percentDriven interactive transition.
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext{
//动画代码在这里写
...

[transitionContext completeTransition:YES];//必须实现这个方法
}
```

``- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext```中的transitionContext，我们可以获得：

源控制器即来自哪个视图控制器：

```
UIViewController *src = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
```

目标控制器，这里是B控制器:  

```
UIViewController *dest = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
```
容器视图：

```
UIView *containerView = [transitionContext containerView];
```
####1.使用pushViewController方式:从A控制器push到B控制器

因为是从A控制器push到B控制器，我们不需要实现UIViewControllerTransitioningDelegate中的方法，我们需要：

>0.在A控制器设置UINavigationDelegate 为 self  
>1.实现UINavigationDelegate  
>2.实现UIViewControllerAnimatedTransitioning协议  

实现UINavigationDelegate中的方法：  

```
- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
animationControllerForOperation:(UINavigationControllerOperation)operation
fromViewController:(UIViewController *)fromVC
toViewController:(UIViewController *)toVC;
```

返回的是一个实现了UIViewControllerAnimatedTransitioning协议的类实例。这个和presentViewController方式一样。

##1.交互式视图控制器过渡

要实现一个交互式视图控制器过渡，除了要是上面的动画外，还要告诉交互控制器动画完成了多少，我们只要知道完成了多少百分比，告诉系统，系统就直接给咱们处理了。

通过计算手势/动作事件或其他驱动过渡的事件，可以得出百分比。例如通过UIScreenEdgePanGestureRecognizer边缘返回手势，我们可以通过平移量来查看返回完成了多少。

还是要分成两种，一种是presentViewController方式，另一种是pushViewController。

####0.使用presentViewController方式

除了上面presentViewController方式实现的协议方法外，我们还需要实现UIViewControllerTransitioningDelegate中的另外两个方法：

```
- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForPresentation:(id <UIViewControllerAnimatedTransitioning>)animator;

- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForDismissal:(id <UIViewControllerAnimatedTransitioning>)animator;
```

交互控制器实现了UIViewControllerInteractiveTransitioning协议。交互式过渡的交互控制器都必须是UIPercentDrivenInteractiveTransition实例或者UIPercentDrivenInteractiveTransition子类实例。

UIPercentDrivenInteractiveTransition中以下方法可以完成交互过渡：

```
- (void)updateInteractiveTransition:(CGFloat)percentComplete;//完成了多少，percentComplete是百分比
- (void)cancelInteractiveTransition;//取消
- (void)finishInteractiveTransition;//完成
```

例如我们给B控制器（A控制器 present to B控制器），B控制器通过缩放到达A控制器，我们给B控制器视图添加了一个缩放手势UIPinchGestureRecognizer。

上面的UIViewControllerTransitioningDelegate我们返回一个 [[UIPercentDrivenInteractiveTransition alloc] init]实例percentDrivenInteractiveTransition。


```
-(void) pinchGestureAction:(UIPinchGestureRecognizer*) gestureRecognizer {

CGFloat scale = gestureRecognizer.scale;
if(gestureRecognizer.state == UIGestureRecognizerStateBegan) {
self.percentDrivenInteractiveTransition = [[UIPercentDrivenInteractiveTransition alloc] init];
self.startScale = scale;
[self.controller dismissViewControllerAnimated:YES completion:nil];
}
if(gestureRecognizer.state == UIGestureRecognizerStateChanged) {

CGFloat completePercent = 1.0 - (scale/self.startScale);
[self updateInteractiveTransition:completePercent];//完成了多少百分比
}
if(gestureRecognizer.state == UIGestureRecognizerStateEnded) {

if(gestureRecognizer.velocity >= 0){
[self cancelInteractiveTransition];//取消
}else{
[self finishInteractiveTransition];//完成
}

self.percentDrivenInteractiveTransition = nil;
}

if(gestureRecognizer.state == UIGestureRecognizerStateCancelled) {
[self cancelInteractiveTransition];//取消
self.percentDrivenInteractiveTransition = nil;
}
}
```

####1.使用pushViewController方式

和present方式不同的一点是，我们使用UINavigationDelegate中的：  

```
- (id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController
```
来代替：

```
- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForPresentation:(id <UIViewControllerAnimatedTransitioning>)animator;

- (id <UIViewControllerInteractiveTransitioning>)interactionControllerForDismissal:(id <UIViewControllerAnimatedTransitioning>)animator;
```

其他的都一样。

##2.实现一个完整的例子

单击图片展示大图，返回慢慢缩放到小图，效果如下:  

![1.gif](/images/TransitionAnimation/1.gif)

将用到：  
0.UINavigationDelegate  
1.返回手势  
2.交互式过渡效果  

###0.新建两个控制器
首页：IYMFirstViewController
喵星人：IYMSecondViewController

###1.使用Storyboard创建两个视图
![2.png](/images/TransitionAnimation/2.png)

###0.设置UINavigationDelegate

IYMFirstViewController.m IYMSecondViewController.m分别重写以下方法：

```
- (void)viewDidAppear:(BOOL)animated {
[super viewDidAppear:animated];

// Set outself as the navigation controller's delegate so we're asked for a transitioning object
self.navigationController.delegate = self;
}

- (void)viewWillDisappear:(BOOL)animated {
[super viewWillDisappear:animated];

// Stop being the navigation controller's delegate
if (self.navigationController.delegate == self) {
self.navigationController.delegate = nil;
}
}
```

###1.实现UINavigationDelegate

IYMFirstViewController.m中：

```
- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
animationControllerForOperation:(UINavigationControllerOperation)operation
fromViewController:(UIViewController *)fromVC
toViewController:(UIViewController *)toVC{
// Check if we're transitioning from this view controller to a DSLSecondViewController
if (fromVC == self && [toVC isKindOfClass:[IYMSecondViewController class]]) {
return self;
}else {
return nil;
}
}
```

IYMSecondViewController.m

```
- (id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController{
return _percentDrivenInteractiveTransition;
}

- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
animationControllerForOperation:(UINavigationControllerOperation)operation
fromViewController:(UIViewController *)fromVC
toViewController:(UIViewController *)toVC{
if (fromVC == self && [toVC isKindOfClass:[IYMFirstViewController class]]) {
return self;
}else {
return nil;
}
}
```

其中_percentDrivenInteractiveTransition是一个UIPercentDrivenInteractiveTransition实例，在边缘手势中实例化。

###2.实现UIViewControllerAnimatedTransitioning

IYMFirstViewController.m

```
- (NSTimeInterval) transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext
{
return 0.3f;//动画时间
}

- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
{
IYMFirstViewController *fromViewContrroller = (IYMFirstViewController *)[transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
IYMSecondViewController *toViewController = (IYMSecondViewController *)[transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
UIView *containerView = [transitionContext containerView];

UIView *snapshotView = [fromViewContrroller.bottomImageView snapshotViewAfterScreenUpdates:NO];//截图
CGRect snapshotViewFromRect = [containerView convertRect:fromViewContrroller.bottomImageView.frame fromView:fromViewContrroller.view];//获取在ContainerView中的frame
snapshotView.frame = snapshotViewFromRect; 
toViewController.view.frame = [transitionContext finalFrameForViewController:toViewController];
toViewController.view.alpha = 0;
[containerView addSubview:toViewController.view];
[containerView addSubview:snapshotView];
fromViewContrroller.bottomImageView.hidden = YES;
toViewController.topImageView.hidden = YES;

CGRect snapshotViewToRect = [containerView convertRect:toViewController.topImageView.frame fromView:toViewController.view];
[UIView animateWithDuration:0.3 animations:^{
toViewController.view.alpha = 1;
snapshotView.frame = snapshotViewToRect;
} completion:^(BOOL finished) {
if (finished) {
[snapshotView removeFromSuperview];
fromViewContrroller.bottomImageView.hidden = NO;
toViewController.topImageView.hidden = NO;

[transitionContext completeTransition:!transitionContext.transitionWasCancelled];
}
}];
}
```


IYMSecondViewController.m

```
- (NSTimeInterval) transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext
{
return 0.3f;
}


- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
{
IYMSecondViewController *fromViewController = (IYMSecondViewController *)[transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
IYMFirstViewController *toViewController = (IYMFirstViewController *)[transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
UIView *containerView = [transitionContext containerView];


UIView *snapshotView = [fromViewController.topImageView snapshotViewAfterScreenUpdates:NO];
CGRect snapshotViewFromRect = [containerView convertRect:fromViewController.topImageView.frame fromView:fromViewController.view];
snapshotView.frame = snapshotViewFromRect; // Setup the initial view states
toViewController.view.frame = [transitionContext finalFrameForViewController:toViewController];
toViewController.view.alpha = 0;
[containerView insertSubview:toViewController.view belowSubview:fromViewController.view];
[containerView addSubview:snapshotView];
fromViewController.topImageView.hidden = YES;
toViewController.bottomImageView.hidden = YES;

CGRect snapshotViewToRect = [containerView convertRect:toViewController.bottomImageView.frame fromView:toViewController.view];
[UIView animateWithDuration:0.3 animations:^{
toViewController.view.alpha = 1;
snapshotView.frame = snapshotViewToRect;
} completion:^(BOOL finished) {
if (finished) {
[snapshotView removeFromSuperview];
fromViewController.topImageView.hidden = NO;
toViewController.bottomImageView.hidden = NO;
// Declare that we've finished
[transitionContext completeTransition:!transitionContext.transitionWasCancelled];
}
}];
}
```

###3.为IYMSecondViewController添加边缘返回手势

```
UIScreenEdgePanGestureRecognizer *screenEdgePanGestureRecognizer = [[UIScreenEdgePanGestureRecognizer alloc] initWithTarget:self action:@selector(handleScreenEdgePanGesture:)];
screenEdgePanGestureRecognizer.edges = UIRectEdgeLeft;
[self.view addGestureRecognizer:screenEdgePanGestureRecognizer];
```

```
- (void) handleScreenEdgePanGesture:(UIScreenEdgePanGestureRecognizer *) recognizer
{
CGFloat progress = [recognizer translationInView:self.view].x / (self.view.bounds.size.width * 1.0);
progress = MIN(1.0, MAX(0.0, progress));

if (recognizer.state == UIGestureRecognizerStateBegan) {
_percentDrivenInteractiveTransition = [[UIPercentDrivenInteractiveTransition alloc] init];
[self.navigationController popViewControllerAnimated:YES];
}else if (recognizer.state == UIGestureRecognizerStateChanged) {
[_percentDrivenInteractiveTransition updateInteractiveTransition:progress];
}else if (recognizer.state == UIGestureRecognizerStateEnded || recognizer.state == UIGestureRecognizerStateCancelled) {
if (progress > 0.5f) {
[_percentDrivenInteractiveTransition finishInteractiveTransition];
}else{
[_percentDrivenInteractiveTransition cancelInteractiveTransition];
}

_percentDrivenInteractiveTransition = nil;
}
}
```