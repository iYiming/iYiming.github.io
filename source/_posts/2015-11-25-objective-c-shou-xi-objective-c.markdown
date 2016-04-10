---
layout: post
title: "Objective-C 2.0 —— 熟悉 Objective-C"
date: 2015-11-25 21:34:58 +0800
comments: true
categories: iOS
---

![1.jpg](/images/objective-c-shou-xi-objective-c/1.jpg)

本系列文章，整理自 Effective Objective-C 2.0 这本书。

书籍购买地址：

[京东地址](http://item.jd.com/11402853.html)

[亚马逊地址](http://t.cn/RUBRt6m)

下文整理自 **第一章 熟悉 Objective-C**：

0.若是用过另一种面向对象语言（C++ 或 Java），那么就能理解 Objective-C 所用的许多范式与模板了。然而语法上也许会显得陌生，因为该语言使用“消息结构”（messaging structure）而非“函数调用”（function calling）。Objective-C 语言由 Smalltalk 演化而来，后者是消息型语言的鼻祖。

1.关键区别在于：**使用消息结构的语言，其运行时所应执行的代码由运行环境来决定；而使用函数调用的语言，则由编译器决定**。

2.对 1 注解：如果范例代码中调用的函数是多态的，那么在运行时就要按照“虚方法表”（virtual table）来查出到底应该执行哪个函数实现。而采用消息结构的语言，不论是否多态，总是在运行时才会去查找索要执行的方法。实际上，编译器甚至不关心接收消息的对象是何种类型。接收消息的对象也要在运行时处理，其过程叫做“动态绑定”（dynamic binding）。

3.Objective-C 的重要工作都由“运行期组件”（runtime component）而非编译器来完成。使用 Objective-C 的面向对象特性所需的全部数据结构及函数都在运行期组件里面。举例来说，运行期组件中含有全部内存管理方法。**运行期组件本质上就是一种与开发者所编写代码相连接的“动态库”（dynamic library），其代码能把开发者编写的所有程序粘合起来。这样的话，只需更新运行期组件，即可提升应用程序性能。**而那种许多工作都在“编译期”（compile time）完成的语言，若想获得类似的性能提升，则要重新编译应用程序代码。

4.初始化一个字符串：

```
NSString *someString = @"The string";
```

这种语法基本上是照搬 C 语言的。它声明了一个名为 ``someString`` 的变量,其类型是 ``NSString*``。也就是说，此变量为指向 ``NSString`` 的 指针。所有 Objective-C 语言的对象都必须这样声明，因为对象所占内存总是分配在“堆空间”（heap space）中，而绝不会分配在“栈”（stack）上。

5.分配在堆中的内存必须直接管理，而分配在栈上用于保存变量的内存则会在其栈帧弹出时自动清理。

6.Objective-C 将堆内存管理抽象出来了。不需要用 ``malloc`` 及 ``free`` 来分配或释放对象所占内存。Objective-C 运行期环境就把这部分工作抽象为一套内存管理架构，名叫“引用计数”。

7.在 Objective-C 代码中，有时会遇到定义里不含 ``*`` 的变量,它们可能会使用“栈空间”（stack space）。这些变量所保存的不是 Objective-C 对象。比如 CoreGraphic 框架中的 ``CGRect`` 是 C 结构体。整个系统框架都在用这种结构体，因为如果改用 Objective-C 对象来做的话，性能会受到影响。与创建结构体相比，创建对象还需要额外开销，例如分配及释放堆内存等。

8.Objective-C 为 C 语言添加了面向对象特性，是其超集。Objective-C 使用动态绑定的消息结构，也就是说，在运行时才会检查对象类型。接收一条消息之后，究竟应执行何种代码，由运行期环境而非编译器来决定。

9.除非确有必要，否则不要引入头文件。一般来说，应在某个类的头文件中使用**向前声明**（如果提前声明 ``Person`` 类，使用：``@class Person;``）来提及别的类，并在实现文件中引入那些类的头文件。这样做可以尽量降低类之间的耦合。**若引入许多根本用不到的内容，会增加编译时间**。

10.看下面这个例子(仅列出.h文件)：

```
//YMPerson.h
#import <Foundation/Foundation.h>
#import "YMEmployer.h"

@interface YMPerson : NSObject

@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, strong) YMEmployer *employer;

@end

//YMEmployer.h
#import <Foundation/Foundation.h>
#import "YMPerson.h"

@interface YMEmployer : NSObject

- (void) addEmployee:(YMPerson *) person;

@end
```

这样会出现问题，虽然使用 ``#import`` 而非 ``#include`` 指令虽然不会导致死循环，但这却意味着两个类**有一个无法被正确编译**。而使用向前声明却可以解决这个问题。

11.有时候必须要在头文件中引入其他头文件：

- 如果你写的类继承自某个超类，则必须引入定义那个超类的头文件。
- 如果要声明你写的类遵从某个协议，那么该协议必须有完整定义，且不能使用向前声明。向前声明只能高度编译器有某个协议，而此时编译器却要知道该协议中定义的方法。

12.引入协议头文件，最好把协议单独放在一个头文件中。然而有些协议，例如“委托协议”，就不用单独写一个头文件了，在这种情况下，协议只有与接收协议委托的类放在一起定义才有意义。

13.每次在头文件中引入其他头文件之前，都要先问问自己这样做是否确有必要。如果可以用向前声明取代引入，那么就不要引入。若因为要实现属性。实例变量或者要遵循协议而必须引入头文件，则应尽量将其一直“ class-continuation 分类”。这样不仅可以缩减编译时间，而且还能降低彼此依赖程度。若是依赖关系过于复杂，则会给维护带来麻烦，而且，如果只要把代码的某个部分开放为公共 API 的话，太复杂的依赖关系也会出现问题。

14.应该使用字面量语法来创建字符串、数值、数组、字典。与创建此类对象的常规方法相比，这么做更加简明扼要。

15.用字面量语法创建数组时要注意，若数组元素对象中有 ``nil``，则会抛出异常，因为字面量语法实际上指示一种语法糖（ syntactic sugar ），其效果等于是先创建了一个数组，然后把方括号内的所有对象都加到这个数组中。这样会让我们提前知道错误，而使用最原始的方法虽不会抛出异常使我们不好找到异常。

16.字典中的对象和键必须都是 Objective-C 对象，所以不能把整数值（如 10 ）直接放进去，而是将其封装在``NSNumber``实例中（ ``@10`` ）才行。

17.使用字面量语法创建出来的字符串、数组、字典对象都是不可变的（immutable）。若想要可变版本的对象，则需复制一份：

```
NSMutableArray *mutable = [[@1, @2, @3] mutableCopy];
```
18.字面量语法有个小小的限制，就是除了字符串以外，所创建出来的对象必须属于 Foundation 框架才行。

19.在编写代码时经常要定义常量。掌握了 Objective-C 与 C 语言基础的人，也许会用这种方法来做：

```
#define ANIMATION_DURATION 0.3
```

上述预处理命令会把源代码中的 ANIMATION_DURATION 字符串替换为 ``0.3`` 。这可能是你想要的效果，不过这样定义出来的常量**没有类型信息**。

20.要想解决上面的问题，应该设法利用编译器的某些特性才对。有个办法比用预处理指令来定义常量更好。比方说，下面这行代码就定义了一个类型为 ``NSTimeInteval`` 的常量：

```
static const NSTimeInterval kAnimationDuration = 0.3;
```

这样定义的常量包含类型信息，其好处是清楚地描述了常量的含义。由此可知该常量类型为 ``NSTimeInterval``,这有助于为其编写开发文档。如果要定义许多变量，那么这种方式能令稍后阅读代码的人更易理解其意图。

21.**若常量局限于某“编译单元”（也就是“实现文件“）之内，则在前面加字母 ``k``；若常量在类之外可见，则通常以类名为前缀**。

22.若不打算公开某个常量，则应将其定义在使用该常量的实现文件里。变量一定要同时用 ``static`` 与 ``const`` 来声明。

- 若视图修改由 ``const`` 修饰符所声明的变量，那么编译器就会报错。
- ``static`` 修饰符则意味着该变量仅在定义此变量的编译单元中可见。编译器每收到一个便一单元，就会输出一份"目标文件"。在 Objective-C 的语境下，“编译单元”一词通常指每个类的实现文件（以 ``.m`` 为后缀名）。假如声明此变量时不加 ``static``,则编译器会为它创建一个“外部符号”（external symbol）。

23.有时候需要对外公开某个常量。比方说，可能要在类代码中调用 ``NSNotificationCenter`` 来通知他人。用一个对象来派发通知，令其他欲接收通知的对象向该对象注册，这样就能实现此功能了。派发通知时，需要使用字符串来表示此项通知的名称，而这个名字就可以声明为一个外界可见的长值变量(constant variable)。这样的话，注册者无须知道实际字符串值，只需以常值变量来注册自己想要接收的通知即可。应该这样定义：

```
//在 .h 文件中
extern NSString *const YMStringConstant;

//在 .m 文件中
NSString *const YMStringConstant = @"Value";
```

**这个常量在头文件中“声明”，且在实现文件中“定义”。注意 ``const`` 修饰符在常量类型中的位置。**

24.注意常量的名字。**为避免名称冲突，最好是用与之相关的类名做前缀**。系统框架中一般都这样做。例如 UIKit 就按照这种方式来声明用作通知名称的全局常量。其中有类似 ``UIApplicationDidEnterBackgroundNotification`` 与 ``UIApplicationWillEnterForegroundNotification`` 这样的常量名。

25.不要用预处理指令定义常量。这样定义出来的常量不含类型信息，编译器只是会在编译前据此执行查找与替换操作。即使有人重新定义了常量值，**编译器也不会产生警告信息**，这样导致应用程序中的常量值不一致。  

26.在实现文件中使用 ``static const`` 来定义”只在编译单元内可见的常量“。由于此类常量不在全局符号表中，所以无需为其名称加前缀。

27.在头文件中使用 ``extern`` 来声明全局变量，并在相关实现文件中定义其值。这种常量要出现在全局符号表中，所以其名称应加以区隔，通常用与之相关的类名称做前缀。

28.应该用枚举来表示状态机的状态、传递给方法的选项以及状态码等值，给这些值起个易懂的名字。

29.如果把传递给某个方法的选项表示为枚举类型，而多个选项又可同时使用。那么就讲各选项值定义为 ``2`` 的幂，以便通过按位或操作将其组合起来。

30.用 ``NS_ENUM`` 与 ``NS_OPTIONS`` 宏来定义枚举类型，并指明其底层数据类型。这样做可以确保枚举是用开发者所选的底层数据类型实现出来的，而不会采用编译器所选的类型。

31.我们总习惯在 ``switch`` 语句中加上 ``default`` 分支。然而，若是用枚举来定义状态机，则最好不要有 ``default`` 分支。这样的话，如果稍后又加了一种状态，那么编译器就会发出警告信息，提示新加入的状态并未在 ``switch`` 分支中处理。假如写上了 ``default`` 分支，那么它就会处理这个新状态，从而导致编译器不发警告信息。

32.枚举分为：普通枚举(``NS_ENUM ``) 和 选项枚举（``NS_OPTIONS ``），其中选项枚举，如系统中的 ``UIViewAutoresizing``，这么定义:

```
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

选项枚举的原理，见下图：

![0.jpg](/images/objective-c-shou-xi-objective-c/0.jpg)

