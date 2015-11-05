---
layout: post
title: "写给大忙人看的Swift（一）"
date: 2015-08-24 10:02:41 +0800
comments: true
categories: iOS
---

![Swift](/images/da-mang-ren-swift1/1.JPG)

关于Swift的书，我买了好几书，有实体书也有电子书，有好书也有烂书。今天买了本《写给大忙人看的 Swift 》，看看名字就知道自己有多功利。准备以此书开启我的 Swift 之旅。

在Swift发布一年后，还没有真正进入 Swift 的世界，我有种种借口：项目中还是使用的Objective-C 无法马上使用 Swift 开发；官方文档是英文的，看英文很费劲；看翻译过来的中文版，感觉太多，无法坚持；Swift自身也在随时发生着改变； 

Swift 一定是未来的趋势，所以必须提前准备。既然感觉项目中无法马上使用 Swift 开发，那就自己建立一个自己的项目，既学习 Swift 又提高自己对整个项目的掌控能力；既然抱怨中英文文档难、多、不能坚持，那就看这本《写给大忙人看的 Swift 》，书很薄，不到 200 页，共 8 个章节。

注：本系列，仅为自己读书摘要。如果你想购买此书，请移步到:

[http://item.jd.com/11732275.html](http://item.jd.com/11732275.html)  

废话不多说，下面是整理的是第一章的内容：

0.Swift 除了常见的注释外，还支持嵌套注释。  

1.枚举或结构体在代码中以值的方式进行传递。也就是说，会创建原始对象的一份副本，并将次副本复制给新的变量，在代码中可以使用新的变量，对其进行修改和删除操作，并且这些操作不会影响到原始变量。反过来也是成立的：原始变量的使用、修改和删除不会影响到新的副本。  

2.类是通过引用的方式进行传递的，在传递过程中，会将指向原始变量的指针赋值给新的变量。无论修改哪个变量，另外一个变量也会受到影响。  

3.任何一种类型都不能隐式的转换成另外一种类型，这就意味着在代码中不能将一个 ``Int`` 和 ``Float`` 值进行相加操作。 如果需要将两个数值加在一起，首先需要确保这两个值的类型相同，或者明确的将某一个值的类型转换为与另一个值相同的类型。这样的规则有助于 Swift 成为一门具有安全性的编程语言：编译器会阻止这种混合类型之间进行操作，并可能引发异常结果。  

4.``Switch`` 结构必须含有 ``default``。

5.在 Swift 中，``Switch`` 结构里，``case`` 分支语句处理方式发生了重大改变。在 C 编程语言中，需要在每个case尾部添加一个 ``break`` 语句，否则程序将继续执行下一个 ``case``。这一直是程序引发许多错误的根源。在 Swift 中，为了预防这样的错误，当遇到下一个 ``case`` 分支开始时，``switch`` 语句会自动结束。 p9  

6.Swift 有一个 Objective-C 中没有的强大功能：给循环语句添加一个标签（ label ），然后再内部循环中指定希望break或continue到哪个循环中。一个标签由标签名称之后跟着一个冒号构成，并且标签位于循环语句关键字之前。 p11  

7.存储整形值的基本类型是 ``Int``。``Int`` 在 32 位的设备中表现为 32 位整形，而在 64 位的设备中则为 64 位整形。（``UInt`` 也类似）  

8.当在设备间传输文件时，为了确保兼容性，写入到文件中（或者通过网络传输）所有的整形变量都应该明确指定长度。当读取存储在文件中的值时，应该强制使用32或64位变量，以避免可能出现的类型不匹配和/或数据损坏。  

9.``Float`` 是 32 位值，``Double`` 是 64 位值，这与设备的架构无关。  

10.当使用小数时，编译器总是将其推断为 ``Double`` 类型，而不是 ``Float``。因此，如果不需要 64 位的高精度，需要显示的将变量声明为 ``Float``。  

11.Swift 中的 ``String`` 是 ``Character``（字符）类型值的集合。每个Character代表一个 Unicode 字符。  

12.在 Swift 中使用 ``Count()`` 函数来判断出一个字符串中有多少个字符。  

13.Swift 中的 ``String`` 也兼容Objective-C中的 ``NSString`` ，由于 ``NSString`` 和 ``String`` 的实现方式有所不同，因此从 ``NSString`` 的``length`` 属性得到的元素总数可能跟 Swift 中的不同。这是因为 ``length`` 返回的是 ``NSString`` UTF-16 版本中 16 位编码的字符数量，而有些Unicode字符使用了多个 16 位。通过使用 ``Count(str.utf16)`` 获得的值（其中 ``str`` 为``String`` 变量）与 ``NSString`` 的 ``length`` 属性就会相同。例如：  

```
Swift:
var myStr = "Tom is a 🐱"

println(count(myStr))//打印出 10
println(count(myStr.utf16))//打印出 11 

Objective-C:
NSString* myStr = @"Tom is a 🐱";
NSLog(@"%@",@(myStr.length));//打印出 11
```

14.在 Swift 中，数组是集合类型中的一种，它用来存储相同类型的有序列表。

15.数组的声明可以用两种方式表示：长形式 和 短形式。这两种方法的效果都是相同的，它们可以互换使用：  
>
>长形式： Array<Type>  
>短形式：[Type]  

16.利用长形式声明和初始化一个数组：  

```
var people: Array<String> = [] //显式类型
//或
var people = Array<String>() //隐式类型
```

17.利用短形式声明和初始化一个数组：

```
var people: [String] = [] //显式类型
//或
var people = [String]() //隐式类型
```

18.由于数组属于结构体，所以数组传递的时候是一份副本，而数组中的元素是副本还是引用，则要取决于该元素是枚举、结构体还是类。  

19.用 ``for-in`` 访问数组中的元素，有两种方式：

```
var people = ["Jim","Tom","Tim"]
//0.不使用索引
for name in people{//name变量名称是随意起的
	print("name : \(name)")
}

//1.使用索引
for (index,name) in people.enumerate(){//index变量名称、name变量名称是随意起的
    print("\(index) : \(name)")
}
```

20.字典是存储无序相同类型的集合，存储在其中的每个对象都有唯一的一个 key。

21.字典的声明，跟数组的声明一样，有两种方式表示：长形式 和 短形式。
>
>长形式： Dictionary<Key Type, Value Type>  
>短形式：[Key Type : Value Type] 

22.利用长形式声明和初始化一个字典：  

```
var people: Dictionary<String,PeopleClass> = [:] //显式类型
//或
var people = Dictionary <String,PeopleClass>() //隐式类型
```

23.利用短形式声明和初始化一个字典：

```
var people: [String:PeopleClass] = [:] //显式类型
//或
var people = [String:PeopleClass]() //隐式类型
```
