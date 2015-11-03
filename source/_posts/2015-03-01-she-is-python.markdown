---
layout: post
title: "She is Python"
date: 2015-03-01 22:32:48 +0800
comments: true
categories: Python
---

最近看到各大牛，都在捣鼓Python。本着羡慕嫉妒恨的节奏，也热闹下。公司有个北大女医学博士，人家都会用Python写东西。跟着学习下，整理下这几天学到的东西和遇到的问题。

##0.安装Python

[官网下载](https://www.python.org)  

Download按钮很明显也很智能，根据你当前的系统，给你推荐下载哪一个。如下图：

![She_is_Python](/images/she_is_python/she_is_python_01.png) 

##1.选择Python版本

**Python 3.X 和 Python 2.X 到底该选择哪个？**  

听别人说改动挺大，本着跟准新技术的原则，我选择的是Python 3.X。

##2.学习教程

[官方教程](https://docs.python.org/3.4/tutorial/#the-python-tutorial) 
 
[官方库](https://docs.python.org/3.4/library/index.html#library-index)  

[第三方库](https://wiki.python.org/moin/UsefulModules)

[3.X 和 2.X版本的区别](http://docs.pythonsprints.com/python3_porting/py-porting.html) 
 
##3.pip

pip是啥，类似iOS开发中的CocoaPods，类库管理工具。

[如何安装pip，需自备梯子](https://pip.pypa.io/en/latest/installing.html) 
 
[如何使用pip](https://pip.pypa.io/en/latest/user_guide.html) 

我在安装的过程中，安装成功，但出现了一个没有pip这个命令的问题，应该是还没安装成功的原因吧。下方答案虽然没打对号，但是用这个``sudo easy_install pip``命令,就解决了。

[StackOverFlow上的解决方法](http://stackoverflow.com/questions/9780717/bash-pip-command-not-found)

##4.Get/POST请求数据

GET请求  

```
import urllib.request
import urllib

data = {}

data['id'] = '1' #请求参数

urlStr = 'http://XXX' #请求地址
urlParameters = urllib.parse.urlencode(data)

print(urlParameters)#打印参数

fullURLString = urlStr + '?' + urlParameters #拼接URL 地址 + 参数

resp = urllib.request.urlopen(fullURLString)
responseStr = resp.read()
responseDecodeStr = responseStr.decode("utf8")
print(responseDecodeStr)
```

POST请求

```
import urllib.request
import urllib

data = {}

data['id'] = '1'

urlStr = 'http://XXX'#请求地址
urlParameters = urllib.parse.urlencode(data)
postdata = urlParameters.encode('utf-8')  

print(urlParameters)
 
response = urllib.request.urlopen(urlStr,postdata)
responseStr = response.read()
responseDecodeStr = responseStr.decode("utf8")
print(responseDecodeStr)
```

##5.爬虫1：查找一个网站所有的链接地址 

这个例子是[这里面的](http://python.jobbole.com/81102/)，光看不敲，不是做技术的态度。参照他的，自己写一哈，不如他写的好，好多东西没考虑到。

```
import urllib.request
import urllib

from bs4 import BeautifulSoup
from collections import deque

#要爬取的地址

urlStr = 'http://www.douban.com'

#创建一个队列 用来存储要爬的URL

spiderQueue = deque()

#将爬取的地址放在队列里

spiderQueue.append(urlStr)

#创建一个集合 用来存储已经爬过的URL

visitedSet = set() 

#创建循环开始扫描

while spiderQueue:
    urlStr = spiderQueue.popleft() #队首元素

    print (urlStr)

    try:
        response = urllib.request.urlopen(urlStr)
        
        responseStr = response.read()#URL网页源码

        soup = BeautifulSoup(responseStr)
        
        linksArray = soup.findAll('a')
        #print('链接地址:' + str(linksArray))

        for link in linksArray:
            hrefStr = link.get('href')
            print('链接地址 ' + hrefStr + '\n')
                
            if 'http' in hrefStr and hrefStr not in visitedSet:
                visitedSet |={hrefStr}
                spiderQueue.append(hrefStr)
                print('加入队列 --->  ' + hrefStr + '\n')

    except:
        print('遇到异常')
        continue
```


##6.爬虫2：爬妹子图片

这个可能有点邪恶，是[这篇文章](http://www.jianshu.com/p/b87413a9307e)教我的， 我多添加了些代码。可以爬出这个[豆瓣妹子网站](http://www.dbmeizi.com)的所有图片。可以再优化下。

```
#抓妹子图片
#根据http://dbmeizi.com/?p=1 
#li class = 'span3'
#img data-bigimg属性

import urllib.request
import urllib

from bs4 import BeautifulSoup

while 1:#这个地方可以优化下 要不爬到Duang
    pageNumber = 1 #页码

    urlStr = 'http://dbmeizi.com/?p=' + str(pageNumber)#URL 地址
    
    response = urllib.request.urlopen(urlStr)
    responseStr = response.read() #网页源代码

    soup = BeautifulSoup(responseStr)

    liTagsArray = soup.findAll('li',attrs = {'class':'span3'})

    for li in liTagsArray:
         imageTags = li.findAll('img')#img标签

         for image in imageTags:
            imageLink = image.get('data-bigimg')#data-src  bigimg
            imageName = image.get('data-id')
            fileSavePath = '/Users/iYiming/Downloads/meizi/%s.jpg' % imageName
            urllib.request.urlretrieve(imageLink,fileSavePath)
            
            print (fileSavePath) 


    pageNumber += 1
    urlStr = 'http://dbmeizi.com/?p=' + str(pageNumber)#URL 地址
```

展示下爬到的东西： 

![She_is_Python](/images/she_is_python/she_is_python_02.png) 

##7.其他

[一个不错教程：用Python 3开发网络爬虫](http://python.jobbole.com/81102/)

中间遇到的问题：

[Python3.4版本 和 2.7版本的不同](http://segmentfault.com/q/1010000000340908/a-1020000000340928)  
 
[IDLE中不能输入中文的问题](http://www.zhihu.com/question/26532408)

有疑惑的问题

0.IDLE如何带自动提示，太难受了  
1.没有提示，这么多类库，怎么用？
  
感谢上面所有链接的作者！

