---
layout: post
title: "WWDC2019之启动时间与Dyld3"
date: 2019-09-13 13:40:48 +0800
comments: true
categories: iOS开发
keywords: 启动时间 iOS开发 WWDC
description: 
---

WWDC从2016，2017到2019都有session对APP启动的过程以及如何优化做过介绍，WWDC2019因为把Dyld3开放给所有APP，所以Apple重新梳理了启动的各个阶段，并给出了对应的优化建议。 

本文内容主要来自WWDC2019，图片取自WWDC的PPT或视频中截图；涉及到Dyld3的内容来自WWDC2017的一个session。 

<!--more-->



## 一、启动各个阶段的介绍 



启动类型 

![](/images/2019/09/13/1.png) 

1.最好是把启动时间控制在400ms内，因为这是启动动画的时间；系统要用100ms的时间来初始化APP，所以留给了你300ms的时间来构建你的第一个view；你可以懒加载或者异步加载数据。 



2.iOS13带来了dyld3，虽然dyld3在WWDC2017的时候介绍过，但是iOS13终于带入了它；dyld3可以缓存runtime的dependency（库）来改善热启动的耗时。 

```
Dynamic Linker loads shared libraries and frameworks
Introduces caching of runtime dependencies to improve warm launch 
```

启动的各个阶段 

![](/images/2019/09/13/2.png) 



3.为了充分利用dyld3带来的优化，苹果建议避免链接不使用的动态库，以及启动时避免使用类似DLOpen和NSBundleLoad的动态库的加载（loading），因为这会抵消缓存所带来的优势。最后就是你需要硬链接你的依赖dependency库，现在这个过程比之前更快了。（硬链接应该是dyld3自带的功能？）

![](/images/2019/09/13/3.png) 



4.libSystemInit是System interface阶段的后面一部分内容，这是在给你的APP初始化低级别的系统组件，这个过程是固定的系统层面的消耗。开发者不必过多关注这个阶段。 

```
Initializes the interfaces with low level system components
System side work with a fixed cost 
```



5.之后是runtime的初始化，这是系统在初始化Objective-C和Swift的runtime。一般的，在这一步你的APP不应该做任何事除非你有静态初始化函数，这些静态函数可能在你的代码里，也可能在你链接的库里。我们也不建议使用静态初始化（函数）。 

```
Initializes the language runtime
Invokes all class static load methods 
```



如果你自己的framework里用到了静态初始化，可以考虑暴露API，以尽早初始化你的堆栈。——言下之意是用户在启动时主动调用该API，而不是写到+load函数里？ 

如果是必须要用到静态初始化，把代码从类的load函数里移动到initialize函数里去。 

![](/images/2019/09/13/4.png) 

6.UIKit 初始化。系统会在这一步实例化你的UIApplication和UIApplicationDelegate。这个阶段大部分是系统的工作，设置事件处理和系统集成/整合。你仍然可以影响这个阶段的耗时，比如子类化UIApplication，或者在UIApplicationDelegate的函数里做一些别的工作（通常是增大耗时o(╯□╰)o）。 

```
Instantiates the UIApplication and UIApplicationDelegate
Begins event processing and integration with the system 
```

![](/images/2019/09/13/5.png) 

7.之后是Application 的初始化。一般的回调顺序如下： 

`application:willFinishLaunchingWithOptions:`

`application:didFinishLaunchingWithOptions:`

iOS13之前

`applicationDidBecomeActive:`

iOS13之后，新增了UISceneDelegate的代理回调函数

`scene:willConnectToSession`

`sceneWillEnterForeground`

`sceneDidBecomeActive`

苹果建议推迟跟第一屏展示不相关的工作，把这些工作放到后台去做或者全部推迟。

如果你采用了UIScenes的API，则可以在多个Scenes之间共享资源，这是为了避免多次去做不必要的工作。

![](/images/2019/09/13/6.png) 

8.然后是Frist Frame Render阶段。首帧渲染过程为创建views，设置好布局，然后进行渲染。 

`loadView`

`viewDidLoad`

`layoutSubviews `



这个阶段可以做的优化有： 

减少视图view的数量； 

减少视图view的层级（flattening your views）； 

懒加载那些在启动过程中不会立即进行展示的view； 

注意你的autolayout，尽量减少约束的数量；——干脆使用手动frame计算 

![](/images/2019/09/13/7.png) 

9.最后进入到Extended的阶段（延长/扩展阶段）。 

从首帧后到最后一帧之间，APP的某个特定阶段； 

异步加载数据的阶段； 

不是所有APP都有这个阶段； 

这个阶段时APP应该是可交互和可响应的； 

可以用操作系统的 signpost API来标记和测量两个时间周期内的消耗。 

`Leverage os_signpost to measure work `

![](/images/2019/09/13/8.png) 



## 二、测量启动 



1.测量启动过程 

要去除掉网络和后台进程的干扰； 

```
Remove sources of variance to produce more consistent results
May result in launch times that are not representative
Use consistent results to evaluate progress
```



2.tips关于设置干净和一致的测试环境 

①重启手机，然后静置几分钟，用来清除任何启动时的工作； 

②设置手机为飞行模式，或者使用Mock网络数据； 

③iCloud在后台工作会干扰APP启动时间的测量，所以测量过程中使用不变的iCloud账号和不变的数据，或者干脆退出iCloud； 

④使用release版本的APP进行测试，避免debug代码的干扰，还能享受编译器的优化（跟线上用户保持一致）； 

⑤测试warm launch的数据，这样子可以保持更好的一致性，一部分APP的数据已经在内存里了，一部分系统服务也已经跑起来了； 

⑥创造多个mock数据是非常重要的，比如用户数据量少和用户数据量多的情况，都要测量到； 

⑦挑选多个设备来测试，并保证他们在测试过程中的一致性；一定要包含一些旧的设备，以及你的APP所支持的最旧的版本（指iOS操作系统版本）； 

Xcode11开始，XCTest也提供了测量启动性能的API。只需要几行代码，Xcode就能重复启动你的APP，并提供启动性能的统计结果。 

![](/images/2019/09/13/9.png) 





3.测量的三个提示和技巧 

①首先最小化你的启动过程； 

最小化过程的时候，应该推迟任何跟展示首帧无关的操作，比如推迟暂时不展示的view或暂时用不到的功能的初始化； 

千万不要block住主线程，不管是网络IO操作，文件IO操作还是其他，把这些移动到后台线程去； 

减少内存的占用，内存的分配和操作是耗时的； 

```
Defer work unrelated to first frame
Move blocking work off main thread
Reduce memory usage 
```



②然后按照优先级来安排你的启动过程；要保证这些工作的安排是合适的； 

这意味着把不同优先级的工作安排到不同的线程去执行比以往任何时候都更重要。 

可以关注一下WWDC 2017对GCD的深入介绍，讲了怎么正确的使用并行队列。 

```
Identify the right QoS for your task
Utilize scheduler optimizations for app launch
Preserve the priority with the right primitives 
```



③最后对这些过程进行优化； 

在启动过程中，应该限制到只去拉去自己需要的数据； 

优化算法和数据结构；——指不要把启动时需要的数据结构搞的太复杂，造成拉取太多并不需要的数据 

你应该缓存你的资源和计算结果，这是为了降低在做多次不必要的操作时产生的CPU和内存消耗； 

```
Simplify or limit existing work
Optimize algorithms and data structures
Cache resources and computations 
```



4.Apple新带来的启动监控方式  **MetricKit**

可以收集电源和性能统计数据，每24小时汇总数据进行上报。

```
Collect custom power and performance metrics

Aggregated results delivered every 24 hours 
```



## 三、使用Instruments和XCTest来优化demo app的启动性能 



1.Instruments对demo APP的启动耗时进行分析 

这一节主要是介绍了这个新的工具是如何使用的，直接看视频即可。 



①Instruments重新编译APP会使用release模式 

`Xcode to recompile your app in release mode, so that you can take the advantages of compiler time optimizations.`



②Xcode 11的Instruments提供了启动时间的模板，可以用来专门做这一块的性能统计和分析。 

`iOS 13, or Xcode 11, we now have the AppLaunchTemplate, which we can use specifically for triage purposes like this, figuring out what's wrong with AppLaunch.`



2.分析各个线程

`The first few phases marked in purple are the phases that occur before your main function is invoked within your app.`

紫色表示pre-main阶段，在main函数执行前的阶段。

`Onto the green phases, these phases of the early phases that occur at the very first of your main function, as your app finishes its launch and draws its first frame in UI.`

绿色表示进入到main阶段。

`Speaking of thread states, gray means it's blocked, meaning that the thread isn't doing any work.`

灰色表示线程被block住了，该线程目前啥都没做。

`Red means it's runnable, meaning that there's work scheduled to be done, but lacking CPU resources.`

红色表示可执行，也就是待调度的状态，但是缺乏CPU资源。

`Orange means it's preempted, meaning that it was doing work but got interrupted in favor of other competing work that has a higher priority.`

orange代表该线程正在执行某个操作，但是被某个更高优先级的线程打断了，高优先级的线程完成了它才能继续执行。

`And last but not least, blue means it's running, meaning that it's actually doing work on the CPU core.`

蓝色表示正在运行中的线程，正在被CPU调度中。

![](/images/2019/09/13/10.png) 



3.iOS系统进行的优化

`Now notice that this initial phase only took 6 milliseconds as it sets up its system interfaces.`

`This is primarily due to the benefits of dyld3 introduction and third-party apps, in addition to other system layer enhancements.`

system interface阶段只花了6ms，这是得益于dyld3和其他第三方APP，以及其他系统层面的优化。开发者可以不用写一行代码就能获得这些优化。



`This discrepancy comes from the overhead of the profiling mechanism itself, which does give us a lot of information and insight, but has a cost of its own.`

虽然这个阶段只花了6ms，但是因为instrumens的测量工具，导致总计花费了149ms，多出来的时间都是测量工具造成的。

但是如果使用XCTest的API来测量启动的性能数据，则不会有这个消耗。在demo APP中，使用Instruments的统计时间是500ms，而使用XCTest测量的时间是300ms，这个差值就是Instruments工具自身的消耗。

XCTest会去掉冷启动带来的数据干扰和误差，默认执行5次热启动，然后把数据汇总。



`As mentioned before, dyld3 brings caching of your runtime dependencies to your apps, which you saw in the demo, that provided a huge improvement.`

dyld3缓存了你的APP runtime的依赖库，这是一个很大的改善。



` The Scheduler has also been optimized to help prioritize the work that happens during launch.`

Scheduler也优化过以支持APP启动时区分优先级的工作。



` We also put Auto Layout and Objective-C under the microscope and made a bunch of optimizations there.`

我们同时优化了autolayout和OC的性能。



`And then finally, we have exciting changes to app packaging coming later this year.`

最后，我们会在今年晚些时候，带来APP打包方面的令人激动的改变。



Apple已经做的和即将要做的（app package）优化：

![](/images/2019/09/13/11.png) 

![](/images/2019/09/13/12.png) 





## 四、Apple给的关于启动的Tips



①不要事后才想起来做优化，应该在开始写代码的时候就要注意，在每一次bug fixed/重构/功能开发的时候去注意性能；积少成多，如果刚开始的不注意一些微小的优化，后面就会积累变成很大的消耗，而且后期会很难找到原因；

②应该经常性的去测量APP的启动性能数据；

③关注一下Xcode organizer，你就能知道你的APP在线上的表现；在iOS13下，用户同意的情况下，Apple会每24小时会把APP的性能数据发送到你的Organizer；iOS13已经支持MetricKit，它可以把收集到的性能数据通过APP内的代理函数回调给开发者自己分析和使用；

![](/images/2019/09/13/13.png) 



## 五、关于Dyld3 

本节内容主要来自WWDC2017。

Dyld3开放给第三方的APP，是iOS13之后第三方APP启动速度会变快的最大原因。 



启动闭包（launch closure）：这是一个新引入的概念，指的是 app 在启动期间所需要的所有信息。比如这个 app 使用了哪些动态链接库，其中各个符号的偏移量，代码签名在哪里等等。 



`perform symbol lookups`这个步骤表示执行符号查找。（例如：如果你使用了printf()函数，就会查找printf是否在库系统中，找到它的地址，将它赋值到你的程序中的函数指针）。



Dyld2和Dyld3的对比 

![](/images/2019/09/13/14.png) 



Dyld2做的事情

![](/images/2019/09/13/15.png) 



Dyld3做的事情

![](/images/2019/09/13/16.png) 



详细展开其在APP进程外和APP进程内做的事情 

上半部分表示进程外 

![](/images/2019/09/13/17.png) 

下半部分表示在进程内做的

![](/images/2019/09/13/18.png) 

![](/images/2019/09/13/19.png) 

dyld2是纯粹的in-process，也就是在程序进程内执行的，也就意味着只有当应用程序被启动的时候，dyld2才能开始执行任务。





#### dyld 2主要工作流程为：

```
•dyld的初始化，主要代码在dyldbootstrap::start，接着执行dyld::main，dyld::main代码较多，是dyld加载的核心部分；
•检查并准备环境，比如获取二进制路径，检查环境变量，解析主二进制的image header等信息；
•实例化主二进制的image loader，校验主二进制和dyld的版本是否匹配；
•检查shared cache是否已经map，没有的话则先执行map shared cache操作；
•检查DYLD_INSERT_LIBRARIES，有的话则加载插入的动态库（实例化image loader）;
•执行link操作。这个过程比较复杂，会先递归加载依赖的所有动态库（会对依赖库进行排序，被依赖的总是在前面），同时在这阶段将执行符号绑定，以及rebase，binding操作；
•执行初始化方法。OC的+load以及C的constructor方法都会在这个阶段执行；
•读取Mach-O的LC_MAIN段获取程序的入口地址，调用main方法。
```



#### 简化版：

①解析 mach-o 文件，找到其依赖的库，并且递归的找到所有依赖的库，形成一张动态库的依赖图。iOS 上的大部分 app 都依赖几百个动态链接库（大部分是系统的动态库），所以这个步骤包含了较大的工作量。

②匹配 mach-o 文件到自身的地址空间

③进行符号查找（perform symbol lookups）：比如 app 中调用了 printf 方法，就需要去系统库中查找到 printf 的地址，然后将地址拷贝到 app 中的函数指针中

④rebase和binding：由于 app 需要让地址空间配置随机加载，所以所有的指针都需要加上一个基地址

⑤运行初始化程序，之后运行 main() 函数



那么这些步骤在性能、安全性和可测试性上应该如何被优化呢？

苹果提出了这样两点思路：

①识别安全性敏感的组件：解析 mach-o 文件并寻找依赖是安全性敏感的，因为恶意篡改的 mach-o 头部可以进行某些攻击，如果一个 app 使用了 @rpath，那么恶意修改路径或者将一些库插入到特定的地方，攻击者就可以毁坏 app。所以这部分工作需要被搬到进程外来完成，比如搬到一个 daemon 进程中。

②识别可以被缓存的部分：符号查找就是其中一个，因为在一个特定的库中，除非软件更新或者这个库被改变，不然每个符号都应该有固定的偏移量。



以上两点思路也是 dyld 3.0 的优化思路。在 dyld 3.0 中，mach-o 头部解析和符号查找工作完成后，这些执行结果会被作为“启动闭包（launch closure）”写入硬盘。



因此iOS操作系统的后台守护进程可以完成所有的这些工作。然后我们确定大量占用资源的部分，也就是占用缓冲的部分。它们是符号查找，因为在给定的库中，除非进行软件更新或者在磁盘上更改库，符号将始终位于库中的相同的偏移位置。



Dyld3中，将这些部分移到上层（图中红色的部分），然后向磁盘写入闭包处理 “Write closure to disk”。这样，启动闭包处理就成了启动程序的重要环节。稍后可以在APP的进程中使用 dyld 3包含的这三个部分，

启动闭包比mach-o更简单。它们是内存映射文件，不需要用复杂的方法进行分析。

我们可以简单的验证它们，这样可以提高速度。



dyld3是部分out-of-process，部分in-process。上图中，虚线之上的部分是out-of-process的，在App下载安装和版本更新的时候会去执行。



#### dyld 3包含三个组件：

①本APP进程外的Mach-O分析器/编译器；

在dyld 2的加载流程中，Parse mach-o headers和Find Dependencies存在安全风险（可以通过修改mach-o header及添加非法@rpath进行攻击），而Perform symbol lookups会耗费较多的CPU时间，因为一个库文件不变时，符号将始终位于库中相同的偏移位置，这两部分在dyld 3中将采用提前写入把结果数据缓存成文件的方式构成一个”lauch closure“（可以理解为缓存文件）。

它处理了所有可能影响启动速度的 search path，@rpaths 和环境变量；它解析 mach-o 二进制文件，分析其依赖的动态库，并且完成了所有符号查找的工作；最后它将这些工作的结果创建成了启动闭包，写入缓存，这样，在应用启动的时候，就可以直接从缓存中读取数据，加快加载速度。 

这是一个普通的 daemon 进程，可以使用通常的测试架构。 

out-of-process是一个普通的后台守护程序，因为从各个APP进程抽离出来了，可以提高dyld3的可测试性。



②本进程内执行”lauch closure“的引擎；验证”lauch closures“是否正确，把dylib映射到APP进程的地址空间里，然后跳转到main函数。此时，它不再需要分析mach-o header和执行符号查找，节省了不少时间。



③”lauch closure“的缓存：

iOS操作系统内置APP的”lauch closure“直接内置在shared cache共享缓存中，我们甚至不需要打开一个单独的文件；

而对于第三方APP，将在APP安装或更新版本时（或者操作系统升级时？）生成lauch closure启动闭包，因为那时候的系统库已经发生更改。这样就能保证”lauch closure“总是在APP打开之前准备好。启动闭包会被写到到一个文件里，下次启动则直接读取和验证这个文件。

在 iOS，tvOS，watchOS 中，一切（生成启动闭包）都是在 app 启动之前做完的。在 macOS 上，由于有 sideload app，进程内引擎会在首次启动时启动一个 daemon，之后就可以使用启动闭包了。总之大部分情景下，这些工作都在 app 启动之前完成了。 

大部分的启动场景都不需要调用这个进程外的 mach-o 解析器。而启动闭包又比 MachO 简单很多，因为它是一个内存映射文件，解析和验证都非常简单，并且经过了良好的性能优化。所以 dyld 3.0 的引入，能让 app 的启动速度得到明显提升。 



总体来说，dyld 3把很多耗时的操作都提前处理好了，极大提升了启动速度。





参考链接：

1.[WWDC2017 session 413](https://developer.apple.com/videos/play/wwdc2017/413/)

2.[WWDC2019 session 423](https://developer.apple.com/videos/play/wwdc2019/423/)

3.[深入理解**iOS App**的启动过程](https://blog.csdn.net/Hello_Hwc/article/details/78317863) 

4.[**App** 启动时间：过去，现在和未来](https://techblog.toutiao.com/2017/07/05/session413/) 

5.[**App** 启动流程以及优化 **WWDC 2017**](https://www.jianshu.com/p/96f66b0c943c) 

6.[iOS 13中的改进和优化](https://easeapi.com/blog/blog/83-ios13-dyld3.html)