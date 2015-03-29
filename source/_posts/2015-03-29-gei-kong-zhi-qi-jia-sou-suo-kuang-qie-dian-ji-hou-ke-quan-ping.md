---
layout: post
title: "给控制器加搜索框且点击后可全屏"
date: 2015-03-29 21:26:20 +0800
comments: true
keywords: SearchBar iOS 搜索框
categories: iOS开发
---

###本文实现给一个普通的控制器加搜索框功能，采用storyboard方式。
<!--more-->

###1.拖一个普通的UIViewController到storyboard。

###2.然后拖一个SearchBar and Search Display控件到控制器里，如下图：

![](/images/2015/03/29/search1.png)

###3.拖一个UITableview到控制器里，对两个控件添加好约束。完成之后效果如下图：

![](/images/2015/03/29/search2.png)

###4.设定tableView的数据源和代理，设定searchBar的代理，更改searchBar的placeholder，storyboard里的设置这些就好。

###5.代码部分，遵守 UISearchDisplayDelegate协议，并提供两个数组分别代表原数据和搜索结果数据:

![](/images/2015/03/29/search3.png)

使用 searchDisplayController的active这个属性来区分搜索状态和常规状态：

![](/images/2015/03/29/search4.png)

实现 UISearchDisplayDelegate代理方法：

![](/images/2015/03/29/search5.png)

第一个方法表示搜索结束后（例如点击cancel或者清除搜索内容）重新刷新数据源以显示原来的数据，当searchBar的搜索内容变化时会调用第二个方法进行实时处理，你的搜索过滤的算法放在这里即可。