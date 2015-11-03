---
layout: post
title: "Xcode 插件不能使用问题"
date: 2015-07-09 20:22:10 +0800
comments: true
categories: iOS
---

最近因为某些原因，重装了一下Xcode，导致了Xcode的插件不能使用。中间捣鼓了一段时间才解决，故用单独的一篇博客来总结下。

前提准备：

[Xcode插件管理器：Alcatraz](https://github.com/supermarin/Alcatraz)

关于Alcatraz看文档就够了，安装方式也很简单。

由于是重装Xcode，中间出现了几个问题没有注意到的问题。

##0.安装Alcatraz前提是需要安装Xcode命令行工具

好久没动过了，文档也没仔细看，直接在命令行里输入了安装命令（如果已经装过，请调过）。但是文档里明明写着需要Xcode命令行工具，太大意了。见下图：

![Alcatraz](/images/xcode_plug/xcode_plug_1.png)

文档可能好久没有更新了，在Xcode的Preference>Downloads中没有发现Xcode Command Line Tools下载入口。于是在网上查到了下面的命令来安装它：  

xcode-select --install  

但是在命令行里输入后回车，出现了如下错误：

![Alcatraz](/images/xcode_plug/2.png)

于是各种搜索，在stackoverflow找到了答案：

[Xcode is not currently available from the Software Update server](http://stackoverflow.com/questions/19907576/xcode-is-not-currently-available-from-the-software-update-server)  

意思就是说：不要使用 xcode-select --install 命令来安装Xcode命令行工具啦，去官网上下载吧，下载地址在这里[Apple Xcode命令行工具下载](https://developer.apple.com/downloads/index.action?name=for%20Xcode%20-)，这里为保险起见，要下载对应Xcode版本的Xcode命令工具。

OK，到这里我们下载完毕并成功安装完Xcode命令工具。

##1.安装Alcatraz

用这个命令行：  

```curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh```

或者直接把[Alcatraz](https://github.com/supermarin/Alcatraz)源码下载下来，直接运行。

安装成功了，会出现下面这个界面：  

![Alcatraz](/images/xcode_plug/3.png)

注意红框里面的内容，重启下Xcode。

可能到这里，你以为安装成功了，我也是这么认为的。但就是没看到下图中的效果：  

![Alcatraz](/images/xcode_plug/4.png)

好吧，接着往下看

##2.一个比较有用的脚本

[脚本地址：https://github.com/cikelengfeng/RPAXU](https://github.com/cikelengfeng/RPAXU)

这个脚本是干什么，看下图：

![RPAXU](/images/xcode_plug/5.png)  

下载下来，进入到这个文件夹，使用 ```sh refreshPluginsAfterXcodeUpgrading.sh```命令，即可安装成功。

重启Xcode，这会儿再看看是不是下图这个结果：  

![Alcatraz](/images/xcode_plug/4.png)