---
layout: post
title: "腾讯体育iOS客户端逆向实践"
date: 2018-11-11 19:40:58 +0800
comments: true
categories: iOS开发
keywords: NBA 腾讯体育 直播 逆向 去广告
description: 
---





#### 最终实现的结果：

可以跳过闪屏+直播间广告，如果是免费的比赛，或者你是会员的话，可以提取直播间的超清直播URL（蓝光画质是服务器鉴权），该URL可以复制到手机Safari或者电脑端播放。



#### 实践总结：

1.所有拉流直播链接都是服务器鉴权的，即使免费的比赛，你未登录时也会给你分配一个key；

2.一个普通非会员账号，是可以蹭几场会员才能看的比赛的，但是因为腾讯体育的服务器对同个账号的拉流有IP数量限制和流量限制，所以一般说来，除非是免费的比赛，否则不要把链接扩散给别人，另外就是蹭了几场比赛后，你的账号拿到的拉流URL，将只能试看1分钟，之后被断流。腾讯体育支持微信和QQ登录，你也可以准备多个账号用来救急某些重要比赛，当然还是买会员支持正版比较直接啦；

3.会员才能看的比赛，非会员也有一个方式可以看直播，清晰度一般但是勉强能看。**还是买会员看蓝光画质美滋滋**；

4.直播间获取直播链接的接口前缀为：`http://info.zb.video.qq.com/`，后面会拼接很多的参数，其中一个key跟用户账号绑定，服务器用这个来做ip打击和流量控制，抓包请认准该接口，返回数据为json格式，**未做二次加密**。在腾讯体育APP里，这个请求是用`NSURLConnection`这个类发出去的，要hook的话也要hook这个类的网络请求，而不是`NSURLSession`；

5.腾讯体育的直播拉流协议使用HLS，你用Charles抓包会看到ts文件不断的被下载下来播放。



<!--more-->



以下是具体操作流程，大致分为越狱，砸壳，静态分析，动态分析，写hook函数这么几个过程。



需要准备的硬件工具：

越狱手机，Windows电脑和Mac电脑。



## iOS9.3.3系统的越狱

PP助手的Mac版已经没有在维护了，用Windows最新版本的pp助手可以实现一键越狱。如果手机越狱完了，重启之后Cydia会闪退，解决方案：

手机Safari打开 [z.25pp.com](http://z.25pp.com) 然后选择越狱版页面，点击iOS9.3.x越狱激活，按照指示的去做就OK（此步骤网络必须要OK）。之后如果出现重启手机后Cydia闪退，按照上述步骤做一次即可打开Cydia。

注意：执行上述步骤时，请退出Mac上的pp助手。

 

## dumpdecrypted砸壳

需要注意的点是，如果砸壳失败，很有可能是dumpdecrypted出了问题，比如更新Xcode后，默认的SDK可能会有更改，重新拉代码make生成dumpdecrypted.dylib一般可以解决。

砸壳后拷贝到电脑端的是类似QQSportsV3.decrypted这种文件，把文件后缀直接去掉，然后chmod 777修改文件的权限为可读可写可执行，然后就可以进行class-dump和拷贝到APP包里替换掉原来的二进制文件了。

 

## 静态分析

①使用class-dump把砸壳后的二进制导出头文件；

②利用monkeyDev把目标APP用Xcode跑起来，在你想要的页面用Xcode查看这个页面的view所属的类，然后通过nextResponder去找响应链可以找到对应页面的控制器或跟这个view相关联的类；

③用sublime打开所有头文件，找到你想要分析的类，观察其属性的函数接口，如果函数很多的话，找到目标函数需要花的时间跟运气直接相关，一般也可以通过一些搜索匹配关键字来过滤；

④hooper或者IDA静态分析二进制文件，搜索定位到对应的类和函数，看其内部实现。

 

#### 恢复符号表（OC+Block）

因为你已经知道一些基本的类和函数名了，Xcode跑起来后可以设置一些符号断点，进行调试和观察。此时需要先恢复符号表，才能看到断点后具体的堆栈调用回溯。

恢复符号表可以参考这篇文章 <http://blog.imjun.net/posts/restore-symbol-of-iOS-app/>

如果用monkeyDeb创建的工程，OC的符号表恢复可以通过targets→beuild settings 的最下面的User-define里设置MONKEYDEV_RESTORE_SYMBOL为YES，但是block符号需要手动去恢复。

 

需要注意的就是，在用Xcode查看调用堆栈是，Xcode左侧边栏可能仅显示1个或2个调用，NSThread的callStackSymbols函数可能会打印出一些字符串很奇怪的函数调用堆栈，这并不代表block符号恢复失败，这个情况，通常要用LLDB的bt指令去检查看下是否OK。



## 动态挂载分析

可以参考这里提供的函数调用打印<http://bbs.iosre.com/t/app-ios/12157> 辅助分析。其实就是使用了一个叫做HookZz的库。

 

动态分析主要是指不通过Xcode，而是通过终端来调试别人的APP，如果是自己的APP，或者自己证书签名的APP，则可以用monkeyDev直接在Xcode里跑起来；

动态分析主要用到了debugserver和LLDB，通常在这一步前先做静态分析，才能知道要打的断点的位置，动态分析通过打断点和读取、修改寄存器值的方式来调试APP，观察APP的行为。

具体过程可以直接参考《iOS应用逆向工程》书里的步骤，需要注意的一点就是，使用WiFi来连接实在是有点慢，所以用iproxy通过USB来调试是比较合适的。

`brew install usbmuxd` 安装好后即可以使用iproxy，

要启动debugserver，需要先ssh到越狱手机上，然后执行：

`debugserver  *:1234 -a WeChat`

这个指令的意思是让手机的1234端口待命，准备接受LLDB的指令，-a表示启动后面微信的process ，同时这个APP会被卡住暂停执行，所以也不会响应用户的手势；

在电脑上执行` iproxy 1234 1234`实现端口转发；

在电脑端直接执行lldb会打开lldb调试器，然后执行

`process connect connect://localhost:1234`

可以连接到手机上。如果不使用iproxy来做转发，则需要把localhost替换为手机的ip，同时手机要跟电脑处于同一个网段；

之后等待，当终端出现 `Process 867 stopped` 类似这种就说明连接成功了，执行c即可让APP继续运行。

连接过程中可能会出现类似`ImportError: cannot import name _remove_dead_weakref`的错误：

![](/images/2018/11/1.png) 



基本上不用理会，继续等就好了，使用USB来调试，大概等个十来秒就可以连接上，如果用WiFi，可能要等挺久的。

如果出现 `error: failed to get reply to handshake packet`  这个错误提示，可以尝试拔掉USB线重新连接，或者重启手机来解决，当然也有另一种尝试的方案，就是先连接下自己的证书签名的APP，然后再去连接App Store下载的APP。

使用LLDB的指令 ` b adress ` 或  `br s -a address`  可以对地址入口打个断点。



## 去除启动闪屏+直播间90秒广告

1.hook `TADSplashManager` 的`splashItemForItem`函数返回nil，可以去掉闪屏广告；

2.hook `TADVideoViewController`的`viewDidAppear`函数，调用其`skipAdPlay`函数可以跳过直播间90秒广告。



这里讲一下去掉闪屏的分析方式：

①直接法，通过Xcode启动APP，从UI入手，找到对应的view和控制器（nextResponder响应链），然后hook相关的函数来实现修改其逻辑。 这个方式在《iOS应用逆向工程》里讲的很清楚；

②间接法，所有的闪屏基本都是APP打开后先发请求去获取闪屏配置信息，然后下载闪屏内容，之后等到下次启动，检查本地有闪屏则显示，无闪屏则直接进入主页面的逻辑，而闪屏的英文是splash，所以用这个关键字在class-dump出来的头文件去搜索，会得到很多很直接的信息，我就是这么试了一下就成功的；

③闪屏信息通过一个接口来获取，可以block掉这个接口。



另外关于闪屏的设计，其实发现腾讯体育和腾讯视频基本是一模一样，连类名和接口名都没变，可能是一个团队做的？



## 破解会员才能看直播的限制

客户端会根据当前登录用户的状态来判断是否可以试看，服务器的接口会返回一个直播的链接和链接可以播放的时效（300秒），可以测试下这个链接是否一直有效：

——接口会返回一个带鉴权key的直播URL，这个链接拉流时间收到服务器限制。

步骤

①把链接地址抓出来看看是否能在Safari上播放；——可以播放

②hook一下客户端对播放时间的限制，去除该限制，看看是否可以一直播放下去。——不行，服务器鉴权。



首页普通全屏视频播放的响应链为：

QSFullScreenMediaPlayerViewController  ——  QSMPVideoLayersViewController。。。

如果不是全屏则从QSMediaPlayerViewController。。。 开始，而且是把这个控制器的view添加到了tableView上面去 



直播间播放器的响应链反推直播间的控制器到播放器的关系为（仅代表响应链，不带表持有关系）：

QSMatchDetailViewController  ——  QSMatchDetailTopViewController  ——   QSMediaPlayerViewController  ——  QSMPVideoLayersViewController ——

QSPlayerViewController  ——  QSRootPlayerViewController  ——  KKMediaRootView  ——  QLVideoView ——  QQPlayerView  ——  QLAVPlayer/QQPlayer。

其中QLVideoView持有两个QQPlayerView，分别为view0和view1，（也许跟试看有关？比如正常播放用一个，试看用另一个？）；

QQPlayerView持有QQPlayer和QQSelfPlayerVideoView（这个只是用来做openGL相关的一个view，没有其他联系）;

QQPlayer持有两个类名为QQPlayerItem的数据源，同时持有QQSelfPlayer和AVPlayer；

QLAVPlayer继承自QQplayer；

QQPlayer的item是QLAVPlayerItem，其继承自QQPlayerItem。



#### 获取直播链接 

获取直播url的请求为 `info.zb.video.qq.com`



hook `NSURLConnection`的`initWithRequest:delegate:`函数之后，发现调用栈为：

```objective-c
frame #0: 0x0000000102fa4b18 libTencentSportsDylib.dylib`$NSURLConnection_initWithRequest$delegate$_method(self=0x000000014fc53cb0, _cmd="initWithRequest:delegate:", arg1=0x000000014fc55b80, arg2=0x000000014fc56550) at TencentSportsDylib.m:189 
frame #1: 0x0000000100ac95e0 QQSportsV3`-[TTRequestLoader connectToURL:] + 312 
frame #2: 0x0000000100acfbf0 QQSportsV3`-[TTURLRequestQueue sendRequest:] + 672 
frame #3: 0x00000001009852c0 QQSportsV3`-[MakeJsonData createDataByRequestURL:] + 1032 
frame #4: 0x0000000100a3598c QQSportsV3`-[KKMediaPlayPreparer makeLiveProgInfo:] + 1428 
frame #5: 0x00000001009af3fc QQSportsV3`-[KKMediaRootViewController startPrepareMediaToPlay] + 720 
frame #6: 0x00000001009b12cc QQSportsV3`-[KKMediaRootViewController startMediaPlayToPlay] + 2176 
frame #7: 0x00000001009788dc QQSportsV3`-[TVKMediaPlayer openMediaPlayerWithChannelID:andPID:] + 516 
frame #8: 0x00000001005ebcec QQSportsV3`-[QSPlayerViewController startMediaPlayer] + 1408 
frame #9: 0x00000001005eac1c QQSportsV3`-[QSPlayerViewController viewDidAppear:] + 64 
```



两个获取直播URL的方法：

①可以hook这个请求，匹配到URL的域名后，多发一次请求，然后解析返回的参数就可以获取直播的URL（已采用）；

②可以hook一下`-[QQMediaPlayerController prepareToPlayAsset:withMediaUrl:withIsAsset:withKeys:withCacheOrder:withMediaQuence:]`这个函数，第1个参数AVURLAsset里包含有直播的完整url。



对试看view的弹出逻辑探索： 

```objective-c
frame #0: 0x0000000188c14acc UIKit`-[UIViewController viewDidLoad]
frame #1: 0x00000001002bdad8 QQSportsV3`-[QSViewController viewDidLoad] + 48 
frame #2: 0x00000001003a504c QQSportsV3`-[QSWindowController viewDidLoad] + 60 
frame #3: 0x0000000100662030 QQSportsV3`-[QSMPAuthenticationResultViewController viewDidLoad] + 64 
frame #4: 0x0000000188b94b40 UIKit`-[UIViewController loadViewIfRequired] + 996 
frame #5: 0x0000000188b94744 UIKit`-[UIViewController view] + 28 
frame #6: 0x00000001003a5b64 QQSportsV3`-[QSWindowController setRootViewController:] + 256 
frame #7: 0x00000001003a5ee8 QQSportsV3`-[QSWindowController setRootViewController:animationType:] + 636 
frame #8: 0x0000000100654b50 QQSportsV3`-[QSMediaPlayerViewController setRootViewController:animationType:] + 76 
frame #9: 0x0000000100654c10 QQSportsV3`-[QSMediaPlayerViewController setRootViewController:animated:] + 52 
frame #10: 0x0000000100658948 QQSportsV3`-[QSMediaPlayerViewController setPlayerState:] + 1280 
frame #11: 0x0000000100659bec QQSportsV3`-[QSMediaPlayerViewController didConsumeEventFromBus:] + 3824 
frame #12: 0x0000000100893120 QQSportsV3`-[QSBusSystem triggerOtherEvent:]_block_block + 44
frame #13: 0x000000010085c858 QQSportsV3`+[QSSystemUtil performBlock:onRunLoop:]_block + 40
```



上述堆栈里设置rootVC的时候，传入的就是QSMPAuthenticationViewController，所以是否应该弹出试看的view，应该就在这个函数  ` -[QSMediaPlayerViewController setPlayerState:]`。

 用IDA打开这个函数可以看到，state有9个值分别对应不同的`switch-case`分支：

```
state=0/1   QSMPPreviewViewController
state=2     QSMPWWANConfirmViewController
state=3     QSMPAuthenticationViewController
state=4     QSMPAuthenticationResultViewController  会弹出限制只能试看的逻辑 
state=5     QSMPAuthenticationResultViewController  会弹出限制只能试看的逻辑
state=6     QSMPADViewController
state=7     [self playerViewControllerWithCurrentPlayerMode]
state=8     QSProjectionControlViewController
```



以勇士某一场比赛为例，需要会员才能看。

进来后state先设置为3，然后请求了一些数据后，设置state为4，弹出了试看限制的view

如果是不需要会员的比赛，则先设置state为6，把广告弹出来，然后广告完了后设置state为7，进行播放比赛。

如果是需要会员才能播放的比赛，则在state=7之后，发送 `info.zb.video.qq.com`  网络请求去获取直播URL，之后state一直为7.



可以通过hook这个函数，然后对state进行监听，当state=4时，修改其值为6，避开试看逻辑的限制： 

```objective-c
CHMethod(1, void, QSMediaPlayerViewController, setPlayerState, int, arg1){ 
    NSLog(@"hook到 %@ 函数啦%@，参数 arg1 = %ld", self, NSStringFromSelector(_cmd), arg1); 
    NSObject *object = self; 
    int state = arg1; 
    if (arg1 == 4) { 
        state = 6; 
        NSLog(@"修改state为6"); 
    } 
    return CHSuper(1, QSMediaPlayerViewController, setPlayerState, state); 
} 
```



因为广告会自动跳过，所以理论上以上修改后就可以正常播放了，但是实践发现试看的URL拉流时被服务器限制了，试看时间过后就断流。 



补充一些其他信息 ：

1.闪屏广告的接口：`http://news.l.qq.com/app`；

2.直播间广告内容的接口案例：

```objective-c
http://lives.l.qq.com/livemsg?unicomtype=2&newnettype=1&ty=web&defn=hd&ad_type=WL_WK&from=3&vid=100209800&dtype=3&sdtfrom=v3004&live=1&pf=iphone&offline=0&guid=E2CF48156FC246B6A3AB28CFD5115335&pu=1&adaptor=1&v=TADIOSPlayerV1.0&openudid=9da01dcc0f93fea12757814c91074ce440526128&appversion=180911&chid=-1&data=Aejy45%2BNeSZwbpL4wK7%2BCY%2B8ZB%2BzakBg6yuDN6tgzCGupUBraVX4YLCLoIFolnAuCgIePnV34J%2FjX09dPQdSGx2xrnB1tknG92ewRhyw2TpdVNffdS2tgVntHcg%2FcDSmDjrvKiEGRPEOE9KItwzX%2F0xyeJnU2Ap0LZnZKnP7CpNbww77jnM2DC3tvD6X9mCn89P3avg%2FyDbRTsJdWYwIAQsFH%2BaGBIpUTl5ixVCCDHYpDBBFYzTNd10bzeZUiaXfD5T6ZDevRiC0TZryNwN%2FWQVh5ME8lt4fy9UmFAR5UYPHAbI9UWuxAk7kNVeLOe8QaLBM2VslgcTVoGL2vVlXxcxiEqapbM6ZiYjssAlW1sk43zagNNW0K8CtgdBAGBrh5tJ7Y61e9HparuJ0YampHjaLCEnnc9vQ7lEVdQWTFScHyPR9Vfew3F%2FTWg5pJ05j%2BNk%2BsuvjtQ4ikHOt5hpB12JbhGeEghJeUxirpFc5NxEeXvVFV8mI5AO4zg4Bs0i4R52Lx5VvsH%2FODH0wGJAX5P%2B5VJNO4KeE2Qb%2F0bGeaDZ4Qk%2FFEAsRTzDp6E64uBDJgwx%2BHtC3ntqGJ5%2FH3dM6mhh8pPngMHr9J0vTeRAjldnEOG%2BMIQZ99OKqN2lXelrlBRbRXJ%2BxHhZx%2BGKoDdZOJ8T45WF9ZW5x%2Bd6Sgt13SE5jbcofbP3BRTCCbop%2BwEISByPLMpw6i1Lpp4DsC4J7Jg1f0t%2BIcWUNL4QZBXun%2Fu7v7Qoogh%2F%2BmXNniJg0sDXWYuVOjy11HjRCbSCHhZqejORcOISjHekJnnrCSeCrjWpRGrVIsPIGRi7Uz7QM7AG6Rhv3NfVdXdVFl2VPEG1WcYAyiwPFeU4SdNFrFGh3cYPe
```

XML数据格式。有广告的视频连接，包含视频广告内容本身和点击跳转的链接，内容很长，全是广告，去广告也可以考虑hook一下这个接口；

3.NBA比赛的直播链接里，默认的域名前缀通常是`http://vzb.tc.qq.com`，通常接口会返回4个播放地址，一个是域名，另外几个是IP地址开头对应不同服务器吧。

其中一个直播 链接为： 

```
http://183.61.13.200:8080/zijian.hls.video.qq.com/45E618DD0290B2A1DCB53C6314D7F0C2047F57A531962422E845E8AD9434D7142A3D50B07F5A37E9C67F86926A772DD94486064D5D7B2EB64B44B5740D0749DF9040E406818BAACCBD29D0108A6B28D8B6180AA8BE3BFC52/100209402.m3u8?cdncode=%2f18907E7BE0798990%2f&time=1540006787&cdn=zijian&sdtfrom=v20431&platform=40403&butype=2&scheduleflag=1&buname=qqlive&useblockid=1&vkey=45E618DD0290B2A1DCB53C6314D7F0C2047F57A531962422E845E8AD9434D7142A3D50B07F5A37E9C67F86926A772DD94486064D5D7B2EB64B44B5740D0749DF9040E406818BAACCBD29D0108A6B28D8B6180AA8BE3BFC52
```

前面的没有用域名，而是IP地址，没啥好处理的。其中的一个关键是中间的URL`zijian.hls.video.qq.com`，抓数据的时候用这个关键字。

上述链接中`vkey`是必不可少，且跟你当前登录账号绑定。 

清晰度不一样主要是通过xxxx.m3u8这个来区分的，往往蓝光的id比超清的数字多1而已，其他清晰度一次递减。比如标清是1001.m3u8，高清是1002.m3u8，超清是1003.m3u8，蓝光则为 1004.m3u8。



#### 问题备注：

1.pp助手客服见这里<https://pro.25pp.com/faq_detail/pc5/20305608>，有不懂的可以问客服，包括新版软件怎么下载，有哪些功能等等；

2.最新版的手机pp助手，可以下载腾讯体育的历史版本，但是最新版的pp助手APP，要连着Windows的pp助手才能下载，应用内直接更新是不行的；

3.IPA包安装到手机上后，bundle里有原版所有文件，用一台越狱的手机，可以直接scp全部拷贝到电脑上用，当然越狱版IPA的可执行文件就是砸壳的，正版的就是加密的，可以自己砸壳生成QQSportsV3.decrypted文件，去掉后缀，再chmod 777改下文件权限就OK了；

4.使用LLDB的bt指令来代替Xcode左边栏的可视化堆栈和NSThread的callStackSymbols函数，bt指令比这两者要准确靠谱得多；

5.普通视频播放和直播的方式不太一样，直播可以直接拿到URL到PC播放，视频的应该是腾讯体育有自定义一个协议，只有播放器内部能播，复制到浏览器无法播放，例如:

```objective-c
streaming://127.0.0.1:16697/playmp4?data_id=e22ee22fd2992c4674950&clip_id=1&guid=3F30E560A8A3459DBA410FFEBA7EFAC3
```

6.LLDB打断点的时候可以考虑对`-[UIViewController viewDidLoad]`打个断点，这样子可以捕获所有的控制器被初始化；如果想知道某个UI页面的按钮被点击后做了啥，也可以对应去打断点；

7.HookZz可以hook `objc_msgSend`函数，打印所有的函数调用，但是新版的ZzReplace接口需要额外研究下怎么用（知道的麻烦告知一下），而且log太多了需要过滤处理；

8.不要修改腾讯体育的包名，否则播放器无法播放视频和看直播，其他功能正常；

9.HTTPS的握手失败是因为手机没有安装Charles的证书，同一台手机连接不同电脑时，都需要安装这台电脑所对应的Charles证书。





#### 参考 

1.[《iOS应用逆向工程》](https://book.douban.com/subject/26363333/)； 

2.[《iOS应用逆向与安全》](https://book.douban.com/subject/30239776/)； 

3.[逆向世界杯直播](http://bbs.iosre.com/t/app-ios/12157)[App](http://bbs.iosre.com/t/app-ios/12157) [央视影音](http://bbs.iosre.com/t/app-ios/12157)[-iOS](http://bbs.iosre.com/t/app-ios/12157)[客户端](http://bbs.iosre.com/t/app-ios/12157) 。



#### 声明：本篇文章仅作为学习和研究逆向技术用途，请勿用于商业行为。腾讯体育给球迷们带来了良好的观看NBA比赛的体验，在此表示感谢~