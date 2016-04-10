---
layout: post
title: "Objective-C 代码规范"
date: 2016-03-29 20:19:21 +0800
comments: true
categories: iOS
---

![objective-c-dai-ma-gui-fan](/images/objective-c-dai-ma-gui-fan/01.jpg)

[纽约时报 Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide)  

[Google Objective-C Style Guide](http://zh-google-styleguide.readthedocs.org/en/latest/google-objc-styleguide/)  

上面是前些天看的关于 Objective-C 代码规范的文章，发现了一些原先代码书写不规范的地方，现整理如下：

0.属性推荐使用点语法

```
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

而不是:

```
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

1.if else 的 写法

```
if (window) {
  [self windowDidLoad:window];
} else {
  [self errorLoadingWindow];
}

if (window) {
  [self windowDidLoad:window];
} else if (self){
  [self errorLoadingWindow];
}
```

2.Error 判断

```
NSError *error;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```
而不是

```
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

3.方法的书写

```
- (void)setExampleText:(NSString *)text image:(UIImage *)image;

- (void)setExampleText:(NSString *)text image:(UIImage *)image {

}
```

4.类别命名应该准确，体现其功能

```
@interface UIViewController (NYTMediaPlaying)
@interface NSString (NSStringEncodingDetection)
```

而不是:

```
@interface NYTAdvertisement (private)
@interface NSString (NYTAdditions)
```

5.添加到类别的方法或者属性 必须要加上前缀

```
@interface NSArray (NYTAccessors)
- (id)nyt_objectOrNilAtIndex:(NSUInteger)index;
@end
```

而不是:

```
@interface NSArray (NYTAccessors)
- (id)objectOrNilAtIndex:(NSUInteger)index;
@end
```

6.注释使用 VVDocumenter 就 OK 了

7.CGRect 函数

```
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

而不是:

```
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

8.常量声明

使用

```
static NSString * const NYTAboutViewControllerCompanyName = @"The New York Times Company";

static const CGFloat NYTImageThumbnailHeight = 50.0;
```

而不是

```
#define CompanyName @"The New York Times Company"

#define thumbnailHeight 2
```

9.如果有多个引入 要去分组

```
// Frameworks
@import QuartzCore;

// Models
#import "NYTUser.h"

// Views
#import "NYTButton.h"
#import "NYTUserView.h"
```

10.在协议或者数据源中，第一个参数应当是发送消息的对象

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
```

而不是:

```
- (void)didSelectTableRowAtIndexPath:(NSIndexPath *)indexPath;
```

11.``@public`` 和 ``@private`` 访问修饰符应该以一个空格缩进

12.异常：每个 ``@`` 标签应该有独立的一行，在 ``@`` 与 ``{}`` 之间需要有一个空格，``@catch`` 与被捕捉到的异常对象的声明之间也要有一个空格。

```
@try {
  foo();
}
@catch (NSException *ex) {
  bar(ex);
}
@finally {
  baz();
}
```

13.Treat Warnings as Errors