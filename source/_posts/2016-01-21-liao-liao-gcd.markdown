---
layout: post
title: "聊聊 GCD"
date: 2016-01-21 00:10:24 +0800
comments: true
categories: iOS
---

![liao_liao_GCD](/images/liao_liao_GCD/0.png)

##什么是 GCD

GCD (Grand Central Dispatch) 是 iOS 多任务的核心。在 Mac OS X 10.6 雪豹中首次推出，后被引入到了 iOS 4.0 中。GCD 是基于 C 的 API，是底层的框架，NSOperationQueue 是在 GCD 的基础上实现的。  

GCD 和 Block 的配合使用，可以方便地进行多线程编程。

##GCD 包含什么内容

分派队列（dispatch queue），分派组（dispatch group），分派屏障（dispatch barrier）。

除了上面几个比较常用的 GCD 还包含下面几个内容：    

分派信号量（dispatch semaphores）、分派源（dispatch sources）、分派数据（dispatch data）以及分派 I/O 。

本文仅介绍 分派队列，分派组，分派屏障 GCD 常用内容。

##同步、异步，并行、串行，并行、并发 概念

同步  

所谓同步，就是在发出一个功能调用时，在没有得到结果之前，该调用就不返回。

同步代码如下：

```
dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_sync(concurrentQueue, ^(){
   NSLog(@"A");
});

dispatch_sync(concurrentQueue, ^(){
   NSLog(@"B");
});

// 先输出 A 后输出 B 。
```

异步

异步的概念和同步相对。当一个异步过程调用发出后，调用者立刻得到结果就执行下面的功能。

异步代码如下：

```
dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_async(concurrentQueue, ^(){
   NSLog(@"A");
});

dispatch_async(concurrentQueue, ^(){
   NSLog(@"B");
});

// 可能先输出 A 后输出 B ，也可能先输出 B 后输出 A。因为异步下是没执行完就执行下面的功能
```

并行与并发区别

当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程,它只能把CPU运行时间划分成若干个时间段,再将时间段分配给各个线程执行,在一个时间段的线程代码运行时,其它线程处于挂起状态.这种方式我们称之为并发(Concurrent).  

当系统有一个以上 CPU 时,则线程的操作有可能非并发.当一个 CPU 执行一个线程时,另一个CPU 可以执行另一个线程,两个线程互不抢占 CPU 资源,可以同时进行,这种方式我们称之为并行(Parallel)

##分配队列

串行队列（一个队列一个队列的执行）：

- 主队列。最常见的串行队列。使用 ``dispatch_get_main_queue()`` 获得。

- 自己创建的串行队列。

```
dispatch_queue_t serialQueue = dispatch_queue_create("me.iYiming.serialQueue", DISPATCH_QUEUE_SERIAL);
```

 或者


```
dispatch_queue_t serialQueue = dispatch_queue_create("me.iYiming.serialQueue", NULL);
```


并发队列（几个队列“同时”执行）：

- 系统队列 

```
#define DISPATCH_QUEUE_PRIORITY_HIGH 2 // 高
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0 // 默认
#define DISPATCH_QUEUE_PRIORITY_LOW (-2) // 低
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN //最低
```

```
dispatch_queue_t concurrentQueue = dispatch_get_global_queue(XXXX, 0); // 并发队列  XXXX 表示 上面的四个参数，
```

- 自己创建的并发队列

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("me.iYiming.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
```

补充：

``dispatch_set_target_queue()`` 用法：  

设置自己创建队列的目标队列，使创建的队列优先级和目标队列一样。

[http://blog.csdn.net/growinggiant/article/details/41077221](http://blog.csdn.net/growinggiant/article/details/41077221)  

上文中说了种情况：

>一般都是把一个任务放到一个串行的queue中，如果这个任务被拆分了，被放置到**多个串行的 queue 中**，但实际还是需要这个任务同步执行，那么就会有问题，因为多个串行 queue 之间是并行的。  
>
>那该如何是好呢？  
>
这是就可以使用 ``dispatch_set_target_queue`` 了。
如果将多个串行的 queue 使用 ``dispatch_set_target_queue`` 指定到了同一目标，那么着多个串行 queue 在目标 queue 上就是同步执行的，不再是并行执行。

除了上面链接中说到的用处外（也可以用别的方式替代），感觉没多大用处？！

##分配组  

关于使用分配组，我用到的情况就是这个情形下：执行 A 任务,B任务后(A B 可以同时做)，最后再做 C 任务。

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("me.iYiming.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);

dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, concurrentQueue, ^{
   // A 任务
});

dispatch_group_async(group, concurrentQueue, ^{
   // B 任务
});

dispatch_group_notify(group, concurrentQueue, ^{
   // C 任务
});
```

##分配屏障

我们使用分配屏障会等当前队列执行处理全部结束后，再将指定的处理追加到该队列上，然后再由分配屏障追加 的处理执行完毕后，当前队列才恢复为一般的动作，追加到该队列的处理又开始执行。

代码如下

```
dispatch_queue_t myQueue = dispatch_get_global_queue(0, 0);
   
dispatch_async(myQueue, ^{
   NSLog(@"123");
});

dispatch_async(myQueue, ^{
   NSLog(@"456");
});

dispatch_barrier_async(myQueue, ^{
   NSLog(@"789");
});

dispatch_async(myQueue, ^{
   NSLog(@"10");
});

```

输出结果，先输出 ``123`` 或者 ``456``，再输出 ``789``，最后才输出 ``10``。


举个例子

我们都会对数据进行读写操作，为了防止多个线程对数据进行安全访问。我们需要使用锁来实现某种同步机制。

在 GCD 出现之前，有两种办法：

- 采用内置的“同步块”

```
 - (void) synchronizationMethod{
      @synchronized(self) {
          // 使用同步块
      }
  }
```

- 直接使用 NSLock 对象（也可以使用 NSRecursiveLock 这种递归锁）。  

```
  _lock = [[NSLock alloc] init];

  - (void) synchronizationMethod{
      [_lock lock];
      //NSLock 对象方式
      [_lock unlock];
  }
```

>互斥锁分为 递归锁 和 非递归锁。
>
>同一个线程可以多次获取同一个递归锁，不会产生死锁。  
如果一个线程多次获取同一个非递归锁，则会产生死锁。

这两种方法都很好，不过也有其缺陷。比方说：  

- 在极端情况下，同步块会导致死锁，另外，其效率也不见得高.

- 滥用 ``@sychronized(self)`` 会很危险，因为所有同步块都会彼此抢夺同一个锁。要是有很多歌属性都这么写的话，那么每个属性的同步块都要等其他所有同步块执行完毕才能执行，这也许不是我们想要的结果。

我们可以使用 “串行同步队列”，将读取操作及写入操作都安排在同一个队列里，即可保证数据同步。

```
_serialQueue = dispatch_queue_create("me.iYiming.serialQueue", DISPATCH_QUEUE_SERIAL);
        
- (NSString *) someString{
    __block NSString *localSomeString;
    
    __weak typeof(self) wakeSelf = self;
    dispatch_sync(_serialQueue, ^{
        localSomeString = wakeSelf.someString;
    });
    return localSomeString;
}

- (void) setSomeString:(NSString *)someString{
    __weak typeof(self) wakeSelf = self;
    dispatch_sync(_serialQueue, ^{
        wakeSelf.someString = someString;
    });
}
```

还可以进一步优化。设置方法并不一定非得是同步的。设置实例变量所用的块，并不需要向设置返回什么值。也就是可以修改成如下：  

```
- (void) setSomeString:(NSString *)someString{
    __weak typeof(self) wakeSelf = self;
    dispatch_async(_serialQueue, ^{
        wakeSelf.someString = someString;
    });
}
```

但经过测试一下程序性能，那么可能会发现这种写法比原来慢，因为执行异步派发时，需要拷贝块。若拷贝所用的时间明显超过执行块所花的时间，则这种做法将比原来更慢。

多个获取方法可以并发执行，而获取方法与设置方法之间不能并发执行，利用这个特点，还能写出更快一些的代码来。


```
_concurrentQueue = dispatch_queue_create("me.iYiming.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);

- (NSString *) someString{
    __block NSString *localSomeString;
    
    __weak typeof(self) wakeSelf = self;
    dispatch_sync(_concurrentQueue, ^{
        localSomeString = wakeSelf.someString;
    });
    return localSomeString;
}

- (void) setSomeString:(NSString *)someString{
    __weak typeof(self) wakeSelf = self;
    dispatch_barrier_sync(_concurrentQueue, ^{ // 同步屏障
        wakeSelf.someString = someString;
    });
}
```

**使用 并发队列 和 分配屏障 可实现高效率的数据库访问和文件访问。**


##GCD VS NSOperation

GCD 的优点：  

1. GCD 提供的 ``dispatch_after`` 支持调度下一个操作的开始时间而不是直接进入睡眠。
2. NSOperation 中没有类似 ``dispatch_source_t``,``dispatch_io``,``dispatch_data_t``,``dispatch_semaphore_t`` 等操作。  

NSOperation 的优点：  

1. GCD 没有操作依赖。我们可以让一个 Operation 依赖于另一个 Operation，这样的话尽管两个 Operation 处于同一个并行队列中，但前者会直到后者执行完毕后再执行；
2. GCD 没有操作优先级（GCD 有队列优先级），能够使同一个并行队列中的任务区分先后地执行，而在 GCD 中，我们只能区分不同任务队列的优先级，如果要区分block任务的优先级，也需要大量的复杂代码；
3. GCD 没有 KVO。NSOperation 可以监听一个 Operation 是否完成或取消，这样能比GCD 更加有效地掌控我们执行的后台任务
4. 在NSOperationQueue 中，我们可以随时取消已经设定要准备执行的任务(当然，已经开始的任务就无法阻止了)，而 GCD 没法停止已经加入 queue 的 Block(其实是有的，但需要许多复杂的代码)
5. 我们能够对 NSOperation 进行继承，在这之上添加成员变量与成员方法，提高整个代码的复用度，这比简单地将 block 任务排入执行队列更有自由度，能够在其之上添加更多自定制的功能。

##使用 ``dispatch_once`` 创建单例

直接上代码：  

```
  //保存单例对象的静态全局变量
  static id _instance;
  + (instancetype)sharedTools {
      return [[self alloc]init];
  }
  //在调用alloc方法之后，最终会调用allocWithZone方法
  + (instancetype)allocWithZone:(struct _NSZone *)zone {
      //保证分配内存的代码只执行一次
      static dispatch_once_t onceToken;
      dispatch_once(&onceToken, ^{
          _instance = [super allocWithZone:zone];
      });
      return _instance;
  }
  //这是个对象方法，既然有对象而且是单例，那么调用者就是这个单例对象了，那就返回调用的对象就行
  - (id)copyWithZone:(NSZone *)zone {
      return self;
  }
  //这是个对象方法，既然有对象而且是单例，那么调用者就是这个单例对象了，那就返回调用的对象就行
  - (id)mutableCopyWithZone:(NSZone *)zone {
      return self;
  }
  #if __has_feature(objc_arc)
  //如果是ARC环境
  #else
  //如果不是ARC环境

  //既然是单例对象，总不能被人给销毁了吧，一旦销毁了，分配内存的代码已经执行过了，就再也不能创建对象了。所以覆盖掉release操作
  - (oneway void)release {
  }
  //这是个对象方法，既然有对象而且是单例，那么调用者就是这个单例对象了，那就返回调用的对象就行
  - (instancetype)retain {
      return self;
  }
  //为了便于识别，这里返回 MAXFLOAT ，别的程序员看到这个数据，就能意识到这是单例了。纯属装逼……
  - (NSUInteger)retainCount {
      return MAXFLOAT;
  }
  #endif
```

当然还有种方式 是使用上面那种 使用 ``@synchronized()`` 方式。但 ``dispatch_once`` 更高效，它没有使用重量级的同步机制，若是那样做的话，每次运行代码前都要获取锁，相反，此函数采用 “原子访问”来查询标记，以判断其所对应的代码原来是否已经执行过。

##参考文章

[http://www.cnblogs.com/NickyYe/archive/2008/12/01/1344802.html](http://www.cnblogs.com/NickyYe/archive/2008/12/01/1344802.html)  

[http://www.tanhao.me/pieces/616.html/](http://www.tanhao.me/pieces/616.html/)  

[http://blog.csdn.net/likendsl/article/details/8568961](http://blog.csdn.net/likendsl/article/details/8568961)  

[http://stackoverflow.com/questions/15629696/why-my-completionblock-never-gets-called-in-an-nsoperation](http://stackoverflow.com/questions/15629696/why-my-completionblock-never-gets-called-in-an-nsoperation)  

[http://blog.csdn.net/hufengvip/article/details/11687699](http://blog.csdn.net/hufengvip/article/details/11687699)  

[http://www.jianshu.com/p/d09e2638eb27](http://www.jianshu.com/p/d09e2638eb27)  

[http://stackoverflow.com/questions/7651551/why-should-i-choose-gcd-over-nsoperation-and-blocks-for-high-level-applications](http://stackoverflow.com/questions/7651551/why-should-i-choose-gcd-over-nsoperation-and-blocks-for-high-level-applications)  