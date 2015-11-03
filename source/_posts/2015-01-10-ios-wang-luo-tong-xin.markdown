---
layout: post
title: "iOS 网络通信"
date: 2015-01-10 15:53:10 +0800
comments: true
categories: iOS
---

最近在研究AFNetworking源码，发现了一些原先没掌握好的东东，整理下...  

##NSURL

对于NSURL的理解，原先指示停留在将NSString转换为NSURL。今天细看文档，发现好多好用的功能，原先还自己封装来获取URL后缀，现在看来是真没文化啊。  

0.经典用法  

```
NSURL *url1 = [NSURL URLWithString:@"http://www.baidu.com/img/bd_logo1.png"];
NSURL *url2 = [NSURL URLWithString:@"/img/bd_logo1.png" relativeToURL:[NSURL URLWithString:@"http://www.baidu.com"]];
```

1.加载项目中本地网页  

```
NSString *urlPath = [[NSBundle mainBundle] pathForResource:@"404" ofType:@"html"];
NSURL *url = [NSURL URLWithString:urlPath];
```

2.获取扩展名  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me/hello.png"];
NSString *pathExtension = [url pathExtension];//扩展名 png
```

3.获取组成部分  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me/index.htm"];
NSArray *pathComponentsArray = [url pathComponents];//所有组成部分  [/,index.htm]
```

4.获取主机  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me/index.htm"];
NSString *host = [url host];//主机 www.iyiming.me
```

5.获取端口号  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me:80/index.htm"];
NSNumber *port = [url port];//端口号 80
```

6.获取查询参数  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me:80/index.htm?key1=value1&key2=value2"];
NSString *query = [url query];//查询参数 key1=value1&key2=value2
```

7.获取协议名称  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me:80/index.htm?key1=value1&key2=value2"];
NSString *scheme = [url scheme];//协议 http
```

8.获取锚定位  

```
NSURL *url = [NSURL URLWithString:@"http://www.iyiming.me:80/index.htm#jumpLocation"];
NSString *fragment = [url fragment];//锚定位 jumpLocation
```

9.获取用户名  

```
NSURL *url = [NSURL URLWithString:@"ftp://yiming@www.iyiming.me"];
NSString *user = [url user];//用户名 yiming
```  

##NSMutableURLRequest   

NSURLMutableRequest是NSURLRequest的子类，名如其类，它是可扩充的NSURLRequest。

0.缓存
```
enum
{
    NSURLRequestUseProtocolCachePolicy  = 0,
    NSURLRequestReloadIgnoringLocalCacheData  = 1,
    NSURLRequestReturnCacheDataElseLoad  = 2,
    NSURLRequestReturnCacheDataDontLoad  = 3,
};
typedef NSUInteger  NSURLRequestCachePolicy;
```

关于缓存的介绍 ，可以看看下面这篇博客，写的很是详细：  
[iOS开发之缓存（一）：内存缓存](http://www.cnblogs.com/zhuqil/archive/2011/07/30/2122127.html)

1.设置请求超时时间

设置timeoutInterval属性的就OK啦。

2.GET请求

```
#pragma mark -获取参数字符串
- (NSString *) parametersStr:(NSDictionary *) parametersDic
{
    NSMutableArray *parametersArray = [[NSMutableArray alloc] init];

    for (id key in parametersDic.allKeys) {
        if ([key isKindOfClass:[NSString class]]) {
            id value = parametersDic[key];
            NSString *parameterStr = [NSString stringWithFormat:@"%@=%@",key,value];
            [parametersArray addObject:parameterStr];
        }
    }

    NSString *parametersStr = [parametersArray componentsJoinedByString:@"&"];
    return parametersStr;
}

#pragma mark -GET Request
- (NSURLRequest *) GET:(NSString *)URLString parameters:(NSDictionary *) parametersDic
{
    NSString *parametersStr = [self parametersStr:parametersDic];//请求参数字符串

    NSString *requestURLStr = [NSString stringWithFormat:@"%@?%@",URLString,parametersStr];//请求URL字符串
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:requestURLStr]];

    //设置请求头部
    ////设置ContentType
    [request addValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    //设置Content-length
    [request addValue:[NSString stringWithFormat:@"%lu",(unsigned long)[parametersStr length]] forHTTPHeaderField:@"Content-Length"];
    //设置请求类型 GET
    [request setHTTPMethod:@"GET"];

    return request;
}
```

3.POST请求

```
#pragma mark -POST Request
- (NSURLRequest *) POST:(NSString *)URLString parameters:(NSDictionary *) parametersDic
{
    NSString *parametersStr = [self parametersStr:parametersDic];//请求参数字符串
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:URLString]];

    //设置请求头部
    ////设置ContentType
    [request addValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    //设置Content-length
    [request addValue:[NSString stringWithFormat:@"%lu",(unsigned long)[parametersStr length]] forHTTPHeaderField:@"Content-Length"];
    //设置请求类型 GET
    [request setHTTPMethod:@"POST"];
    //设置请求Body
    [request setHTTPBody:[parametersStr dataUsingEncoding:NSUTF8StringEncoding]];

    return request;
}
```

4.发送图片等多媒体数据  

未完成


##NSURLConnection

0.同步加载数据  

```
    NSURLRequest *request = [self GET:urlStr parameters:parametersDic];

    NSError *error = nil;
    NSURLResponse *response = nil;
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    if (!error) {
        NSString *dataStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"获取的字符串 %@",dataStr);
}   
```

1.异步加载数据  

1.1 +connectionWithRequest:delegate:

```
 NSURLRequest *request = [self GET:urlStr parameters:parametersDic];
[NSURLConnection connectionWithRequest:request delegate:self];
```

1.2 -initWithRequest:delegate:  

```
[[NSURLConnection alloc] initWithRequest:request delegate:self];
```

1.3 -initWithRequest:delegate:startImmediately:  

>startImmediately参数为YES时，会立刻执行。如果startImmediately参数为NO时，将不会在当前RunLoop中运行，需要手动调用```scheduleInRunLoop:forMode:```方法。不调用```scheduleInRunLoop:forMode:```方法，也可以调用```start```方法来执行,此时的RunLoop是当时的RunLoop。关于RunLoop可以参考[这篇文章](http://iyiming.me/blog/2015/01/03/run-loops/)  

```
NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
[connection start];
```

1.4 -sendAsynchronousRequest:queue:completionHandler:  

>queue参数是指completionHandler Block执行所在的队列

```
[NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
    if (!connectionError) {
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            NSString *dataStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
            dispatch_async(dispatch_get_main_queue(), ^{
                NSLog(@"获取的字符串 %@",dataStr);
            });
        });
    }else{
        NSLog(@"error:%@",connectionError.localizedDescription);
    }
}];
```

2.取消Connection

>调用cancel方法,如果取消之后，还想继续建立连接，需要重新实例化connection对象  

```
[connection cancel];
```

##NSURLConnectionDelegate

这个协议经常用到，但是不知道它是用来干啥的。相关资料也很少，只好查看[官方文档](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLConnectionDelegate_Protocol/index.html)  

NSURLConnectionDelegate是配合NSURLConnectionDataDelegate、NSURLConnectionDownloadDelegate协议使用的。常见的```connection:didFailWithError: ```方法就属于NSURLConnectionDelegate。  

NSURLConnectionDelegate中声明的方法，有些方法都已经废除了，见下图：

![网络通信]({{ root_uri }}/images/ios_wang_luo_tong_xin/ios_wang_luo_tong_xin_01.png)  

0.-connection:willSendRequestForAuthenticationChallenge:

>Tells the delegate that the connection will send a request for an authentication challenge.  
告诉协议，连接（connection）将会为身份认证询问发送一个请求

1.- (BOOL)connectionShouldUseCredentialStorage:(NSURLConnection *)connection

>URL加载者是否使用凭证存储来认证连接
Sent to determine whether the URL loader should use the credential storage for authenticating the connection.    

>如何返回NO，连接不会自动询问凭证存储，不会存储凭证，然而，在```connection:didReceiveAuthenticationChallenge```方法中，更具需要可以自己实现询问凭证存储，存储凭证。  

>在iOS7 和 OS X v10.9之前，实现```connection:willSendRequestForAuthenticationChallenge:```方法不会调用```connectionShouldUseCredentialStorage:```方法。也就是说，在以后的操作系统（iOS7 和 OS X v10.9后）中，实现```connection:willSendRequestForAuthenticationChallenge:```方法会调用```connectionShouldUseCredentialStorage:```方法。  

2.例子：实现访问无线路由器设置页面

一般无线路由器设置了密码，访问路由器设置页面会出现如下图页面（有的是[http://192.168.0.1](http://192.168.0.1)或者是[http://192.168.1.1](http://192.168.1.1)）：  

![网络通信]({{ root_uri }}/images/ios_wang_luo_tong_xin/ios_wang_luo_tong_xin_02.png)  

```
//步骤一  建立连接
NSURLRequest *request = [self POST:@"http://192.168.0.1" parameters:nil];
connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
[connection start];

//步骤二 实现NSURLConnectionDelegate  
#pragma mark -NSURLConnectionDataDelegate
- (void)connection:(NSURLConnection *)connection willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge {
    if ([challenge previousFailureCount] == 0) {
        NSLog(@"received authentication challenge");
        NSURLCredential *newCredential = [NSURLCredential credentialWithUser:@"这个地方输入用户名"
        password:@"这个地方输入密码"
        persistence:NSURLCredentialPersistenceForSession];
        NSLog(@"credential created");
        [[challenge sender] useCredential:newCredential forAuthenticationChallenge:challenge];
        NSLog(@"responded to authentication challenge");
    }else {
        NSLog(@"previous authentication failure");
    }
}

//步骤三 实现NSURLConnectionDataDelegate
- (void) connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    [responseData setLength:0];

    NSLog(@"接收响应");
}

- (void) connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    [responseData  appendData:data];

    NSLog(@"接收数据...");
}

- (void) connectionDidFinishLoading:(NSURLConnection *)connection
{
    NSString *newStr = [[NSString alloc] initWithData:responseData encoding:NSUTF8StringEncoding];

    //简单测试 用于显示路由器设置页面
    UIWebView *mainWebView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, 320, 400)];
    [mainWebView loadHTMLString:newStr baseURL:[NSURL URLWithString:@"http://192.168.0.1"]];
    [self.view addSubview:mainWebView];

    NSLog(@"完成 %@",newStr);
}
```

##NSURLConnectionDataDelegate

未完成

参考文章  

[苹果官方文档：NSURL](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSURL_Class/)  
[苹果官方文档：NSMutableURLRequest](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSMutableURLRequest_Class/index.html)  
[苹果官方文档：NSURLConnection](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSURLConnection_Class/index.html)  
[iOS 开发 中级：HTTP请求 GET POST 网络编程实现](http://blog.csdn.net/songrotek/article/details/8830372)  
