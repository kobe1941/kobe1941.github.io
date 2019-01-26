---
layout: post
title: "PLCrashreporter源码分析其一"
date: 2019-01-19 22:53:32 +0800
comments: true
categories: iOS开发
keywords: iOS 崩溃 崩溃日志收集 crash PLCrashreporter plc
description: 

---



大概总共会分三篇文章来分析PLC，第一篇讲整体的工作流程，第二篇梳理PLC内部使用到的一些特殊函数的含义，第三篇会尝试做一个实践。 

不定期更新，我想明白PLC的某个细节可能就会来更新一下。 



<!--more-->



代码在GitHub的[仓库地址]( <https://github.com/plausiblelabs/plcrashreporter> )，只不过GitHub上的代码只是一份镜像，几年都没有更新了。 

有一个[官网](<https://www.plcrashreporter.org/>)可以下载最新的代码和查看部分文档。 

本次以GitHub上的代码为主。clone到本地后打开工程，可以看到以下工程目录： 

![](/images/2019/01/19/1.png) 

CrashDemo目录下的main.m是入口，了解PLC的原理和流程，从这里开始就可以了。



整体流程： 

1.入口 

①main函数里配置好PLCrashReporterConfig，然后初始化一个PLCrashReporter，保存下任何已经存在的崩溃日志save_crash_report(reporter); 

②设置好崩溃时，收集完崩溃日志后的回调PLCrashReporterCallbacks，

③调用PLCrashReporter的 enableCrashReporterAndReturnError函数开启崩溃信号拦截和崩溃日志收集服务。

![](/images/2019/01/19/2.png) 



2.注意上述流程的设置callback这一步，PLCrashReporter内部有一个静态全局变量用来保存该callback函数指针，当崩溃发生时PLC会收集崩溃日志，收集完成后会来回调这个callback，这个是PLC提供给外部的额外回调，你可以在这里做一些类似统计打点或者数据落地的工作。demo里这个callback就仅仅只是打印了一个log而已；

![](/images/2019/01/19/3.png) 

![](/images/2019/01/19/4.png) 

![](/images/2019/01/19/5.png) 



3.PLCrashReporter内部的enableCrashReporterAndReturnError函数，是用来启动崩溃信号捕获的函数；

①首先第一段代码用来确保，单个APP进程仅有一份PLCrashReporter的实例存在；

![](/images/2019/01/19/6.png) 



②创建文件目录，当后面收集到崩溃日志时要立马写文件；

![](/images/2019/01/19/7.png) 



③创建一个崩溃之前的page-guarded allocator，我自己理解是崩溃时用来保护内存的page避免崩溃时的信息丢失。

signal_handler_context是一个全局结构体变量，内部有好几个变量，包括writer，path，allocator，dynamic_loader，如果定义的是Mach的异常，则还有一个mach的port变量，这些变量都会在下面逐一进行初始化赋值。

先看看这个结构体，见下图：

![](/images/2019/01/19/8.png) 



在此时创建的正是这个allocator，如下图，取这个结构体的一个allocator的地址传给构造函数

![](/images/2019/01/19/9.png) 



C++的函数在内部创建好后，对该指针进行赋值：

![](/images/2019/01/19/10.png) 



同时把文件目录赋值给该静态的结构体的一个内部文件指针：

![](/images/2019/01/19/11.png) 

创建dynamic loader，取结构体的该变量的地址，在构造函数内部对该指针进行赋值：

![](/images/2019/01/19/12.png) 

![](/images/2019/01/19/13.png) 



④之后是初始化结构体里的另外一个变量writer。

这个是当崩溃时收集到线程堆栈信息后用来写文件用的，当然还要传入另一个符号化策略的参数PLCrashReporterSymbolicationStrategy，有三个值，None，Table或者Objc，或者Table和Objc，一般选None比较合适，因为你崩溃的时候去做符号化，一来耗时比较长，二来有些APP在Xcode的编译选项里设置了strip symbol的话（见另一篇安装包size优化的文章），这里是拿不到符号的，另外就是网上有人反应，这里的符号化不够准确，没有代码行数，另外也没有系统动态库比如UIKit的符号。所以直接选None就好了，崩溃日志收集到之后，再用符号表统一进行符号化即可。

这个初始化writer的操作跟之前的差不太多，都是传入指针，构造函数new一个对象出来后，对指针进行赋值。多的一点就是writer是一个结构体，内部很多基本类型变量的初始化，也是在这个函数里做的，同时因为类的封装问题，内部还有一个plcrash_async_symbol_strategy_t的枚举跟PLCrashReporterSymbolicationStrategy一一对应，这里也一起做了转换： 

![](/images/2019/01/19/14.png) 



⑤注册C和C++的异常处理 

这里分两种，BSD层和Mach层，不同的层的信号处理策略不太一样，在使用的时候通过枚举变量选择使用哪一层的异常处理。 

PLC默认仅处理这6中信号的异常 

![](/images/2019/01/19/15.png) 



这里有一个分支分别处理BSD层和Mach层的异常，使用的时候可以二选一。 



BSD层 

对信号数组进行遍历，分别给对应的异常信号注册回调和context，这个context是上方提到的全局静态变量，其实就是个单例。 

![](/images/2019/01/19/16.png) 



Mach层 

就像注释里所讲的，Mach层也需要额外对SIGABRT信号做注册。 

Mach层对其他信号的捕获是通过启动一个machServer来实现的，在创建这个machServer的时候，传入了三个变量的地址，方便在构造函数里对这几个变量进行赋值。 

mach_exception_callback是mach层捕获到异常信号时的回调，跟BSD层的signal_handler_callback回调函数想对应。

![](/images/2019/01/19/17.png) 

在开启Mach层的server后，传入的_previousMachProts也被创建好了，之后要设置一下静态全局变量的port_set:

![](/images/2019/01/19/18.png) 



⑥注册OC的异常处理handler函数

![](/images/2019/01/19/19.png) 

PLC没有对OC的异常做多SDK兼容性处理，常见的兼容性处理是调用 NSGetUncaughtExceptionHandler()函数获取之前别的SDK注册的handler，用一个全局变量保存，当OC异常进行回调时，自己处理完后，对之前的handler也进行一次回调，然后再去调用abort函数。

![](/images/2019/01/19/20.png) 

OC的NSException里有异常名字，原因和线程回溯。

![](/images/2019/01/19/21.png) 

上面就是所有的注册信号处理函数的所有流程了，还是比较清晰的。





4.下面来看具体注册信号的细节，这些实现决定了发生崩溃时，怎么进行回调。 

从三个方面来分析，以此是BSD层，Mach层和OC的异常处理。 

①BSD层的信号处理细节 

最外层传入的回调函数是signal_handler_callback，传入之后会被保存到静态全局变量shared_handler_context的callbacks里。

![](/images/2019/01/19/22.png) 

具体的注册信号函数在PLCrashSignalHandler类里registerHandlerWithSignal:函数里。

shared_handler_context则是另一个静态的结构体，也可以认为是个单例。这里保存了传入的context和callBack，同时内部用一个List的结构来支持保存多个信号的注册回调callback和context。

![](/images/2019/01/19/23.png) 

需要注意的是，上方有一个nasync_prepend()的函数，在实际的注册函数里，还会调用nasync_append()函数。这两个函数的差别是，append是拼接到List的末尾，而prepend则是插入到首部的位置。如果已经注册过该信号到List里则直接返回，不做额外处理。

具体在执行上，prepend函数是把传进来的callback和context封装的结构体插入到了callbacks的List的最前面的位置。

而注册函数内部的append函数的操作，其结构体的context是NULL，然后callback是 previous_action_callback，而不是传进来的callback，其是把一个新的结构体，插入到callbacks的List的最后面： 

![](/images/2019/01/19/24.png) 

所以当异常信号发生时，外部传入的callback函数，和这个previous_action_callback函数都会被调用。 

这里要注意一下previous_action_callback这个函数，下面会做分析。 

具体的注册信号函数在PLCrashSignalHandler类里registerHandlerWithSignal:函数里。

每一个信号注册过后，信号和这个信号之前旧的处理函数（ sa_prev）都会被构造成一个结构体，该结构体被append到shared_handler_context全局静态变量的previous_actions结构里，这个previous_actions跟callbacks是一样的List。 

当然，如果一个信号已经被PLC注册过了（就是说已经在previous_actinos里），是不会重复注册的。

![](/images/2019/01/19/25.png) 



如果该信号没注册就进行注册，注册信号使用sigaction函数，关联上signo和sa变量就算注册好了，信号过来时，会回调赋值给sa变量的sa_sigaction的函数。下图中就是 plcrash_signal_handler这个函数，所以崩溃时，最开始的回调入口就是plcrash_signal_handler函数。 

但是如果该信号之前被别的SDK注册过，PLC会保存下来，之后当异常信号发生时再统一进行回调，这里是把之前别的SDK注册该信号的handler添加到了shared_handler_context.previous_actions的List里：

![](/images/2019/01/19/26.png) 

注册的时候，sa_sigaction是取了plcrash_signal_handler这个函数的地址，也就是说，如果有异常信号过来，会回调plcrash_signal_handler这个函数。 

看一下这个函数的实现

![](/images/2019/01/19/27.png) 

注释告诉我们，除非你是进行单元测试，否则这个函数不要去手动调用。 

同时看注释可以知道，如果崩溃发生时，callbacks的List里，没有任何人来处理这个收到的信号，则会重新把该信号抛出来。 

问题：如果处理了，是否信号就不会被再次抛出来了？那么如果APP内有多个SDK都有类似PLC的崩溃收集服务，如何兼容呢？ ——PLC内部做了兼容，sigaction函数会把之前的信号处理handler函数一起保存到callbacks里，崩溃发生时会统一进行回调。但是据说实际工作可能有问题，需要测试才知道。 



继续往下看，这个函数对静态全局变量的callbacks的List进行遍历，对每一个元素进行递归调用，最终每个被添加到callbacks里的handler都会被调用到。 



函数里的prev指针，指向context，而context外部传入的是NULL，next()函数传入NULL，那么会返回_head指针，也就是List的头结点，所以current初始值就是List的头结点。 

如果List里有多个值，也就是说有多个处理信号的callback函数，则递归的调用该函数，其实也就是相当于一个数组，对数组遍历，让数组里的每个元素都能处理该信号。 

![](/images/2019/01/19/28.png) 

上方提到了previous_action_callback这个函数，作为callback加入到shared_handler_context.callbacks的List的尾部。

所以当崩溃发生时遍历callbacks的List时，除了回调之前外部传入的callback函数，也会回调previous_action_callback这个函数。 

我们来看下previous_action_callback这个函数的实现：

就像注释里说的，如果在PLC注册信号之前，进程内已经有别的handler注册了该信号，那么此处也会递归的调用这些handler：

![](/images/2019/01/19/29.png) 

如果没有的话，则执行默认的处理逻辑： 

遍历previous_actions的List，如果信号类型能比对上，则调用之前注册时设置的sa_sigaction函数。这里可以对应最初保存别的SDK注册信号的处理： 

![](/images/2019/01/19/30.png) 

需要注意的是，遍历的时候，只要信号类型（比如SIGABRT）对的上，那么就去找sa_flags对应的标记，如果你注册的是sigaction的sa_sigaction那么回调这个，如果注册的是sa_handler，则对应进行回调。SIG_IGN表示忽略则不进行额外处理，SIGDFL是一个空函数，会在下一篇文章里进行分析，目前知道一下就好。 

if-else的逻辑进行了区分处理。但是不管怎样，只要找到一个handler处理完毕后就直接break跳出循环了。 

![](/images/2019/01/19/31.png) 

关于BSD层的信号回调函数signal_handler_callback，这个函数会在callbacks的List遍历时进行回调，它也是PLC抓线程堆栈信息和写文件的核心。 

注册的时候，取这个函数的地址作为参数传入注册函数，后面崩溃信号来的时候，就会对这个函数进行回调。



下图这个函数的第一步更像是一个清理信号的操作？——崩溃信号过来时，清理掉所有信号的注册handler。正常流程也是收集完崩溃日志后就让APP崩溃，留着这些信号handler也没用。

![](/images/2019/01/19/32.png) 

后面一步则是获取线程状态，去初始化context，然后进行一些BSD和Mach层的信息转换：

![](/images/2019/01/19/33.png) 

之后就是去抓取线程堆栈的状态信息然后写文件了，写完文件后去回调用户额外的一个callback，这个callback是main函数里设置好的：

![](/images/2019/01/19/34.png) 

plcrash_write_report是核心，暂停线程，抓线程堆栈信息，写文件和恢复线程都在这个函数里。

如下图所示，基本步骤是打开指定目录的文件，初始化writer，用writer写文件，然后关闭writer，之后再把数据落地，然后关闭文件。open()，close()，write()这些函数都是Unix标准的系统调用。

![](/images/2019/01/19/35.png) 

具体抓线程信息和写文件的关键实现在plcrash_log_writer_write这个函数里，这个函数内部会去读image_list，获取所有线程，暂停除了当前线程之外的所有线程，

![](/images/2019/01/19/36.png) 

然后写数据。写数据包括写文件头部信息，写手机的硬件和操作系统等信息，然后写线程堆栈，写Binary Images信息，

![](/images/2019/01/19/37.png) 

如果有额外的OC异常信息，也会把OC的异常信息写入到文件，之后还会写一下信号的信息到日志里，

![](/images/2019/01/19/38.png) 

最后就是恢复所有线程让其继续执行，然后一些清理内存和端口的工作。

![](/images/2019/01/19/39.png) 





②Mach层的信号处理细节 

传入一个MachPort的地址和callback函数的地址，供开启server函数的内部赋值用： 

![](/images/2019/01/19/40.png) 

Mach层的开启server的函数内部的实现，exc_mask是一个exception_mask_t类型的枚举变量，用来表示Mach层的异常类型用的。

![](/images/2019/01/19/41.png) 



用外部传入的callback和context作为传入参数创建了一个server，然后用exc_mask返回了一个异常端口port，之后用这个端口去注册了一个task，然后就结束啦，虽然也看不懂具体在做啥，反正就是注册异常信号，有异常信号过来时就回调之类的。

![](/images/2019/01/19/42.png) 

Mach层不太能看得懂里边的具体实现细节，比如上图中的server的构造函数，里边全是mach_port开头的函数，这个后面等我看懂了再来补充吧。

总之有一点原则就是，最初构造server时传入的callback一定会在异常信号过来时会被回调，至于是怎么回调的以后看懂再说嘛。。。

最开始注册Mach异常时外部传入的 _previousMachPorts 在这里进行了赋值： 

![](/images/2019/01/19/43.png) 

接下来看看Mach层的异常回调函数mach_exception_callback的实现，跟BSD层的signal_handler_callback大致是类似的，有几个不同的点：

一是在回调main函数设置的callback之前做了些信号之间的转换，

![](/images/2019/01/19/44.png) 

![](/images/2019/01/19/45.png) 



中间的这一部分，看注释也能明白，就是写线程堆栈信息的，然后Mach层读取线程状态和写文件不是走的C函数，而是一个汇编语言实现的函数plcrash_async_thread_state_current 。

![](/images/2019/01/19/46.png) 



二是多了个PLCrashMachExceptionForward函数，目的是让其他注册的server也能够处理收到的异常信号，当然这个函数内部要么调用exception_raise函数针对端口重新抛出异常信号，要么返回失败让程序继续往下走，如果重新抛出异常，则mach_exception_callback这个函数会根据raise函数的返回值（正常流程是返回失败，所以要看exception_raise的返回值）决定是否需要直接返回。

![](/images/2019/01/19/47.png) 

![](/images/2019/01/19/48.png) 



③OC的异常处理函数比较简单，上方已经贴过代码实现。简而言之就是把 signal_handler_context这个静态全局变量的writer传入到OC的异常处理函数里，当系统给OC异常做回调时，拿到writer，把NSException的name和reason还有backtrace这些值写到writer里。在writer写文件落地时，就可以把这些数据一起写到崩溃日志里了。 

实际上你看崩溃日志也可以发现OC的异常，是有reason的，比如常见的数组越界，不能响应的方法之类的，都会在崩溃名字里直接显示出来，而C和C++的崩溃日志，其名字里就只有崩溃的信号名称比如SIGSEGV，而没有更多的原因和信息。



关于OC的异常处理，这里有个疑问点，APP发生了OC的异常崩溃后，是怎么抓取所有线程的堆栈的？——可能OC的异常，一样会触发BSD或者Mach层的异常信号。看代码逻辑是这样子，待验证。

下图中，OC异常的信息写入是在线程和Binary Images信息之后的位置的， 

![](/images/2019/01/19/49.png) 

这是OC的异常过来时，uncaught_exception_handler这个函数调用的。

![](/images/2019/01/19/50.png) 





5.PLC的自定义异常日志收集 

除了真的崩溃的时候可以收集到所有线程堆栈的日志，PLC还支持自定义错误信息收集，此时会去抓所有线程的堆栈，预留了接口来实现。 

![](/images/2019/01/19/51.png) 



需要注意的是，生成自定义日志的接口需要传入一个thread对象，这个thread表示crash的线程。函数里边做了个判断，如果传入的thread是当前正在执行代码的线程，则走的是汇编的函数实现，否则才去走跟BSD层一样的C函数实现逻辑。PLC的单元测试里都是新建一个thread传入这个接口。

![](/images/2019/01/19/52.png) 

事实上， plcrash_log_writer_write这个函数内部也用断言做了判断，注释里也写的比较清楚，如果crash的线程是当前执行代码的线程，则必须提供context：

![](/images/2019/01/19/53.png) 

要打印当前执行代码的线程，必须要提供context，否则无法抓取到线程的堆栈信息

![](/images/2019/01/19/54.png) 





6.其他信息 

原始崩溃日志的文件格式是protobuf，PLC有一个专门的格式化的类可以用来处理格式的转换，上传崩溃日志的时候用的上。 

另外protobuf格式的文件比json格式的文件压缩比要高，同一份文件protobuf格式比json格式的数据量要小，所以一些公司在TCP长连接的时候会采用这种格式来传输数据，消耗的流量更小一些，对弱网的支持会更好。 





TODO: 

1.把PLC放到demo里跑起来，测试一下从收到信号到重新抛出信号，PLC的总耗时和各个阶段的耗时； 

2.测试一下崩溃发生时，OC异常和C++异常的回调时序是怎样的？ BSD或Mach层的信号回调和OC的信号回调，这两者会有一个先后顺序的问题，需要测试一下先后顺序，是否会影响到崩溃日志的收集？ 

3.在PLC的基础上尝试封装完成一个检测ANR和捕获到ANR时抓堆栈信息的模块，测试抓堆栈和写文件的时间，也许可以改成异步写文件；

4.OC的异常处理函数这里处理完后直接调用了abort()函数，理论上APP这个时候会崩溃，但是这个信号按照原理来说同样会被PLC捕获，具体的处理还有待分析。 