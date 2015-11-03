---
layout: post
title: "使用 Github 搭建博客"
date: 2014-10-26 01:26:53 +0800
comments: true
categories: IT
---

今天折腾了一晚上，终于把自己的博客搭建好了，原先就弄过一遍，重新搭建，还是花费了大把脑细胞,主要原先没有做笔记，导致还是和第一次搭建一样啥也不会啥也不懂，不过确实遇见了原先没有遇见的坑。现在记录一下，提醒自己，好记性不如烂笔头！  

###0.注册Github账户
###1.创建一个仓库
假如你的Github用户名为：xiaoming，那你创建的仓库名为：xiaoming.github.io 详情请参照
[Github官网文档](https://pages.github.com/)  
###2.使用Octopress博客引擎
[Octopress](http://octopress.org/docs/)是一个为专门管理博客的引擎工具，可以通过他方便的创建管理博客文章。
#####2.1使用Octopress前的配置
0.需要提前[安装Git](http://git-scm.com/download)  
1.安装Ruby 1.9.3 或者更高版本 [rbnv](http://octopress.org/docs/setup/rbenv/)或者 [RVM](http://octopress.org/docs/setup/rvm/)  
#####2.2初始化Octopress  
0.开启终端输入命令(一行一行的输入)
>0.克隆 octopress源码到octopress文件夹  
>git clone git://github.com/imathis/octopress.git octopress  
>1.进入 octopress文件夹  
>cd octopress  
>2.安装依赖  
>sudo gem install bundler  
>bundle install  
>3.安装Octopress默认主题  
>rake install  

1.Github的设置  
>0.关联你的上面Github官网中创建的仓库  
>rake setup_github_pages  
>1.上面的命令输入完毕回车后，会提示输入Github仓库的地址 格式例如：git@github.com:XXX/XXX.github.io.git（XXX是GitHub用户名）  
>2.生成博客  
>rake generate  
>3.发布博客  
>rake deploy  
>4.使用Github Pages
>rake setup_github_pages 
>5.上面的命令会要求输入你的仓库url地址  

2.保存你的source代码到Github(有两个分支：master是现实你的博客内容，source是octopress生成代码)
>git add .
>git commit -m 'your message'
>git push origin source  

3.设置完等待10分钟后就可以访问 http://xxx.github.io 就可以看到自己的博客啦  

4.如果以上遇见问题可以看看参考文章里是否有回答  

5.自定义域名 (如果新域名请参看[新申请域名连接到Github博客](http://jser.me/2012/11/17/%E8%87%AA%E5%AE%9A%E4%B9%89github-pages%E7%9A%84%E5%9F%9F%E5%90%8D.html)  设置好10分钟后才能看到效果)  
>echo 'xxx.com' >> source/CNAME  
>rake generate  
>rake deploy 




###参考文章
[Octopress官网文档](http://octopress.org/docs)  
[Github官网文档](https://pages.github.com/)  
[象写程序一样写博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)  
[本人博客使用的Octopress主题](https://github.com/lucaslew/whitespace)  
[第三方Octopress主题](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)  
[StackOverflow解决中间出现的问题](http://stackoverflow.com/questions/17609453/rake-gen-deploy-rejected-in-octopress/18037705)  
[新申请域名连接到Github博客](http://jser.me/2012/11/17/%E8%87%AA%E5%AE%9A%E4%B9%89github-pages%E7%9A%84%E5%9F%9F%E5%90%8D.html)  
[Generating SSH keys](https://help.github.com/articles/generating-ssh-keys/)