---
layout: post
title: "FFExtension"
date: 2018-10-15 22:20:23 +0800
comments: true
categories: iOS开发 
keywords: crash iOS开发 iPhone 
description: 
---



写这个库的初衷是为了防止一些常见的崩溃，扫了一圈github上已有的库，都不太合适，像avoidCrash虽然能用，但是我不喜欢它用try-catch来做防崩溃的方式，它GitHub上的issue页也有很多问题是不好去定位和解决的。所以决定自己来是实现一遍。



主要原理就是用Method Swizzle去hook系统类包括私有类的函数， 对常见的容器类，数组字典字符串这些有做保护，顺带新增了 NSSet，NSCache，NSUserDefaults，NSData ，NSAttributedString等几个类的保护，支持拦截 `unrecognized selector sent to instance` 异常，设置好要拦截的类即可。

一些使用代码规范就能解决的崩溃，比如 NSTimer，通知和 KVO 等等，本项目并未做额外处理，这种低级的失误，还是用代码规范来限制比较好。

已经在自己的项目用上了，目前工作稳定，iOS8.x 到 iOS12 都测试通过。

<!--more-->



##接入方式

使用cocoapod接入

`$ pod 'FFExtension'`

导入`FFManager.h`头文件，如果你需要拦截部分类的`unrecogzied selector sent to instance` 崩溃，你可以传入一个包含类前缀的数组，比如下面的SSZ，那么所有以SSZ开头的类，都不会再有这个类型的崩溃。

初始化：

```objective-c
__weak typeof(self) weakSelf = self;
[[FFManager sharedInstance] startWorkWithOption:FFHookOptionAll unrecogziedSelectorClassPrefixs:@[@"SSZ"] callBackBlock:^(NSDictionary *exceptionDic) {
        [weakSelf reportExecptionToBugly:exceptionDic];
}];
```

上传错误日志到bugly：

```objective-c
#pragma mark - Report Exception To Bugly
- (void)reportExecptionToBugly:(NSDictionary *)exceptionDic
{
    if (exceptionDic) {
        NSString *name = [exceptionDic objectForKey:FF_Name];
        NSString *reason = [exceptionDic objectForKey:FF_Reason];
        NSDictionary *extraDic = [exceptionDic objectForKey:FF_ExtraDic];
        NSArray *callStack = [exceptionDic objectForKey:FF_CallStackSymbols];
       
        [Bugly reportExceptionWithCategory:3 name:name reason:reason callStack:callStack                        extraInfo:extraDic terminateApp:NO];
    }
}
```



##设计原理和hook函数的原则

method swizzle的正确姿势 ，一定要理解method swizzle的原理 ：

```objective-c
+ (void)ff_instancenSwizzleWithClass:(Class)class originSelector:(SEL)originSelector swizzleSelector:(SEL)swizzleSelector
{
    Method originMethod = class_getInstanceMethod(class, originSelector);
    Method swizzleMethod = class_getInstanceMethod(class, swizzleSelector);
    if (!originMethod || !swizzleMethod) {
        return;
    }
    
    class_addMethod(class,
                    originSelector,
                    method_getImplementation(originMethod),
                    method_getTypeEncoding(originMethod));
    class_addMethod(class,
                    swizzleSelector,
                    method_getImplementation(swizzleMethod),
                    method_getTypeEncoding(swizzleMethod));
    
    
    ///< 添加完了之后要重新赋值，因为原来的两个method都是父类的。参考自见https://github.com/rentzsch/jrswizzle/blob/semver-1.x/JRSwizzle.m
    Method originMethod2 = class_getInstanceMethod(class, originSelector);
    Method swizzleMethod2 = class_getInstanceMethod(class, swizzleSelector);
    
    method_exchangeImplementations(originMethod2, swizzleMethod2);
}

```

上图中，这两步是很重要的，第一步判空保护自然不用说，如果你都没有实现这个函数，交换必然也是无效的。 

第二步则是重点，在分别调用了两次class_addMethod之后，做method_exchange时，是不能直接传递最初的Method指针的，因为可能class并没有去实现originSelector，而是其父类实现的，此时originMethod获取到的就是父类的实现指针。当class_addMethod函数调用以后，class这个类本身也有了originSelector的实现，所以后面交换的时候需要重新取一下值。而且，如果你不做这一步操作的话，就很有可能把父类的实现指针拿去交换了，这是后面如果其他的派生子类去hook同一个函数时是会出问题的，可以看源码里关于NSArray的本类和其的多个派生类对同一个函数的hook实现对比： 

![](/images/2018/10/1.png) 

更详细原因和推理过程可以看[这里](<https://juejin.im/entry/5a1fceddf265da43310d9985>)。 

有一点想吐槽AvoidCrash的就是，虽然这个项目star数量比较多，但是它的实现中，类之间的循环依赖到处都是。。。 



##踩过的坑

我发现在你hook系统的函数之前，系统是可以给你正确识别异常并报错的，hook之后，很多正常的数组越界，字符串超长问题，就会给你包BAD_ACCESS，SIGBRT之类的错误了。而且除了自己写的native代码会有问题，react native从js转换过来的RCT开头的那一堆类，也有不少各种各样的问题，用上这个库后，react native的崩溃也能有一部分下降。

1.SIGABRT

```objective-c
Assertion failure in -[_UITraitBasedAppearance _beginListeningForAppearanceEventsForSetter:], /BuildRoot/Library/Caches/com.apple.xbs/Sources/UIKit/UIKit-3600.9.1/UIAppearance.m:1575
Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Bad selector setup for -[UIPickerView _setTextColor:]'
```

原因是手抖导致的代码错误，`rangeOfReceiverToSearch.location + rangeOfReceiverToSearch.length <= self.length`少了个`=`号导致崩溃

2.不要使用这种初始化函数:

```objective-c
[NSDictionary dictionaryWithObjects:objects forKeys:keys count:2]
[[NSArray alloc] initWithObjects:(const id _Nonnull [_Nullable])objects count:(NSUInteger)cnt]
```

count和前面的指针不见得是一一对应的；



3.跟bugly的冲突:

`EXC_BAD_ACCESS` 、` SIGABRT` 或者` EXC_BAD_INSTRUCTION`:

```objective-c
Terminating app due to uncaught exception 'JCEBaseObjectException', reason: 'Invalid JCE ext string: __b0x9i_M09ONSStringONSString'
```

![](/images/2018/10/2.png) 

这是hook NSString时边界条件处理不好导致的问题。



4.字典空值：

```objective-c
 +[__NSDictionaryM setObject:forKeyedSubscript:], key https://ios.bugly.qq.com/rqd/sync?aid=688DD054-8A1A-4076-897A-EEC89B4D8923 or object (null) can not be nil
```



5.字符串hook函数边界条件不对导致的`unrecognized selector sent to instance`：

```objective-c
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[RCTImageView setMAGESOURCESmageSources:]: unrecognized selector sent to instance
```



以上几个问题，除了部分手抖写错函数调用之外，全靠以下几次提交解决：

![](/images/2018/10/3.png) 

![](/images/2018/10/4.png) 



6.`NSUserdefaults` 的`setobject:forKey:`函数和`NSDictionaryM`的`setObject:forkeysubscript:`函数，object都是可以为nil的，会调用`removeObjectForKey`移除该key。所以在进行hook的时候，要注意不要对object做判空保护。 



7.`NSData`的 `appendBytes: length:`函数，length为0时，bytes是可以为nil的:

![](/images/2018/10/5.png) 



8.进入直播间崩溃，`NSString` 的 `substringFromIndex`这个函数，index是可以==self.length的。同时如果越界了，尽量不要直接返回nil，而是取安全区间的字符串返回：

```objective-c
+[__NSCFString substringFromIndex:], index 24 is out of bounds 0...23
```

![](/images/2018/10/6.png) 



9.`[NSThread callBackSymbols]`线程回溯返回了空的数组 

见[链接1](<http://www.gnustep.org/resources/documentation/Developer/Base/Reference/NSThread.html>)和下图

![](/images/2018/10/7.png) 

以及[链接2](<http://landonf.bikemonkey.org/code/objc/Reliable_Crash_Reporting.20110912.html>)和下图

![](/images/2018/10/8.png) 



10.`NSAttributedString`相关的崩溃

![](/images/2018/10/9.png) 

同时，在hook系统的两个函数时发现 

①`NSAttributedString`的`attribute:atIndex:longestEffectiveRange:inRange:`函数，如果`attrName为`空或`range`的值越界了，Xcode10会直接停住不往下执行，但是不会显示任何的异常或断点，就相当于APP卡住了;

②`attributesAtIndex:effectiveRange:`这个函数如果hook了，在UILabel初始化的时候，可能会导致`EXC_BAD_ACCESS`错误，如下图所示，原因不明：

![](/images/2018/10/10.png) 

最终把相关获取属性的函数（见源码）全部注释了（正常使用很少会手动去调用这几个函数），问题没有再出现。当然治本的方法还是要靠苹果大爷。如果你有什么好方法，麻烦评论区告诉一下我~



11.由于出于好意，`FFExtension`最初的设计里，保留了对部分系统类的`unrecogzied selector sent to instance` 崩溃拦截，但是实践发现一个案例： 

对`NSNull`对象的`unrecognized selector sent to instance`做了拦截，目的是防止服务器接口返回一些空的对象时引起的崩溃，比如字符串这种。但是这里没崩溃，后面字符串复制的时候还是崩了，copy函数不能简单通过增加一个函数来解决。而且后面的崩溃会打乱你的堆栈，破坏Xcode对异常的捕获：

![](/images/2018/10/11.png) 

`FFExtension`在最新的版本中去掉了上述默认的对系统类的拦截，但依然保留接口，使用者可以设置自定义类的拦截。



其他：

1.KVC和storyboard、xib可能会遇到的`setUndefinedValueForKey`这种崩溃，因为我们项目里用纯代码来实现，所以暂时也还没有遇到过，相关问题可以参考[这里](<https://tbd.ink/2018/04/04/iOS/18040401.TPreventKVC%E7%9A%84%E4%BD%BF%E7%94%A8%E5%92%8C%E5%AE%9E%E7%8E%B0/index/>)。 



参考链接： 

1. [iOS 界的毒瘤：Method Swizzle](<https://juejin.im/entry/5a1fceddf265da43310d9985>); 

2. [Objective-C Method Swizzling](http://yulingtianxia.com/blog/2017/04/17/Objective-C-Method-Swizzling).