---
layout: post
title: "iOS 并发编程之 NSOperation"
date: 2014-12-28 06:16:43 +0800
comments: true
categories: iOS
---
本文翻译自[苹果官方文档]((https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperation_class/index.html)，以便于加深理解。

##NSOperation

0.NSOperaiton 类是一个抽象类，因为它是一个抽象类，所以不能直接使用它，可以自定义一个它的子类，或者使用系统定义好的子类(NSInvocationOperation 或者 NSBlockOperation)来执行任务。  

1.一个Operation对象是一个一次性对象，就是说它只能执行一个任务一次，执行完毕之后就不能再使用。通常把他放到Operation Queue中来执行。Operation Queue不是直接就执行他，而是把他放在二级线程中，或者用libdispatch类库（GCD）来直接执行。  

2.如果不想使用Operation Queue，可以通过直接调用Operation的start方法来执行。手动执行Operation通常加重代码负担，因为在一个没准备好的状态下执行Operation会触发异常，read属性来说报告这个Operation是否准备就绪。  


##Operaiton 依赖

依赖是一种方便的方式 来执行操作在特定的顺序中。可以通过addDependency:和removeDependency:方法来添加和删除操作依赖。默认情况下，Operation 对象被置为ready直到它所有的依赖完成执行。一旦最后一个依赖执行完成，操作对象会准备就绪，开始执行。  

NSOperation 的依赖没有区分一个依赖操作是否成功完成（比如cancel了一个操作，也被标记为这个操作已经完成）。操作A依赖操作B，如果想等到B成功执行后再来执行A，那就徐璈额外的代码来实现这一功能。  



##支持KVO机制的属性

NSOperation类为多个属性实现了KVC和KVO。可以通过监视这些属性来完成你想完成的东西。这些属性如下图：  


![Size Classes]({{ root_uri }}/images/ios-bing-fa-bian-cheng-zhi-nsoperation/ios-bing-fa-bian-cheng-zhi-nsoperation_01.png)  

尽管可以监听这些属性，但是不能通过Cocoa绑定来绑定到应用程序界面上去，因为用户界面必须运行在主线程，而一个Operation可以运行在任何线程。  

如果自定义实现以上的属性，必须保持KVC 和 KVO兼容，如果自定义其他额外属性，推荐也保持这种兼容。  


##多核心注意事项

NSOperation本身就是考虑多核心的。  


##同步 和 异步

如果手动执行一个操作对象，而不是把它放到一个队列，可以设置操作对象是同步还是异步行为。默认情况下，操作对象是同步的，操作队形不会创建一个独立的线程来执行任务。当调用start方法时，直接执行该操作在当前线程。  

如果想通过调用start方法来执行一个异步操作，这个方法会在彻底执行完毕前返回相关状态值。  

把NSOperation对象直接放到NSOPeration Queue里面，会自动异步执行，如果想实现一个异步的NSOperation 需要做很多工作，需要监视运行状态 等等，相关信息可以参考下方的子类化注意事项  


##子类化注意事项

NSOperation类提供了基本的逻辑和执行状态，因为它是抽象类，你需要子类化它才能正常使用。然后创建一个NSOperation子类，取决于这个操作室并发的还是非并发的。  


###0.创建非并发的NSOperation子类

只需要重写main方法就可以了  

###1.创建并发的NSOperation子类

至少需要重写以下方法和属性（main方法为可选）：  

MyOperation.h  
```
#import <Foundation/Foundation.h>

@interface MyOperation : NSOperation
{
    BOOL        executing;
    BOOL        finished;
}

- (void)completeOperation;

@end

```
MyOperation.m

```
#import "MyOperation.h"

@implementation MyOperation

- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}

- (void)start {
    // Always check for cancellation before launching the task.
    if ([self isCancelled])
    {
        // Must move the operation to the finished state if it is canceled.
        [self willChangeValueForKey:@"isFinished"];
        finished = YES;
        [self didChangeValueForKey:@"isFinished"];
        return;
    }

    // If the operation is not canceled, begin executing the task.
    [self willChangeValueForKey:@"isExecuting"];
    [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
    executing = YES;
    [self didChangeValueForKey:@"isExecuting"];
}

- (void)main {
    @try {
        // Do the main work of the operation here.

        [self completeOperation];
    }
    @catch(...) {
        // Do not rethrow exceptions.
    }
}

- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];

    executing = NO;
    finished = YES;

    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}



- (BOOL)isConcurrent {
    return YES;
}

- (BOOL)isExecuting {
    return executing;
}

- (BOOL)isFinished {
    return finished;
}

@end

```


![Size Classes]({{ root_uri }}/images/ios-bing-fa-bian-cheng-zhi-nsoperation/ios-bing-fa-bian-cheng-zhi-nsoperation_02.png)  

如何在NSOperation子类化类中使用操作的状态，如下图：

![Size Classes]({{ root_uri }}/images/ios-bing-fa-bian-cheng-zhi-nsoperation/ios-bing-fa-bian-cheng-zhi-nsoperation_03.png)  



参考文章：  

[官方文章：Concurrency Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html)  
[官方文章：NSOperation Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperation_class/index.html)