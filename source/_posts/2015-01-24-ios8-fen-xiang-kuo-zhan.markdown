---
layout: post
title: "iOS8 分享扩展"
date: 2015-01-24 23:16:58 +0800
comments: true
categories: iOS
---

今天研究了下，iOS8的分享扩展，iOS8有好几个扩展，如下：

0.通知界面widget  
1.分享扩展  
2.Action  
3.照片编辑扩展  
4.资源存储扩展  
5.自定义键盘  

本文主要整理下分享扩展。


##创建项目并添加分享扩展

0.新建一个项目，命名为YMShareDemo：  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_01.png) 

1.添加分享扩展  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_02.png) 

选择Share Extentsion(分享扩展)  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_03.png)  

命名为YMShareExtension：  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_04.png) 

选择Finish，下图选择Active：  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_05.png) 

OK，到这里，项目创建成功！

##分享扩展plist文件

如下图所示，查看分享扩展的plist文件：  

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_06.png)  


NSExtensionActivationSupportsAttachmentsWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持最多附件数量 

NSExtensionActivationSupportsAttachmentsWithMinCount  

> 向系统和其他应用程序指示自己的应用程序支持的最小附件数量

NSExtensionActivationSupportsFileWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持文件分享，数值是一个Number类型的，表示能分享的最多文件数量

NSExtensionActivationSupportsImageWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持图片分享，数值是一个Number类型的，表示能分享的最多图片数量

NSExtensionActivationSupportsMovieWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持视频分享，数值是一个Number类型的，表示能分享的最多视频数量  

NSExtensionActivationSupportsText  

> 向系统和其他应用程序指示自己的应用程序支持文本分享，数值是一个Boolean类型的，表示是否能分享文本

NSExtensionActivationSupportsWebURLWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持链接分享，数值是一个Number类型的，表示能分享的最多链接数量

NSExtensionActivationSupportsWebPageWithMaxCount  

> 向系统和其他应用程序指示自己的应用程序支持网页分享，数值是一个Number类型的，表示能分享的最多网页数量  

示例（支持2个图片、1个视频、1个链接并支持文本分享）：

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_07.png)  

##分享扩展中的方法和属性

分享扩展控制器ShareViewController，继承自SLComposeServiceViewController（UIViewController的子类）。  

除了UIViewController的标准方法外，SLComposeServiceViewController还提供了一些与分享功能生命周期相关的属性和方法：  

###0. -(void) presentationAnimationDidFinish  

>在这个方法里，我们可以获得，要分享的图片、视频、链接等内容

示例(获取图片、视频、链接)：  

```
- (void) presentationAnimationDidFinish
{
    for (NSExtensionItem *item in self.extensionContext.inputItems) {
        for (NSItemProvider *itemProvider in item.attachments) {
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0), ^{
                if ([itemProvider hasItemConformingToTypeIdentifier:(NSString *) kUTTypeURL]) {//URL地址
                    [itemProvider loadItemForTypeIdentifier:(NSString *)kUTTypeURL options:nil completionHandler:^(NSURL *URL, NSError *error) {
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"URL:%@",URL);
                            });
                    }];
                }else if([itemProvider hasItemConformingToTypeIdentifier:(NSString *)kUTTypeImage]){//图片
                    [itemProvider loadItemForTypeIdentifier:(NSString *)kUTTypeImage options:nil completionHandler:^(UIImage *image, NSError *error) {
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"图片:%@",image);
                            });
                    }];
                }else if([itemProvider hasItemConformingToTypeIdentifier:(NSString *)kUTTypeMovie]){//视频
                    [itemProvider loadItemForTypeIdentifier:(NSString *)kUTTypeMovie options:nil completionHandler:^(NSURL *path,NSError *error){
                        if (path){
                            dispatch_async(dispatch_get_main_queue(), ^{
                                NSLog(@"视频地址:%@",path);
                            });
                        }
                    }];
                }
            });
        }
    }
}
```

###1.contentText  

>NSString类型属性，是分享文本框内的文字  

示例（获取分享文本框内文字）：

```
self.contentText
```
###2. -(void) didSelectPost  

>点击Post按钮时触发

###3. -(void) didSelectCancel  

>点击Cancel按钮时触发。  

###4. -(BOOL) isContentValid  

>编辑框文字变化时触发。

示例（只能在分享文本框内输入140字）：

```
- (BOOL)isContentValid {
    NSInteger messageLength = [[self.contentText stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]] length];
    NSInteger charactersRemaining = 140 - messageLength;
    self.charactersRemaining = @(charactersRemaining);

    if (charactersRemaining >= 0) {
        return YES;
    }

    // Do validation of contentText and/or NSExtensionContext attachments here
    return YES;
}
```

###5. charactersRemaining  

>Int类型属性，表示分享文本框还能填写的字数，如果超出字数，该属性展示的字数会变红，并用负数表示已经超出了多少字数。 

具体用法可以参照上方```-(BOOL) isContentValid```  


##实战  

未完成

##注意及遇到的问题

0.如果想要调试的话，需要选择第三方应用 如Safari、照片应用测试，如下图： 

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_08.png) 

1.本来想使用AFNetworking作为网络库来上传图片数据，但是不知道什么原因，不能在扩展中使用，至今不知道啥原因，还需研究下。  
2.如果想要自定义分享视图，如Tumblr、微信：

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_09.png)  

官方文档上说：

![分享扩展]({{ root_uri }}/images/ios8_fen_xiang_kuo_zhan/ios8_fen_xiang_kuo_zhan_10.png)  

不过貌似已经取消了，需要将```ShareViewController : SLComposeServiceViewController```转变为```ShareViewController : UIViewController```，详见[stackoverflow上的回复](http://stackoverflow.com/questions/25922118/making-share-extension-with-custom-layout)

参考文章

[苹果官方文档1](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/ShareSheet.html)  
[苹果官方文档2](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/SystemExtensionKeys.html#//apple_ref/doc/uid/TP40014212-SW2)