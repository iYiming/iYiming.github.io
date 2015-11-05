---
layout: post
title: "写给大忙人看的Swift（二）"
date: 2015-09-01 09:33:02 +0800
comments: true
categories: iOS
---

![Swift](/images/da-mang-ren-swift1/1.JPG)

注：本系列，仅为自己读书摘要。如果你想购买此书，请移步到[http://item.jd.com/11732275.html](http://item.jd.com/11732275.html)  

下面是 《第二章 深入学习Swift语法》的摘抄内容

0.在代码中，变量值可能为空，而错误的使用控制会导致出现错误。为了解决这个问题Swift提供了可空类型。可空类型能够安全地检查某个值是否为空，当值不为空时，才会被访问。  

1.在运行时，可空类型有两种不同的表示方式：Some 或者 None。当值不为空时，可空类型用 Optional.Some(value) 表示，否则用 Opational.None表示，当变量需要存放这两种不同的情况时，我们必须将变量声明为可空类型，在类型说明后面添加一个"?"修饰符（例如，原本的Int将变成 Int?）。不限于类，任意类型都可以徐世伟可空类型。  

2.可空类型对值进行了装箱处理，因此不能像对待非可空类型一样对待它们。例如，你不能将一个String?当成String来用：  

```
let dic = ["name" : "jim"]      
let gender = dic["gender"] //gender为String?可空类型
let say = "Your gender is" + gender //出错：可空类型‘String？’无法拆箱 应改为：let say = "Your gender is" + gender！
```

3.在运行时，如果你对一个不存在的值进行拆箱，应用程序将崩溃。  

4.对于可空类型，在拆箱操作之前，必须检查可空类型是否包含有效值（不为空）,类似的代码非常繁琐，为了使代码更加简洁和可读，Swift使用了一种将检查和拆箱操作合并到一个步骤的语法，即所谓的可空绑定。它为数据的处理提供了一种简短安全的方法，如下代码：

```
if someOptional != nil{
	let someValue = someOptional!
	//正常使用someValue
}
```

可以被可空绑定替换为：

```
if let someValue = someOptional{
	//正常使用someValue
}
```

5.泛型结构如下：  

```
func maxValue<T: comparable>(lhs: T, rhs: T) -> T {
	if lhs > rhs{
		return lhs
	}

	return rhs
}
```

上面的代码中，声明了一个新的函数，并将其命名为 ``maxValue``，与一般的函数不同，在参数列表前面我们使用了一个 ``<T: Comparable>``,这表明我们正在定义一个泛型函数。``<T: Comparable>``说明这个函数要使用一个泛型 ``T`` ，同时 ``T`` 必须实现 ``Comparable`` 协议（该协议定义了运算符函数，运算会在这个函数中用到）。  

6.Swift 标准库的许多函数和类都使用了泛型。``Array`` 和 ``Dictionary`` 就是两个泛型集合。  

7.编译器有时也无法推断出正确的类型信息，这时你可以显式指明值的类型，比如：

```
protocol Speaks{
	func speak() -> String
}

struct Duck: Speaks{
	func speak() -> String{
		return "quack"
	}
}

struct Dog: Speaks{
	func speak() -> String{
		return "bark"
	}
}

let fido = Dog()
let donald = Duck()

let pets = [fido, donald] //错误：无法恒却判断出数组的类型

//需要将上面的代码改为：
let pets: [Speaks] = [fido, donald] 
```

8.闭包提供了一种创建自包含代码片段的函数式方法，同时还能从上下文中自动捕获值。闭包一次取义于"从上下文中捕获变量常量"，即"封闭"值的意思。Swift的闭包类似Objective-C中的块（block），只不过在使用上更加灵活。  

9.闭包有三种不同的形式：全局函数、嵌套函数、闭包表达式，每一种都有各自的用途。

10.闭包表达式提供了一种轻量级的隔离代码的方法（不需要对代码块进行命名），同时从上下文中捕获值。下面是一个闭包表达式的例子：  

```
var aFewNumbers = [1, 2, 3]  
var squares = aFewNumbers.map({ (num: Int) -> Int in 
	return num * num
})
//squares = [1, 4, 9]

```

11.Swift 中的元组，是一种轻巧实用的数据结构，允许你将多个值结合成一个对象。  

12.要访问元组的成员，你可以通过索引或者将远足分解到多个变量中。例如：

```
let iceCreamCone = (2, "chocolate", "cone")

print("I'd like \(iceCreamCone.0) scoop(s) of \(iceCreamCone.1) in a \(iceCreamCone.2)")
```

13.也可以在创建元组时，为每个成员进行命名，以便通过名称来访问元组成员，这种方法能清楚的描述元组所储存的数据，对于简单的类，可以直接用元组代替。  

```
let iceCreamCone = (scoops:2, flavor:"chocolate", style:"cone")

println("I'd like \(iceCreamCone.scoops) scoop(s) of \(iceCreamCone.flavor) in a \(iceCreamCone.style)")
```

14.返回值可以指定名称也可以不指定名称：

```
//不指定名称
func numberCounts(nums: [Int]) -> (Int, Int)
//指定名称
func numberCounts(nums: [Int]) -> (even: Int, odd: Int)
```

15.如果元组中的某个值你不需要，在分解时你可以用下划线“_”替代。

16.在Swift中，Switch结构，如果你需要代码贯穿到下一分支，你必须显式地实用fallthrough关键字。例如：  

```
func colorForIndexPath(indexPath: NSIndexPath) -> UIColor {
	switch (indexPath.section){
		case 0:
			fallthrough
		case 2:
			fallthrough
		case 4:
			return UIColor.blueColor()
		default:
			return UIColor.whiteColor()
	}
}
```

17.在 ``case`` 中实用 ``where`` 子句，这为 ``switch`` 语句提供了更多的灵活性。你还可以在 ``case`` 语句中使用值绑定，从而使代码更加紧凑，可读性更好。如：  

```
let indexSizeAndValue = (9, 10, "")
switch indexSizeAndValue{
	case (0,_,let value):
		print("The first value is \(value)")
	case let (index,size,"") where index == (size - 1):
		print("The last value is empty")
	case let (index,size,value) where index == (size-1):
		print("The last value is \(value)")
	case let (index,_,value):
		print("The value at index \(index) is \(value)")
}
```



