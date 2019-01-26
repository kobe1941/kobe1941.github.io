---
layout: post
title: "PLCrashreporter源码分析其二"
date: 2019-01-25 23:26:26 +0800
comments: true
categories: iOS开发
keywords: iOS 崩溃 崩溃日志收集 crash PLCrashreporter plc
description: 

---

本篇文章主要对PLC内部用到的一些当时看源码时不太理解的底层函数进行分析，以及对一些为了理解PLC的原理需要了解的概念进行汇总。



<!--more-->



1.BSD和Mach层，POSIX层的区别和含义 。

先看YYKit的作者对这一段的[表述]( <https://blog.ibireme.com/2015/05/18/runloop/>) 。

![](/images/2019/01/25/1.png) 

XNU 内核的内环被称作 Mach，其作为一个微内核，仅提供了诸如处理器调度、IPC (进程间通信)等非常少量的基础服务。 

BSD 层可以看作围绕 Mach 层的一个外环，其提供了诸如进程管理、文件系统和网络等功能。 

IOKit 层是为设备驱动提供了一个面向对象(C++)的一个框架。 

Mach 本身提供的 API 非常有限，而且苹果也不鼓励使用 Mach 的 API，但是这些API非常基础，如果没有这些API的话，其他任何工作都无法实施。在 Mach 中，所有的东西都是通过自己的对象实现的，进程、线程和虚拟内存都被称为”对象”。和其他架构不同， Mach 的对象间不能直接调用，只能通过消息传递的方式实现对象间的通信。”消息”是 Mach 中最基础的概念，消息在两个端口 (port) 之间传递，这就是 Mach 的 IPC (进程间通信) 的核心。 

为了实现消息的发送和接收，mach_msg() 函数实际上是调用了一个 Mach 陷阱 (trap)，即函数mach_msg_trap()，陷阱这个概念在 Mach 中等同于系统调用。当你在用户态调用 mach_msg_trap() 时会触发陷阱机制，切换到内核态；内核态中内核实现的 mach_msg() 函数会完成实际的工作，如下图：

![](/images/2019/01/25/2.png) 

Mach 是一个 XNU 的微内核核心，Mach 异常是指最底层的内核级异常，被定义在 下 。每个 thread，task，host 都有一个异常端口数组，Mach 的部分 API 暴露给了用户态，用户态的开发者可以直接通过 Mach API 设置 thread，task，host 的异常端口，来捕获 Mach 异常，抓取 Crash 事件。 

所有 Mach 异常都在 host 层被ux_exception转换为相应的 Unix 信号，并通过threadsignal将信号投递到出错的线程。iOS 中的 POSIX API 就是通过 Mach 之上的 BSD 层实现的。 

因此，`EXC_BAD_ACCESS (SIGSEGV)`表示的意思是：Mach 层的`EXC_BAD_ACCESS`异常，在 host 层被转换成 `SIGSEGV` 信号投递到出错的线程。 

![](/images/2019/01/25/3.png) 



2.注册信号的函数在`PLCrashSignalHandler`类里`registerHandlerWithSignal:`函数里。这个函数会首先调用`sigaltstack`函数，它的具体作用？

![](/images/2019/01/25/4.png) 

`sigaltstack`函数的作用：

① 分配一块内存区，当然是从堆中分配，这块内存区就称为“可替换信号栈”(alternate signal stack)，顾名思义，我们就是希望将信号处理函数的栈挪到堆中，而不和进程共用一块栈区。 

② 使用 `sigaltstack()` 系统调用通知内核“可替换信号栈”已经建立。 

③ 接着建立信号处理函数，此时需要对 `sigaction()` 函数的 `sa_flags` 成员设立 `SA_ONSTACK` 标志，该标志告诉内核信号处理函数的栈帧就在“可替换信号栈”上建立。 



回到 sigaltstack() 函数，该函数的第 1 个参数 sigstack 是一个 stack_t 结构的指针，该结构存储了一个“可替换信号栈” 的位置及属性信息。第 2 个参数 old_sigstack 也是一个 stack_t 类型指针，它用来返回上一次建立的“可替换信号栈”的信息(如果有的话)。 

要想创建一个新的可替换信号栈，ss_flags必须设置为0，ss_sp和ss_size分别指明可替换信号栈的起始地址和栈大小。系统定义了一个常数SIGSTKSZ，该常数对极大多数可替换信号栈来说都可以满足需求，MINSIGSTKSZ规定了可替换信号栈的最小值。

如果想要禁用已存在的一个可替换信号栈，可将ss_flags设置为SS_DISABLE。

而`sigaltstack`第一个参数为创建的新的可替换信号栈，第二个参数可以设置为NULL，如果不为NULL的话，将会将旧的可替换信号栈的信息保存在里面。函数成功返回0，失败返回-1.



见stack_t的数据结构：

![](/images/2019/01/25/5.png) 

这两个参数都可以设置为 NULL。比如说，如果只是想找出已有的“可替换信号栈”而不需要去改变它，那么可以将 sigstack 这个参数指定为 NULL 。 

stack_t 结构中的 ss_sp 成员指定了“可替换信号栈”的起始地址(内核在分配这个地址时会根据不同的硬件平台而自动对齐的，这个无需担心)，ss_size 成员指出了该栈的大小。 

一般的，“可替换信号栈” 既可以动态分配，也可以静态分配。通常，我们可以利用 SIGSTKSZ 这个常数(iOS里为128k)指定该栈的大小，另一个常数 MINSIGSTKSZ 则表示该栈最小分配需求(在iOS里为32k)。 

![](/images/2019/01/25/6.png) 

需要注意的是，内核并不会再重新划分“可替换信号栈”的大小。如果所分配的栈溢出了，那么结果将是错乱的。但是这种情况通常也无需担心，因为我们一般只用“可替换信号栈”来处理一些标准栈(主进程所用的栈)溢出这种特殊情况，因此在这种情形下，我们往往只是在 SIGSEGV 信号处理函数里做些清理，或是结束进程，抑或是使用非局部跳转用以解除标准栈溢出的问题，因此在“可替换信号栈”中也就只会分配一个或者少数几个栈帧，故而不用太担心会在此间造成溢出问题。 



stack_t 结构中的 ss_flags 成员可以使用下面两个值： 

SS_ONSTACK

如果在从当前建立的“可替换信号栈”(old_sigstack)中获取相关信息时设置该标志，那么表示进程当前正在“可替换信号栈”中执行，如果此时试图去建立一个新的“可替换信号栈”，那么会遇到 EPERM (禁止该动作) 的错误。 

SS_DISABLE

如果在返回的 old_sigstack 中看到此标志，那么说明当前没有已建立的“可替换信号栈”。如果在 sigstack 中指定该标志，那么当前禁止建立“可替换信号栈”。 



结论：`sigaltstack`函数的作用就是在在堆中为函数分配一块区域，作为该函数的栈使用。所以，虽然递归函数将系统默认的栈空间用尽了，但是当调用我们的信号处理函数时，使用的栈是它实现在堆中分配的空间，而不是系统默认的栈，所以它仍旧可以正常工作。

![](/images/2019/01/25/7.png) 

![](/images/2019/01/25/8.png) 

一般来说，使用可替换信号栈的步骤如下：

①在内存中分配一块区域作为可替换信号栈

②使用`sigaltstack()`函数通知系统可替换信号栈的存在和内存地址

③使用`sigaction()`函数建立信号处理函数的时候，通过将`sa_flags`设置为`SA_ONSTACK`来告诉系统信号处理函数将在可替换信号栈上面运行。



3.`Sigaction`信号处理相关函数的含义 。

函数原型：

`intsigaction(int signum, const struct sigaction * act, struct sigaction * oldact); `

第一个参数signum指明我们想要改变其信号处理函数的信号值。注意，这里的信号不能是`SIGKILL` 和`SIGSTOP`。这两个信号的处理函数不允许用户重写，因为它们给超级用户提供了终止程序的方法（ `SIGKILL` and `SIGSTOP` cannot be caught, blocked, or ignored）。 

第二个和第三个参数是一个struct sigaction的结构体，该结构体在 `<signal.h>` 中定义，用来描述信号处理函数。如果act不为空，则其指向信号处理函数（修改信号的处理函数为新的act里的函数指针）。如果oldact不为空，则之前的信号处理函数将保存在该指针中。PLC就是通过这个函数来获取别的SDK注册的信号处理函数，并将其函数指针保存下来，在崩溃时进行回调。也就是说PLC本身的源码除了OC的异常外，其他信号异常都支持多个SDK的崩溃收集服务共存。

如果act为空，则之前的信号处理函数不变。我们可以通过将act置空，oldact非空来获取当前的信号处理函数。

![](/images/2019/01/25/9.png) 

![](/images/2019/01/25/10.png) 

Union联合体的意思是，这个数据结构里，在不同时候只会有一个数据有值，可能是其中的任何一个。目的是为了节省内存。 

`sa_handler`是一个函数指针，指向我们定义的信号处理函数，该值也可以是SIG_IGN（忽略信号）或者SIG_DEL（使用默认的信号处理函数）。

sa_mask字段说明了一个信号集，信号处理函数执行期间这一信号集要加到进程的信号屏蔽字中。仅当从信号处理函数返回时再将进程的信号屏蔽字复位为原先的值。这样在调用信号处理函数时就能阻塞某些信号。在信号处理函数被调用时，操作系统建立的新信号屏蔽字包括正在被递送的信号。因此保证了在处理一个给定信号时，如果这种信号再次发生，那么它会被阻塞到对前一个信号的处理结束为止。

sa_flags字段指定对信号处理的一些选项，有两个PLC里用到的选项，其含义说明如下（在 <signal.h>中定义）： 

SA_SIGINFO

此选项对信号处理程序提供了附加信息：一个指向siginfo结构的指针以及一个指向进程上下文标识符的指针。

SA_ONSTACK

若用`sigaltstack`声明了以替换栈，则将此信号递送给替换栈上的进程。

`sa_sigaction`是一个替代的信号处理函数，当sa_flags字段设置为SA_SIGINFO时，使用该信号处理函数。需要注意的是，对于`sa_sigaction`和`sa_handler`字段，其实现可能使用同一存储区，所以应用程序只能一次使用这两个字段中的一个。



4.这一段代码的作用是什么？

![](/images/2019/01/25/11.png) 

`SIG_DFL`是一个空函数，清掉所有信号的注册`sa_handler`，注意下方注册信号的handler函数时，用的是`sa_sigaction`而不是`sa_handler`。 

`SIG_DFL`这个宏就是把0强制转换给所定义的函数指针，意思就是一个空函数，没有任何实现。 

这是C语言的知识点，详细推理见底部的参考链接。



所以这一段代码的意思就是把数组里定义的信号，其默认的信号处理动作全部清掉。 

在PLC的实现中，这一段代码是放在`signal_handler_callback`函数里的，所以我猜原作者的意思是，当有异常信号被PLC捕捉到，调用到指定的callback函数时，把所有之前初始化PLC时注册的信号handler函数全部清掉，这是为了避免之后PLC写完线程堆栈信息文件后，重新抛出信号时再次捕获相同的信号？PLC的正常流程是写完崩溃日志后重新抛出异常信号，一般这个时候APP就崩溃了。——这是猜测，因为这个机制好像只对BSD层有效，对Mach层是怎么处理的还不知道？？

另外一点就是，清理所有向操作系统的信号注册，也意味着，如果你的APP集成了PLC，那么如果有第三方的SDK也有崩溃收集服务，只要你在别的SDK初始化后继续调用一下这段清理代码，别的SDK的崩溃收集服务功能就被干掉啦。本身如果自己已经集成了PLC的话，就不再需要第三方的崩溃收集服务了，而且只有你自己才有对应的符号表，所以别的SDK就算收集到了崩溃日志，也解析不出来，没啥用。当然使用PLC的`PLCrashReporterSymbolicationStrategy`不为`PLCrashReporterSymbolicationStrategyNone`那么还是会有一点符号信息的，不过，在做安装包size优化的时候，编译选项里大多数都已经去掉了符号了，所以很可能很多APP内部是没有符号的，你不选`PLCrashReporterSymbolicationStrategyNone`的枚举选项，拿到的日志也可能全部都是地址没法看。





5.`raise()`函数抛出异常的含义；.

——标准的Linux函数调用。 

发生信号的函数: kill()、raise()。 

捕捉信号的函数: alarm()、pause()。

处理信号的函数: signal()、sigaction()。

与kill()函数不同的是，raise()函数允许进程向自身发送信号。

PLC默认的处理是，当没有PLC没有callback来处理信号，就重新抛出信号，如果处理了就不抛出了。所以可能市场上的各个SDK，可能自己修改的PLC的代码实现。 

![](/images/2019/01/25/12.png) 



6.`mach_task_self()`函数是做什么的？ 

——获取当前所在的进程，下图的`task_threads`函数，传入当前进程后，可以获取当前进程的所有线程。 

![](/images/2019/01/25/13.png) 





7.`pl_mach_thread_self()`这个函数是做什么的？有点像是获取当前所在的线程？

![](/images/2019/01/25/14.png) 

![](/images/2019/01/25/15.png) 

——是的，`mach_thread_self()`函数可以获取当前线程。

```
kern_return_t task_threads
(
    task_inspect_t target_task,
    thread_act_array_t *act_list,
    mach_msg_type_number_t *act_listCnt
);
```

iOS的操作系统是基于Darwin内核实现的，这个内核提供了task_threads接口让我们获取所有的线程列表（注意这里的线程是最底层的 mach 线程。），以及接口thread_info来获取单个线程的信息。 

`task_threads`的第一个参数的`target_task`传入进程标记，这里使用`mach_task_self()`获取当前进程，后面两个传入两个指针分别返回线程列表和线程个数。

对于每一个线程，可以用 `thread_get_state` 方法获取它的所有信息，信息填充在 _STRUCT_MCONTEXT 类型的参数中。

在 _STRUCT_MCONTEXT 类型的结构体中，存储了当前线程的 Stack Pointer 和最顶部栈帧的 Frame Pointer，从而获取到了整个线程的调用栈。 

进程的内存使用信息同样放在了另一个结构体`mach_task_basic_info`中，存储了包括多种内存使用信息

详情见底部的参考链接。 



8.`pthread_cond_init()`这个函数是什么意思？ 

——条件锁。一般和 `pthread_cond_signal()` 以及` pthread_cond_wait()` 配套使用。 

![](/images/2019/01/25/16.png) 

![](/images/2019/01/25/17.png) 



9.set_reading(true)和set_reading(false)经常成对出现，这是某种形式的锁？ 

——对，其实对其包围的代码片段进行加锁。见下方的解释 

![](/images/2019/01/25/18.png) 





10.`OSMemoryBarrier()`函数的作用。

为了达到最佳性能，编译器通常会将汇编级别的指令进行重新排序，从而保持处理器的指令流水线尽可能的满。作为优化的一部分，编译器可能会对内存访问的指令进行重新排序(在它认为不会影响数据的正确性的前提下)，然而，这并不一定都是正确的，顺序的变化可能导致一些变量的值得到不正确的结果。如果看似独立的变量实际上 是相互影响,那么编译器优化有可能把这些变量更新成了错误的顺序,导致潜在不不正确结果。 

Memory Barriers是一种不会造成线程block的同步工具，它用于确保内存操作的正确顺序。Memory Barriers像一道屏障，迫使处理器在其前面完成必须的加载或者存储的操作。Memory Barriers常被用于确保一个线程中可被其他线程访问的内存操作按照预期的顺序执行。

Barriers严格限制了内存访问顺序。所有出现在barriers之前的加载和存储操作完成后，才会运行barriers之后的加载和存储操作。

大多数代码都应该使用barrier函数来确保在线程间共享的内存是正确同步的。

例如，如果我们想要初始化一个共享的数据结构，然后自动增加某个变量值来标识初始化操作完成，则我们必须使用OSAtomicIncrement32Barrier来确保数据结构的存储操作在变量自动增加前完成。

同样的，该数据结构的消费者也必须使用`OSAtomicIncrement32Barrier`，以确保在自动递增变量值之后再去加载这些数据。另一方面，如果我们只是简单地递增一个全局计数器，那么使用`OSAtomicIncrement32`会更安全且可能更快。

如果不能确保我们使用的是哪个版本，则使用barrier变量以保证是安全的。

值得注意的是，大部分锁类型都合并了内存屏障，来确保在进入临界区之前它前面的加载和存储指令都已经完成。

另外，自旋锁和队列操作总是包含一个barrier。

`OSMemoryBarrier()`函数就是用来设置内存屏障，它即可以用于读操作，也可以用于写操作。

相较于`@synchronized`，OSAtomic原子操作更趋于数据的底层，从更深层次来对单例进行保护。同时，它没有阻断其它线程对函数的访问。





11.崩溃日志里的Binary Images是什么，有什么作用？ 

崩溃日志里的这部分内容列出了在进程被终止时加载在进程中的二进制文件（binary images）。

image加载的时候都会相对基地址进行重定位，并且每次加载的基地址都不一样，函数栈frame的地址是重定位后的绝对地址，我们要的是重定位前的相对地址。

可以看到Crash Log的Binary Images块包含每个image加载起止地址、image名、arm架构、uuid、image路径。

Crash时刻App加载的所有的库，其中第一行是Crash发生时我们App可执行文件的信息，可以看出为armv7，可执行文件的包得uuid位c0f……cd65，解析Crash的时候dsym文件的uuid必须和这个一样才能完成Crash的符号化解析。



每一行都包含了一个二进制文件的以下细节信息：

①在进程内的二进制文件的地址空间

②一段二进制的名称或者bundle id（仅针对macOS）。一个MacOS的crash report，如果二进制是OS的一部分，会在前面加上a。

③（仅针对macOS）二进制的短版本(short version)和bundle版本，通过破折号来分割。

④（仅针对iOS）二进制文件的架构名。一个二进制可能包含多个分片，每一个架构它都支持。其中只有一个可以被加载到进程中。

⑤一个可以唯一标示二进制文件的id，即UUID。这个值会随每一次构建而发生变化，并且它会用来定位需要符号化时的dSYM文件。

⑥磁盘上二进制文件的path。





12.PLC源码里C++实现的aync_list是数组还是链表，或者是其他的什么容器？ 

——看代码实现应该是链表 



13.hpp文件的含义。

——hpp,其实质就是将.cpp的实现代码混入.h头文件当中，定义与实现都包含在同一文件，则该类的调用者只需要include该hpp文件即可，无需再 将cpp加入到project中进行编译。而实现代码将直接编译到调用者的obj文件中，不再生成单独的obj,采用hpp将大幅度减少调用 project中的cpp文件数与编译次数，也不用再发布烦人的lib与dll,因此非常适合用来编写公用的开源库。——百度百科 





补充知识点： 

1.abort函数会先清除对SIGABRT信号阻塞（如果有阻塞的话），然后调用raise函数向调用进程发送信号。注意：如果abort函数使得进程终止了，那终止前会刷新和关闭所有打开的流。

kill 函数发送信号给进程或进程组， raise 函数发送信号给自己。

abort 函数可以发送 SIGABRT 信号到调用进程，相当于 raise(SIGABRT) ，它使程序终止。



2.如何检测后台线程中更新UI？

从Xcode9开始，诊断选项里有个叫"Main Thread checker"的，默认是打开的，在程序运行期间，如果检测到了主线程之外的线程中更新UI，那么会在控制台中打出警告。但问题是，很多开发者选择无视，需要依赖于开发者的自觉，才能避免之类的问题。

也可以自己去实现一套机制，原理是通过hook UIView的-setNeedsLayout, -setNeedsDisplay, -setNeedsDisplayInRect三个方法，确保它们都是在主线程中执行。如果不是，那么让程序发生崩溃，可以强制开发者去修改。 



3.关于Mach和BSD层，以及Mach层的异常处理见底部念茜的文章。

Mach异常是什么？它又是如何与Unix信号建立联系的？

Mach是一个XNU的微内核核心，Mach异常是指最底层的内核级异常，被定义在下 。每个thread，task，host都有一个异常端口数组，Mach的部分API暴露给了用户态，用户态的开发者可以直接通过Mach API设置thread，task，host的异常端口，来捕获Mach异常，抓取Crash事件。 

所有Mach异常都在host层被ux_exception转换为相应的Unix信号，并通过threadsignal将信号投递到出错的线程。iOS中的 POSIX API 就是通过 Mach 之上的 BSD 层实现的。 

![](/images/2019/01/25/19.png) 





参考链接： 



1.[深入理解runloop](<https://blog.ibireme.com/2015/05/18/runloop/> )

2.[sigaltstack() -- 替换信号处理函数栈](<http://www.groad.net/bbs/forum.php?mod=viewthread&tid=7336>)

3.[Linux 信号应用之黑匣子程序设计](<http://blog.jobbole.com/101619/>) 

4.[信号之sigaction函数](<https://www.cnblogs.com/nufangrensheng/p/3515945.html>)

5.[#define SIG_DFL ((void(*)(int))0)](<https://www.cnblogs.com/liulipeng/p/3555395.html>)

6.[* (*(void(*) ())0)();------这是什么？](<https://blog.csdn.net/yishizuofei/article/details/78351119>)

7.[质量监控-资源使用](<https://www.jianshu.com/p/22a077fd51f1>)

8.[获取任意线程调用那些事](<https://bestswifter.com/callstack/>)

9.[OSAtomic原子操作](<http://southpeak.github.io/2014/10/17/osatomic-operation/>)

10.[漫谈iOS crash收集框架](<http://www.cocoachina.com/ios/20150701/12301.html>)