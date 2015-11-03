---
layout: post
title: "iOS 图片调整技术"
date: 2015-07-12 17:53:04 +0800
comments: true
categories: iOS
---

前几天看到[NSHisper](http://nshipster.com/)上的一篇关于图片缩放的文章挺好，到[中文版](http://nshipster.cn/)上找对应的中文版本看，没有找到。索性就自己翻译一下。下面是原文地址：

[http://nshipster.com/image-resizing/](http://nshipster.com/image-resizing/)

翻译如下：

自iOS出现以来，iOS开发者一直被一个问题所困扰：“如何调整一张图片”，在开发者和平台相互不信任的刺激下，这个问题变得含糊不清。在搜索出的一千条垃圾代码样例里，每一个
都自称是正确的答案，而其他的是假先知。

这是令人尴尬的，真的。

本周的文章努力对在iOS上调整图片的多种方式做出清晰解释（在OS X，需要一个UImage到NSImage适当的转换），用事实证明每种方式的性能特征，而不是简单的适用于所有情况的任何一种方式。

**在进一步阅读之前，请注意下面的这段话：**

>当设置UIImageView的Image， 对绝大多数的用例来手动调整图片是没有必要的。相反，可以简单的设置contentMode属性，无论是.ScaleAspectFit 来确保整个图片可以在UIImageView中都看得到，或者是设置.ScaleAspectFill，根据需要从中心来裁剪图片，来确保图片填充在UIImageView里。

>```
>imageView.contentMode = .ScaleAspectFit 
>imageView.image = image
>```

##确定缩放尺寸

在做图片调整之前，首先一个必须要决定的目标是要缩放的尺寸。

###0.通过系数缩放

缩放图片最简单的一种方式，是通过常量系数。通常，这个调用是除以一个整数，来减少原始尺寸（而不是乘以一个整数来放大）

新的尺寸可以通过单独缩放宽度和高度组件来计算。

```
let size = CGSize(width: image.size.width / 2, height: image.size.height / 2)  

...或者通过运用CGAffineTransform:
let size = CGSizeApplyAffineTransform(image.size, CGAffineTransformMakeScale(0.5, 0.5))  

```

###1.通过长宽比缩放

这样常常有用来缩放原始尺寸用这种方式，来适应一个矩形不用改变原始长宽比。AVFoundation框架中的AVMakeRectWithAspectRatioInsideRect是一个有用的函数，负责为你计算：

```
import AVFoundation
let rect = AVMakeRectWithAspectRatioInsideRect(image.size, imageView.bounds)
```

##缩放图片

有许多不同的方式来缩放图片，每一种具有不同的能力和性能特征

###0.UIGraphicsBeginImageContextWithOptions & UIImage -drawInRect:

在UIKIt框架中可以发现高级的图片调整API。提供一个UIImage对象，一个临时图形上下文，用来渲染一个缩放版本，使用UIGraphicBeginImageContextWithOptions() 和 UIGraphicsGetImageFromCurrentImageContext():

```
let image = UIImage(contentsOfFile: self.URL.absoluteString!)

let size = CGSizeApplyAffineTransform(image.size, CGAffineTransformMakeScale(0.5, 0.5))
let hasAlpha = false
let scale: CGFloat = 0.0 // Automatically use scale factor of main screen

UIGraphicsBeginImageContextWithOptions(size, !hasAlpha, scale)
image.drawInRect(CGRect(origin: CGPointZero, size: size))

let scaledImage = UIGraphicsGetImageFromCurrentImageContext()
UIGraphicsEndImageContext()
```

UIGraphicsBeginImageContextWithOptions()创建一个临时渲染上下文，在这上面绘制原始图片。第一个参数，size，是缩放图片的尺寸，第二个参数，isOpaque 是用来决定透明通道是否被渲染。对没有透明度的图片设置这个参数为false，可能导致图片有粉红色调。第三个参数scale是显示缩放系数。当设置成0.0，主屏幕的缩放系数将被使用，对视网膜屏显示是2.0或者更高（在iPhone6 Plus 上是 3.0）

###1.CGBitmapContextCreate & CGContextDrawImage

Core Graphic/Quartz 2D 提供了一个底层的API集合，可以提供更高级的配置。提供一个CGImage对象，一个临时位图上下文，用来渲染缩放的图片，使用CGBitmapContextCreate()和CGBitmapContextCreateImage():

```
let cgImage = UIImage(contentsOfFile: self.URL.absoluteString!).CGImage

let width = CGImageGetWidth(cgImage) / 2
let height = CGImageGetHeight(cgImage) / 2
let bitsPerComponent = CGImageGetBitsPerComponent(cgImage)
let bytesPerRow = CGImageGetBytesPerRow(cgImage)
let colorSpace = CGImageGetColorSpace(cgImage)
let bitmapInfo = CGImageGetBitmapInfo(cgImage)

let context = CGBitmapContextCreate(nil, width, height, bitsPerComponent, bytesPerRow, colorSpace, bitmapInfo.rawValue)

CGContextSetInterpolationQuality(context, kCGInterpolationHigh)

CGContextDrawImage(context, CGRect(origin: CGPointZero, size: CGSize(width: CGFloat(width), height: CGFloat(height))), cgImage)

let scaledImage = CGBitmapContextCreateImage(context).flatMap { UIImage(CGImage: $0 }
```

CGBitmapContextCreate需要几个参数来构建上下文，期望的尺寸和每个给定色彩空间内部的信道。在这个例子中，这些值从 CGImage中获取。下一步，CGContextSetInterpolationQuality允许上下文在各个保真度等级插入像素。在这个例子中，传递kCGInterpolationHigh参数用来获得最优的结果。CGContextDrawImage允许在制定的尺寸和位置上画图，允许在特定边缘或者适应一组图片特征比如faces，裁剪图片。最终，CGBitmapContextCreateImage从上下文中创建CGImage


###2.CGImageSourceCreateThumbnailAtIndex

Image I/O是一个强大，也是一个很少知道可以用来处理图片的框架。依赖Core Graphic，它可以在许多不同格式下读写，访问图片媒体，执行通用图片处理。在这个平台上框架提供了快速编解码图片，高级缓存机制和快速加载图片能力。

CGImageSourceCreateThumbnailtIndex提供了一个简介的AP,比同样的Core Graphics调用有不同的选项：

```
import ImageIO

if let imageSource = CGImageSourceCreateWithURL(self.URL, nil) {
    let options: CFDictionary = [
        kCGImageSourceThumbnailMaxPixelSize: max(size.width, size.height) / 2.0,
        kCGImageSourceCreateThumbnailFromImageIfAbsent: true
    ]

    let scaledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options).flatMap { UIImage(CGImage: $0) }
}
```

提供了CGImageSource和一组选项，CGImageSourceCreateThumbnailAtIndex创建一个缩略图。调整是通过kCGImageSourceThumbnailMaxPixelSize完成。指定最大的尺寸初一一个常量系数来缩放图片，同时保持原始的长宽比。通过指定不论是kCGImageSourceCreateThumbnailFromImageIfAbsent 或者 kCGImageSourceCreateThumbnailFromImageAlways，后续调用，Image I/O将自动缓存缩放的结果。

###3.Core Image Lanczos 采样

Core Image 提供一个使用CILanczosScaleTransform滤镜内置的Lanczos 采样函数。虽然是比UIKit更高级的API，在Core Image普及的使用KVC，使他用起来比较笨拙。

这就是说，设计模式是一致的。创建变换滤镜，配置和渲染输出图片的过程就像是其他Core Image流程：

```
let image = CIImage(contentsOfURL: self.URL)

let filter = CIFilter(name: "CILanczosScaleTransform")
filter.setValue(image, forKey: "inputImage")
filter.setValue(0.5, forKey: "inputScale")
filter.setValue(1.0, forKey: "inputAspectRatio")
let outputImage = filter.valueForKey("outputImage") as CIImage

let context = CIContext(options: nil)
let scaledImage = UIImage(CGImage: self.context.createCGImage(outputImage, fromRect: outputImage.extent()))
```

CILanczosScaleTransform接受inputImage，InputScale，和inputAspectRatio，所有的这些不言自明。CIContext通过CGImageRef间接表示方式，被用来创建UIImage，既然UIImage（CIImage:）不像预期那样经常工作。

##性能测试

那么，如何让这些不同途径组合成另一个？

这里有一组在运行iOS 8.0GM的iPod Touch(第五代)性能测试结果，使用XCTestCase.measureBlock():

###JPEG

缩放一个来自NASA Visible Earth，大的，高分辨率（12000 * 12000 px 20 MB JPEG）原图片。

![1.png](/images/image-resizing/1.png)
￼

###PNG

缩放一个相当大（1024 * 1024 px 1MB PNG）来渲染，即Postgres.app的应用Icon 所方程成0.1倍尺寸：

![2.png](/images/image-resizing/2.png)

###结论

>UIKit,Core Graphic 和Image I/O 对大多数图片缩放操作表现良好

>Core Image优于图片缩放操作。实际上，在这篇文章里特别推荐使用Core Graphic或者 Image I/O
函数来对图片缩放或者原先采样

>对不需要额外功能的一般图片缩放，UIGraphicsBiginImageContextWithOptions可能是最好的选项

>如果图片质量是一个考虑，考虑使用CGBitmapContextCreate与CGContextSetInterpolationQuality组合方式

>当以显示缩略图为目的的缩放图片，CGImageSourceCreateThumbnailAtIndex提供了一个对图片渲染和缓存的完整解决方式

注明：  

第一次翻译整篇文章，有些地方，翻译的有点粗糙，不明白的地方请[参照原文](http://nshipster.com/)。