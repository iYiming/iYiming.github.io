---
layout: post
title: "Swift 2.0 —— 字符串和字符"
date: 2015-11-03 11:58:34 +0800
comments: true
categories: iOS
---

![Swift 字符串和字符](/images/swift-zi-fu-chuan-he-zi-fu/0.jpg)

0.字符串字面量是由双引号 ( ``""`` ) 包裹着的具有固定顺序的文本字符集。每一个字符串都是由编码无关的 Unicode 字符组成，并支持访问字符的多种 Unicode 表示形式（representations）。  

```
let someString = "Some string literal value"
```

1.要创建一个空字符串作为初始值，可以将空的字符串字面量赋值给变量，也可以初始化一个新的String实例：

```
var emptyString = ""               // 空字符串字面量
var anotherEmptyString = String()  // 初始化方法
// 两个字符串均为空并等价。
```

2.您可以通过检查其Boolean类型的isEmpty属性来判断该字符串是否为空：

```
if emptyString.isEmpty {
    print("Nothing to see here")
}
// 打印输出："Nothing to see here"
```

3.Swift 的 ``String`` 类型是**值类型**。 如果您创建了一个新的字符串，那么当其进行常量、变量赋值操作，或在函数/方法中传递时，会进行值拷贝。 任何情况下，都会对已有字符串值创建新副本，并对该新副本进行传递或赋值操作。

4.``String`` 是例如 ``"hello, world"`` ，``"albatross"`` 这样的有序的``Character`` （字符）类型的值的集合。通过 ``String`` 类型来表示。 一个 ``String`` 的内容可以用变量的方式读取，它包括一个 ``Character`` 值的集合。

5.您可通过for-in循环来遍历字符串中的characters属性来获取每一个字符的值：

```
for character in "Dog!🐶".characters {
    print(character)
}
// D
// o
// g
// !
// 🐶
```

6.创建一个字符

```
let exclamationMark: Character = "!"
```

7.字符串可以通过传递一个值类型为Character的数组作为自变量来初始化：

```
let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
let catString = String(catCharacters)
print(catString)
// 打印输出："Cat!🐱"
```

8.字符串可以通过加法运算符（+）相加在一起（或称“连接”）创建一个新的字符串：

```
let string1 = "hello"
let string2 = " there"
var welcome = string1 + string2
// welcome 现在等于 "hello there"
```

9.您可以用 ``append()`` 方法**一个字符**附加到一个字符串变量的尾部：

```
let exclamationMark: Character = "!"
welcome.append(exclamationMark)
// welcome 现在等于 "hello there!"
```

10.如果想要获得一个字符串中 ``Character`` 值的数量，可以使用字符串的``characters`` 属性的 ``count`` 属性：  

```
let unusualMenagerie = "Koala 🐨, Snail 🐌, Penguin 🐧, Dromedary 🐪"
print("unusualMenagerie has \(unusualMenagerie.characters.count) characters")
// 打印输出 "unusualMenagerie has 40 characters"
```

另外需要注意的是通过 ``characters`` 属性返回的字符数量并不总是与包含相同字符的 ``NSString`` 的 ``length`` 属性相同。``NSString`` 的 ``length`` 属性是利用 ``UTF-16`` 表示的十六位代码单元数字，而不是 ``Unicode`` 可扩展的字符群集。作为佐证，当一个 ``NSString`` 的 ``length`` 属性被一个 ``Swift`` 的 ``String`` 值访问时，实际上是调用了 ``utf16Count``。

11.每一个String值都有一个关联的索引( ``index`` )类型，``String.Index``，它对应着字符串中的每一个 ``Character`` 的位置。  

12.不同的字符可能会占用不同数量的内存空间，所以要知道Character的确定位置，就必须从String开头遍历每一个 Unicode 标量直到结尾。**因此，Swift 的字符串不能用整数(integer)做索引（索引类型为实质是一个结构体，别名为Index）**。

13.使用 ``startIndex`` 属性可以获取一个 ``String`` 的**第一个Character的索引**。使用 ``endIndex`` 属性可以获取最后一个 ``Character`` 的**后一个位置的索引**。因此，``endIndex`` 属性不能作为一个字符串的有效下标。**如果String是空字符串，``startIndex`` 和 ``endIndex`` 是相等的**。

14.通过调用 ``String.Index`` 的 ``predecessor()`` 方法，可以立即得到前面一个索引，调用 ``successor()`` 方法可以立即得到后面一个索引。任何一个 ``String`` 的索引都可以通过锁链作用的这些方法来获取另一个索引，也可以调用``advancedBy(_:)`` 方法来获取。但如果尝试获取出界的字符串索引，就会抛出一个运行时错误。

```
let greeting = "Guten Tag!"
greeting[greeting.startIndex]
// G
greeting[greeting.endIndex.predecessor()]
// !
greeting[greeting.startIndex.successor()]
// u
let index = greeting.startIndex.advancedBy(7)
greeting[index]
// a
```

15.使用 ``characters`` 属性的 ``indices`` 属性会创建一个包含全部索引的范围( ``Range`` )，用来在一个字符串中访问单个字符。

```
for index in greeting.characters.indices {
   print("\(greeting[index]) ", terminator: " ")
}
// 打印输出 "G u t e n   T a g !"
```

16.调用 ``insert(_:atIndex:)`` 方法可以在一个字符串的指定索引插入**一个字符**：

```
var welcome = "hello"
welcome.insert("!", atIndex: welcome.endIndex)
// welcome now 现在等于 "hello!"
```

17.调用insertContentsOf(_:at:)方法可以在一个字符串的指定索引插入一个字符串。

```
welcome.insertContentsOf(" there".characters, at: welcome.endIndex.predecessor())
// welcome 现在等于 "hello there!"
```

18.调用 ``removeAtIndex(_:)`` 方法可以在一个字符串的指定索引删除**一个字符**。

```
welcome.removeAtIndex(welcome.endIndex.predecessor())
// welcome 现在等于 "hello there"
```

19.调用 ``removeRange(_:)`` 方法可以在一个字符串的指定索引删除一个子字符串。

```
let range = welcome.endIndex.advancedBy(-6)..<welcome.endIndex
welcome.removeRange(range)
// welcome 现在等于 "hello"
```
20.Swift 提供了三种方式来比较文本值：字符串字符相等、前缀相等和后缀相等。

21.字符串/字符可以用等于操作符( ``==`` )和不等于操作符( ``!=`` ):

```
let quotation = "We're a lot alike, you and I."
let sameQuotation = "We're a lot alike, you and I."
if quotation == sameQuotation {
    print("These two strings are considered equal")
}
// 打印输出 "These two strings are considered equal"
```

如果两个字符串（或者两个字符）的可扩展的字形群集是标准相等的，那就认为它们是相等的。在这个情况下，即使可扩展的字形群集是有不同的 ``Unicode`` 标量构成的，只要它们有同样的语言意义和外观，就认为它们标准相等。

22.通过调用字符串的 ``hasPrefix(_:)/hasSuffix(_:)``方法来检查字符串是否拥有特定前缀/后缀，两个方法均接收一个String类型的参数，并返回一个布尔值。
