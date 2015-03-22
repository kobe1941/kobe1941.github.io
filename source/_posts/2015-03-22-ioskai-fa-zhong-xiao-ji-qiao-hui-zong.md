---
layout: post
title: "iOS开发中小技巧汇总"
date: 2015-03-22 18:12:04 +0800
comments: true
keywords: iOS开发 技巧 汇总
categories: iOS开发
---
##本文汇总了一些iOS开发中的小技巧，不定时更新。
<!--more-->



##1.让UITableView滚动到指定位置
可使用如下方法使UITableView滚动到指定位置：

![hehe](/images/2015/03/22/UITableView1.png)

![hehe](/images/2015/03/22/UITableView2.png)

其中，UITableViewScrollPosition代表，滚动完成时，目标行indexPath在屏幕的顶部、中间还是底部。




##2.iOS7.0.3系统的几个坑	
1）加约束的时候不能直接设置为`MAXFLOAT`：

![](/images/2015/03/22/Constraint1.png)

2）UITableViewCell的contentView的高度问题：
iOS7.0.3系统如果不手动改变高度，有时候(非必现)会一直为44，更高的系统版本(比如iOS7.0.4及以上)则无此问题。



##3.让scrollView上的UIButton在选中后手指滑动依然可以滚动scrollView
在scrollView或者tableView上添加了button，选中button，不松手的情况下，手指继续滑动，这个时候scrollView是无法滑动的，因为事件被button拦截了，要想不松手的情况下继续滑动使scrollView滚动，同时button取消选中状态，可继承scrollView并作如下处理：

![](/images/2015/03/22/UIScrollView1.png)



##4.给UITextView加上placeHolder和禁止回车键
1）加一个UILabel到UITextView上方，调整好间距。
然后在代理方法中作如下处理即可：

![](/images/2015/03/22/UITextView1.png)

2）让UITextView禁止回车键，可在代理方法中做如下处理：

![](/images/2015/03/22/UITextView2.png)

可自行控制是否收起键盘。根据项目需要进行封装以实现代码复用。


##5.区域截图

目的是在self.imageView这张大图片上截图一个区域。

代码：

![](/images/2015/03/22/CutPicture1.png)

注意：
self.selectView是一个控件，透明，用于显示要截取的具体区域。因其是添加到self.view上的，故截图前需要转换frame。
opaque表示不透明度，opaque=yes表示不透明，配合alpha使用。

理由见这里：

![](/images/2015/03/22/CutPicture2.jpg)


##6.Core Graphics绘制股票曲线方法

![](/images/2015/03/22/stock1.jpg)

画上面这样的图，有两种方式：

1）底部的基准线和虚线用一个UIView进行重绘，然后股票的走势曲线和填充的红色用一个UIView来绘制，再进行叠加，可以设置上面view的透明度。这样在填充的时候，可以一步到位。即直接封闭股票走势曲线，但是把不显示的曲线画到UIView的界面外边，以实现效果。

2）直接用一个UIView进行重绘，填充部分的绘制分两个步骤，第一先画股票走势曲线，第二，再设定很小的线宽(可以设置为0)画一个封闭的图形，该封闭图形的上边界与之前画的股票走势曲线完全重合。这样所有的绘制可以在一个控件上实现。


##7.模拟器的各种路径
1）模拟器路径
Xcode6以后模拟器在Mac上的位置为：/Users/username/Library/Developer/CoreSimulator
其中，username表示用户名，Library表示资源库，资源库文件夹默认是隐藏的。

![](/images/2015/03/22/Simulator1.png)

2）应用沙盒地址路径
在Xcode6中，应用程序文件、Document文件夹、Library文件夹、tmp文件夹这四个文件放在了不同的目录中。应用程序文件路径：/Users/username/Library/Developer/CoreSimulator/Devices/模拟器UDID/data/Containers/Bundle/Application文件夹下；Document文件夹、Library文件夹、tmp文件夹路径：/Users/username/Library/Developer/CoreSimulator/Devices/模拟器UDID/data/Containers/Data/Application文件下。但是不幸的是，这两个路径打开后的文件名，还是经过编码过的，而且，同一个应用中的应用程序文件和D、L、t文件夹所在的文件夹的文件名是不同的。只能自己找。

3）NSUserDefault的文件存储位置路径
在Xcode6中，程序对使用NSUserDefault方式创建的plist文件的位置进行了更换，具体路径为：/Users/username/Library/Developer/CoreSimulator/Devices/模拟器UDID/data/Library/Preferences文件夹下。
      
这里特别说一下，如果按照在Finder里打开的路劲来看，并不是这样的，但通过 Finder，前往文件夹，通过该路径查找是可以查到的。上述的路径地址是通过查看Preferences文件夹的显示简介获得的。
        通过上述的路径可以看出，通过NSUserDefault创建的plist文件夹还是在Library文件下，但不同的是，真正存放的位置变了，成了在模拟器的资料库文件夹下，这样的改变所产生的变化就是，当我们在删除模拟器中的应用程序后，plist文件还是会保留，并不会删除。但在真机中，也是会删除的。

备注：
打开Mac隐藏文件命令：

```
defaults write com.apple.finder AppleShowAllFiles -bool true
```
关闭Mac隐藏文件命令：

```
defaults write com.apple.finder AppleShowAllFiles -bool false
```


##8.事件传递和响应链
1）关于事件分发和响应看[这里](http://blog.csdn.net/wzzvictory/article/details/9264335)。

2）关于scrollView或者tableView，滑动手势不会被其子View拦截的原因：

UIScrollView在手势上有很复杂的运算，针对某个手势，首先判断是否应该处理，若不该自己处理，就往下传递，直到找到第一响应者。

你所说的那个滑动，是应该UIScrollView滚动的，是你的手势对UIScrollView来说就是滚动的手势，这也满足在一个有着很多控件的UIScrollView上，你迅速在界面上扫动，就是UIScrollView的滑动，不然你一旦手势在某个控件上，滑动就被控件接受了，那样滑动UIScrollView就会很困难了。 实际上这是一个时间问题（scrollview.delayContentTouch），你在按下到滑动的时间若小于某个值，就是UIScrollView的滚动，若大于某个值，就是控件去处理，你可以试试的。

hitTest方法是一个消息传递的试探方法，去找第一响应者，这个里面你可以来决定当前view是否处理消息，无法判断是否拖动还是单击的

该解读来自CocoaChina，链接在[这里](http://www.cocoachina.com/bbs/read.php?tid=253004)。
