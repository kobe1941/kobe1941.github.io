---
layout: post
title: "下拉刷新框架HFRefresh介绍"
date: 2016-09-27 08:10:17 +0800
comments: true
categories: iOS开发
keywords: 下拉刷新 MJRefresh HFRefresh
description: 
---

代码在github上的仓库[HFRefresh](https://github.com/kobe1941/HFRefresh)。

先上本文的思维导图：

![](/images/2016/09/HFRefresh.png)

<!--more-->

##github上已有刷新框架分析

产品形态上，除了loading的动画有所不同，其他比较一致。上拉加载大体上有两种，一种是类似MJRefresh的自动回弹，另一种就是类似HFRefresh的滑动到一定位置直接刷新。

技术实现上，一般分为两种：

```
①KVO监听UIScrollView的contentOffset等属性，比如SVPullToRefresh；
②控制器通过UIScrollViewDelegate实现监听contentOffset，完成刷新状态的封装，然后通过子类化该控制器实现上下拉刷新。
```
第二种实现方式不能通用，UITableView和UICollectionView各需要一个控制器来实现，且都需要把UITableView的datasource和delegate设置成self，父类控制器才能监听到UIScrollViewDelegate的方法。

通常来说，第一种实现方式相对简单直接，通用性好。不过github上的一些已存在的下拉刷新框架基本存在如下几个问题：

```
①循环引用，UIScrollView添加header或footer时，两者相互强引用；
②解除KVO的时机不对，造成控制器销毁页面返回时crash，而且很遗憾的是，由于测试demo使用UIWindow的根控制器，很多作者并未发现这一问题；
③实现机制太复杂或者类太多，比如MJRefresh功能虽然很强大很完整，但是类太多，你要去研究他的实现原理会比较折腾，另外像SVPullToRefresh把所有的类都写进两个文件的方式，也不是很方便阅读，而且冗余代码较多；
④github上很多代码其实不怎么讲究代码规范，关注点都在功能实现上，所以你会看到一大堆的属性和风格迥异的代码混在一起。
```

##HFRefresh优缺点分析

优势：

```
①轻量级，类文件少，在保证功能完整的情况下类和代码行数都尽量少；
②关键部分有注释，阅读成本低；
③接口简洁实用简单，可以快速集成无副作用。
```

不足：

```
①没有酷炫的loading动画效果（不打算集成进来）；
②如果需要设置scrollView的contentInset，则对设置时机有一定要求，需要在添加上下拉之前设置好；
③未测试UIWebView，以及未测试iOS7下的产品形态，也许有bug。
```

##适用对象

HFRefresh适用于需要快速上线产品，而又不需要太多动画效果的企业。可能创业公司和外包公司比较合适。

具体技术实现原理其实很简单，下拉刷新只需要在不同的时机设置好UIScrollView的contentInset.top即可，上拉刷新则根据产品形态去区别对待即可，这里不做赘述。


##关于技术实现

技术实现上需要注意的几个细节点：

```
①UIScrollView的contentInset设置时间；
②iOS7之后导航栏的自适应；
③解除KVO的技巧，具体可查看github上的仓库，有写注释说明；
④严谨的代码规范和风格一致性。
```

##展望

如果大家在使用过程中有任何问题，欢迎到github去提issue。

本次抱着练手的心态试着实现了下拉刷新的功能，完成之后感觉还不错，以后继续挑选适合的项目练手玩玩~~