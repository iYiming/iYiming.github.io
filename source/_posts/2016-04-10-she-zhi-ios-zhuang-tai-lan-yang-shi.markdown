---
layout: post
title: "iOS 状态栏样式"
date: 2016-04-10 12:07:30 +0800
comments: true
categories: iOS
---

![/images/she-zhi-ios-zhuang-tai-lan/0.jpg](/images/she-zhi-ios-zhuang-tai-lan/0.jpg)  

一直也没有使用新 API 来设置状态栏，在使用的过程中，遇到了问题，整理下：

关于状态栏设置,原先一直使用系统提供的如下方法： 

```
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];

// 加动画
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent animated:YES];
```
或者

```
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleDefault];

// 加动画
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleDefault animated:YES];
```

**注意：使用这个方法，还需要设置项目 Info.plist 文件中的 ``UIViewControllerBasedStatusBarAppearance`` 为 ``NO``。**

遗憾的是，这个 API 在 iOS 9 已经被废弃了。

早在 iOS 7 就出现了新的设置状态栏的 API:

在 ``UIViewController`` 添加了 ``- (UIStatusBarStyle)preferredStatusBarStyle;``

###如何使用

新建一个 Demo, 其中 ``ZYQRootViewController `` 是一个 ``UIViewController`` 的子类，代码如下：


``AppDelegate.m`` 代码如下：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    
    ZYQRootViewController *rootViewController = [[ZYQRootViewController alloc] init];
    self.window.rootViewController = rootViewController;
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

在 ``ZYQRootViewController`` 实现 ``- (UIStatusBarStyle)preferredStatusBarStyle; `` 方法：

```
- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleLightContent;
}
```
运行发现没问题，设置成功！

###在 UINavigationController 中使用出现的问题

如果把 ``AppDelegate.m`` 代码改为：

 ``ZYQRootViewController`` 包一个 ``UINavigationController``,将 ``window.rootViewController`` 设置为 ``UINavigationController``:

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    
    ZYQRootViewController *rootViewController = [[ZYQRootViewController alloc] init];
    UINavigationController *navController = [[UINavigationController alloc] initWithRootViewController:rootViewController];
    self.window.rootViewController = navController;
    [self.window makeKeyAndVisible];
    
    return YES;
}
```

运行发现,在 ``ZYQRootViewController`` 实现 ``- (UIStatusBarStyle)preferredStatusBarStyle; `` 方法没有调用。Why？

**因为 ``UINavigationController`` 将自己处理如何设置状态栏，而不是传递给它的子控制器。**


如何解决这个问题？

在 UIViewController 类别里找到下面方法：

```
// Override to return a child view controller or nil. If non-nil, that view controller's status bar appearance attributes will be used. If nil, self is used. Whenever the return values from these methods change, -setNeedsUpdatedStatusBarAttributes should be called.
- (nullable UIViewController *)childViewControllerForStatusBarStyle NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
```

如果重写了这个方法，返回我们想使用的控制器来控制状态栏样式。我们可以为 ``UINavigationController`` 添加类别，来重写这个方法：

```
#import "UINavigationController+ZYQStatusBarStyle.h"

@implementation UINavigationController (ZYQStatusBarStyle)

- (UIViewController *)childViewControllerForStatusBarStyle {
    return self.topViewController; // 返回最顶部的控制器
}

@end
```

添加 ``UINavigationController (ZYQStatusBarStyle)`` 类别后，我们就可以正常调用 ``ZYQRootViewController`` 实现 ``- (UIStatusBarStyle)preferredStatusBarStyle; `` 的方法了。