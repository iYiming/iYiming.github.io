---
layout: post
title: "Swift 2.0 —— Swift 初见"
date: 2015-11-01 23:38:27 +0800
comments: true
categories: iOS
---

![Swift 初见](/images/swift-chu-jian/0.jpg)

Swift 更新实在是太快，以至于不得不按照官方文档来进行学习，买过的书，看过的文档已有比较大的变化，加之还没系统看完，更是囫囵吞枣。故还是得按照官方文档来进行系统学习，跟紧新特性。  

Swift 2.0 官方文档，中文版已有翻译过来的：  

[http://wiki.jikexueyuan.com/project/swift/](http://wiki.jikexueyuan.com/project/swift/)  

本系列文章，仅是整理自己阅读过程中觉得重要的且一开始不是很理解的内容。  

Let's start!  

0.如果你写过 C 或者 Objective-C 代码，那你应该很熟悉下面这种形式——在 Swift 中，这行代码就是一个完整的程序。你不需要为了输入输出或者字符串处理导入一个单独的库。**全局作用域中的代码会被自动当做程序的入口点**，所以你也不需要 ``main()``函数。  


```
print("Hello, world!")
```

**注：如果不想换行请使用``print("Hello", terminator:"")``**


1.不需要在每个语句结尾写上分号。

2.使用 ``let`` 来声明常量，使用 ``var`` 来声明变量。

```
var myVariable = 42
myVariable = 50
let myConstant = 42
```

3.一个常量的值，在编译的时候，并不需要有明确的值，但是只能为它**赋值一次**。

4.常量或者变量的**类型**必须和赋给它们的值**一样**。  

5.不用明确地声明类型，声明的同时赋值的话，编译器会自动推断类型。如果初始值没有提供足够的信息（或者没有初始值），那你需要在变量后面声明类型，用冒号分割。  

```
let implicitInteger = 70
let implicitDouble = 70.0
let explicitDouble: Double = 70
```

6.值永远不会被隐式转换为其他类型。如果你需要把一个值转换成其他类型，请显式转换。

```
let label = "The width is"
let width = 94
let widthLabel = label + String(width)
```

7.有一种更简单的把值转换成字符串的方法：把值写到括号中，并且在括号之前写一个反斜杠。  

```
let apples = 3
let oranges = 5
let appleSummary = "I have \(apples) apples."
let fruitSummary = "I have \(apples + oranges) pieces of fruit."
```

8.使用方括号 ``[]`` 来创建数组和字典，并使用下标或者键（key）来访问元素。**最后一个元素后面允许有个逗号**。  

```
var shoppingList = ["catfish", "water", "tulips", "blue paint"]
shoppingList[1] = "bottle of water"

var occupations = [
    "Malcolm": "Captain",
    "Kaylee": "Mechanic",
]
occupations["Jayne"] = "Public Relations"
```

9.要创建一个空数组或者字典，使用初始化语法。

```
let emptyArray = [String]()
let emptyDictionary = [String: Float]()
```

10.使用 ``if`` 和 ``switch`` 来进行条件操作，使用 ``for-in``、``for`` 、``while`` 和 ``repeat-while`` 来进行循环。**包裹条件和循环变量括号可以省略，但是语句体的大括号是必须的**。

```
let individualScores = [75, 43, 103, 87, 12]
var teamScore = 0
for score in individualScores {
    if score > 50 {
        teamScore += 3
    } else {
        teamScore += 1
    }
}
print(teamScore)
```

**注：``do-while`` 被 ``repeat-while`` 给取代。``do``关键字用来处理异常**。

11.在 ``if`` 语句中，条件必须是一个布尔表达式。  

12.可以一起使用 ``if`` 和 ``let`` 来处理值缺失的情况。这些值可由可选值来代表。一个可选的值是一个具体的值或者是 ``nil`` 以表示值缺失。在类型后面加一个**问号**来标记这个变量的值是可选的。  

```
var optionalString: String? = "Hello"
print(optionalString == nil)

var optionalName: String? = "John Appleseed"
var greeting = "Hello!"
if let name = optionalName {
    greeting = "Hello, \(name)"
}
```

13.另一种处理可选值的方法是通过使用 ``??`` 操作符来提供一个默认值。如果可选值缺失的话，可以使用默认值来代替。   

```
let nickName: String? = nil
let fullName: String = "John Appleseed"
let informalGreeting = "Hi \(nickName ?? fullName)"
```

14.``switch`` 支持任意类型的数据以及各种比较操作——不仅仅是整数以及测试相等。

```
let vegetable = "red pepper"
switch vegetable {
case "celery":
    print("Add some raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):
    print("Is it a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}
```

15.运行 ``switch`` 中匹配到的子句之后，程序会退出 ``switch`` 语句，并不会继续向下运行，所以不需要在每个子句结尾写 ``break``。 

**注：如果你需要代码贯穿到下一分支，你必须显式地实用 ``fallthrough`` 关键字**

```
let fullName = "John"

switch fullName{
case "John":
    fallthrough
case "John Appleseed":
    print(fullName)
default:
    print("hello stranger")
};
```

上面会打印出 ``John``;

16.使用 ``while`` 来重复运行一段代码直到不满足条件。  

```
var n = 2
while n < 100 {
    n = n * 2
}
print(n)

var m = 2
repeat {
    m = m * 2
} while m < 100
print(m)
```

17.可以在循环中使用 ``..<`` 来表示范围，也可以使用传统的写法，两者是等价的：

```
var firstForLoop = 0
for i in 0..<4 {
    firstForLoop += i
}
print(firstForLoop)

var secondForLoop = 0
for var i = 0; i < 4; ++i {
    secondForLoop += i
}
print(secondForLoop)
```

18.使用 ``func`` 来声明一个函数，使用名字和参数来调用函数。使用 ``->`` 来指定函数返回值的类型。  

```
func greet(name: String, day: String) -> String {
    return "Hello \(name), today is \(day)."
}
greet("Bob", day: "Tuesday")
```

19.使用元组来让一个函数返回多个值。

```
func calculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int) {
    var min = scores[0]
    var max = scores[0]
    var sum = 0

    for score in scores {
        if score > max {
            max = score
        } else if score < min {
            min = score
        }
        sum += score
    }

    return (min, max, sum)
}
let statistics = calculateStatistics([5, 3, 100, 3, 9])
print(statistics.sum)
print(statistics.2)
```

20.函数可以带有可变个数的参数，这些参数在函数内表现为数组的形式。

```
func sumOf(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
}
sumOf()
sumOf(42, 597, 12)
```

20.函数可以嵌套。

```
func returnFifteen() -> Int {
    var y = 10
    func add() {
        y += 5
    }
    add()
    return y
}
returnFifteen()
```

21.函数是第一等类型，这意味着函数可以作为**另一个函数的返回值**。

```
func makeIncrementer() -> (Int -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}
var increment = makeIncrementer()
increment(7)
```

22.函数也可以当做参数传入另一个函数。

```
func hasAnyMatches(list: [Int], condition: Int -> Bool) -> Bool {
    for item in list {
        if condition(item) {
            return true
        }
    }
    return false
}
func lessThanTen(number: Int) -> Bool {
    return number < 10
}
var numbers = [20, 19, 7, 12]
hasAnyMatches(numbers, condition: lessThanTen)
```

23.可以使用 ``{}`` 来创建一个匿名闭包。使用 ``in`` 将参数和返回值类型声明与闭包函数体进行分离。

```
numbers.map({
    (number: Int) -> Int in
    let result = 3 * number
    return result
})
```

24.**创建更简洁的闭包 1** —— 如果一个闭包的类型已知，比如作为一个回调函数，你可以忽略参数的类型和返回值。

```
let mappedNumbers = numbers.map({ number in 3 * number })
print(mappedNumbers)
```

25.**创建更简洁的闭包 2** —— 可以通过参数位置而不是参数名字来引用参数。 当一个闭包作为**最后一个参数**传给一个函数的时候，它可以直接跟在括号后面。当一个闭包是传给函数的**唯一参数**，可以完全忽略括号。 

```
let sortedNumbers = numbers.sort { $0 > $1 }
print(sortedNumbers)
```

26.声明类。

```
class Shape {
    var numberOfSides = 0
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}
```

27.要创建一个类的实例，在类名后面加上括号。使用点语法来访问实例的属性和方法。  

```
var shape = Shape()
shape.numberOfSides = 7
var shapeDescription = shape.simpleDescription()
```

28.一个构造函数来初始化类实例。使用 ``init`` 来创建一个构造器。

```
class NamedShape {
    var numberOfSides: Int = 0
    var name: String

    init(name: String) {
        self.name = name
    }

    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}
```

29.每个属性都需要赋值——无论是通过声明（就像 ``numberOfSides`` ）还是通过构造器（就像 ``name`` ）。  

30.如果你需要在删除对象之前进行一些清理工作，使用 ``deinit`` 创建一个析构函数。  

31.子类的定义方法是在它们的类名后面加上父类的名字，用冒号分割。  

32.子类如果要重写父类的方法的话，需要用 ``override`` 标记——如果没有添加 ``override`` 就重写父类方法的话编译器会报错。编译器同样会检测 ``override`` 标记的方法是否确实在父类中。  

```
class Square: NamedShape {
    var sideLength: Double

    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 4
    }

    func area() ->  Double {
        return sideLength * sideLength
    }

    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)."
    }
}
let test = Square(sideLength: 5.2, name: "my test square")
test.area()
test.simpleDescription()
```

33.除了储存简单的属性之外，属性可以有 getter 和 setter 。

```
class EquilateralTriangle: NamedShape {
    var sideLength: Double = 0.0

    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 3
    }

    var perimeter: Double {
        get {
            return 3.0 * sideLength
        }
        set {
            sideLength = newValue / 3.0
        }
    }

    override func simpleDescription() -> String {
        return "An equilateral triagle with sides of length \(sideLength)."
    }
}
var triangle = EquilateralTriangle(sideLength: 3.1, name: "a triangle")
print(triangle.perimeter)
triangle.perimeter = 9.9
print(triangle.sideLength)
```

在 ``perimeter`` 的 ``setter`` 中，新值的名字是 ``newValue``。你可以在 ``set`` 之后显式的设置一个名字。

注：如果想修改 ``newValue`` 这个默认的参数名为 ``newPerimeter ``，可以这么写：

```
   var perimeter: Double {
        get {
            return 3.0 * sideLength
        }
        set(newPerimeter) {
            sideLength = newPerimeter / 3.0
        }
    }
```

注意 ``EquilateralTriangle`` 类的构造器执行了**3**步：

- 设置子类声明的属性值
- 调用父类的构造器
- 改变父类定义的属性值。其他的工作比如调用方法、getters和setters也可以在这个阶段完成。

34.如果你不需要计算属性，但是仍然需要在设置一个新值之前或者之后运行代码，使用 **willSet** 和 **didSet**。

```
class TriangleAndSquare {
    var triangle: EquilateralTriangle {
        willSet {
            square.sideLength = newValue.sideLength
        }
    }
    var square: Square {
        willSet {
            triangle.sideLength = newValue.sideLength
        }
    }
    init(size: Double, name: String) {
        square = Square(sideLength: size, name: name)
        triangle = EquilateralTriangle(sideLength: size, name: name)
    }
}
var triangleAndSquare = TriangleAndSquare(size: 10, name: "another test shape")
print(triangleAndSquare.square.sideLength)
print(triangleAndSquare.triangle.sideLength)
triangleAndSquare.square = Square(sideLength: 50, name: "larger square")
print(triangleAndSquare.triangle.sideLength)
```

35.处理变量的可选值时，你可以在操作（比如方法、属性和子脚本）之前加 ``?``。如果 ``?`` 之前的值是 ``nil``，``?`` 后面的东西都会被忽略，并且整个表达式返回 ``nil`` 。否则，``?`` 之后的东西都会被运行。 

```
let optionalSquare: Square? = Square(sideLength: 2.5, name: "optional square")
let sideLength = optionalSquare?.sideLength
```

36.使用 ``enum`` 来创建一个枚举。就像类和其他所有命名类型一样，枚举可以包含方法。  

```
enum Rank: Int {
    case Ace = 1
    case Two, Three, Four, Five, Six, Seven, Eight, Nine, Ten
    case Jack, Queen, King
    func simpleDescription() -> String {
        switch self {
        case .Ace:
            return "ace"
        case .Jack:
            return "jack"
        case .Queen:
            return "queen"
        case .King:
            return "king"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.Ace
let aceRawValue = ace.rawValue
print(Rank.Jack.simpleDescription())
```

在上面的例子中，枚举原始值的类型是 ``Int``，所以你只需要设置第一个原始值。剩下的原始值会按照顺序赋值。你也可以使用字符串或者浮点数作为枚举的原始值。使用 ``rawValue`` 属性来访问一个枚举成员的原始值。  

37.使用 ``init?(rawValue:)`` 初始化构造器在原始值和枚举值之间进行转换。  

```
if let convertedRank = Rank(rawValue: 3) {
    let threeDescription = convertedRank.simpleDescription()
}
```

38.结构体和类有很多相同的地方，比如方法和构造器。它们之间最大的一个区别就是**结构体是传值，类是传引用**。  

39.使用 ``protocol`` 来声明一个协议。

```
protocol ExampleProtocol {
    var simpleDescription: String { get }
    mutating func adjust()
}
```

40.类、枚举和结构体都可以实现协议。

```
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class."
    var anotherProperty: Int = 69105
    func adjust() {
        simpleDescription += "  Now 100% adjusted."
    }
}
var a = SimpleClass()
a.adjust()
let aDescription = a.simpleDescription

struct SimpleStructure: ExampleProtocol {
    var simpleDescription: String = "A simple structure"
    mutating func adjust() {
        simpleDescription += " (adjusted)"
    }
}
var b = SimpleStructure()
b.adjust()
let bDescription = b.simpleDescription
```

**注意声明SimpleStructure时候mutating关键字用来标记一个会修改结构体的方法。SimpleClass的声明不需要标记任何方法，因为类中的方法通常可以修改类属性（类的性质）。**


41.使用 ``extension`` 来为现有的类型添加功能，比如新的方法和计算属性。你可以使用扩展在别处修改定义，甚至是从外部库或者框架引入的一个类型，使得这个类型遵循某个协议。

```
extension Int: ExampleProtocol {
    var simpleDescription: String {
        return "The number \(self)"
    }
    mutating func adjust() {
        self += 42
    }
}
print(7.simpleDescription)
```

42.可以像使用其他命名类型一样使用协议名——例如，创建一个有不同类型但是都实现一个协议的对象集合。当你处理类型是协议的值时，协议外定义的方法不可用。

```
let protocolValue: ExampleProtocol = a
print(protocolValue.simpleDescription)
```
即使 ``protocolValue`` 变量运行时的类型是 ``simpleClass``，编译器会把它的类型当做 ``ExampleProtocol``。这表示你不能调用类在它实现的协议之外实现的方法或者属性。

43.在尖括号里写一个名字来创建一个泛型函数或者类型。

```
func repeatItem<Item>(item: Item, numberOfTimes: Int) -> [Item] {
    var result = [Item]()
    for _ in 0..<numberOfTimes {
        result.append(item)
    }
    return result
}
repeatItem("knock", numberOfTimes:4)
```

44.也可以创建泛型函数、方法、类、枚举和结构体。

```
// Reimplement the Swift standard library's optional type
enum OptionalValue<Wrapped> {
    case None
    case Some(Wrapped)
}
var possibleInteger: OptionalValue<Int> = .None
possibleInteger = .Some(100)
```

45.在类型名后面使用 ``where`` 来指定对类型的需求，比如，限定类型实现某一个协议，限定两个类型是相同的，或者限定某个类必须有一个特定的父类。 

```
func anyCommonElements <T: SequenceType, U: SequenceType where T.Generator.Element: Equatable, T.Generator.Element == U.Generator.Element> (lhs: T, _ rhs: U) -> Bool {
    for lhsItem in lhs {
        for rhsItem in rhs {
            if lhsItem == rhsItem {
                return true
            }
        }
    }
    return false
}
anyCommonElements([1, 2, 3], [3])
```


