---
layout: post
title: "Swift 2.0 —— 控制流"
date: 2015-11-05 18:49:40 +0800
comments: true
categories: iOS
---

0.Swift 的 ``switch`` 语句比 C 语言中更加强大。在 C 语言中，如果某个 ``case`` 不小心漏写了 ``break``，这个 ``case`` 就会贯穿至下一个 ``case``，Swift 无需写 ``break``，所以不会发生这种贯穿的情况。``case`` 还可以匹配更多的类型模式，包括区间匹配（range matching），元组（tuple）和特定类型的描述。``switch`` 的 ``case`` 语句中匹配的值可以是由 ``case`` 体内部临时的常量或者变量决定，也可以由 ``where`` 分句描述更复杂的匹配条件。

1.``for`` 循环中，如果你不需要知道区间序列内每一项的值，你可以使用下划线（``_``）替代变量名来忽略对值的访问：

```
let base = 3
let power = 10
var answer = 1
for _ in 1...power {
    answer *= base
}
print("\(base) to the power of \(power) is \(answer)")
// 输出 "3 to the power of 10 is 59049"
```

2.``while`` 循环的另外一种形式是 ``repeat-while``，它和 ``while`` 的区别是在判断循环条件之前，先执行一次循环的代码块，然后重复循环直到条件为 ``false``。Swift语言的repeat-while循环合其他语言中的do-while循环是类似的。

```
repeat {
    // 顺着梯子爬上去或者顺着蛇滑下去
    square += board[square]
    // 掷骰子
    if ++diceRoll == 7 { diceRoll = 1 }
    // 根据点数移动
    square += diceRoll
} while square < finalSquare
print("Game over!")
```

3.我们可以使用元组在同一个 ``switch`` 语句中测试多个值。元组中的元素可以是值，也可以是区间。另外，使用下划线（ ``_`` ）来匹配所有可能的值。

```
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// 输出 "(1, 1) is inside the box"
```

4.``case`` 分支的模式允许将匹配的值绑定到一个临时的常量或变量，这些常量或变量在该 ``case`` 分支里就可以被引用了——这种行为被称为值绑定（``value binding``）。  

```
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// 输出 "on the x-axis with an x value of 2"
```

5.``case`` 分支的模式可以使用 ``where`` 语句来判断额外的条件。

```
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// 输出 "(1, -1) is on the line x == -y"
```

6.当在一个 ``switch`` 代码块中使用 ``break`` 时，会立即中断该 ``switch`` 代码块的执行，并且跳转到表示 ``switch`` 代码块结束的大括号(``}``)后的第一行代码。

7.在 ``switch`` 中 如果某一分支没有语句，必须使用 ``break``,不能不写任何东西，就再写 ``case`` :

```
let myNumber = 1

switch myNumber{
case 1:
    break
case 2:
    print("2")
default:
    print("nil")
}
```

8.如果你确实需要 C 风格的**贯穿**的特性，你可以在每个需要该特性的 case 分支中使用 ``fallthrough`` 关键字。下面的例子使用 ``fallthrough`` 来创建一个数字的描述语句。

```
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
print(description)
// 输出 "The number 5 is a prime number, and also an integer."
```

9.在 Swift 中，你可以在循环体和 ``switch`` 代码块中嵌套循环体和 ``switch`` 代码块来创造复杂的控制流结构。然而，循环体和 ``switch`` 代码块两者都可以使用 ``break`` 语句来提前结束整个方法体。因此，显式地指明 ``break`` 语句想要终止的是哪个循环体或者 ``switch`` 代码块，会很有用。类似地，如果你有许多嵌套的循环体，显式指明 ``continue`` 语句想要影响哪一个循环体也会非常有用。

为了实现这个目的，你可以使用标签来标记一个循环体或者 ``switch`` 代码块，当使用 ``break`` 或者 ``continue`` 时，带上这个标签，可以控制该标签代表对象的中断或者执行。

10.产生一个带标签的语句是通过在该语句的关键词的同一行前面放置一个标签，并且该标签后面还需带着一个冒号。下面是一个 ``while`` 循环体的语法，同样的规则适用于所有的循环体和 ``switch`` 代码块。

```
labelName: while condition {
    statements
}
```

```
gameLoop: while square != finalSquare {
    if ++diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // 到达最后一个方块，游戏结束
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // 超出最后一个方块，再掷一次骰子
        continue gameLoop
    default:
        // 本次移动有效
        square += diceRoll
        square += board[square]
    }
}
print("Game over!")
```

11.像 ``if`` 语句一样，``guard`` 的执行取决于一个表达式的布尔值。我们可以使用 ``guard`` 语句来要求条件必须为真时，以执行 ``guard`` 语句后的代码。不同于``if`` 语句，一个 ``guard`` 语句总是有一个 ``else`` 分句，如果条件不为真则执行 ``else`` 分句中的代码。

```
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }

    print("Hello \(name)")

    guard let location = person["location"] else {
        print("I hope the weather is nice near you.")
        return
    }

    print("I hope the weather is nice in \(location).")
}

greet(["name": "John"])
// prints "Hello John!"
// prints "I hope the weather is nice near you."
greet(["name": "Jane", "location": "Cupertino"])
// prints "Hello Jane!"
// prints "I hope the weather is nice in Cupertino."
```

12.Swift 有检查 API 可用性的内置支持，这可以确保我们不会不小心地使用对于当前部署目标不可用的 API。  

```
//可用性条件指定了在 iOS 系统上，if段的代码仅会在 iOS 9 及更高版本的系统上执行；
//在 OS X，仅会在 OS X v10.10 及更高版本的系统上执行。
//最后一个参数，*，是必须写的，用于处理未来潜在的平台。

if #available(iOS 9, OSX 10.10, *) {
    // 在 iOS 使用 iOS 9 的 API, 在 OS X 使用 OS X v10.10 的 API
} else {
    // 使用先前版本的 iOS 和 OS X 的 API
}
```