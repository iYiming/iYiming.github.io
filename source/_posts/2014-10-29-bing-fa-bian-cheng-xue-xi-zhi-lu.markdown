---
layout: post
title: "并发编程学习之路"
date: 2014-10-29 07:50:55 +0800
comments: true
categories: iOS
published: false
---

###并发和程序设计

###操作队列

###分配队列（Dispatch Queues）

Cocoa中的操作（operation）是一种面向对象的方式来使你想要完成的任务异步。Operation既可以结合operation queue使用也可以单独使用，他们都是基于Objective-C的，在OS X 和 iOS中使用不尽相同。  

####0.关于 Operation 对象  

NSOperation是一个抽象类，必须继承它才可以工作。Foundation framework提供了两种实现好的子类来供我们使用（NSInvocationOperation、NSBlockOperation），详情见下图：  

![NSOperation]({{ root_uri }}/images/concurrency_guid/concurrency_guid_1.png)  

###分配资源（Dispatch Sources）

###从Thead迁移至新多线程技术


###参考文章  
[0.苹果官方并发编程学习指南](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html)  