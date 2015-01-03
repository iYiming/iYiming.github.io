---
layout: post
title: "iOS 并发编程之 GCD"
date: 2014-12-28 07:10:30 +0800
comments: true
categories: iOS
published: false
---
串行队列（同时也被称为私有分配队列），按照添加进队列的顺序，在一段时间里只执行一条任务。
你可以创建多个串行队列，每一个队列并发执行。换言之，如果你创建了4个串行队列，每个对队列值在一段时间里执行一个任务，但这4条任务是并发执行的。

全局队列（也被称为全局分配队列），并发执行一个或多个任务，但是任务依然按照添加进队列的顺序开始执行。
在iOS5以后，你可以创建并发的分配队列，指定DISPATCH_QUEUE_CONCURRENT作为队列类型，另外，还有4种提前定义好的全局并发队列(DISPATCH_QUEUE_PRIORITY_HIGH, DISPATCH_QUEUE_PRIORITY_DEFAULT, DISPATCH_QUEUE_PRIORITY_LOW, DISPATCH_QUEUE_PRIORITY_BACKGROUND
)供你的应用程序使用。

主分配队列是一个全局可用的串行队列，在应用程序的主线程中执行任务。这个队列运行在应用程序的run loop中。

##队列

0.Serial Queue 串行队列
dispatch_queue_t queue;

queue = dispatch_queue_create("com.example.MyQueue", NULL);


1.Concurrent Queue 并发队列
dispatch_queue_t concurrentQueue = dispatch_queue_create(“com.example.MyQueue”,DISPATCH_QUEUE_CONCURRENT);
dispatch_get_global_queue(long identifier, unsigned long flags);

DISPATCH_QUEUE_PRIORITY_HIGH
DISPATCH_QUEUE_PRIORITY_DEFAULT
DISPATCH_QUEUE_PRIORITY_LOW
DISPATCH_QUEUE_PRIORITY_BACKGROUND

2.Main dispatch Queue 主线程队列
dispatch_get_main_queue()