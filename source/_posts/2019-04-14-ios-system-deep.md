---
layout: post
title: "《深入解析Mac OS X & iOS操作系统》"
date: 2019-04-14 21:14:53 +0800
comments: true
categories: iOS开发 读书笔记
keywords: 操作系统 内核 iOS Mach BSD
description: 

---



整体上还算不错，对内核，Mach和BSD的介绍，包括一些函数和调用等，以及对异常处理的描述，对于理解PLC这个崩溃日志生成的库还是很有帮助的。同时一些内核线程、用户线程的概念也算是填补部分知识空白。引导过程虽然讲的很详细，不过兴趣不大。



<!--more-->

## 名词解释



ASLR：Address Space Layout Randomization 地址空间布局随机化 

XNU：X is Not UNIX 

POSIX：Portable Operating System Interface 可移植操作系统接口 

XPC：高级IPC框架，实现进程间特权的分离 

ELF：Executable and Library Format ，Unix下的可执行文件格式 

__LINKEDIT：由dyld使用，这个区包含了字符串表、符号表以及其他数据 

pthread：就是BSD层实现的POSIX规范的线程，Mach层则是mach_thread 

DFU：设备固件更新模式，这个模式用于更新iOS镜像 

HFS：Hierarchical File System 层次文件系统 

FAT：File Allocation Table 文件分配表 

NFS：Network File System网络文件系统 

VFS：Virtual File System虚拟文件系统 

LR寄存器：保存了当前函数的返回地址 

PC寄存器：指令指针，程序计数器，PC寄存器中的内容，是下一条要取来执行的指令的地址

ABI：Application Binary Interface 应用程序二进制接口 





## 第一部分 第一到三章 高级用户指南 



1.iOS系统的GUI是springboard，这是大家熟知的触屏应用加载器。 

2.Darwin是操作系统的类UNIX核心，本身由内核，XNU和运行时组成。 

3.XNU实际上是由两种技术混合在一起的：Mach和BSD，此外还添加了一些其他的组件，主要是IOKit。 

4.在Mac OS中，“bundle”这个词实际上描述的是两种不同的术语：第一种是本节中讨论的目录结构；第二种是共享库目标的一种文件目标格式，共享库由进程显示的加载（普通的库是隐式加载的）。这个词有时候也表示一个插件。 

5.OS X是在Mach内核的基础上构建的，而Mach是NeXTSTEP的遗产。BSD层是对Mach内核的包装，但是Mach系统调用仍然可以在用户态访问。 

6.内核XNU是Darwin的核心，也是整个OS X的核心。XNU本身由以下几个组件构成：Mach微内核，BSD层，libKern，IOKit。 

7.Mach微内核仅能处理操作系统最基本的职责：进程和线程抽象，虚拟内存管理，任务调度，进程间通信和消息传递机制。 

8.BSD层建立在Mach层之上，BSD层提供了更高层次的抽象，其中包括：UNIX进程模型，POSIX线程模型（Pthread）及其相关的同步原语，UNIX用户和组，网络协议栈（BSD Socket API），文件系统访问，设备访问（通过/dev目录访问）。 

9.OS X提供了一个系统级的通知机制。这是分布式IPC的一种形式，进程可以通过这种机制广播或监听事件。通知机制的核心部分在于notifyd(8) 守护程序，这个守护程序是在系统引导时启动的，这是Darwin的通知服务器。还有一个守护程序 distnoetd(8) 的作用是分布式通知服务器。notify(8) 默认使用Mach消息并注册Mach端口 com.apple.system.notification_center，能够处理大部分通知。notify(8)还有一个有意思的特点，这个API允许通过Mach消息传递文件描述符。 

10.苹果通过代码签名，沙盒以及entitlement授权文件来保证iOS和OS X的安全。 

11.沙盒机制也有一个专用的守护程序usr/libexec/sandboxd，这个程序运行在用户态，提供了跟踪功能以及内核扩展所需要的辅助服务，这个守护程序是根据需要启动的。 





## 第四章 进程线程和Mach-O 



1.尽管同一个可执行程序可以并发的启动多个实例，但是每一个实例都有一个不同的PID。 

2.子进程返回的整数由其父进程收集。进程将要返回的值传递给exit(2) 系统调用（或者从main()函数返回）。 

3.一个进程内的所有线程都共享虚拟内存空间，文件描述符和各种句柄。 

4.进程的生命周期开始于SIDL状态，在这个状态中，进程仍被定义为”正在初始化“，不会响应任何信号，也不会进行任何操作。 

![](/images/2019/04/1.png) 

5.睡眠的进程也会被信号唤醒。通过一个特殊的信号（TSTOP或TOSTOP）可以使一个进程停止执行。这相当于”冻结“了进程（即同时挂起这个进程的所有线程），将这个进程置于”深度睡眠“的状态。恢复这个进程的唯一方法就是发送另一个信号（CONT），这个信号可以将进程切换回可运行状态，使得进程中的每一个线程都可以被重新调度了。 

6.终止一个进程会同时终止其所有线程。但是在进程完成终止之前，会短暂的处于僵尸（zombie）状态。每一个进程在安息之前都会有很短暂的时间处于这个状态。僵尸进程都是绝对死亡的进程，僵尸进程只是进程的空壳，所有的资源都被释放了，但是仍然占用着PID。 

7.pid_suspend”冰冻“一个进程，pid_resume”解冻“一个进程。冰冻背后的挂起操作是在更为底层的Mach任务的层次实现的，而不是进程的层次实现的，所以这个睡眠更深。iOS的springboard大量使用了这些调用，比如用户按下Home键时。 

8.iOS还添加了一个私有的系统调用 pid_shutdown_sockets，这在OS X是没有的。这个系统调用可以在进程之外关闭这个进程所有的套接字。只有springboard使用了这个调用，多半是挂起进程时使用的。 

9.信号指发送给程序的异步通知，其中不包括数据（或只包括非常少量的数据）。信号是操作系统发送给进程的，用于表示发生了某种条件，而这种条件通常是因为某类硬件错误或程序异常而产生的。除了SIGKILL之外，进程可以通过一些系统调用屏蔽或处理以下的所有错误。 

![](/images/2019/04/2.png) 

![](/images/2019/04/3.png) 

10.通用二进制（胖二进制）格式只不过是其支持的各种架构的二进制文件的打包文件。也就是说，这种格式的文件包含一个非常简单的文件头，文件头后面依次拷贝了每一种支持架构的二进制文件。lipo这个工具可以提取、删除或替换通用二进制文件中制定架构的二进制代码，因此可以用于对通用二进制文件进行”瘦身“。这个工具还可以显示胖二进制文件头的详细信息。 

11.调用一个二进制文件时，Mach加载器会首先解析胖二进制文件头，确定其中可用的架构，然后只加载最适合的架构的代码，因此不相关架构的代码不会占用任何内存。 

12.Mach-O格式具有一个固定的文件头，文件头一开始是一个魔数值，加载器可以通过这个魔数值快速判断这个二进制文件用于32位还是64位，在魔数值之后跟着的是CPU类型及子类型字段。 

13.在iOS中，没有暴露sysct接口，堆和栈都默认不可执行。 

14.Mach-O文件头的主要功能在于加载命令load command。otool工具可用于分析Mach-O文件。 

15.加载过程在内核的部分负责新进程的基本设置——分配虚拟内存，创建主线程，以及处理任何可能的代码签名/加密的工作。然而对于动态链接的可执行文件（大部分可执行文件都是动态链接的）来说，真正的库加载和符号解析的工作都是通过LC_LOAD_DYLINKER命令指定的动态链接器在用户态完成的。控制权会转交给链接器。 

16._PAGEZERO段（空指针陷阱）、_TEXT段（程序代码）、_DATA段（程序数据）和_LINKEDIT（链接器使用的符号和其他表）段提供了LC_SEGMENT命令。段有时候也可以进一步分解为区（section）。 

17.当所有的库都完成加载之后，dyld的工作也完成了，之后由LC_UNIXTHREAD命令负责启动二进制程序的主线程（因此主线程总是在可执行文件中，而不会在其他二进制文件中例如库文件）。 

18.LC_THREAD用于核心转储文件，Mach-O核心转储文件实际上是一组LC_SEGMENT命令的集合，这些命令负责建立起进程的内存镜像。 

19.LC_MAIN命令的作用是设置程序主线程的入口点地址和栈大小。 

20.Mach-O二进制文件有一个重要特性就是可以进行数字签名。LC_CODE_SIGNATURE包含了Mach-O二进制文件的代码签名，如果这个签名和代码本身不匹配（或者在iOS上这条命令不存在），那么内核会立即给进程发送一个SIGKILL信号，将进程杀死，没有商量的余地。 

21.dyld是一个用户态的进程，dyld不属于内核的一部分，而是作为一个开源的项目由苹果单独维护的。 

22.可执行文件很少是独立的，除了极少数的一些静态链接的可执行文件，大部分可执行文件都是动态链接的。 

23.内核加载器执行的设置工作包括根据段的描述初始化进程地址空间以及执行其他命令。然而，仅有非常少量的进程只需要内核加载器就可以完成加载，而OS X上几乎所有的程序都是动态链接的。也就是说，Mach-O镜像中有很多”空洞“——即对外部的库和符号的引用——这些空洞要在程序启动时填补。这项工作就需要由动态链接器来完成。这个过程有时候也称为符号绑定（binding）。 

24.动态链接器是内核执行LC_DYLINKER 加载命令时启动的。通常情况下，使用的是 /usr/lib/dyld 作为动态链接器，不过这条加载命令可以指定任何程序作为参数。链接器接管刚创建的进程的控制权，因为内核将进程的入口点设置为链接器的入口点。链接器索要完成的工作，就是查找进程中所有的符号和库的依赖关系，然后解决这些关系。这个过程必须递归的完成，因为通常情况下库还会依赖于其他的库。 

25.通过otool -L命令可以显示库的依赖关系。 

26.LC_LOAD_DYLIB命令告诉链接器在哪里可以找到这些符号。链接器要加载每一个指定的库，并且搜寻匹配的符号。被链接的库有一个符号表，符号表将符号名称和地址关联起来。符号表在Mach-O目标文件中的地址可以通过LC_SYMTAB加载命令指定的symoff找到。 

27.libSystem库是系统上所有二进制代码的绝对先决条件，不论是C、C++还是Objective-C的程序。这是因为这个库是对底层系统调用和内核服务的接口，如果没有这些接口就什么事也干不了。 

28.共享库缓存指的是一些库经过预先链接，然后保存在磁盘上的一个文件中。iOS中大部分常用的库都被缓存了。这个概念有点类似安卓的prelink-map，在prelink-map中的库被提前链接到地址空间中的固定偏移处。 

29.为了节省加载的时间，iOS的dyld采用了一个共享库链接缓存，苹果从iOS3.0开始将所有的基础库都移到了这个缓存中。 

30.在OS X中，dyld共享库缓存保存在/private/var/db/dyld目录下。在iOS中，共享库缓存可以在/System/Library/Caches/com.apple.dyld 中找到。这个缓存是一个单独的文件，即dyld_shared_cache_armv7。共享库缓存都会增长到非常庞大，OS X包含整整200多个文件，而iOS包含500多个文件，大小约为200M。 

问题：共享库缓存到底是一个文件还是多个文件的组合？如果APP使用的第三方的动态库，是APP启动的时候操作系统才去加载，还是手机开机后自动加载这个共享库缓存目录下的文件？ 

31.OS X的dyld支持两级名称空间。这个特性是10.1引入的，指的是符号名称还包含其所在库的信息。这种方法更具有优势，因为允许两个不同的库导出相同的符号——而这在其他UNIX中会产生链接错误。有时候又需要禁用这种行为，通过将DYLD_FORCE_FLAT_NAMESPACE 环境变量设置为非零的值即可禁用。 

32.函数拦截是传统ld没有而dyld有的特性。DYLD_INTERPOSE宏定义允许一个库将其库函数实现替换为另一个函数的实现。——这也是追踪函数调用和修改函数实现的原理 

![](/images/2019/04/4.png) 

33.用户态的一个优点在于虚拟内存的隔离。进程独享一个私有的地址空间。 

34.和所有标准的C语言程序一样，OS X中的可执行文件也有一个标准的入口点，默认名称为”main“。不过除了三个标准的参数——argc、argv和envp——之外，Mach-O程序还接受第四个参数：名为”apple“的char**。在Snow Leopard系统之前，”apple“参数只包含一个字符串——程序的完整路径，即启动这个程序所用的execve()系统调用传入的第一个参数。dyld在进程加载的过程中使用了这个参数。从Lion系统开始，”apple“参数被扩展为一个完整的向量，其中包括两个新加入的参数。 

35.Cocoa应用程序的入口也是标准的C main()，不过常见做法是将main实现为NSApplicationMain()的包装，然后通过其进入Objective-C的编程模型。 

36.重写内存最常用的方法是采用缓冲区溢出（即利用未经保护的内存复制操作越过栈上数组的边界），将函数的返回地址重写为自己的指针。不仅如此，黑客还有更具创意的技术，例如破坏pringtf()格式化字符串以及基于堆的缓冲区溢出等。 

37.ASLR：进程每一次启动时，地址空间都会被简单的随机化——只是偏移，而不是搅乱。实现方法是通过内核将Mach-O的段平移某个随机系数。 

38.在64位模式下，由于内存空间巨大，所以也就可以遵循其他操作系统采用的模型了，即将内核的地址空间映射到每一个进程的地址空间中。这是64位地址空间和传统的OS X模型的不同之处，传统的OS X内存模型中内核有自己的地址空间，但是新的地址空间允许更快速的用户态/内核态切换（共享CR3寄存器，CR3寄存器是包含了页表指针的控制寄存器）。 

39.在内核层面，既没有用户堆也没有栈存在，所有的内存相关操作都要归约为页面。 

40.尽管栈在传统上一直是用来保存自动变量的，但是在某些情况下，程序员也可以选择使用栈来动态分配内存，方法是使用鲜为人知的alloca()。如果发生了栈溢出，alloca()会返回NULL，进程会收到SIGSEGV信号。 

41.iOS中VM压力释放机制依赖于Jetsam，Jetsam是一种类似于Linux的Out-Of-Memory killer的机制。 

42.OS X有一个来自于Mach的独特之处在于，交换空间不是直接在内核层次管理的，而是由一个专用的用户进程dynamic_pager()处理所有的交换请求。这个进程在引导时由launchd通过一个属性列表文件cpm.apple.dynamic_pager.plist启动。 

43.线程，作为最大化利用进程时间片的方法应运而生：通过使用多个线程，程序的执行可以分割为表面看上去并发执行的子任务。 

44.进程中线程的抢占开销比多任务系统对进程抢占的开销要小。因此从这个角度看，大部分操作系统开始将调度策略从进程转换到线程是有意义的。 

45.多处理器更是特别适合线程，因为多个处理器核心共享同样的cache和RAM——这为多线程之间的共享虚拟内存提供了基础。 

46.GCD自己维护了一个底层的线程库实现，以支持并发和异步的执行模型，减轻开发者处理并发问题的负担，以及减少类似于死锁之类的潜在错误。GCD的另一个优势是能够自动的随着逻辑处理器的个数而扩展。 ——YYDispatchQueuePool可以根据设备的物理核心数量来创建对应的线程数量。 



## 第五章  进程跟踪和调试



1.OS X中的调试工具首先要介绍的就是 DTrace。DTrace是一个重要的调试平台，移植自Sun的Solaris。DTrace中的D指的是D语言，这是一门完整的跟踪语言，通过这个语言可以创建专用的跟踪器，或称为探测器。D语言脚本被编译后由内核执行。

2.iOS中根本就没有提供DTrace。Linux上的ptrace提供了完整的进程跟踪和调试能力，因此称为了Linux下strace，gdb的基础。

3.DTrace神器的调试功能来源于能在内核中执行探测器的能力。DTrace的用户态部分由/usr/lib/dtrace.dylib 负责，Instruments和脚本解释器/usr/sbin/dtrace都使用了这个库。这是编译D脚本的运行时系统。然而，对于大部分有用的脚本来说，实际的执行都在内核态。DTrace通过一个特殊的字符设备(/dev/device)和内核组件进行通信。

![](/images/2019/04/5.png) 

4.sysctl机制在之前的章节中已经讨论过了，sysctl提供 一些显示进程统计数据的变量。sysctl获得进程ID列表的机制非常重要（事实上，ps和top指令都是通过这个机制获得进程列表的）。

5.除了DTrace和Instruments之外，在OS X中海油一些工具可以获得系统或进程状态的快照：system_profiler，sysdiagnose，allmemory，stackshot，stack_snapshot系统调用。

6.allmemory工具的作用是捕获用户进程的所有内存使用情况。运行时，这个工具遍历系统中的每一个进程，然后将其内存映射导出至/tmp/allmemoryfiles（可以通过-o指定其他文件）。获得了所有进程内存快照之后，allmemory会显示每一个进程的汇总统计数据，还会显示框架的内存使用。 

7.stack_snapshot这个系统调用可以捕获指定进程中所有线程的状态。 

8.sc_usage工具显示每一个进程的系统调用信息。fs_usage可以显示系统调用，但是显示的是与文件、套接字和目录相关的应用，这个工具可以显示系统范围内的跟踪（除非调用时提供了PID或命令参数）。 

9.latency工具显示中断和调度的延迟值。这个工具展示落在阈值内的上下文切换和中断处理程序计数，这两个阈值分别可以通过-st和-it参数设置。 

10.XNU包含一个称谓kdebug的内建内核跟踪设施。kdebug利用内核缓冲器来记录日志，而内核缓冲器的空间极为有限。 

11.在UNIX中，崩溃和一个信号有关。崩溃的真正原因来自于内核，内核发现进程无法继续执行时，生成这个信号作为最后的补救办法。 

12.当一个进程崩溃时，可以选择是否生成核心存储文件。iOS和OS X都没有选择创建巨大的核心转存文件，而是包含了一个CrashReporter，当进程异常终止时自动触发Crash Reporter，生成详细的崩溃日志。这个机制在进程消亡之前进行快速简单的分析，并且在崩溃日志中记录重要的内容。 

13.有没有可能在一个应用程序崩溃时自动运行另一个应用程序？在iOS和OS X中，将异常端口绑定至BSD进程底层的Mach任务的机制则能实现这一点。 

14.spindump和sample命令的采样方法都是类似的——挂起进程，记录栈跟踪（spindump使用之前描述的stack_snapshot系统调用），然后恢复进程。采样间隔通常大约为10毫秒，整个采样过程通常持续10秒，这两个值是可以配置的。 

15.应用程序崩溃的主要原因就是缓冲区溢出（既包含栈也包含堆）和堆内存的破坏。 

![](/images/2019/04/6.png) 

16.libgmalloc.dylib这个库可以截获并调试内存分配，这个强大的库的工作原理是截获LibSystem中的分配函数。 

17.任何读/写操作如果越过了缓冲区的尾部就会导致读写操作越过页边界，从而导致一个未处理的页错误，使得进程收到总线错误的信号（SIGBUS）而崩溃。 

18.标准的UNIX命令ps可以显示进程列表。UNIX的top命令是一个获得当前系统运行状况的关键工具，在OS X和iOS上都可用，而且相比标准版本有所修改。这些修改都和底层的Mach架构有关，使得top既能显示UNIX的信息（来自XNU的BSD层），也能显示Mach的信息。 

19.有时候需要查看某个进程在使用哪些文件，或某个文件正在被哪些进程使用。lsof()和fuser()这两个工具就分别能够实现以上两个功能。lsof()是对之前描述的fs_usage的补充，因为后者只能看到新打开文件的操作，而看不到已经打开的文件。lsof()能显示一个进程所有文件描述符（包括套接字）的映射。换句话说，fs_usage可以持续运行，lsof产生的是一个快照。 

20.fuser()提供的是反向的映射——从文件到拥有这个文件的进程。这个工具的主要作用是诊断文件锁定或者“文件被占用”的问题。 

21.尽管XNU在用户态提供了完整的POSIX API，展示了UNIX兼容的人格，但是底层的实现却主要依靠Mach的基本原语。 

![](/images/2019/04/7.png) 

22.随着苹果转向LLVM-gcc转换，还引入了LLDB作为GDB的替代品。LLDB的语法基本上和GDB类似，但是调试功能大有增强。 









## 第六章 引导过程 第七章 贯穿始终——launchd 

1.固件（firmware）可以看作是一种软件，这种软件因为被写入了芯片，所以是“固化”的。固件代码本身可以保存在只读存储器（ROM）中，也可以保存在电可擦除只读存储器EEPROM中 

2.BIOS是一个固定的程序，而且通常都是封闭的。EFI是一套接口。EFI更像是一个运行时环境，规范了一组应用程序编程接口，基于EFI的程序可以利用这些接口实现功能。 

3.苹果的EFI实现的另一个重要特性是Boot Camp。Boot Camp是苹果双重引导的解决方案，这个解决方案再Mac硬件上运行非苹果的操作系统（主要是Windows）。 

![](/images/2019/04/8.png) 

4.一旦苹果关闭了某个iOS版本的时间窗口，将安全服务器配置为拒绝这个版本的签名时，就不可能降级固件了。 

5.OS X和iOS中，用户环境始于launchd。launchd作为系统中的第一个用户态进程，负责直接或间接的启动系统中的其他进程。launchd仍然属于Darwin的范畴。 

6.launchd是由内核直接启动的。负责加载BSD子系统的主内核线程创建一个线程来执行bdsinit_task。这个线程获得PID 1，并且临时命令为“init”，这是一项来源于BSD的遗产。bsdinit_task然后调用load_init_program()，这个函数调用execve()系统调用（在内核空间中执行）执行守护程序。 

7.系统范围的launchd（PID 1）是不可能终止的。事实上，这个launchd是系统上唯一不朽的进程。当系统关闭的时候，launchd也是最后一个退出的进程。 

8.launchd的核心职责是根据预定的安排或实际的需要加载其他应用程序或作业。launchd区分两种类型的后台作业： 

①守护程序（daemon）和传统的UNIX概念一样，是后台服务，通常和用户没有交互。守护程序由系统自动启动，不考虑是否有用户登录进系统。 

②代码程序（agent）是一类特殊的守护程序，只有在用户登录的时候才启动。和守护程序的不同之处在于，代理程序可以和用户交互，有的代理程序还有GUI。 

③iOS不支持用户登录的概念，因此只有LaunchDaemon。 

④守护程序和代理程序都是通过自己的属性列表文件（.plist）声明的。 

9.launchd是用户态出现的第一个进程。当系统还在启动初期的时候，launchd是系统上唯一的进程（尽管这个状态很短暂）。这意味着系统启动和功能的方方面面都和launchd直接或间接相关。 

10.launchd的第一个，也是主要的职责就是init守护程序的职责。init的职责是派生出各种各样的后台守护程序，设置好系统，然后转入后台，确保这些守护程序都活着。如果有程序死亡了，launchd要负责重新派生出新的守护程序。 

11.传统意义上，init是用户态的第一个进程，从init fork出其他进程（转而可能fork出更多进程），init设置的资源限制会被所有后代继承。而launchd（新时代意义上的用户态第一个进程）还会设置Mach异常端口，内核内部通过异常端口处理异常情况并生成信号。 

12.UNIX传统上包含两个守护程序——atd和crond——来定时作业，即在指定时间运行指定的命令。第一个守护程序atd负责一次运行的作业，第二个守护程序crond提供了重复执行作业的支持。 

13.inetd/xinetd的用途是启动网络服务。这个守护程序的职责是绑定一些端口（UDP端口或TCP端口），当有连接请求到达的时候，根据需要启动相应的服务程序，并且将服务程序的输入输出描述符（stdin，stderr，stdout）连接到对应的套接字。 

14.Mach的IPC服务依赖于“端口”的概念（有点类似TCP和UDP端口的概念），端口是通信的端点。 

15.launchd还整合了iOS的Jetsam机制，Jetsam机制可以强制施行虚拟内存使用率的限制，这项特性在没有交换空间的iOS中特别重要。 

16.ReportCrash守护程序是默认的崩溃处理程序，截获所有应用程序的崩溃。通过设置作业的Mach异常端口，在发生崩溃的时候自动运行。 

17.iOS中amfid守护程序，可以阻止一切无签名无entitlement的代码在iOS中运行。 

18.apsd（ApplePushService.framework）守护程序则是苹果推送服务的守护程序，以mobile用户运行。 

19.atc守护程序则用于空中流量限制。crash_mover守护程序用于将崩溃日志移动到 `/var/Mobile/Library/Logs` 目录。 

20.mobile_obliterator守护程序则用于远程擦除设备。 

21.lockdownd就像是用户态的狱警，是所有越狱者的头号敌人。lockdownd由launchd启动，负责处理设备激活、备份、崩溃报告、设备同步以及其他服务。 

22.OS X下的图形shell环境是Finder，iOS使用的择时SpringBoard。 

23.和Finder不太一样，SpringBoard几乎全部靠自己完成所有的工作，在CoreServices目录下也只有少量几个可加载的bundle。 

lockbundle提供了锁屏时的功能。NowPlayingArtLockScreen.lockbundle负责提供当音乐播放器正在运行且屏幕锁定的时候的锁屏画面，PictureFramePlugin负责显示用户照片库中的图片。iPhone还有一个名为VoiceMemosLockScreen的bundle，负责显示语音信息和未接电话指示器。 

24.SpringBoard是一个多线程的应用程序，线程数远多于Finder。如果SpringBoard超过几分钟没有响应，那么看门狗会重启系统。 

25.SpringBoard注册的端口中，最重要的是PurpleSystemEventPort，这个端口通过GSEvent消息的方式处理UI事件。SpringBoard中的主线程调用GSEventRun()，GSEventRun()是一个处理UI消息的CFRunloop。其他线程都类似的运行循环，处理SpringBoard中的其他Mach端口。 

26.XPC是Lion和iOS5新引入的轻量级进程间通信原语。libxpc.dylib提供了各种各样的C语言层次的XPC原语。默认情况下XPC的消息是异步发送的，应答也是异步的，通过reply_sync函数可以阻塞知道收到应答消息。XPC是通过Mach消息机制实现的。 



#### 第二部分 内核 



## 第八章 内核架构 



1.内核既是一个操作系统，也是一个调度器，还是一个仲裁器，内核同时也提供安全服务。 

2.内核从架构上分为巨内核，微内核和混合内核。 

3.巨内核采取的方式是将所有的内核功能——不论是基础功能还是高级功能——全部放在一个地址空间中。在这种架构的内核中，线程调度和内存管理，以及文件系统、安全管理、甚至设备驱动全都在一起。 

所有的内核功能都实现在同一个地址空间中吗。为了进一步优化，巨内核不进将所有的工呢过都组织在同一个地址空间中们还将这个地址空间映射到每一个进程的内存中。 

在巨内核架构中，从用户态到内核态的切换非常搞笑，基本上就是一次线程切换的开销。这是因为内核的你内存页面映射在所有进程的地址空间中，也就是说，除了硬件强制的内核态和用户态之间的隔离外，两者之间其实没有任何分别。所有的进程，不论所有者或功能，都包含一份内核内存的拷贝，就好像包含共享库的拷贝一样。此外，这些拷贝（同样类似于共享库）都映射到了同一组物理页面，而且是常驻内存的物理页面。 

4.XNU的核心组件Mach是一个微内核系统。 

一个微内核值包含最核心的内核功能，代码量也最精简。内黑只负责完成最最关键的部分——通常包括任务调度和内存管理，其他的功能都交给外部服务程序（通常是用户态）完成。 

微内核有几个巨内核没有的有点：正确性，稳定性和健壮性，灵活性（移植）。 

尽管微内核架构有着种种优势，但是却有一个致命的缺点——性能。微内核的消息传递在底层需要通过内存复制操作以及数次上下文的切换来实现，而这些操作对计算速度的影响都不小。 

5.混合内核试图结合两种内核的好处。内核最核心部分支持底层服务，包括调度、进程间通信和虚拟内存，是自包含的，这一部分就像微内核一样。所有其他的服务都实现在这个核心之外，但是也在内核态中，而且和这个核心在同一个内存空间中。 

这种内核不强制要求消息传递。其他组件可以调用这个“内部核心”的服务，但是这个“内部核心”不能调用外部的组件。 

6.从技术上说，XNU是一个混合内核。Windows内核也被认为是一个混合型的内核，但是两者差别巨大。Windows更接近巨内核，所以死亡蓝屏概率高，而XNU更接近于微内核。 

7.XNU的Mach最早是一个真正的微内核，现在Mach的原语仍然是围绕着消息传递的基础构建的。然而，消息通常是以指针的形式传递的，因此没有昂贵的复制操作。这是因为大部分服务现在都在同一个地址空间中执行（因为也会被归为巨内核）。类似的，建立在Mach之上的BSD层一直都是一个巨内核，而且这个子系统也在同一个地址空间中。 

8.32位的OS X应用程序可以享用完整的没有内核预留的地址空间——内核有自己的地址空间。然而在64位的OS X中，苹果却顺从了，就像其他巨内核的系统一样，内核空间和用户空间是共享的。在iOS中也是如此。 

9.内核是一个受信任的系统组件。内核的功能和应用程序的功能之间需要有一种严格的分离，否则应用程序的崩溃会使整个系统本科鬼。这种分离需要由硬件强制支持，因为基于软件的强制实施不但会产生很大的开销，也不可靠。区分内核态和用户态非常重要，因此这个功能是由硬件提供的。 

10.用户态和内核态的切换有两种类型： 

①自愿转换，比如系统调用； 

②非自愿转换，当发生异常、中断或处理器陷阱的时候，代码的执行会被挂起，并且保留发生错误时候的完整状态。控制权被转交给预定义的内核态错误处理程序或中断服务程序。 

11.一共有三种类型的异常： 

①错误（fault）：指令遇到一个可以纠正的异常，并且处理器可以重新启动这条出现异常的指令。 

②陷阱（trap）：类似于错误，但是错误处理完成后返回发生陷阱指令之后的那条指令。 

③中止（abort）：不可重启指令。 

12.BSD系统调用，可以通过current_task获得当前BSD进程的数据结构。 







## 第九章 内核引导和内核崩溃 



1.苹果只开源了针对OS X编译的XNU版本，iOS的版本则是闭源的。 

2.和Linux内核类似，Linux可以针对特定架构编译，Mach也能。 

3.如果要在一堆源码文件中查找某个特定的函数名、变量名或其他符号，grep是一个不错的工具，grep可以接受任何正则表达式，并且在.h和.c文件中寻找匹配。 

4.vstart是i386/x64架构下的“官方”的内核初始化函数，标志着从汇编代码到C语言代码的转换。 

5.除了虚拟内存之外，kernel_bootstrap还初始化Mach的一些关键抽象： 

IPC——进程间通信是Mach构建的根基，IPC要求一些重要的资源，例如内存、同步对象和Mach接口生成器； 

时钟clock——通过时钟抽象实现闹铃； 

账本——账本是Mach系统的记账工具； 

任务task——任务是Mach的容器，类似BSD的进程； 

线程thread——线程是实际的执行单元。任务只不过是一个资源容器，真正被调度和执行的是线程。 

6.关于异常处理 

![](/images/2019/04/9.png) 





## 第十章 Mach原语：一切以消息为媒介 



1.XNU的核心是苹果从NeXTSTEP带来的Mach微内核。尽管Mach核心被BSD层包装起来了，而且主要的内核接口是标准的POSIX系统调用，但是这个Mach核心具有一组独特的API和原语。 

消息传递原语：讨论消息和端口，这是Mach IPC的基础。 

同步原语：锁和信号量是两种内核对象，这些对象用于确保并发执行的安全。 

2.Mach采用的是极简主义的概念。Mach和其他操作系统不同，其他操作系统提供了用户态进程实现所基于的完整模型，而Mach只提供了一个极简的模型，操作系统本身可以在这个模型的基础上实现。 

在Mach中，所有的东西都是通过自己的对象实现的。进程（在Mach中称为任务）、线程和虚拟内存都是对象，所有对象都有自己的属性。 

Mach的独特之处在于选择了通过消息传递的方式实现对象和对象之间的通信。Mach对象不能直接调用另一个对象，而是必须传递消息。源对象发送一条消息，然后这条消息被加入到目标对象的队列中等待处理。类似的，消息处理中可能会产生一个应答，这个应答通过另一条消息被发送回源对象。消息是以FIFO的方式可靠传输的（如果消息被发送出去，那么一定能被收到）。 

3.Mach的首要设计目标也是最重要的目标就是要将所有功能移出内核，并且放在用户态中，将内核保持在极简的状态。 

4.Mach的设计有一个非常强大的优点——在设计中考虑了多处理。从理论上说，Mach可以轻松扩展成计算机集群使用的操作系统。 

5.Mach中最基本的概念是消息，消息在两个端点或端口之间传递。任何两个端口之间都可以传递消息——不论是同一台机器上的端口还是远程主机的端口。Mach消息的设计考虑了参数串行化、对齐、填充和字节顺序的问题，这些问题都被消息实现隐藏了。 

6.Mach消息的发送和接收都是通过同一个API函数mach_msg()进行的，这个函数在用户态和内核态都有实现。 

7.Mach消息原本是为真正的微内核架构而设计的，也就是说，mach_msg()函数必须在发送者和接受者之间复制消息所在的内存。但是XNU通过单一内核的方式来“作弊”：所有的内核组件都共享同一个地址空间，因此消息传递时只需要传递消息的指针就可以了，从而省去了昂贵的内存复制操作。 

8.为了实现消息的发送和接收，mach_msg()函数调用了一个Mach陷阱（trap）。Mach陷阱就是Mach中跟系统调用等同的概念，在用户态调用mach_msg_trap()会引发陷阱机制，切换到内核态，在内核态中，内核实现的mach_msg()会完成实际的工作。 

9.消息在端口之间传递。消息从某个端口发送到另一个端口。每个端口都可以接收来自任意发送者的消息，但是只能有一个指定接收者。向一个端口发送消息时实际上是将消息放在一个队列中，直到消息能被接收者处理。 

10.所有的Mach原生对象都是通过对对应的端口访问的。 

11.端口和权限也可以从一个实体传递到另一个实体。实际上，通过复杂消息将端口从一个任务传递到另一个任务并不罕见。这是IPC设计中的一个非常强大的特性，有一点类似于主流UNIX的domain socket，允许在进程之间传递文件描述符。 

12.Mach的消息传递模型是远程过程调用（RPC）的一种实现。 

13.IPC所需要的基本原语：消息、发送和接收消息的端口，以及确保安全并发的信号量和锁。 

14.每一个Mach任务（Mach任务是一个对应于进程的高层次抽象）包含一个指针指向自己的IPC名称空间，在名称空间中保存了自己的端口。此外，任务也可以获得系统范围内的端口。 

15.同步机制的根本是排他访问的能力：当别人在使用一个资源时，排除其他人对这个资源的访问。因此最基本的同步原语是互斥对象，也称为互斥体。互斥体只不过是内核内存中的普通变量，通常是机器字大小的证书，但是有一个特殊要求——硬件必须对这些变量进行原子操作。“原子”的意思就是说，对互斥体的操作决不允许被打断——即使是硬件中断也不能打断。在SMP系统上，对物理互斥还有一个要求，就是要求硬件实现某种内存屏障。 

16.Mach的锁依赖两个层次组合而成：硬件相关层——依赖于硬件的特殊性质，并且通过特定的汇编指令实现原子性和互斥性；硬件无关层——通过统一的API包装硬件特定的调用，这些API使得Mach之上的层完全不用关心实现的细节。 

17.互斥体有一个最大的缺点，就是一次只能有一个线程持有锁。读写锁就是这个问题的解决方案，读写锁能够区分读访问和写访问，多个读者可以同时持有锁，而一次只能有一个写者。当一个写者持有锁时，所有其他线程都被阻塞。 ——pthread_rwlock_t就是POSIX层的API在iOS下的读写锁，区分pthread_rwlock_rdlock读和pthread_rwlock_wrlock写的加锁，解锁统一用pthread_rwlock_unlock。

18.阻塞一个线程意味着放弃线程的时间片，把处理器让给调度器认为下一个要执行的线程。当锁可用时，调度器会得到通知，然后根据自己的判断将线程从等待队列中取出并重新调度。 

19.Mach提供了信号量，信号量是泛化的互斥体。互斥体的值只能是0和1，而信号量的取值可以达到某个正数，即允许并发持有信号量的持有者的个数。信号量可以在用户态使用，而互斥体只能在内核态使用。 

20.在XNU上，POSIX信号量的底层实现是通过Mach信号量实现的。 

21.信号量可以转换为端口，也可以由端口转换而来。 

22.锁集就是锁的数组，通过给定的锁ID可以访问锁。锁也可以传递给其他线程。交出一个锁会阻塞交出锁的线程，并唤醒接受锁的线程。 

23.锁集的有趣之处在于允许锁的传递。锁的传递指的是将锁从一个任务传递给另一个任务的过程。Mach在调度中也使用了传递的概念，允许一个线程放弃处理器但是指定哪一个线程接替运行。 

24.Mach提供了一组异常丰富的API调用用于查询机器信息，所有这些调用都要求获得主机端口才能工作。 

主机API最重要的一个方面就是能提供其他方式几乎无法获得的信息。Mach API是获得内核模块信息，内存映射表信息以及其他POSIX（BSD层）无法获得的信息的最直接方法。 

25.所有的用户都可以通过mach_host_self()获得主机端口，但是只有特权用户才能通过调用host_get_host_priv_port()获得特权端口。 

26.host_set_exception_ports()获得/设置或交换主机层次的异常处理程序。 

27.一个或多个processor_t对象可以分组为处理器集，或称为pset。pset中的处理器通过两个队列进行维护：一个是active_queue，保存当前正在执行线程的处理器；另一个是idle_queue，用于保存当前空闲的处理器。 







## 第十一章  Mach调度 



1.和所有现代的操作系统一样，内核调度的对象是线程，而不是进程。Mach使用了比进程更轻量级的概念：任务task。 

2.线程定义了Mach中最小的执行单元。一个或多个线程包含在一个任务中。 

3.Mach将任务定义为线程的容器，因此资源是在任务这个层次处理的。线程只能（通过端口）访问包含这个线程的任务中分配的资源和内存。 

4.任务task是一种容器对象，虚拟内存空间和其他资源都是通过这个容器对象管理的。这些资源包括设备和其他句柄。资源进一步被抽象为端口。因此资源的共享实际上相当于允许对对应端口进行访问。 

5.严格的说，Mach的任务并不是其他操作系统中所谓的进程，因为Mach作为一个微内核的操作系统，并没有提供“进程”的逻辑，而只提供了最基本的实现。不过在BSD的模型中，这两个概念有1：1的简单映射，每一个BSD进程（也就是OS X进程）都在底层关联了一个Mach任务对象。实现这种映射的防范是指定一个透明的指针bsd_info，Mach对bsd_info完全无知。 

6.任务是没有生命的，任务存在的目的就是要成为一个或多个线程的容器。大部分针对任务的操作实际上就是遍历给定任务中的所有线程，并对这些线程进行对应的线程操作。 

7.在任何时刻，内核都必须能够蝴蝶当前任务和当前线程的句柄。内核分别通过current_task()和current_thread()函数完成这两个任务。 

8.thread_suspend/thread_resume表示挂起/恢复线程，会递增/递减挂起计数器。线程只有在其suspend计数器和所在的任务的suspend计数器都为0时才能执行。 

9.当调用pthread_create()时，底层会转而调用Mach的API调用thread_create()，并且使用mach_task_self()作为第一个参数。 

10.由于每一个处理器核心在同一时刻只能运行一个线程，所以内核必须具有抢占一个线程的执行，将处理器让给另一个线程的能力，从而实现上下文切换。 

11.由于Mach具有处理器集的抽象，所以从某种角度说，Mach比Linux和Windows更擅长管理多核处理器：Mach可以将同一个CPU的多个核心放在同一个pset中管理，并且通过不同的pset管理不同的CPU。 

12.上下文切换是暂停某个线程的执行，并且将其寄存器状态记录在某个预定义的内存位置中。当一个线程被抢占时，CPU寄存器中会加载另一个线程保存的线程状态，从而恢复那个线程的执行。 

13.一个线程在CPU上可以执行任意长时间。执行指的是这样一个事实：CPU寄存器中填满了线程的状态，因此CPU执行该线程函数的代码。这个执行过程一直持续，直到①线程终止；②线程自愿放弃CPU；③外部中断打断了线程执行（时间片用完或更高优先级的线程被唤醒）。 

14.每一个操作系统都提供了一个优先级的范围：Windows有32个优先级，Linux有140个优先级，而Mach有128个优先级。 

内核线程的最低优先级为80，比用户态线程的优先级要高，可以保证内核以及系统维护管理的线程能够抢占用户态的线程。 

通过 ps -l 命令可以查看优先级 

15.Mach会针对每一个线程的CPU利用率和整体系统负载动态吊证每一个线程的优先级。因此线程会在自己的优先级范围中“漂移”，如果耗CPU太多则降低优先级，如果不能得到足够的CPU资源则提升优先级。 

16.在使用多核、SMP或超线程的现代架构中，还可以设置某个线程和一个或多个指定CPU的亲缘性。这种亲缘性对于线程和系统来说都是有好处的，因为当线程回到同一个CPU上执行时，线程的数据可能还留在CPU的缓存中，从而提升性能。用Mach的说法，线程对CPU的亲缘性的意思就是绑定。 

17.Mach含有的特殊特性： 

①控制权转交（handoff）允许一个线程主动放弃CPU，但不是将CPU放弃给任何其他线程，而是降CPU转交给自己选择的某个特定的线程。 

②使用续体可以使线程不用管理自己的栈，线程可以丢弃自己的栈，系统恢复线程执行时不需要恢复线程的栈。 

③异步软件陷阱AST是软件对底层硬件陷阱机制的补充完善。通过使用AST，内核可以响应需要得到关注的带外（out-of-band）事件，例如调度事件。 

18.Mach对yield做了改进，允许选择将CPU转交给谁。控制权转交并不是对调度器的强制要求，调度器还可以选择将控制权转交给其他线程（例如，如果指定的线程处于不可运行的状态）。作为控制权转交的结果，当前线程剩下的时间片也会被转交给新调度的线程。 

19.如果线程要进行控制权转交而不是简单的yield操作，那么需要调用thread_switch()，Mach将thread_switch()导出为一个陷阱，因此也可以从用户态调用这个函数。 

20.上下文切换采用的是每一个线程都有自己独有栈的模型，当线程自愿请求一次上下文切换时可以选择指定一个续体。如果指定了续体，那么当线程恢复执行时，系统会以续体作为入口点重新加载线程，并创建新的栈，之前的状态都不会得到保留。这样可以明显加快上下文切换的速度，因为不需要保存和加载寄存器（此外还能显著地节省内核栈的空间，内核栈本身很小，只有4个页面，即16kb）。续体中的线程只需要4-5kb的空间来保存线程状态，将16kb中的其他空间节省下来用作其他用途。使用续体不需要保存完整的寄存器状态和线程栈，只需要保存续体以及参数。 

21.续体是缓解上下文切换开销的简单有效的机制，主要由Mach的内核线程使用。内核线程特别喜欢使用续体，Mach的内核线程是通过续体启动的。 

22.Mach支持两种不同模式的抢占——显示抢占和隐式抢占，续体模式只能用于显示抢占。 

23.系统中的线程可能会被两种方式抢占：一种是显示的抢占，即线程放弃CPU的控制权或进入阻塞的操作；另一种是隐式的抢占，这种抢占是由中断引起的。显示抢占有时候也被认为是同步的，因为这种抢占是事先可以预知的。而由于中断不可预测的本质，所以隐式的抢占是异步的。 

24.发生显示抢占的原因可能是等待某个资源，等待某个IO，或睡眠一定的时间。当用户态的线程调用阻塞的系统调用（例如read(),select()和sleep()）时会发生显示抢占。 

25.thread_invoke()函数负责执行上下文切换并负责处理续体。 

26.显示抢占本身是有局限性的，将放弃CPU的选择权交给运行的线程是极为不可靠的。线程会陷入费时的处理操作中，甚至会进入死循环。 

27.Mac OS X是一个抢占式的多任务系统。简单的说，Mach具有随时抢占一个线程的权利，不论这个线程是否准备好了抢占。和显示的抢占不同，这种隐式的抢占对于线程来说是不可见的。线程可以对这种抢占完全不知情，线程的状态会被透明的保存并恢复。大部分线程不会受到太大的影响，因为大部分线程都是IO密集型的。但是对于CPU密集型的线程来说，这种抢占可能会造成一些问题，特别是要求时间关键的性能时（例如视频和音频解码都是这种类型的任务）。 

28.Mach是一个分时系统，而不是一个实时系统。 

29.THREAD_AFFINITY_POLICY策略定义了线程的L2缓存亲缘性，这些线程共享同一块缓存。这意味着这些线程很可能运行在同一个CPU上，不论这个CPU有多少核心（毕竟同一个CPU上所有核心都共享同一块L2缓存）。 

30.异步软件陷阱AST就是为了支持隐士抢占的。AST是人工引发的非硬件触发的陷阱。AST是内核操作的关键部分，而且是调度时间的底层机制，也是BSD信号的实现基础。 

31.ast_taken函数（内核陷阱中和内核线程终止时也可以调用）负责处理除了内核idle线程之外的所有线程的AST。否则，这个函数会检查AST_BSD，这原本是对Mach的一个临时修改，使其能够处理BSD事件（例如信号），但是被永久的保留了。如果设置了AST_BSD，则调用bsd_ast处理信号。 

32.Mach的线程调度算法高度可扩展，而且允许更换用于线程调度的算法。不过通常情况下，只启用了一个调度器，即传统调度器。调度器大量使用了AST机制。 

33.对于要提供抢占式多任务的系统来说，必须有某种机制允许调度器能够首先得到CPU的控制权，从而抢占当前正在执行的线程，然后才能执行调度算法，并且通过调度算法决定当前的线程可以继续恢复执行还是要抢夺其CPU给更重要的线程使用。 

34.为了能够从当前运行的线程抢夺CPU，现在的操作系统都利用了现有的硬件中断机制。由于中断的特点是强迫CPU在发生中断时“放下手中所有的任务”，并longjmp跳转到中断处理程序（也称为中断服务例程ISR）执行，因此可以通过中断机制在发生中断时运行调度器。但是问题是，中断是异步的。 

35.内核可以配置时钟使其在给定数目的周期之后产生一个中断。这个中断源通常称为定时器中断。 

36.解决方案是采用另一种不同的模型：无tick内核。在这种模型中，每一次定时中断发生时，定时器都会被重新设置为调度器认为需要下一次中断的时刻。这意味着在每一次定时器中断时，中断处理程序都要（非常快）扫描还没超期的截止时间线的列表。相比大量不必要的中断，每一次定时器中断中多做的这些处理工作还是值得的，而且通过只跟踪那些最紧急的截止时间线可以将这些处理工作的开销降到最低。 

37.在添加非严格的定时器事件是会加上一个所谓的“宽限slop”值，通过宽限值可以合并一些定时器事件，从而增加这些定时器事件同时超时的概率（从而减少了定时器中断的总数）。 

38.Mach只提供了一个异常处理机制用于处理所有类型的异常——包括用户定义的异常、平台无关的异常、以及平台特定的异常。 

39.在Mach中，异常是通过内核中的基础设施——消息传递机制——处理的。异常由出错的线程或任务（通过msg_send()）抛出，然后由一个处理程序（通过msg_recv()）捕捉。处理程序可以处理异常，也可以清除异常，可以决定终止线程。 

40.Mach的异常处理程序在不同的上下文中运行异常处理程序，出错的线程向预先指定好的异常端口发送消息，然后等待应答。每一个任务都可以注册一个异常端口，这个异常端口会对同一个任务中的所有线程起效。单个的线程还可以通过 thread_set_exception_prots 注册自己的异常端口。通常情况下，任务和线程的异常端口都是NULL，也就是说异常不会被处理。 

41.发生异常时，首先尝试将异常抛给线程的异常端口，然后尝试抛给任务的异常端口，最后再抛给主机的异常端口（即主机注册的默认端口）。如果没有一个端口返回 KERN_SUCCESS，那么整个任务被终止。Mach不提供异常处理逻辑——只提供传递异常通知的框架。 

42.exception_triage()负责主要的异常处理逻辑，这个逻辑在两种架构上的Mach消息层面都是一样的。这个函数尝试根据前文描述的方式——线程、任务、最终到达主机——利用exception_deliver()投递异常。 

43.每一个线程或任务对象，以及主机本身，都有一个异常端口数组，这个数组中的端口通常初始化为IP_NULL。通过xxx_set_exception_ports()调用可以设置这些异常端口，其中的xxx为thread、task或host。 

44.[mach]_exception_raise 用于EXCEPTION_DEFAULT，[mach]_exception_state_raise 用于EXCEPTION_STATE。[PLCrashReporter里有相关的使用] 这些函数最终通过调用ux_exception将异常转换为响应的UNIX信号，并且通过threadsignal将信号投递到出错的线程。 

45.OS X的最重要的特性之一崩溃报告器（crash reporter）就是利用异常端口的机制实现的。launchd注册了异常端口，然后将所有子进程都应用同样的异常端口，因为异常端口是随着进程fork集成的。launchd将ReportCrash设置为MachExceptionHandler。通过这种方式，当一个launchd作业发生异常时，崩溃报告器会自动根据需要启动。调试器也可以利用异常端口捕捉异常并且在发生错误时中断。 

46.Mach异常处理会先于UNIX异常处理发生。 

47.使用mach_msg在异常端口上创建一个活动监视者。异常处理可以由同一个程序中的另一个线程来完成，不过更有意思的做法是在另一个程序中实现异常处理的部分。 

48.Mach是XNU的微内核核心。XNU暴露给用户的主要接口：BSD层。BSD层使用了Mach作为底层的原语和抽象，向应用程序暴露出流行的POSIX API，使得OS X能够和很多其他的UNIX实现兼容。 





常见的架构无关的Mach异常 

EXC_BAD_ACCESS 内存访问异常，代码包含发生内存访问异常的地址。 

EXC_BAD_INSTRUCTION  指令异常，非法或未定义的指令或操作数。 

EXC_BREAKPOINT  和跟踪、断点相关的异常 

EXC_SYSCALL  系统调用 

EXC_CRASH  异常的进程退出 









## 第十二章  虚拟内存 



1.Mach和所有内核一样，代码中有很大一部分都在负责高效的管理虚拟内存。 

2.vm_map：表示任务地址空间内的一个或多个虚拟内存区域。每一个区域都由一个独立的条目vm_map_entry表示，这些条目由一个双向链表vm_map_links维护。 

vm_map_entry: 每一个vm_map_entry 都表示了虚拟内存中一块连续的区域。每一个这样的区域都可以通过指定的访问保护权限进行保护（和虚拟内存页面采用同样的权限，即r/w/x权限）。vm_map_entry 通常指向一个vm_object，但是也可以指向一个嵌套的vm_map，即子映射。 

vm_object: 用于将vm_map_entry 和实际支撑的内存关联起来。这个数据结构包含一个 vm_page 的链表。 

vm_page：vm_page真正表示了vm_object或部分vm_object。vm_page可以有多种状态：驻留内存、交换出、加密、干净和脏等。 

3.每一个Mach任务都有一个自己的虚拟内存空间，任务的struct task 中的map字段保存的就是这个虚拟内存空间。 

4.purgeable的对象在内存低的情况下可能会丢失，即直接释放，而不是提交到后备存储。 

5.Mach的API比POSIX提供的等同API要强大得多，主要是因为Mach API允许一个任务入侵到另一个任务的地址空间。为了访问其他任务的地址空间，要求有相应的权限（具体来说就是任务的端口）。除了这点要求之外，这些调用几乎是无所不能的。事实上，在OS X中很多进程入侵和线程注入技术都依赖这些Mach调用，而不是依赖BSD提供的调用。 

6.vmmap()的例子可以很容易扩展得更具有入侵性，比如可以将进程内存映射导出到磁盘，甚至可以写入内存映射。 

7.pmap可以嵌套（即包含其他pmap）。这是一个非常常见的技术，共享内存严重依赖这项技术——包括隐式的共享内存（共享库）和显式的共享内存（mmap()）。 

8.Mach Zone的概念相当于Linux的内存缓存和Windows的Pool。Zone是一种内存区域，用于快速分配和释放频繁使用的固定大小的对象。Zone的API是内核内部使用的，在用户态不可使用。内核中的Zone和malloc的zone完全不同，后者是C运行时库的一部分，在用户态使用，而且具有很好的文档。 

BSD内核zone 直接构建与Mach的zone之上。 

9.如果系统内存不足，zone可能会进行垃圾回收。垃圾回收是一个两趟的过程，系统首先扫描所有的zone（跳过标记为不可回收的zone），检查这些zone的空闲列表，判断哪些对象是可以回收的。在第二趟中，将这些对象转换为页面：和非空闲对象共享了一个页面的对象不能被释放，只有页面全部空闲的对象才能被释放。最后，当判定好了可以释放的页面之后，通过kmem_free()释放。 

10.所有的内核分配（除了连续物理内存的分配）的路径最终都会到达一个函数，那就是kernel_memory_allocate()。 

11.进程的内存需求早晚会超过可用的RAM，系统必须有一种方法能将不活动的页面备份起来并且从RAM中删除，腾出更多的RAM给活动的页面使用，至少暂时能够满足活动页面的需求。在其他的操作系统中，这个工作是由专门的内核线程完成的。例如，Linux中的pdflush和wswapd内核线程。在Mach中，这些专用的任务成为分页器，分页器可以是内核线程，甚至可以是外部的用户态服务程序。 

这里提到的分页器及基金实现了各自负责的内存对象的分页操作。这些分页器不会控制系统的分页侧路。分页策略是由vm_pageour守护线程负责的，而vm_pageout守护线程是kernel_bootstrap_thread()完成所有任务之后最后变成的。 

12.Mach通过Universal Page List（统一页列表）这个数据结构来维护页的信息，这个列表和分页器的具体实现无关。UPL是连接虚拟地址和实际的物理页面的纽带，有一点类似于Windows的Memory Descriptor List和IOKit的IOMemoryDescriptor。 

13.Vnode分页器负责支持文件的内存映射。当内存映射了文件，这些文件的内容需要从文件系统中读取。当内存映射的文件在内存中“脏”了，那么这些脏的页面需要写回文件系统。解密的页面永远不会被标记为脏，因此永远都不会被换出到磁盘上（如果可以从交换文件中提取到明文，那么整个加密就没有意义了）。 

14.通过DYLD_INSERT_LIBRARIES强制注入一个库，然后直接从任务中读取内存。这也是为什么尽管App Store的二进制文件被加密，但是iOS应用程序的破解依然很繁荣的原因。 

15.pageout守护程序其实不是一个真正的守护程序，而是一个线程。vm_pageout永不返回。初始化之后会派生出两个线程：外部的iothread，和一个垃圾回收线程（其实还有第三个线程，内部的iothread，是默认分页器注册时创建的）。设置完成后，vm_pageout()最终调用vm_pageout_continue()，这个函数周期性的唤醒并执行vm_pageout_scan()。 

16.BSD层的Jetsam机制类似于Linux的Low Memory Killer。 

17.在iOS中，物理内存非常紧缺而且没有交换空间，这个宏调用了vm_check_memorystatus()，而后者负责唤醒kernel_memorystatus线程，这属于Jetsam机制的一部分。 

18.vm_fault()函数调用vm_page_fault()处理实际发生错误（缺页中断）的页面，并且从后备存储中将这个页面取回。实现方法是：查找vm_page对应的vm_object，然后从中获得分页器的端口，分页器的data_request函数负责从后备存储中读入要换入的页面。如果需要的话，换入操作还会对页面进行解密（如果页面在加密的交换文件中），并且验证代码签名。 

19.非法访问：访问一个没有映射到进程地址空间（即任务的vm_map）的地址。解引用一个野指针时通常会发生这种错误。发生这种错误时进程会收到SIGSEGV信号。 

20.页面保护错误：访问一个映射的地址，但是页面的保护掩码拒绝请求的访问。通常引发这个错误的原因包括跳转到数据段或试图写入（或读取）一个不允许写入（或不允许读取）的页面。发生这种错误时进程会收到SIGBUS信号。 

21.dynamic_pager()是一个用户态的守护程序，负责维护系统交换文件，默认情况下交换文件在/private/var/vm/swapfile目录下。内核的default_pager分页器需要在用户态的干预下调整或修改交换条件时，从内核态调用这个守护程序。 





## 第十三章 BSD层 



1.Mach只是一个微内核。尽管Mach的部分应用程序编程接口（API）也暴露给了用户态，但是开发者主要使用的还是更为流行的POSIX API，而这一套API是通过Mach之上的BSD层实现的。 

2.ptrace()属于进程控制相关的调用。 

3.在Mach提供的这些原语之上还需要建立一个层次提供像文件、设备、用户和组等重要从抽象。Mach最早选择的这个层次就是BSD，而且延续在XNU中了。 

4.BSD采用了两个原语，并且组织成立UNIX世界上著名的进程和线程的概念。 

5.BSD的进程可以唯一的映射到Mach任务，但是包含的信息比Mach任务提供的基本调度和统计信息要丰富。其中最值得注意的是，BSD进程包含了文件描述符和信号处理程序的数据。进程还支持复杂的谱系，将进程和其父进程、兄弟进程和子进程连接起来。 

6.进程就是容器，二进制代码的实际执行单元是线程。 

7.用户态的线程始于对pthread_create的调用。这个函数做的工作并不多，因为主要工作是由bsdthread_create()系统调用完成的，bsdthread_create()只不过是对Mach线程创建的复杂包装。真正的线程创建是由底层的Mach层完成的。bsdthread_create()负责的工作是设置线程栈（如果指定了自定义栈），设置（机器相关的）线程状态，以及设置自定义调度参数（如果提供了的话）等。 

8.在UNIX中，进程不能被创建出来，只能通过fork()系统调用复制出来。如果fork()操作失败，fork()只会在调用的进程中返回-1。 

9.子进程是父进程的完整复制，除了一下几个重要的例外： 

①文件描述符，尽管数值和指向的文件都是一样，但只是原始描述符的副本。这意味着后续对这些描述符修改的调用（例如lseek()和close()）只会影响创建这些描述符的进程。 

②资源限制，子进程会继承资源限制，但是资源利用率都设置为0。 

③子进程的内存映像看上去是子进程私有的，但是事实上子进程和父进程使用的是内存中相通的物理页面。虚拟内存的私有性是通过设置页面的写时复制标志位来保证的，因此不论是哪个进程试图写入页面时都会引发页错误，从而创建页面的副本，并且重新建立映射。 

10.vfork()的进程没有对应的Mach任务和线程。只有在接下来调用了execve()之后才会创建任务和线程。事实上，除了有一个execve()跟在后面之外，vfork()没有存在的意义，因为这个系统调用最初的设计就是为了这个目的。子进程的 task_t 和 thread_t （可以分别通过mach_task_self()和mach_thread_self()获得）完全就是父进程的 task_t 和 thread_t ，vm_map也是如此，只有以后调用 execve()载入一个Mach-O镜像才能最终真正创建一个Mach任务和进程。 

11.如果将一个进程比作是一个人体，那么在进程中执行的二进制程序就是这个人体的大脑。只是通过fork()新创建出来的进程也没有多大作用，除非执行镜像通过exec()替换为另一个可执行程序。因此，进程创建的核心在于二进制文件的加载和执行。 

12.execsw镜像加载器。进程执行镜像的流程太长了，这里不做记录，还是直接看书吧， 

13.所有的镜像加载的路径要么终止在一个错误上，要么最终完成加载Mach镜像。 

14.XNU中的Mach-O加载逻辑基本上和NeXT在1988年发明这个格式时差不多。经过这么多年苹果对这个过程做了一些修改，其中主要针对代码解密部分进行修改，但是Mach-O文件格式基础基本没什么变化。 

苹果将这个修改封装在 exec_mach_imgact()中了，这是Mach二进制文件注册的处理程序。这个函数首先读取Mach文件头，然后解析其架构（32位或64位）和标志位。这个函数拒绝接受Dylib和Bundle文件——这些文件是由用户态的dyld动态链接器负责的。然后再应用posix_spawn()中的参数（如果有的话）。之后，对二进制进行评估以确保满足当前架构的需求。 

处理Mach-O加载的主要函数是load_machfile()。load_machfile()函数负责设置内存映射，这个映射最终会加载各种 LC_SEGMENT 命令加载的数据。 

Load_machfile()的核心在于parse_machfile。这个函数负责实际解析加载命令的繁杂工作。 

其中，load_code_signature 也是load_machfile()函数里众多流程中的一个步骤，也就是验证代码签名。 

经过三趟扫描后，在dlp变量中有一个保存的动态链接器命令，将动态链接器加载到新的映射中，可能要根据ASLR偏移进行调整。load_dylinker()函数会递归的调用parse_machfile()。 

如果load_machfile()成功返回了，exec_mach_imgact会继续完成后续的工作。具体操作如下： 

①通过调用vm_map_set_user_write_limit设置ulimit-m； 

②设置代码签名的标志： 

​    CS_HARD：拒绝加载无效页； 

​    CS_KILL：如果有任何无效页则杀掉进程； 

​    CS_EXEC_*：和上面两个标志位一样，只不过来自execve()。 

③设置新的进程名称等等。 

需要注意的是，这里并没有强制任何事情：真正的代码签名实施是在Mach的VM页错误处理程序中，通过调用 cs_invalid_page 来强制实施策略。 

15.Mach提供了丰富的跟踪机制，其中最重要的就是DTrace。另一个机制ptrace()，这个机制在OS X和iOS（故意的）上只有部分功能有效。 

16.BSD和其他UNIX系统提供了一个名为ptrace()的一站式系统调用，这个调用支持进程跟踪和调试。这个系统调用对于调试和逆向工程来说非常有用，例如在Linux中，gdb，系统调用跟踪（strace）和库函数调用跟踪（ltrace）就是用了这个系统调用。 

17.在Linux中，ptrace的真正实力在于能够读写其他进程的内存，而XNU的ptrace实现则忽略了这些选项。不过Mach的API也能实现类似的功能。 

18.挂起一个进程相当于停止一个进程的执行无限长时间，知道这个进程被恢复。冷冻和解冻的决定权通常都在iOS的加载器SpringBoard手中。 

19.Mach已经通过异常机制提供了底层的陷阱处理，而BSD则在异常机制之上构建了信号处理机制。硬件产生的信号被Mach层捕捉，然后转换为对应的UNIX信号。为了维护一个统一的机制，操作系统和用户产生的信号首先被转换为Mach异常，然后再转换为信号。 

20.当一个BSD进程（也是用户态进程）被bsdinit_task()函数启动时，这个函数还调用了 ux_handler_init()函数，这个函数设置了一个名为 ux_handler 的Mach内核线程。只有在 ux_handler_init()函数返回之后，bsdinit_task() 才能够注册使用 ux_exception_port。 

通过调用 host_set_exception_ports()函数，bsdinit_task() 将所有的Mach异常消息都重定向到 ux_exception_port，这个端口由 ux_handler() 线程持有。由于所有后创建的用户态进程都是PID 1的后台，所以这些进程都会自动继承这个异常端口，相当于 ux_handler() 线程要负责处理系统上 UNIX 进程产生的每一个Mach异常。 

ux_handler()函数非常简单，这个函数在进入时首先设置好 ux_handler_port，然后进入一个无限的Mach消息循环。消息循环接收Mach异常消息，然后调用mach_exc_server()处理异常。 

mach_exc_server会调用mach_exception_raise()，然后会被mach_catch_exception_raise()捕获，信号处理逻辑就在这里。 

21.硬件产生的信号始于处理器陷阱。处理器陷阱是平台相关的。ux_exception负责将陷阱转换为信号。 

如果信号不是由硬件产生的，那么这个信号来源于两个API调用：kill()或pthread_kill()。这两个函数分别向进程和线程发送信号。 

![](/images/2019/04/10.png) 



## 第十四章 BSD的高级功能 



1.虚拟内存管理是在Mach层进行的，Mach控制了分页器，并且向用户态导出了各种vm_和mach_vm_消息接口。而用户态的开发者大部分都只知道标准的POSIX调用，因此需要对这些Mach调用进行封装。类似的，BSD层也使用了自己的内存管理函数。 

2.OS X和iOS实现了一个低内存情形的处理机制，成为Jetsam，或者称为Memorystatus。这个机制有点类似于Linux的“out-of-memory”杀手，最初的用途就是杀掉消耗太多内存的进程。Jetsam的名字来源于杀掉消耗内存最多的进程并且抛弃这些进程占用的内存页面的过程。 

3.Memorystaus维护了两个列表：一个是快照列表，这个列表保存了系统中所有进程的状态以及消耗的内存页面数；还有一个优先级列表，保存了要杀掉的备选进程。 

4.用户态也可以通过pid_suspend()和pid_resume()控制进程的休眠。 

5.ASLR：Address Space Layout Randomization 内核地址空间布局随机化。 

ASLR对内核代码的影响非常小：代码不再使用固定地址，而是转变为使用相对地址，相对地址是针对程序的当前位置确定的。 

6.工作队列是OS X中开发的一项机制，作用是为应用程序提供多线程并扩展到多处理器支持。工作队列是GCD的基础机制。 

7.overcommit位表示这个队列可以创建新的线程。通常情况下不建议使用这个策略，因为线程数多于CPU数会降低程序的运行速度。 

GCD仅通过dispatch_get_global_queue调用可以接受的一个标志（DISPATCH_QUEUE_OVERCOMMIT）来支持overcommit，但是苹果的文档掩盖了这个事实，宣称这个标志位必须为0。 

GCD和libdispatch在工作队列不存在或被禁用时也能工作，在这种情况下，GCD和libdispatch会退而使用线程池模型。 

补充： 

_dispatch_get_root_queue 会获取一个全局队列，它有两个参数，分别表示优先级和是否支持 overcommit。一共有四个优先级，LOW、DEFAULT、HIGH 和 BACKGROUND，因此共有 8 个全局队列。带有 overcommit 的队列表示每当有任务提交时，系统都会新开一个线程处理，这样就不会造成某个线程过载(overcommit)。——参考自<https://bestswifter.com/deep-gcd/> 



阅读过 GCD 源码的同学会知道，所有默认创建的 GCD queue 都有一个优先级，但其实每个优先级对应两个 queue，比如一个是 default-priority， 那么另一个就是 default-priority-overcommit。dispatch_async 的时候，会首先将任务丢进 default-priority 队列，如果队列满了，就转而丢进 default-priority-overcommit。 

在 Mac 系统里，GCD 允许 overcommit，意味着每次 dispatch_async 都会创建一个新线程，即使 over commit 了，这些过量的线程会根据优先级来竞争 CPU 资源。 

而在 iOS 系统里，GCD 会控制 overcommit，如果某个优先级队列 over commit 里，那么排在后面的任务就会处于等待状态。移动设备 CPU 资源比较紧张，这种设计合乎常理。 

所以如果在 iOS 里创建过多的 serial queue，那么后面提交的任务可能就会一直处于等待状态。这也是为什么我们需要严格控制 queue 的数量和层级关系，最好是 App 当中每个子系统只能分配固定数量和优先级的 queue，从而避免 thread explosion 导致的代码无法及时执行问题。 

——参考自<https://zhuanlan.zhihu.com/p/37463055> 



8.MAC：Mandatory Access Control 强制访问控制。 

9.iOS的安全机制比OS X的安全机制严格得多。OS X中代码签名是可选的，而iOS会通过kill-9杀掉任何代码签名不正确的进程。 

iOS中“坏警察”是由AMFL扮演的，AMFL在用户态也有一个守护程序：/usr/libexec/amfid。这个守护程序是由launchd启动的。也注册了一个主机特殊端口。 











## 第十五/十六章 文件系统 





1.ACL：Access Control List 访问控制列表。OS X允许通过chmod()设置和修改ACL。通过 ls -e 可以显示访问控制列表。 ——比如著名的chomd 777就是把文件改为可读可写可执行的状态 

2.Unix要求维护3个时间戳：创建事件、修改时间和访问时间。 

3.FIFO是UNIX对“具名管道”的实现。通过pipe()系统调用可以创建匿名管道，但是匿名管道不能在无关的进程间共享。 

4.底层的文件系统可以是基于表的（例如FAT），也可以基于B树的（例如NTFS和HFS+）。 

5.每一个操作系统都有一个自己的原生文件系统。DOS的原生文件系统是FAT，Windows的原生文件系统是NTFS，HFS+是OS X的原生文件系统。 

6.内核不能执行压缩操作，而且也没有提供对外部压缩的支持：在内核层只支持解压缩。 

7.HFS+使用的是UTF_16编码——双字节Unicode。HFS+也是大小写不敏感的文件系统。OS X 默认使用HFS+，iOS使用启动了大小写敏感的HFSX。 

8.日志是磁盘中一块特殊的区域，用户看不见这个区域，文件系统在向磁盘提交事务之前会将事务记录在这个区域中。如果修改事务被成功提交，那么这些事务就会从日志中删除。但是如果发生了崩溃，文件系统可以快速恢复到一致的状态——要么重放日志（即提交所有记录的事务），要么回滚日志（如果包含未完成的事务）。 

日志并不是解决数据丢失的灵丹妙药。然而，日志可以显著的减少系统崩溃导致文件系统无法使用的情况。 

9.HFS+有一项很有意思很特别的特性是能够动态适应频繁访问的文件。HFS+为每一个文件维护一个热度值。HFS+能够在工作时进行碎片整理工作。 

10.B树是一些文件系统构建的基础，例如NTFS（Windows）、Ext4（Linux）和苹果的HFS及HFS+。 

11.任何文件系统最基本的概念就是用于保存和取得文件的机制。需要满足的需求包括：搜索、插入、更新和随机访问。 

大部分文件系统都采用了基于树的方案。根据树形结构的设计，上述的要求都能满足，而且还很自然的提供了层次结构，这是平坦的表示结构无法提供的。 

12.B数可以看成是二叉树的扩展，相似的地方在于都采用了树形结构，而不同的地方在于B树的节点可以有任意数据的子节点——定义为m——而不只是两个子节点。这种结构可以帮助限制树的深度，从log2(N)（典型的二叉树搜索时间复杂度），到最优情况的logm(N)以及最坏情况的logm/2(N)。 

13.和所有的树一样，B树由节点组成，但是和其他树不一样的地方在于，B树的节点可以有具体的子类型，或称为kind。不同的节点类型可以保存不同的数据，但是所有类型的节点都来源于一个基本类型（可以看成是一个基类）。 

为了遍历所有的记录，在节点尾部向头部方向依次保存了指向每一条记录的指针，其中也包含节点中包含的任何空闲空间使用的空记录的指针。 

14.HFS+的B树总是有一个固定的深度。也就是说，所有的叶子节点都在同一层上。 

15.当HFS+挂载时启动了日志功能，那么还会启用一个日志文件。 





#### 其他部分  网络协议栈/内核扩展模块/IOKit 



1.一般的Cocoa开发者并不需要了解套接字相关的知识。因为CoreFoundation通过CFNetwork提供了封装了套接字的CFSocket和CFStream，此外还提供了一些协议的封装，例如CFFTP、CFHTTP等。尽管如此，BSD套接字是XNU中所有网络组件的核心（实际上也是所有现代操作系统的核心）。 

![](/images/2019/04/11.png) 

2.模块化设计是可扩展性之母。 

3.代码签名，如今已经被大多数系统采纳为标准。Windows是一个典型实例，只允许加载具有合法数字签名的驱动程序。在控制权转交给模块入口点之前，内核会验证代码签名，代码签名保存在附加的证书中。证书必须通过私钥签名，内核已知公钥，内核也可以通过一个信任链获得这样一个秘钥。 

早在iBoot阶段，未经苹果签名的代码就不能被加载。 

4.预链接是苹果在OS X和iOS中使用的方法。引导加载器不按照先加载内核，再以一定顺序加载kext的方式进行加载，而是加载一个kernelcache文件。——好处是加载的速度快得多，并且kernelcache可以添加签名，甚至还可以加密，一旦加载了kernelcache，就可以禁止所有kext加载，这样可以阻断代码进入iOS内核的合法通道。 

5.IOKit有一个顶层的抽象基类是OSObject。 

6.IOKit提供了一个工作循环（work loop）模型，有一点类似于Objective-C的runloop（或Mach的消息循环）。简而言之，工作循环是一个不断处理事件的消息处理循环。通过使用工作循环可以极大的简化并发问题，而且通常可以避免对锁的使用，锁是会影响性能的。 

7.IOKit采用了NeXT的runloop模型，用户态开发者应该会想到CFRunLoop。IOKit版本的runloop成为IOWorkloop，基本思想是一样的：提供一个单线程且线程安全的机制处理所有类型的事件，如果不采用这种机制则是异步的。工作循环的访问被一个互斥体保护，因此不需要考虑可重入的问题以及线程安全的问题。不过要注意的是，不能保证工作循环确实是一个线程。也就是说，工作循环的迭代可能会运行在系统中另一个线程的上下文中。 

8.上下文切换：是另一种类型的控制权转移，上下文切换指的是将当前正在执行的线程的上下文切换为另一个线程的上下文。 

9.ARM和Intel处理器都在处理器层次提供了线程的支持。事实上，这也是为什么现代操作系统都不调度进程，而是调度线程的原因。 

10.在现代操作系统中，实现并发的先决条件是具有能够提供安全锁定机制的能力，通过这种方式能够同步共享资源的访问。同步机制通常都依赖于硬件的支持，因此在ARM和Intel架构上的实现是不同的。Mach底层的hw_lock_lock()函数的实现时候一个很好的示例。从内核的角度看，这个函数提供的方法总是一直的：快速的自旋锁。 

11.原子操作是和锁非常接近的功能。院子操作是一类保证了原子性（atomicity，几不可中断性）的操作。原子操作常作为锁操作的底层机制（因为必须以原子的方式访问锁），而且经常可以替代锁的使用（当要保护的对象为机器字大小时）。 

原子不一定意味着单周期，原子的意思只是说CPU保证在访问的过程中不会被打断。 

12.为了能够最优化的利用内部组件（例如ALU，FPU和加载/存储组件），现代的CPU会采用乱序的方式执行指令。然而在某些情况下，乱序执行可能会在程序中引入bug。在这种情况下，可以使用屏障（barrier）指令来确保程序执行到某个点时所有访问操作都完成了。——比如iOS的内存屏障 