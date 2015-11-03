---
layout: post
title: "Swift 2.0 —— 基础部分"
date: 2015-11-02 15:50:55 +0800
comments: true
categories: iOS
---

![基础部分](/images/swift-ji-chu-bu-fen/0.jpg)

0.Swift 包含了 C 和 Objective-C 上所有基础数据类型，``Int`` 表示整型值；``Double`` 和 ``Float`` 表示浮点型值；``Bool`` 是布尔型值；``String`` 是文本型数据。Swift 还提供了三个基本的集合类型: ``Array`` ，``Set`` 和``Dictionary``。

1.Swift 还增加了 Objective-C 中没有的高阶数据类型比如元组（Tuple）。元组可以让你创建或者传递一组数据，比如作为函数的返回值时，你可以用一个元组可以返回多个值。

2.Swift 还增加了可选（Optional）类型，用于处理值缺失的情况，它可以用在**任何类型上**。  

3.常量和变量必须在使用前声明，用 ``let`` 来声明常量，用 ``var`` 来声明变量。

4.可以在一行中声明多个常量或者多个变量，用逗号隔开。

```
var x = 0.0, y = 0.0, z = 0.0
```

5.当声明常量或者变量的时候可以加上类型标注（type annotation），说明常量或者变量中要存储的值的类型。如果要添加类型标注，需要在常量或者变量名后面加上**一个冒号和空格，然后加上类型名称**。

6.可以在一行中定义多个同样类型的变量，用逗号分割，并在最后一个变量名之后添加类型标注：

```
var red, green, blue: Double
```

7.一般来说你很少需要写类型标注。如果你在声明常量或者变量的时候赋了一个初始值，Swift可以推断出这个常量或者变量的类型。

```
var i = 0.0 //将i变量推断为Double类型
```

8.你可以用任何你喜欢的字符作为常量和变量名，包括 Unicode 字符:

```
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

9.你可以用print(_:separator:terminator:)函数来输出当前常量或变量的值。

**``print(_:separator:terminator:)`` 是一个用来输出一个或多个值到适当输出区的全局函数。如果你用 Xcode，``print(_:separator:terminator:)`` 将会输出内容到“console”面板上。``separator`` 和 ``terminator`` 参数具有默认值，因此你调用这个函数的时候可以忽略它们。默认情况下，该函数通过添加换行符来结束当前行。如果不想换行，可以传递一个空字符串给 ``terminator`` 参数--例如，``print(someValue, terminator:"")``。**

10.Swift 用字符串插值（string interpolation）的方式把常量名或者变量名当做占位符加入到长字符串中，Swift 会用当前常量或变量的值替换这些占位符。将常量或变量名放入圆括号中，并在开括号前使用反斜杠将其转义：

```
let friendlyWelcome = "Hello!"
print("The current value of friendlyWelcome is \(friendlyWelcome)")
```

11.单行注释以双正斜杠（ ``//`` ）作为起始标记;你也可以进行多行注释，其起始标记为单个正斜杠后跟随一个星号（``/* ``），终止标记为一个星号后跟随单个正斜杠（ ``*/`` ）;也可以进行嵌套注释。

12.与其他大部分编程语言不同，Swift 并不强制要求你在每条语句的结尾处使用分号（;），当然，你也可以按照你自己的习惯添加分号。**有一种情况下必须要用分号，即你打算在同一行内写多条独立的语句**：  

```
let cat = "🐱"; print(cat)
// 输出 "🐱"
```

13.**当推断浮点数的类型时，Swift 总是会选择 ``Double`` 而不是 ``Float``**。

```
let pi = 3.14159
// pi 会被推测为 Double 类型
```

14.类型别名（type aliases）就是给现有类型定义另一个名字。你可以使用 ``typealias`` 关键字来定义类型别名:

```
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min
```

15.Swift 有一个基本的布尔（Boolean）类型，叫做 ``Bool``。布尔值指逻辑上的值，因为它们只能是真或者假。Swift 有两个布尔常量，``true`` 和 ``false``。

16.元组（tuples）把多个值组合成一个复合值。**元组内的值可以是任意类型，并不要求是相同类型**。  

```
let http404Error = (404, "Not Found")
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// 输出 "The status code is 404"
print("The status message is \(statusMessage)")
// 输出 "The status message is Not Found"
```

17.如果你只需要一部分元组值，分解的时候可以把要忽略的部分用下划线（_）标记。

```
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// 输出 "The status code is 404"
```

18.你还可以**通过下标来访问元组中的单个元素，下标从零开始**

```
print("The status code is \(http404Error.0)")
// 输出 "The status code is 404"
print("The status message is \(http404Error.1)")
// 输出 "The status message is Not Found"
```

19.你可以在定义元组的时候给单个元素命名

```
let http200Status = (statusCode: 200, description: "OK")

print("The status code is \(http200Status.statusCode)")
// 输出 "The status code is 200"
print("The status message is \(http200Status.description)")
// 输出 "The status message is OK"
```

20.可选类型

```
var serverResponseCode: Int? = 404
// serverResponseCode 包含一个可选的 Int 值 404
serverResponseCode = nil
// serverResponseCode 现在不包含值
```

21.可选绑定格式

```
if let constantName = someOptional {
    statements
}
```

22.可以包含多个可选绑定在 ``if`` 语句中，并使用 ``where`` 子句做布尔值判断。

```
if let firstNumber = Int("4"), secondNumber = Int("42") where firstNumber < secondNumber {
    print("\(firstNumber) < \(secondNumber)")
}
// prints "4 < 42"
```

23.有时候在程序架构中，第一次被赋值之后，可以确定一个可选类型总会有值。在这种情况下，每次都要判断和解析可选值是非常低效的，因为可以确定它总会有值。这种类型的可选状态被定义为**隐式解析可选类型**（implicitly unwrapped optionals）。把想要用作可选的类型的后面的问号（String?）改成感叹号（String!）来声明一个隐式解析可选类型。

24.一个隐式解析可选类型其实就是一个普通的可选类型，但是可以被当做非可选类型来使用，并不需要每次都使用解析来获取可选值。下面的例子展示了可选类型String和隐式解析可选类型String之间的区别：

```
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // 需要惊叹号来获取值

let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  // 不需要感叹号
```

25.错误处理

**注：在Swift中，错误用遵循ErrorType协议类型的值来表示。**这个空协议表示一种可以用做错误处理的类型。 Swift的枚举类型尤为适合塑造一组相关的错误情形(error conditions)。

```
enum VendingMachineError: ErrorType {
    case InvalidSelection   //选择无效
    case InsufficientFunds(coinsNeeded: Int)    //金额不足
    case OutOfStock         //缺货
}

func canThrowAnError() throws {
    // 这个函数有可能抛出错误
    
    guard var item = "error" else {
            throw VendingMachineError.InvalidSelection
   }
}

do {
    try makeASandwich()
    eatASandwich()
} catch Error.OutOfCleanDishes {
    washDishes()
} catch Error.MissingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```

26.可选类型可以让你判断值是否存在，你可以在代码中优雅地处理值缺失的情况。然而，在某些情况下，如果值缺失或者值并不满足特定的条件，你的代码可能没办法继续执行。这时，你可以在你的代码中触发一个断言（assertion）来结束代码运行并通过调试来找到值缺失的原因。

```
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
// 因为 age < 0，所以断言会触发
```

如果不需要断言信息，可以省略，就像这样:

```
assert(age >= 0)
```
