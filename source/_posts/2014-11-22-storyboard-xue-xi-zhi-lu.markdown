---
layout: post
title: "Storyboard 学习之路"
date: 2014-11-22 22:30:55 +0800
comments: true
categories: iOS
published: false
---

不知道什么原因，一开始做iOS时就不喜欢用拖控件的方式来做界面布局，可能是深受.NET的影响。原先看过[大牛喵神的一篇文章](http://onevcat.com/2013/12/code-vs-xib-vs-storyboard/)，
来介绍纯代码、xib、storyboard布局的优缺点。现在随着iPhone6和iPhone 6 plus的发布，越来越感觉到使用代码布局的麻烦和缺点，
最近WatchKit的发布，更感觉到Storyboard的重要性，所以写篇博客简单整理下，自己也算是个StoryBoard的新手，从头来学！

##0.Storyboard简介

![Layer Subclass]({{ root_uri }}/images/storyboard_guid/storyboard_guid_01.png)  

红色圈出的是初始化控制器  
蓝色圈出的是segue  

control拖拽：按住Control拖拽（包括添加segue、Autolayout约束）  

两个操作：  
Editor > Resolve Auto Layout Issues > “Reset to Suggested Constraints” Xcode选择恰当的约束  
Editor > Resolve Auto Layout Issues > Clear Constraints 清除所有约束

##1.使用AutoLayout

选择控件，按住control键向不同的方向拖到控件的父视图会看到以下几种：  
向上拖动：Top Space to Top Layout Guide 距顶部  
向下拖动：Bottom Space to Bottom Layout Guide 距底部  
向左拖动：Leading Space to container margin 距左侧  
向右拖动：Trailing Space to container margin 距右侧  


##2.实例：UITableViewCell自适应高度  

0.两种不同Identifier的UITableViewCell：GreenIdentifier，RedIdentifier（两种Cell中颜色不同）  
1.为UILabel添加 居上、居下、宽度、居左约束，如下图：  
![Layer Subclass]({{ root_uri }}/images/storyboard_guid/storyboard_guid_02.png)  

2.主要代码如下：
```
//获取UITableViewCell
- (UITableViewCell *) cellForIndexPath:( NSIndexPath *)indexPath
{
    NSUInteger row = [indexPath row];
    NSString *identifier = nil;
    if (row%2 == 0) {
        identifier = @"GreenIdentifier";
    }else{
        identifier = @"RedIdentifier";
    }

    UITableViewCell *cell = [self.tableView dequeueReusableCellWithIdentifier:identifier];

    UILabel *cellLabel = (UILabel *)[cell viewWithTag:1];
    cellLabel.text = [listArray objectAtIndex:row];
    [cellLabel setNeedsLayout];
    [cellLabel setNeedsUpdateConstraints];

    return cell;
}

#pragma mark - Table view data source
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    // Return the number of sections.
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return [listArray count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 获取缓存的 Cell
    UITableViewCell *cachedCell = [_cellCache objectForKey:@(indexPath. row)];

    // 如果没有缓存再次调用 getCellFromIndexPath 来创建 Cell
    if (!cachedCell){
        return [self cellForIndexPath:indexPath];
    }
    
    return cachedCell;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 获取 Cell
    UITableViewCell *cell = [self cellForIndexPath:indexPath];

    // 缓存 Cell
    [_cellCache setObject:cell forKey: @(indexPath.row) ];

    [cell setNeedsLayout];
    [cell layoutIfNeeded];

    CGSize size = [cell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];
    NSLog(@"h=%f", size.height + 1);
    return 1  + size.height;//加1是因为UITableViewCell的分割线高度为1
}
```

参考文章  

[苹果官方教程](https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOS/SecondTutorial.html)  
[Storyboard的简单使用](http://my.oschina.net/plumsoft/blog/53886)  
[动态计算UITableViewCell高度详解](http://www.ifun.cc/blog/2014/02/21/dong-tai-ji-suan-uitableviewcellgao-du-xiang-jie/)  
[Autolayout和Storyboard中动态UITableViewCell的高度](http://www.tuicool.com/articles/FZN3q2)  

