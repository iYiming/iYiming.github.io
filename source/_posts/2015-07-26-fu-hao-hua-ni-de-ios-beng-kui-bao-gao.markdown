---
layout: post
title: "符号化iOS崩溃报告"
date: 2015-07-26 16:44:54 +0800
comments: true
categories: iOS
---

最近看到一篇关于符号化iOS崩溃报告的文章挺好，决定翻一下，下方链接是原文地址：  

[https://possiblemobile.com/2015/03/symbolicating-your-ios-crash-reports/](https://possiblemobile.com/2015/03/symbolicating-your-ios-crash-reports/)    

##开始

你已经递交你的应用程序的崩溃报告，但是栈回溯包含了难懂的内存地址。开发者该怎么办？简而言之，你需要调试符号来应用于堆栈追踪，从而使它人类可读，这个过程我们称作符号化。

在开始之前，我们依照使用Crasher，它提供了一个简单崩溃报告，让你破译。

你应该有.crash文件。如果没有，你可以从iTunes Connect中攫取，或者直接通过Xcode从连接的设备中攫取（Windows > Devices）,或在一个连接的设备（Settings > Privacy > Diagnostics & Usage）,或使用 PLCrashReporter 框架。你可能已经使用了第三方的崩溃报告服务，在你配置正确后，它将符号化你的崩溃。

根据你的应用创建是如何配置的，你需要一个或者全部下面的东西：

>0.导致崩溃的应用程序构建的.app文件。这个包包含了应用二进制文件，以及可能包含测试符号。（如果你有一个.ipa文件，你可以把它当做一个.zip文件，通过解压这个文件把它提取出来）

>1.导致崩溃的应用程序构建的.dSYM文件。这是你的应用程序的副产品，包含调试符号化。  

你需要哪一个? 在 Xcode中，查找Build Setting里的 "Strip Debug Symbols During Copy"(COPY_PHASE_STRIP)。当允许时，调试符号将会从你的.app省去，把它放到了.dSYM文件。否则你的.app包含这些符号。（默认情况下， 由于某些不清楚的原因，调试符号从释放的创建中剥离，你可能不应该改变这个发布配置）  

##但是等等，什么是调试符号？

为了让调试符号变成人类可读的即程序员给方法命名的名称，编译器通过使用它自己的符号来减少这些命名的调试符号，从而使代码模糊。即使两次相同代码创建，你也不能依赖这些符号。

##检查崩溃

如果通过Xcode的Organizer从设备中提交崩溃报告，你的崩溃报告可能将会对UIKit和其他iOS框架自动符号化。如果Xcode知道你的创建，他将自动符号化你的崩溃。

如果不是这种情况，你需要自己符号化。

##使用"Symbolicatecrash"工具来符号化

幸运的是，苹果提供给了我们脚本来检索调试符号化，将他们应用于崩溃报告。

对 Xcode 6，你可以在下面目录里找到这个工具：    

```
/Applications/Xcode.app/Contents/SharedFrameworks/DTDeviceKitBase.framework/Versions/Current/Resources/symbolicatecrash
```

或者如果你使用的是Xcode 5 从下方路径获得：  

```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/Current/Resources/symbolicatecrash
```

为了使用这个工具，你需要用安装Xcode恰当路径导出 ```DEVELOPER_DIR``` 环境变量：  

```
export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"  
```

把你的.crash文件.app文件和.dSYM文件放在同一个目录下然后运行：  

```
symbolicatecrash -v ScaryCrash.crash > Symbolicated.crash
```

你可能需要显示应用应用程序二进制：  

```
symbolicatecrash -v ScaryCrash.crash ./Crasher.app/Crasher > Symbolicated.crash
```
如果你使用在Crasher例子中发现的便利脚本，需要冗长的标记：  

```
sh symbolicate6.sh ScaryCrash.crash ./Crasher.app/Crasher > Symbolicated.crash
```

如果你使用在Crasher例子中发现的便利脚本，需要冗长的标记：  

```
sh symbolicate6.sh ScaryCrash.crash ./Crasher.app/Crasher > Symbolicated.crash
```

##验证符号化
如果符号化不能工作，仔细检查你攫取的.dSYM或者.app文件是否正确。你可以通过交叉引用崩溃的报告中的UUID和应用程序二进制中的UUID：

```
dwarfdump –uuid Crasher.app/Crasher
```  
结果：  

![1.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/1.png)  


在dSYM文件中：  

```
dwarfdump –uuid Crasher.app.dSYM/Contents/Resources/DWARF/Crasher
```  

结果：  

![2.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/2.png)  

和你的崩溃报告中的UUID进行比较：  

![3.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/3.png)   

从symbolicatecrash工具冗长的日志里，同样列出它找出的UUID。

##排查 "Symbolicatecrash" 工具

如果你还是困惑，请认真检查symbolication日志。symbolication工具尝试找出你应用中与UUID相匹配的相应文件以及每个动态框架，寻找着你的应用程序名称或者UUID 并看出是否匹配。  

![4.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/4.png)  

如果Spotlight不能找到你的.dSYM文件，你可能碰到这样一个日志样例：  

![5.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/5.png)  

或者有一个无效的.dSYM：  

![6.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/6.png)  

或者有任何其他无效的输入：  

![7.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/7.png)  

在Xcode 6 版本中的symbolicatecrash工具尝试解决一些再Xcode 5 版本中面临的Spotlight问题。如果你遇到问题，他可能是Spotlight中一个文件的索引问题，尝试：

```
mdimport -g /Applications/Xcode.app/Contents/Library/Spotlight/uuid.mdimporter . 
```

##使用命令行工具链

我们可以深入 和使用开发者命令行工具链来一行行的符号化栈追踪的内存地址。如果你在过去使用这个方法中遇到问题，那可能是因为在最近几年.crash文件根式改变了。

首先，让我们再看看我们崩溃报考中的栈追踪：  

![8.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/8.png)   

最左边的十六进制值，0x000aeef6,是栈地址。第二个十六进制值，0xa8000,是应用程序的加载地址。以加号为开始的数字，28406，是栈地址和加载地址的差值。 0x00aeef6 = 0xa8000 + 0x6EF6
。你将会发现在我们的崩溃报告底部的“Binary Images”，它是动态库内存地址范围列表。我们的二进制文件的起始地址在堆栈跟踪加载地址相匹配。  

![9.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/9.png)   

下一步，为了更好的测量，我们将验证可执行文件包含蹦溃的架构，不论使用file或者lipo -info命令：

```
file Crasher.app/Crasher
```  

结果：  

![10.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/10.png)  

现在我们全副武装。我们经查实atos命令来将我们的地址转换成调试符号。注意我们如何提供加载地址，它在我们指定崩溃的架构的栈地址后面：  

```
atos -arch armv7 -o Crasher.app/Crasher -l 0xa8000 0x000aeef6  
```

结果：  

![11.png](/images/fu-hao-hua-ni-de-ios-beng-kui-bao-gao/11.png)  

就是这样。如果你有兴趣继续钻研，研读Mach-O对象文件格式以及Mach-O命令行实用工具，也就是otool和lipo命令。  

##更多阅读

更多全面的文档，请查看：  
[Technical Note TN2151: Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html#//apple_ref/doc/uid/DTS40008184)  
[Technical Q&A QA1765: How to Match a Crash Report to a Build](https://developer.apple.com/library/ios/qa/qa1765/_index.html#//apple_ref/doc/uid/DTS40012196)  
[Mach-O Programming Topics](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html)  
[Objc.io on Mach-O Executables](http://www.objc.io/issues/6-build-tools/mach-o-executables/)


