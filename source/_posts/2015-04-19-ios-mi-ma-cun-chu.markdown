---
layout: post
title: "iOS 密码存储"
date: 2015-04-19 22:05:07 +0800
comments: true
categories: iOS
---

本文主要整理下iOS密码存储的相关知识。

毫无疑问，密码存储是一个比较老生常谈的问题，原先项目中使用的是[SFHFKeychainUtils](https://github.com/kamiro/SFHFKeychainUtils),不知道什么原因使用的它。不过还算好用，没出现什么问题。

本着猎奇心理，在github上搜索```ios keychain```关键字，找出了星星最多的前两个开源包：  

[SSKeychain](https://github.com/soffes/sskeychain)

[UICKeyChainStore](https://github.com/kishikawakatsumi/UICKeyChainStore)

当然使用就使用星星最多的SSKeychain，pod install下，就应用到了项目里。但是使用了一段时间经常会发生[errSecDefault问题](https://github.com/soffes/sskeychain/issues/71)  

出于无奈，我开始看UICKeyChainStore，但它貌似不如SSKeychain，没有提供account。又想起原先使用的[SFHFKeychainUtils](https://github.com/kamiro/SFHFKeychainUtils)。

基于以上原因，也或许原先仅是拿过来使用，而没有理解里面的实现细节，故整理下：

##苹果官方相关信息

>
[苹果官方：Keychain Services Programming Guide](https://developer.apple.com/library/ios/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html#//apple_ref/doc/uid/TP30000897)   

>
[苹果官方： Keychain Services Reference](https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/index.html#//apple_ref/doc/constant_group/Keychain_Import_Export_Options) 

>[苹果官方代码](https://developer.apple.com/library/ios/samplecode/GenericKeychain/Introduction/Intro.html)

由于上方文档，即有Mac OS X的，又有iOS的，仅摘抄下关于iOS的重要部分：

>
A keychain is an encrypted container that holds passwords for multiple applications and secure services. Keychains are secure storage containers, which means that when the keychain is locked, no one can access its protected contents. In OS X, users can unlock a keychain—thus providing trusted applications access to the contents—by entering a single master password. In iOS, each application always has access to its own keychain items; the user is never asked to unlock the keychain. Whereas in OS X any application can access any keychain item provided the user gives permission, in iOS an application can access only its own keychain items.

钥匙链是一个加密的容器，它保存了多个程序和安全服务的密码。钥匙链是一个安全的存储容器，这意味着没有应用程序或服务在它锁住的情况下，可以访问它的内容。在Mac OS X下，用户可以解锁钥匙链，因此可以通过输入主密码的形式提供给信任的应用程序访问里面的内容。在iOS中，每个应用程序只能访问自己的钥匙链项；用户不能解锁钥匙链。相反在Mac OS X里，用户给予了许可，每个应用程序都能访问任何一个钥匙链项。在iOS里，一个应用程序只能访问自己的钥匙链项。

>
Note： On iPhone, Keychain rights depend on the provisioning profile used to sign your application. Be sure to consistently use the same provisioning profile across different versions of your application.

注意：在iPhone中，钥匙链依赖于 provisioning profile 文件来标识你的应用程序。确保应用程序在不同版本中一贯的使用同一个 provisioning profile 文件

>
In iOS,there is a single keychain accessible to applications. Although it stores the keychain items of all the applications on the system, an application can access only its own keychain items (with the possible exception of a keychain item for which the application that created it obtained a persistent reference).

在iOS里，有一个单独的钥匙链供应用程序访问。尽管他存储了系统里全部应用程序的钥匙链项，一个应用程序只能并且仅能访问它自己的钥匙链项（包括因为应用程序创建包含了一个持久的引用二引发的可能的钥匙链项异常）

>
Structure of a Keychain
>
Each keychain can contain any number of keychain items. Each keychain item contains data plus a set of attributes. For a keychain item that needs protection, such as a password or private key (a string of bytes used to encrypt or decrypt data), the data is encrypted and protected by the keychain. For keychain items that do not need protection, such as certificates, the data is not encrypted.


钥匙链结构  

每一个钥匙链可以包含任意数量的钥匙链项。每个钥匙链项包含数据加上一些属性集。对一个钥匙链项来说需要进行保护，比如像一个密码或这个私有键（加密或者揭秘的二进制数据字符串），数据被钥匙链加密和保护。对多个钥匙链项来说不需要保护，比如像证书，它的数据是不加密的。

>
The iOS gives an application access to only its own keychain items.

在iOS中，仅给一个用程序访问它自己的钥匙项权限。

>
The iOS Keychain Services API uses a different paradigm. This API has a single function (SecItemAdd) for adding an item to a keychain.

在iOS钥匙链服务API使用了一个不同的范式（相对于Mac OS X）。这个API有一个单独的函数（SecItemAdd）来添加一个钥匙链项。

>
In iOS, you call the SecItemCopyMatching function to find a keychain item owned by your application. In this case there’s only one keychain and the user is never prompted to unlock it.

在iOS中，你调用SecItemCopyMatching函数来查找你应用程序自己的钥匙链项目。在这种情况下，仅有一个钥匙链并且用户不需要被提示来解锁它（相对于Mac OS X）。

>
![天天向上](/images/iOS_mi_ma_cun_chu/iOS_mi_ma_cun_chu.jpg) 

iOS钥匙链服务搜索字典

在iOS，钥匙链服务使用键值对字典方式来制定钥匙链项的属性，这样你就能查找或者创建。

经典的搜索字典组成：

class key：用来指定要搜索的钥匙链项的类型（比如，互联网密码 或者密码密钥）。

一个或多个键值对来指定匹配属性数据（比如标签或者创建日期）

一个或多个搜索键值对，用来指定值来精确搜索，比如发行证书或者邮件地址来匹配。

一个返回值键值对，来指定你想要的数据（比如。一个字典或者是一个偏好引用）。


什么样的属性值被指定，取决于你要搜索的钥匙链项的类型。举例，如果你指定键位kSecClass的值为kSecClassGenericPassword，然后你可以指定创建时间的值或这个修改时间的值，但是不能指定主题或发行者（这个被用于证书）。

举例，如果你想执行一个对AppStore账户名为'ImaUser'密码不敏感情况的搜索,你可以在SecItemCopyMatching函数，使用上面表格中的键值对。

kSecReturnData键是函数返回钥匙链项的数据，如果你想得到键值对的值，你需要使用kSecReturnAttributes返回类型键并且值为kCFBooleanTrue.


##制作一个类似SSKeychain的翻版

###0.设置密码（添加、更新密码）

```
/**
*  @author iYiming, 15-04-19 22:20:51
*
*  设置密码（添加密码、修改密码）
*
*  @param password    密码
*  @param serviceName 服务名称
*  @param account     账户名称
*
*  @return 是否设置成功
*/
+ (BOOL) setPassword:(NSString *)password forService:(NSString *)serviceName account:(NSString *)account{
    NSMutableDictionary *keyChainDic = [[NSMutableDictionary alloc] init];
    [keyChainDic setObject:account forKey:(__bridge id)kSecAttrAccount];//账户
    [keyChainDic setObject:serviceName forKey:(__bridge id)kSecAttrService];//服务名
    [keyChainDic setObject:(__bridge id)kSecClassGenericPassword forKey:(__bridge id)kSecClass];//类型
    [keyChainDic setObject:[password dataUsingEncoding:NSUTF8StringEncoding] forKey:(__bridge id)kSecValueData];//密码数据
    [keyChainDic setObject:(__bridge id)kCFBooleanTrue forKey:(__bridge id)kSecReturnAttributes];//返回数据键值对
    [keyChainDic setObject:(__bridge id)kCFBooleanTrue forKey:(__bridge id)kSecReturnData];//返回Data

    CFMutableDictionaryRef outDictionary = nil;
    OSStatus keychainErr = SecItemCopyMatching((__bridge CFDictionaryRef)keyChainDic,(CFTypeRef *)&outDictionary);

    BOOL resultState = NO;
    if (keychainErr == noErr) {
        NSMutableDictionary *returnDictionary = [NSMutableDictionary dictionaryWithDictionary:(__bridge_transfer NSMutableDictionary *)outDictionary];
        [returnDictionary setObject:(__bridge id)kSecClassGenericPassword forKey:(__bridge id)kSecClass];

        [keyChainDic removeObjectForKey:(__bridge id)kSecAttrAccount];
        [keyChainDic removeObjectForKey:(__bridge id)kSecAttrService];
        [keyChainDic removeObjectForKey:(__bridge id)kSecClass];
        [keyChainDic removeObjectForKey:(__bridge id)kSecReturnAttributes];
        [keyChainDic removeObjectForKey:(__bridge id)kSecReturnData];
        OSStatus errorCode = SecItemUpdate((__bridge CFDictionaryRef)returnDictionary, (__bridge CFDictionaryRef)keyChainDic);

        if (errorCode == noErr) {
            resultState = YES;
        }else{
            resultState = NO;
        }
    }else if (keychainErr == errSecItemNotFound) {
        //添加
        OSStatus errorCode = SecItemAdd((__bridge CFDictionaryRef)keyChainDic,NULL);

        if (errorCode == noErr) {
            resultState = YES;
        }else{
            resultState = NO;
        }
    }

    return resultState;
}
```

###1.获取密码

```
/**
*  @author iYiming, 15-04-19 22:21:38
*
*  获取密码
*
*  @param serviceName 服务名称
*  @param account     账户名称
*
*  @return 密码
*/
+ (NSString *) passwordForService:(NSString *)serviceName account:(NSString *)account
{
    NSMutableDictionary *keyChainDic = [[NSMutableDictionary alloc] init];
    [keyChainDic setObject:account forKey:(__bridge id)kSecAttrAccount];
    [keyChainDic setObject:serviceName forKey:(__bridge id)kSecAttrService];
    [keyChainDic setObject:(__bridge id)kSecClassGenericPassword forKey:(__bridge id)kSecClass];
    [keyChainDic setObject:(__bridge id)kCFBooleanTrue forKey:(__bridge id)kSecReturnData];

    CFDataRef passwordData = NULL;
    OSStatus keychainError = noErr; 
    keychainError = SecItemCopyMatching((__bridge CFDictionaryRef)keyChainDic,(CFTypeRef *)&passwordData);

    NSString *pwdStr = nil;
    if (keychainError == noErr){
        NSString *passwordString = [[NSString alloc] initWithBytes:[(__bridge_transfer NSData *)passwordData bytes]
        length:[(__bridge NSData *)passwordData length] encoding:NSUTF8StringEncoding];
        NSLog(@"%@",passwordString);
        pwdStr = passwordString;
    }else if (keychainError == errSecItemNotFound) {
        pwdStr = nil;
    }

    return pwdStr;
}

```

###2.删除密码

```
/**
*  @author iYiming, 15-04-19 22:22:10
*
*  删除密码
*
*  @param serviceName 服务名称
*  @param account     账户名称
*
*  @return 是否删除成功状态
*/
+ (BOOL) deletePasswordForService:(NSString *)serviceName account:(NSString *)account {
    NSMutableDictionary *keyChainDic = [[NSMutableDictionary alloc] init];
    [keyChainDic setObject:account forKey:(__bridge id)kSecAttrAccount];
    [keyChainDic setObject:serviceName forKey:(__bridge id)kSecAttrService];
    [keyChainDic setObject:(__bridge id)kSecClassGenericPassword forKey:(__bridge id)kSecClass];
    [keyChainDic setObject:(__bridge id)kCFBooleanTrue forKey:(__bridge id)kSecReturnAttributes];

    CFMutableDictionaryRef outDictionary = nil;

    OSStatus keychainError = SecItemCopyMatching((__bridge CFDictionaryRef)keyChainDic,(CFTypeRef *)&outDictionary);
    BOOL resultState = NO;

    if (keychainError == noErr){
        NSString *pwd = [YMKeyChain passwordForService:serviceName account:account];//YMKeyChain 自己写的是一个类，调用的上面的方法

        [keyChainDic setObject:[pwd dataUsingEncoding:NSUTF8StringEncoding] forKey:(__bridge id)kSecValueData];

        OSStatus keychainError =  SecItemDelete((__bridge CFDictionaryRef)keyChainDic);//密码正确才能删除成功
        if (keychainError == noErr){
            resultState = YES;
        }else{
            resultState = NO;
        }
    }

    return resultState;
}
```

已经放到 [GitHub](https://github.com/iYiming/YMKeyChain) 上。
