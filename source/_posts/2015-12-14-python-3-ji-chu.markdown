---
layout: post
title: "Python 3 基础"
date: 2015-12-14 23:23:07 +0800
comments: true
categories: Python
---

![python-3-ji-chu](/images/python-3-ji-chu/0.jpg)

记忆力不好，为方便以后查看，特整理之。  

书籍资料为廖雪峰的教程：  

[http://t.cn/R2PDyWN](http://t.cn/R2PDyWN)  

0.打印字符串

```
#!/usr/bin/env python3  # 为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释
# -*- coding: utf-8 -*- # 为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。

a = 'Sam'
b = 'Jim'

print('a is %s,b is %s' % (a,b))
print('a is',a,',b is',b)
```

1.条件判断

```
a = 'Tom'
b = 'Jim'

if a== 'Sam':
    print('a is Sam')
elif a == "Tom":
    print('a is Tom')

if b == 'Sam':
    print('b is Sam');
else:
    print('b is not Sam')
```

2.``for`` 循环

```
c = ['Sam','Jim','Tom']
d = {'name' : 'Jim','age' : 21}

for name in c:
    print(name)


for index,name in enumerate(c):
    print(index,name)

for key in d:
    print('key =',key)

for value in d.values():
    print('value = ',value)

for key,value in d.items():
    print('%s : %s' % (key,value))
```

3.元组

```
def sayHello(name):
    print('Hello %s!' % name)

classmates = ('Jim','Tom','Sam')

student = classmates[-1]
print(student)
sayHello(student)
```

4.输入  

```
print('请输入姓名:')
name = input()
sayHello(name)

```

5.list  

Python 内置的一种数据类型是列表：list。list 是一种有序的集合，可以随时添加和删除其中的元素。

```
names = ['Jim','Tom']
names.append('Mike')
names.insert(1,'Sam')
names.pop(0)
names[0] = 'Same'
print('names = %s and names have %d values' % (names,len(names)))
```

6.dict  

Python 内置了字典：dict 的支持，dict全称dictionary，在其他语言中也称为map，使用键-值（key-value）存储，具有极快的查找速度。

```
info = {'name':'Tom','age' : 25}
info['gender'] = 'M'
info['age'] = 24
info.pop('age')
print(info)
```

7.set   

set 和 dict 类似，也是一组 key 的集合，但不存储 value。由于 key 不能重复，所以，在 set 中，没有重复的 key。

```
names = set(['Jim','Sam'])
names.add('Mike')
names.remove('Jim')
print(names)
```

8.``range()`` 函数，可以生成一个整数序列。

```
>>> list(range(5))
[0, 1, 2, 3, 4]
```

9.Python 是对大小写敏感的。

10.Python使用缩进来组织代码块，请务必遵守约定俗成的习惯，坚持使用 **4** 个空格的缩进。

11.空值是 Python 里一个特殊的值，用 None 表示。None 不能理解为 0，因为 0 是有意义的，而 None 是一个特殊的空值。  

12.当Python解释器读取源代码时，为了让它按UTF-8编码读取，我们通常在文件开头写上这两行：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

第 1 行注释是为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；  

第 2 行注释是为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。  

13.``%d``表示整数;``%f``表示浮点数;``%s``表示字符串;``%x``表示十六制整数; 

```
>>> '%2d-%02d' % (3, 1)
' 3-01'
>>> '%.2f' % 3.1415926
'3.14'
```

14.在一个模块中，我们可能会定义很多函数和变量，但有的函数和变量我们希望给别人使用，有的函数和变量我们希望仅仅在模块内部使用。在 Python 中，是通过 ``_`` 前缀来实现的。

15.模块搜索路径

当我们试图加载一个模块时，Python会在指定的路径下搜索对应的.py文件，如果找不到，就会报错：

```
>>> import mymodule
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named mymodule
```

默认情况下，Python 解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在 ``sys`` 模块的 ``path`` 变量中：

```
>>> import sys
>>> sys.path
['', '/Library/Frameworks/Python.framework/Versions/3.4/lib/python34.zip', '/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4', '/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/plat-darwin', '/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/lib-dynload', '/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/site-packages']
```

如果我们要添加自己的搜索目录，有两种方法：

一是直接修改sys.path，添加要搜索的目录：

```
>>> import sys
>>> sys.path.append('/Users/michael/my_py_scripts')
```
这种方法是在运行时修改，运行结束后失效。

第二种方法是设置环境变量PYTHONPATH，该环境变量的内容会被自动添加到模块搜索路径中。设置方式与设置Path环境变量类似。注意只需要添加你自己的搜索路径，Python自己本身的搜索路径不受影响。

16.默认情况下，``dict`` 迭代的是 ``key``。如果要迭代 ``value``，可以用 ``for value in d.values()``，如果要同时迭代 ``key`` 和 ``value``，可以用 ``for k, v in d.items()`` 。

```
>>> d = {'a': 1, 'b': 2, 'c': 3}
>>> for key in d:
...     print(key)
...
a
c
b
```

17.当我们使用 ``for`` 循环时，只要作用于一个可迭代对象，``for`` 循环就可以正常运行，而我们不太关心该对象究竟是 ``list`` 还是其他数据类型。那么，如何判断一个对象是可迭代对象呢？方法是通过 ``collections`` 模块的 ``Iterable`` 类型判断：

```
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```

18.Python内置的 ``enumerate`` 函数可以把一个 ``list`` 变成索引-元素对，这样就可以在``for`` 循环中同时迭代索引和元素本身：  

```
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C

>>> for x, y in [(1, 1), (2, 4), (3, 9)]:
...     print(x, y)
...
1 1
2 4
3 9
```

19.切片

切片操作十分有用。我们先创建一个 ``0-99`` 的数列：

```
>>> L = list(range(100))
>>> L
[0, 1, 2, 3, ..., 99]
```

可以通过切片轻松取出某一段数列。比如前10个数：

```
>>> L[:10]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

后10个数：  

```
>>> L[-10:]
[90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
```

前11-20个数：  

```
>>> L[10:20]
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

前 10 个数，每两个取一个：  

```
>>> L[:10:2]
[0, 2, 4, 6, 8]
```

所有数，每 5 个取一个：  

```
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```

甚至什么都不写，只写 ``[:]`` 就可以原样复制一个 ``list``：  

```
>>> L[:]
[0, 1, 2, 3, ..., 99]
```

tuple也是一种 ``list``，唯一区别是tuple不可变。因此，tuple也可以用切片操作，只是操作的结果仍是tuple：  

```
>>> (0, 1, 2, 3, 4, 5)[:3]
(0, 1, 2)
```

字符串'xxx'也可以看成是一种list，每个元素就是一个字符。因此，字符串也可以用切片操作，只是操作结果仍是字符串：  

```
>>> 'ABCDEFG'[:3]
'ABC'
>>> 'ABCDEFG'[::2]
'ACEG'
```
在很多编程语言中，针对字符串提供了很多各种截取函数（例如，``substring``），其实目的就是对字符串切片。Python 没有针对字符串的截取函数，只需要切片一个操作就可以完成，非常简单。

20.列表生成式  

如果要生成 ``[1x1, 2x2, 3x3, ..., 10x10]`` 怎么做？  

方法一是循环：

```
>>> L = []
>>> for x in range(1, 11):
...    L.append(x * x)
...
>>> L
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```
但是循环太繁琐，而列表生成式则可以用一行语句代替循环生成上面的list：  

```
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

写列表生成式时，把要生成的元素 ``x * x`` 放到前面，后面跟 ``for`` 循环，就可以把``list`` 创建出来，十分有用，多写几次，很快就可以熟悉这种语法。
