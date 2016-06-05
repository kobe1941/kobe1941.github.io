---
layout: post
title: "iOS设置启动图的两种方式和bug汇总"
date: 2016-06-05 18:35:31 +0800
comments: true
categories: iOS开发
keywords: iOS LaunchImage LaunchScreen.storyboard 启动图 
description: 
---
前段时间一直在折腾启动图的设置，忙到最后发现居然是 Apple 的锅。。。

先讲一下正常的设置启动图的流程，有两种方式，一种是LaunchImage的方式，另一种就是LauchScreen.storyboard的方式。见下图：

其一：

![](/images/2016/06/1.png)

其二：
![](/images/2016/06/2.png)
![](/images/2016/06/3.png)

<!--more-->

以下是遇到的问题：

1.单独使用 LaunchImage 来设置启动图时：

①iOS9 系统的手机，在 APP 从后台通过 openURL 拉起到前台时，不会出现启动图；

②但 iOS7 和 iOS8 正常。

2.单独使用 LaunchScreen.storyboard 来设置启动图时：

①iOS7 不支持该方式；

②iOS8 系统会导致 APP 在从后台被拉起到前台时，先出现黑屏再出现启动画面的情况，

③iOS9 系统，如果更换过启动图， APP 在后台被拉起到前台时出现的启动图不会更新。

3.使用 LaunImage 和 LaunchScreen.storyboard 混用的方式，问题如下：

①iOS8 系统会导致 APP 在从后台被拉起到前台时，先出现黑屏再出现启动画面的情况；

②iOS9 系统，如果更换过启动图， APP 在后台被拉起到前台时出现的启动图不会更新。


补充：

另如果某个版本使用了 LaunchScreen.storyboard 方式来设置启动图后，后续的升级版本不可再更改为 LaunchImage 的方式，如果有更改，则启动图依然会使用之前 LaunchScreen.storyboard 里的内容。见 stackoverflow 的[这个问题](http://stackoverflow.com/questions/33730168/how-to-change-launch-screen-file-to-launch-images-source-when-app-update/37434449#37434449)。

关于 iOS8 下 APP 从后台拉起到前台会先短时间黑屏的问题，测试发现微信和微博也同样存在。

解决方案：
如果可以像微信和微博一样，容忍iOS8系统下拉起时的短暂黑屏，而且确定以后不会更改启动图，也不会再转回使用LaunchImage来设置启动图的方式，那么就可以使用LaunchScreen.storyboard这种方式来设置启动图，否则还是乖乖使用LaunchImage这种方式吧，而且，如果你的应用需要支持iOS7，那么也要使用LaunchImage方式，当然你可以在应用中同时使用两种设置启动图的方式来兼容iOS7和以后的系统。

两种启动图方式同时设置见下图：

![](/images/2016/06/4.png)