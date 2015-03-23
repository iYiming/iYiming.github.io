---
layout: post
title: "从加密说起"
date: 2015-01-04 20:34:02 +0800
comments: true
categories: 挨踢
---

最近遇到了些安全问题，发现问题就要解决问题，关于对称加密和非对称加密的概念都没搞清楚，着实该好好整理下。

##何为加密

摘自[维基百科:加密](http://zh.wikipedia.org/wiki/%E5%8A%A0%E5%AF%86)  

>加密算法就是加密的方法。  
>加密算法可以分为两类：[对称加密](http://zh.wikipedia.org/wiki/%E5%B0%8D%E7%A8%B1%E5%AF%86%E9%91%B0%E5%8A%A0%E5%AF%86)和[非对称加密](http://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)  
>在密码学中，加密是将明文信息隐匿起来，使之在缺少特殊信息时不可读。  
>对称加密就是将信息使用一个密钥进行加密，解密时使用同样的密钥，同样的算法进行解密。  
>非对称加密，又称公开密钥加密，是加密和解密使用不同密钥的算法，广泛用于信息传输中。  

##对称加密（对称密钥加密）
摘自[百度百科:对称加密](http://baike.baidu.com/view/7591.htm)  

>对称加密算法是应用较早的加密算法，技术成熟。在对称加密算法中，数据发信方将明文（原始数据）和加密密钥（mi yao）一起经过特殊加密算法处理后，使其变成复杂的加密密文发送出去。收信方收到密文后，若想解读原文，则需要使用加密用过的密钥及相同算法的逆算法对密文进行解密，才能使其恢复成可读明文。在对称加密算法中，使用的密钥只有一个，发收信双方都使用这个密钥对数据进行加密和解密，这就要求解密方事先必须知道加密密钥。  

常见对称密钥加密算法有：[AES](http://baike.baidu.com/subview/133041/5358738.htm)（Advanced Encryption Standard 高级加密标准），[DES](http://baike.baidu.com/view/7510.htm)（DES，Data Encryption Standard 数据加密标准）  

##非对称加密（公开密钥加密）
摘自[百度百科:非对称加密](http://baike.baidu.com/view/1490349.htm),看看原理知道是啥了，如下图：  

![加密]({{ root_uri }}/images/cong_jia_mi_shuo_qi/cong_jia_mi_shuo_qi_01.png)    


常见非对称加密算法有：[RSA](http://baike.baidu.com/link?url=5PT7OFiecao970jOMruzJ9CjTSeSrMC8fNpv-CQnvKixadXCy8c-hqy0AKrcKObdYCLYrSURCj2xlbdIx18PE6cmtP7601mUDaTfLmHy2RgYzqGudkEJ6qk39WDK3G12)

##iOS中的对称加密和非对称加密

iOS提供了<CommonCrypto/CommonCryptor.h>类库来实现加密(对称加密和非对称加密)功能。

[详情请参考:使用 DES 算法对数据加密](http://www.cnblogs.com/YouXianMing/p/3608078.html)  
[Github上源码](https://github.com/alfaromeodev/Cryptor)  