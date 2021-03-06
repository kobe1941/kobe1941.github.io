---
layout: post
title: "《深入理解计算机系统》"
date: 2019-12-27 22:34:42 +0800
comments: true
categories: iOS开发 读书笔记
keywords: 
description: 
---

这本书应该是要配合实验才能理解的更加深刻。不过可惜只是过了一遍理论。。

第三章，第七章和第九章的内容比较有意思，相对来说对实际编程的帮助会更大一些，整体上本书可以当做参考书，有需要的时候来查一下还是很不错的。



ps：循环是用类似goto语句的原理（jump指令）实现的。

<!--more-->



## 名词解释



ALU：算数逻辑单元。

ISA：instruction Set Architecture 指令集体系结构或指令集架构

CISC：复杂指令集计算机

RISC：精简指令集计算机

PCI：Peripheral Component Interconnect 外围设备互联

USB：Universal Serial Bus 通用串行总线

NFS：Network File System 网络文件系统

LFU：Least Frequency Used 最不常使用

LRU：Least Recently Used 最近最少使用

ELF：Execute and Linkable Format 可执行可链接格式

ECF：ExceptionalControl Flow 异常控制流

HTTP：Hypertext Transfer Protocol  超文本传输协议

HTML：Hypertext Markup Language 超文本标记语言

MIMIE：Multipurpose Internet Mail Extension 多用途国际邮件扩充协议

URL：Universal Resource Locator  通用资源定位符

URI：Uniform Resource Identifier  统一资源标识符





## 第一章  计算机系统漫游

1.源程序实际上就是一个由值0和1组成的位（又称为比特）序列，8个位被组织成一组，称为字节。每个字节表示程序中的某些文本字符。



2.像hello.c这样只由ASCII字符构成的文件称为文本文件，所有其他文件都称为二进制文件。



3.系统所有的信息——包括磁盘文件、内存中的程序、内存中存放的用户数据以及网络上传送的数据，都是由一串比特表示的。区分不同数据对象的唯一方法是我们读到这些数据对象时的上下文。比如，在不同的上下文中，一个同样的字节序列可能表示一个整数、浮点数、字符串或者机器指令。



4.为了在系统上运行hello.c程序，每条c语句都必须被其他程序转化为一系列的低级机器语言指令。然后这些指令按照一种称为可执行目标程序的格式打好包，并以二进制磁盘文件的形式存放起来。目标程序也称为可执行目标文件。



5.在Unix系统上，从源文件到目标文件的转化是由编译器驱动程序完成的。

汇编器将hello.s翻译成机器语言指令。

![](/images/2019/12/1.jpg) 



6.贯穿整个系统的是一组电子管道，称作总线。通常总线被设计成传送定长的字节块，也就是字（word）。字中的字节数（即字长）是一个基本的系统参数，各个系统不尽相同。现在大多数机器字长要么是4个字节（32位），要么是8个字节（64位）。



7.控制器是IO设备本身或者系统的主印制电路板（主板）上的芯片组。而适配器则是一块主板插槽上的卡。它们的功能都是在IO总线和IO设备之间传递信息。



8.主存是一个临时存储设备，从逻辑上来说，存储器是一个线性的字节数组，每个字节都有其唯一的地址（数组索引）。



9.处理器CPU是解释（或执行）存储在主存中指令的引擎。处理器的核心是一个大小为一个字的存储设备（或寄存器），称为程序计数器（PC）。在任何时刻，PC都指向主存中的某条机器语言指令（即含有该条指令的地址）。



10.指令集架构描述的是每条机器代码指令的效果；而微体系结构描述的是处理器实际上是如何实现的。



11.利用DMA（直接存储器存取）技术，数据可以不通过处理器而直接从磁盘到达主存。



12.L1和L2高速缓存是用一种叫做静态随机访问存储器（SRAM）的硬件技术实现的。系统可以获得一个很大的存储器，同时访问速度也很快，原因是利用了高速缓存的局部性原理，即程序具有访问局部区域里的数据和代码的趋势。通过让高速缓存里存放可能经常访问的数据，大部分的内存操作都能在快速的高速缓存中完成。



13.我们可以把操作系统看成是应用程序和硬件之间插入的一层软件，所有应用程序对硬件的操作尝试都必须通过操作系统。



14.操作系统有两个基本功能：（1）防止硬件被失控的应用程序滥用；（2）向应用程序提供简单一致的机制来控制复杂而又通常大不相同的低级硬件设备。操作系统通过几个基本的抽象概念（进程、虚拟内存和文件）来实现这两个功能。



15.进程是操作系统对一个正在运行的程序的一种抽象。



16.无论是在单核还是在多核系统中，一个CPU看上去都像是在并发的执行多个进程，这是通过处理器在进程间切换来实现的。操作系统实现这种交错执行的机制称为上下文切换。



17.从一个 进程到另一个进程的转换是由操作系统内核管理的。内核是操作系统代码常驻内存的部分。内核不是一个独立的进程。相反，它是系统管理全部进程所用代码和数据结构的集合。



18.虚拟内存是一个抽象概念，它为每个进程提供了一个假象，即每个进程都在独占的使用主存。每个进程看到的内存都是一致的，称为虚拟地址空间。



在Linux中，地址空间最上面的区域是保留给操作系统中的代码和数据的（内核），这对所有进程来说都是一样。地址空间的地步区域存放用户进程定义的代码和数据。



大约在地址空间的中间部分是一块用来存放像C标准库和数学库这样的共享库的代码和数据的区域。



位于用户虚拟地址空间顶部的是用户栈。

![](/images/2019/12/2.jpg) 

![](/images/2019/12/3.jpg) 



19.虚拟内存的运作需要硬件和操作系统软件之间精密复杂的交互，包括对处理器生成的每个地址的硬件翻译。基本思想是把一个进程虚拟内存的内容存储在磁盘上，然后用主存作为磁盘的高速缓存。



20.文件就是字节序列，仅此而已。每个IO设备，包括键盘鼠标显示器甚至网络等等，都可以看成是文件。



21.数字计算机的整个历史中，有两个需求是驱动进步的持续动力：一个是我们想要计算机做的更多，另一个是我们想要计算机运行的更快。



22.并发是一个通用的概念，指一个同时具有多个活动的系统；并行指的是用并发来使一个系统运行得更快。



23.多核处理器是将多个CPU集成到一个集成电路芯片上。



24.超线程，有时称为同时多线程，是一项允许一个CPU执行多个控制流的技术。它涉及CPU某些硬件有多个备份，比如程序计数器和寄存器文件，而其他的硬件部分只有一份。Intel Core i7 处理器可以让每个核执行两个线程，所以一个4核的系统实际上可以并行的执行8个线程。



25.在较低的抽象层次上，现代处理器可以同时执行多条指令的属性称为指令级并行。



26.在流水线（pipelining）中，将执行一条指令所需要的活动划分为不同的步骤，将处理器的硬件组织成一系列的阶段，每个阶段执行一个步骤。这些阶段可以并行的操作，用来处理不同指令的不同部分。如果处理器可以达到比一个周期一条指令更快的执行速率，就称之为超标量处理器。



27.在最低层次上，许多现代处理器拥有特殊的硬件，允许一条指令产生多个可以并行执行的操作，这种方式称为单指令、多数据，即SIMD并行。



28.在处理器里，指令集架构提供了对实际处理器硬件的抽象。



29.三个抽象：文件是对IO设备的抽象，虚拟内存是对程序存储的抽象，而进程是对一个正在运行的程序的抽象。



30.虚拟机，它提供对整个计算机的抽象，包括操作系统，处理器和程序。





### 第一部分   程序结构和执行



## 第二章 信息的表示和处理



1.无符号（unsigned）编码基于传统的二进制表示法，表示大于或等于0的数字。补码编码是表示有符号整数的最常见的方式。浮点数编码是表示实数的科学计数法的以2为基数的版本。



2.大多数计算机使用8位的块，或者字节，作为最小的可寻址的内存单位，而不是访问内存中单独的位。机器级程序将内存视为一个非常大的字节数组，称为虚拟内存。内存中的每个字节都由一个唯一的数字来标识，称为它的地址，所有可能的地址的集合就称为虚拟地址空间。



3.我们将程序称为“32位程序”或“64位程序”时，区别在于该程序是如何编译的，而不是其运行的机器类型。



4.在几乎所有的机器上，多字节对象都被存储为连续的字节徐磊，对象的地址为所使用字节中最小的地址。



5.反汇编器（disassembler）是一种确定可执行程序文件所表示的指令序列的工具。



6.表达式sizeof（T）返回存储一个类型为T的对象所需要的字节数。



7.有一个相应的右移运算 x>>k，逻辑右移在左端补k个0，算术右移是在左端补k个最高有效位的值。实际上，几乎所有的编译器/机器组合都对有符号数使用算术右移，而对于无符号数，右移必须是逻辑的。



8.C和C++都支持有符号（默认）和无符号数。Java只支持有符号数。



9.C语言标准并没有要求要用补码形式来表示有符号整数，但是几乎所有的机器都是这么做的。



10.当执行一个运算时，如果它的一个运算数是有符号的而另一个是无符号的，那么C语言会隐式的将有符号参数强制类型转换为无符号数，并假设这两个数都是非负的。



11.有符号数到无符号数的隐式转换，会导致错误或者漏洞的方式，避免这类错误的一种方法就是绝不使用无符号数。



12.由于整数乘法比移位和加法的代价要大得多，许多C语言编译器试图以移位、加法和减法的组合来消除很多整数乘以常数的情况。



## 第三章 程序的机器级表示



1.GCC C语言编译器以汇编代码的形式产生输出，汇编代码是机器代码的文本表示，给出程序中的每一条指令。然后GCC 调用汇编器和链接器，根据汇编代码生成可执行的机器代码。



2.使用现代的优化编译器，最大的优点是，用高级语言编写的程序可以在很多不同的机器上编译和执行，而汇编代码则是与特定机器密切相关的。



3.优化编译器能够重新排列执行顺序，消除不必要的计算，用快速操作替换慢速操作，甚至将递归计算变换成迭代计算。



4.大多数ISA，包括x86-64，将程序的行为描述成好像每条指令都是按顺序执行的，一条指令结束后，下一条再开始。处理器的硬件远比描述的精细复杂，它们并发的执行许多指令，但是可以采取措施保证整体行为与ISA指定的顺序执行的行为完全一致。



5.程序计数器PC给出将要执行的下一条指令在内存中的地址。



6.x86-64的虚拟地址是由64位的字来表示的。在目前的实现中，这些地址的高16位必须设置为0，所以一个地址实际上能够指定的是2^48或256TB范围内的一个字节。操作系统负责管理虚拟地址空间，将虚拟地址翻译成实际处理器内存中的物理地址。



7.机器执行的程序只是一个字节序列，它是对一系列指令的编码。机器对产生这些指令的源代码几乎一无所知。



8.反汇编器（disassembler）只是基于机器代码文件中的字节序列来确定汇编代码。它不需要访问该程序的源代码或汇编代码。



9.16个寄存器如下图，最特别的是栈指针%rsp，用来指明运行时栈的结束位置。



ps：函数的返回值也是存在寄存器里，函数传参一般也用寄存器，不够时才会用栈。

![](/images/2019/12/4.jpg) 



10.栈可以实现为一个数组，总是从数组的一端插入和删除元素。这一端被称为栈顶。——也可以实现为链表



11.因为栈和程序代码以及其他形式的程序数据都是放在同一内存中，所以程序可以用标准的内存寻址方法访问栈内的任意位置。



12.处理器通过使用流水线pipelining来获得高性能。



13.汇编中没有循环的指令存在，可以用条件测试和跳转组合起来实现循环的效果。——也就是类似goto或jump指令



14.使用跳转表是一种非常有效的实现多重分支的方法。在我们的例子中，程序可以只是用一次跳转表引用就分支到5个不同的位置。甚至当switch语句有上百种情况的时候，也可以只用一次跳转表访问去处理。



15.C语言过程调用机制的一个关键特性（大多数其他语言也是如此）在于使用了栈数据结构提供的后进先出的内存管理原则。



16.**x86-64的栈向低地址方向增长，而栈指针%rsp指向栈顶元素。当x86-64过程需要的存储空间超出寄存器能够存放的大小时，就会在栈上分配空间。这个部分称为过程的栈帧（stack frame）。大多数过程的栈帧都是定长的，在过程的开始就分配好了。但是有些过程需要变长的栈帧。**



17.**将控制从函数P转移到函数Q只需要简单的把程序计数器PC设置为Q的代码的起始位置。不过稍后从Q返回的时候，处理器必须记录好它需要继续P的执行的代码的位置。在x86-64机器中，这个信息是用指令call Q 调用过程Q来记录的。call Q指令会把地址A压入栈中，并将PC设置为Q的起始地址。压入的地址A被称为返回地址，是紧跟在call 指令后面的那条指令的地址。对应的指令ret 会从栈中弹出地址A，并把PC设置为A。——对比下上面函数返回值，函数的返回地址和返回值是不同的**



18.x86-64中，可以通过寄存器最多传递6个整形（即整数和指针）参数。如果一个函数有大于6个整形参数，超出6个的部分就要通过栈来传递。有n个整形参数，且n>6，那么就把参数1-6复制到对应的寄存器，把参数7-n 放到栈上，而参数7就位于栈顶。

![](/images/2019/12/5.jpg) 



19.作为过程调用的一部分，返回地址被压入栈中。



20.运行时栈提供了一种简单的、在需要时分配、函数完成时释放局部存储的机制。



21.寄存器组是唯一被所有过程共享的资源。虽然在给定时刻只有一个过程是活动的，我们仍然必须确保当一个过程（调用者）调用另一个过程（被调用者）时，被调用者不会覆盖调用者稍后会使用的寄存器值。



根据惯例，寄存器%rbx、%rbp和%r12-%r15被划分为被调用者保存寄存器。当过程P调用过程Q时，Q必须保存这些寄存器的值不变，保证它们的值在Q返回到P时与Q被调用时是一样的。



所有其他的寄存器，除了栈指针%rsp，都分类为调用者保存寄存器：过程P在某个此类寄存器中有局部数据，然后调用过程Q。因为Q可以随意修改这个寄存器，所以在调用之前首先保存好这个数据是P（调用者）的责任。



22.二维数组在内存中按照“行优先”的顺序排列，意味着第0行的所有元素，可以写作A[0]。A[5][3]则可以被看成一个5行3列的二维数组。

![](/images/2019/12/6.jpg) 



23.ISO C99引入了一种功能，允许数组的维度是表达式，在数组被分配的时候才计算出来。



24.C语言提供了两种将不同类型的对象组合到一起创建数据类型的机制：结构，用关键字struct来声明，将多个对象集合到一个单位中；联合，用关键字union来声明，允许用几种不同的类型来引用一个对象。



25.类似于数组的实现，结构的所有组成部分都存在在内存中一段连续的区域内，而指向结构的指针就是结构第一个字节的地址。编译器维护关于每个结构类型的信息，指示每个字段的字节偏移。它以这些偏移作为内存引用指令中的位移，从而产生对结构元素的引用。



26.一种情况是，我们事先知道对一个数据结构中的两个不同字段的使用是互斥的，那么将这两个字段声明为联合的一部分，而不是结构的一部分，就会减少分配空间的总量。



27.许多计算机系统对基本数据类型的合法地址做了一些限制，要求某种类型对象的地址必须是某个值K（通常是2、4或8）的倍数。无论数据是否对齐，x86-64硬件都能正确工作。不过Intel还是建议要对齐数据以提高内存系统的性能。对齐原则是任何K字节的基本对象的地址必须是K的倍数。



28.如果数据没有对齐，某些型号的Intel或AMD处理器对于有些实现多媒体操作的SSE指令，就无法正确执行。任何试图以不满足对齐要求的地址来访问内存都会导致异常，默认的行为是程序终止。



因此，任何针对x86-64处理器的编译器和运行时系统都必须保证分配用来保存可能会被SSE寄存器读或写的数据结构的内存，都必须满足16字节对齐。



29.函数指针的值是该函数机器代码表示中第一条指令的地址。



30.如果存储的返回地址的值被破坏了，那么ret指令会导致程序跳转到一个完全意想不到的位置。



31.缓冲区溢出的一个更加致命的使用就是让程序执行它本来不愿意执行的函数。



32.栈随机化的思想使得栈的位置在程序每次运行时都有变化。



在Linux系统中，栈随机化已经变成了标准行为，它是ASLR（地址空间布局随机化）技术中的一种。采用ASLR，每次运行时程序的不同部分，包括程序代码、库代码、栈、全局变量和堆数据，都会被加载到内存的不同区域。



ASLR技术能够增加成功攻击一个系统的难度，但是也不能提供完全的安全保障。



33.随机化，栈保护和限制哪部分内存可以存储可执行代码，是用于最小化程序缓冲区溢出攻击漏洞三种最常见的机制。



34.为了管理变长栈帧，x86-64代码使用寄存器%bp作为帧指针。



35.JOT即时编译，动态的将字节代码序列翻译成机器指令。当代码要执行多次时，这种方法执行起来更快。





## 第四章  处理器体系结构



1.与同一时刻只执行一条指令相比，通过同时处理多条指令的不同部分，处理器可以获得更高的性能。



2.寄存器%rsp被入栈、出栈、调用和返回指令作为栈指针。程序计数器PC存放当前正在执行指令的地址。



3.内存从概念上来说就是一个很大的字节数组，保存着程序和数据。



4.call 指令将返回地址入栈，然后跳到目的地址。ret 指令从这样的调用中返回。nop 指令只是简单的经过各个阶段，除了要将PC加1，不进行任何处理。halt 指令使得处理器状态被设置成HLT，导致处理器停止运行。



5.流水线化的一个重要特性就是提高了系统的吞吐量，也就是单位时间内服务的顾客综述，不过它也会轻微的增加延迟。



6.猜测分支方向并根据猜测开始取指的技术称为分支预测。

分支预测错误会极大的降低程序的性能，因此这就促使我们在可能的时候，要使用条件数据传送而不是条件控制转移。



7.我们的指令集体系结构包括三种不同的内部产生的异常：（1）halt指令；（2）有非法指令和功能码组合的指令；（3）取指或数据读写试图访问一个非法地址。



## 第五章  优化程序性能



1.编写高效程序需要做到以下几点：①我们必须选择一组适当的算法和数据结构。②我们必须编写出编译器能够有效优化以转换成高效可执行代码的源代码。③针对处理运算量特别大的计算，将一个任务分成多个部分，这些部分可以在多核和多处理器的某种组合上并行的计算。



C语言的有些特性，例如执行指针运算和强制类型转换的能力，使得编译器很难对它进行优化。

即使是要利用并行性，每个并行的线程都以最高性能执行也是非常重要的。



2.程序优化的第一步就是消除不必要的工作，这包括不必要的函数调用，条件测试和内存引用。（消除循环的低效率，减少过程调用）



程序优化的第二步，利用处理器提供的指令级并行能力，同时执行多条指令。



3.用内联函数替换优化函数调用。



4.实际的处理器中，是同时对多条指令求值的，这个现象称为指令级并行。在某些设计中，可以有100或更多条指令在处理中。



## 第六章   存储器层次结构



1.具有良好局部性的程序倾向于一次又一次的访问相同的数据项集合，或是倾向于访问邻近的数据项集合。



2.基本的存储技术：SRAM存储器，DRAM存储器，ROM存储器以及旋转的和固态的硬盘。



3.随机访问存储器RAM分为两类：静态的和动态的。静态RAM（SRAM）比动态RAM（DRAM）更快，但也贵的多。SRAM用来作为高速缓存存储器，既可以在CPU芯片上，也可以在片下。DRAM用来作为主存以及图形系统的帧缓冲区。



4.SRAM将每个位存储在一个双稳态的存储器单元里。每个单元是用一个六晶体管电路来实现的。——从不稳定状态开始，电路会迅速的转移到两个稳定状态中的一个。



由于SRAM存储器单元的双稳态特性，只要有电，它就会永远的保持它的值。即使有干扰来扰乱电压，当干扰消除时，电路就会恢复到稳定值。



5.DRAM将每个位存储为对一个电容的充电。这个电容非常小。DRAM存储器可以制造得非常密集——每个单元由一个电容和一个访问晶体管组成。但与SRAM不同，DRAM存储器单元对干扰非常敏感。当电容的电压被扰乱之后，它就永远不会恢复了。暴露在光线下会导致电容电压改变。实际上，数码照相机和摄像机中的传感器本质上就是DRAM单元的阵列。



很多原因会导致漏电，使得DRAM单元在10-100ms的时间内失去电荷。内存系统必须周期性的通过读出，然后重写来刷新内存每一位。

![](/images/2019/12/7.jpg) 



6.DRAM芯片封装在内存模块中，它插到主板的扩展槽上。



7.SDRAM 称为同步DRAM。双倍数据速率同步DRAM称为DDR SDRAM，它是SDRAM的一种增强，它通过使用两个时钟沿作为控制信号，从而使DRAM的速度翻倍。不同类型的DDR SDRAM是用提高有效带宽的很小的预取缓冲区的大小来划分的：DDR（2位）、DDR2（4位）、DDR3（8位）。



8.如果断电，DRAM和SRAM会丢失它们的信息。



PROM可编程ROM只能被编程一次。



EPROM可擦写可编程ROM能够被擦除和重编程的次数的数量级可以达到1000次。EEPROM电子可擦除PROM类似于EPROM，能够被编程的次数的数量级可以达到10^5次。



闪存是一类非易失性存储器，基于EEPROM。基于闪存的磁盘驱动器，称为固态硬盘SSD。



存储在ROM设备中的驱动程序通常称为固件firmware。



9.数据流通过称为总线的共享电子电路在处理器和DRAM主存之间来来回回。



10.磁盘是由一个或多个叠放在一起的盘片组成的。每个盘片有两面或者称为表面，每个表面是由一组称为磁道的同心圆组成，每个磁道被划分为一组扇区。



11.磁盘控制器必须对磁盘进行格式化，然后才能在该磁盘上存储数据。格式化包括用标识扇区的信息填写扇区之间的间隙，标识出表面有故障的柱面并且不使用它们，以及在每个区中预留出一组柱面作为备用。如果区中一个或多个柱面在磁盘使用过程中坏掉了，就可以使用这些备用的柱面。因为存在着这些备用的柱面，所以磁盘制造商所说的格式化容量比最大容量要小。



12.CPU使用一种称为内存映射 I/O 的技术来向 I/O 设备发射命令。在使用内存映射 I/O 的系统中，地址空间中有一块地址是为与I/O设备通信保留的。每个这样的地址称为一个 I/O 端口。



13.在磁盘控制器收到来自CPU的读命令后，它将逻辑块号翻译成一个扇区地址，读该扇区的内容，然后将这些内容直接传送到主存，不需要CPU的干涉。设备可以自己执行读或者写总线事务而不需要CPU干涉的过程，称为直接内存访问DMA（Direct Memory Access）。



在DMA传送完成，磁盘扇区的内容被安全的存储在主存中以后，磁盘控制器通过给CPU发送一个中断信号来通知CPU。基本思想是中断会发信号到CPU芯片的一个外部引脚上。这会导致CPU暂停它当前正在做的工作，跳转到一个操作系统例程，这个程序会记录下I/O已经完成，然后将控制返回到CPU被中断的地方。



14.固态硬盘是一种基于闪存的存储技术。一个SSD封装由一个或多个闪存芯片和闪存翻译层组成。



15.随机写很慢，有两个原因。首先，擦除块需要相对较长的时间，1ms级的，比访问页所需时间要高一个数量级。其次，如果写操作试图修改一个包含已经有数据的页p，那么这个块中所有带有用数据的页都必须被复制到一个新（擦除过的）块，然后才能进行对页p的写。



16.不同的存储技术有不同的价格和性能折中。快速存储总是比慢速存储要贵的。SRAM比DRAM贵，DRAM又比磁盘贵的多，SSD位于DRAM和旋转磁盘之间。



DRAM和磁盘的性能滞后于CPU的性能。而且差距实际上是在加大的。



17.多核处理器：无法再像以前一样迅速的增加CPU的时钟频率了，因为如果那样芯片的功耗会太大。解决方法是用多个小处理器核取代单个大核处理器，从而提高性能，每个完整的处理器能够独立的，与其他核并行的执行程序。



18.局部性原理：它们倾向于引用邻近与其他最近引用过的数据项的数据项，或者最近引用过的数据项本身。



局部性通常有两种不同的形式：时间局部性和空间局部性。在一个具有良好时间局部性的程序中，被引用过一次的内存位置很可能在不远的将来再被多次引用。在一个具有良好空间局部性的程序中，如果一个内存位置被引用了一次，那么程序很可能在不远的将来引用附近的一个内存位置。



有良好局部性的程序比局部性差的程序运行的更快。



19.重复引用相同变量的程序有良好的时间局部性。对于具有步长为k的引用模式的程序，步长越小，空间局部性越好。对于取指令来说，循环有好的时间和空间局部性。循环体越小，循环迭代次数越多，局部性越好。



20.固态硬盘在存储器层级结构中扮演着越来越重要的角色，连接起DRAM和旋转磁盘之间的鸿沟。

![](/images/2019/12/8.jpg) 



21.直写，就是立即将w的高速缓存块写回到紧接着的低一层中。虽然简单，但是直写的缺点是每次都会引起总线的流量。



写回，尽可能的推迟更新，只有当替换算法要驱逐这个更新过的块时，才把它写到紧接着的低一层中。由于局部性，写回能显著的减少总线流量，但是它的缺点是增加了复杂性。



虚拟内存系统（用主存作为存储在磁盘上的块的缓存）只使用写回。我们在现代系统的所有层次上都能看到写回缓存，其与处理读的方式相对称。



一般而言，高速缓存越往下层，越可能用写回而不是直写。



22.存储器系统的性能不是一个数字就能描述的，相反，它是一座时间和空间局部性的山。





#### 第二部分  在系统上运行程序



## 第七章 链接



1.链接器把程序的各个部分联合成一个文件，处理器可以将这个文件加载到内存，并且执行它。



链接是将各种代码合数据片段收集并组合成为一个单一文件的过程。



链接可以执行于编译时，也就是在源代码被翻译成机器代码时；也可以执行于加载时，也就是在程序被加载器加载到内存并执行时；甚至执行于运行时，也就是由应用程序来执行。现代系统中，链接是由叫做链接器的程序自动执行的。



2.链接器在软件开发中扮演着一个关键的角色，因为它们使得分离编译成为可能。我们不用将一个大型的应用程序组织为一个巨大的源文件，而是可以把它分解为更小、更好管理的模块，可以独立的修改和编译这些模块。



3.许多软件产品在运行时使用共享库来升级压缩包装的二进制程序。大多数web服务器都依赖于共享库的动态链接来提供动态内容。



4.无论是什么样的操作系统，ISA或者目标文件格式，基本的链接概念是通用的。



5.大多数编译系统提供编译器驱动程序，它代表用户在需要时调用语言预处理器、编译器、汇编器和链接器。

6.静态链接：像Linux LD 程序这样的静态链接器以一组可重定位目标文件和命令行参数作为输入，生成一个完全

链接的、可以加载和运行的可执行目标文件作为输出。输入的可重定位目标文件由各种不同的代码合数据节组成，每一节都是一个连续的字节序列。指令在一节中，初始化了的全局变量在另一节中，而未初始化的变量又在另外一节中。



为了构造可执行文件，链接器必须完成两个主要任务：



①符号解析。目标文件定义和引用符号，每个符号对应于一个函数、一个全局变量或一个静态变量（即C语言中任何以static属性声明的变量）。符号解析的目的是将每个符号引用正好和一个符号定义关联起来。



②重定位。编译器和汇编器生成从地址0开始的代码和数据节。链接器通过把每个符号定义与一个内存位置关联起来，从而重定位这些节，然后修改所有对这些符号的引用，使得它们指向这个内存位置。



7.关于链接器的一些基本事实：目标文件纯粹是字节块的集合。这些块中，有些包含程序代码，有些包含程序数据，而其他的则包含引导链接器和加载器的数据结构。



8.目标文件由三种形式：



①可重定位目标文件，包含二进制代码和数据，其形式可以在编译时与其他可重定位目标文件合并起来，创建一个可执行目标文件。



②可执行目标文件，包含二进制代码和数据，其形式可以被直接复制到内存并执行。



③共享目标文件，一种特殊类型的可重定位目标文件，可以在加载或者运行时被动态的加载进内存并链接。



9.编译器和汇编器生成可重定位目标文件（包括共享目标文件）。链接器生成可执行目标文件。从技术上来说，一个目标模块就是一个字节序列，而一个目标文件就是一个以文件形式存放在磁盘中的目标模块。



10.一个典型的ELF可重定位目标文件包含下面几个节：



.text：已编译程序的机器代码。



.rodate：只读数据，比如printf语句中的格式串和开关语句的跳转表。



.data：已初始化的全局和静态C变量。局部C变量在运行时被保存在栈中，既不出现在.data节中，也不出现在.bss节中。



.bss：未初始化的全局和静态C变量，以及所有被初始化为0的全局或静态变量。在目标文件中这个节不占据实际的空间，它仅仅是一个占位符。在目标文件中，未初始化的变量不需要占据任何实际的磁盘空间。运行时，在内存中分配这些变量，初始值为0。



.symtab：一个符号变，它存放在程序中定义和引用的函数和全局变量的信息。实际上，每个可重定位目标文件在.symtab中都有一张符号表（除非程序员特意用STRIP 命令去掉它）。然而，和编译器中的符号表不同，.symtab符号表不包含局部变量的条目。



11.每个可重定位目标模块m都有一个符号表，它包含m定义和引用的符号的信息。在链接器的上下文中，有三种不同的符号：

①由模块m定义并能被其他模块引用的全局符号。

②由其他模块定义并被模块m引用的全局符号。

③只被模块m定义和引用的局部符号。



有趣的是，定义为带有C static属性的本地过程变量是不在栈中管理的。相反，编译器在.data或.bss中为每个定义分配空间，并在符号表忠创建一个有唯一名字的本地链接器符号。



在C中，源文件扮演模块的角色。任何带有static属性声明的全局变量或者函数都是模块私有的。类似的，任何不带static属性声明的全局变量和函数都是公共的，可以被其他模块访问。



12.符号表是由汇编器构造的，使用编译器输出到汇编语言.s文件中的符号。



13.链接器解析符号引用的方法是将每个引用与它输入的可重定位目标文件的符号表忠的一个确定的符号定义关联起来。静态局部变量也会有本地链接器符号，编译器还要确保它们拥有唯一的名字。



14.对全局符号的符号解析很棘手，还因为多个目标文件可能会定义相同名字的全局符号。在这种情况下，链接器必须要么标志一个错误，要么以某种方法选出一个定义并抛弃其他定义。



15.C++和Java中能使用重载函数，是因为编译器将每个唯一的方法和参数列表组合编码成一个对链接器来说唯一的名字。这种编码过程叫做重整，而相反的过程叫做恢复。



16.在编译时，编译器向汇编器输出每个全局符号，或者是强strong或者是弱weak。函数和已初始化的全局变量是强符号，未初始化的全局变量是弱符号。



根据强弱符号的定义，Linux链接器使用下面的规则来处理多重定义的符号名：

①不允许有多个同名的强符号；

②如果有一个强符号和多个弱符号同名，那么选择强符号；

③如果有多个弱符号同名，那么从这些弱符号中任意选择一个。



17.所有的编译系统都提供一种机制，将所有相关的目标模块打包称为一个单独的文件，称为静态库。它可以用作链接器的输入。当链接器构造一个输出的可执行文件时，它只复制静态库里被应用程序引用的目标模块。



静态库概念被提出来，以解决这些不同方法的缺点。相关的函数可以被编译为独立的目标模块，然后封装成一个单独的静态库文件。在链接时，链接器将只复制被程序引用的目标模块，这就减少了可执行文件在磁盘和内存中的大小。



18.在Linux系统中，静态库以一种称为存档archive的特殊文件格式存放在 磁盘中。存档文件是一组连接起来的可重定位目标 文件的集合，有一个头部用来描述每个成员目标文件的大小和位置。存档文件名由后缀.a标识。



19.任何Linux程序都可以通过调用execve函数来调用加载器，加载器将可执行目标文件中的代码合数据从磁盘复制到内存中，然后通过跳转到程序的第一条指令或入口点来运行该程序。这个将程序复制到内存并运行的过程叫做加载。



20.所谓内核就是操作系统驻留在内存的部分。



21.加载器的工作流程：

**当shell运行一个程序时，父shell进程生成一个子进程，它是父进程的一个复制。子进程通过execve系统调用启动加载器。加载器删除子进程现有的虚拟内存段，并创建一组新的代码、数据、堆和栈段。新的栈和堆段被初始化为零。通过将虚拟地址空间中的页映射到 可执行文件的页大小的片chunk，新的代码和数据段被初始化为可执行文件的内容。最后，加载器跳转到_start地址，它最终会调用应用程序的main函数。除了一些头部信息，在加载过程中没有任何从磁盘到内存的数据复制。直到CPU引用一个被映射的虚拟页时才会进行复制，此时，操作系统利用它的页面调度机制自动将页面从磁盘传送到内存。**



22.共享库是致力于解决静态库缺陷的一个现代创新产物。共享库是一个目标模块，在运行或加载时，可以加载到任意的内存地址，并和一个在内存中的程序链接起来。这个过程称为动态链接，是由一个叫做动态链接器的程序来执行的。共享库也称为共享目标，在Linux系统中通常用.so后缀来表示。微软的操作系统大量的使用了共享库，它们称为DLL动态链接库。



23.动态链接器本身就是一个共享目标。加载器不会像它通常所做的那样将控制传递给应用，而是加载和运行这个动态链接器。最后动态链接器将控制传递给应用程序。



24.应用程序还可能在它运行时要求动态链接器加载和链接某个共享库，而无需在编译时将那些库链接到应用中。



25.现代高性能的Web服务器可以使用基于动态链接的更有效和完善的方法来生成动态内容。其思路是将每个生成动态内容的函数打包在共享库中。当一个来自Web浏览器的请求到达时，服务器动态的加载和链接适当的函数，然后直接调用它，而不是使用fork额execve在子进程的上下文中运行函数。函数会一直缓存在服务器的地址空间中，所以只要一个简单的函数调用的开销就可以处理随后的请求了。在运行时无需停止服务器，就可以更新已存在的函数，以及添加新的函数。



26.Linux系统为动态链接器提供了一个简单的接口，允许应用程序在运行时加载和链接共享库：dlopen()。



dlsym()函数的输入是一个指向前面已经打开了的共享库的句柄和一个symbol名字，如果该符号存在，就返回符号的地址，否则返回NULL。



如果没有其他共享库还在使用这个共享库，dlclose()函数就卸载该共享库。



27.JNI Java本地接口，它允许Java程序调用本地的C和C++函数。JNI的基本思想是将本地C函数（如foo）编译到一个共享库中（如foo.so）。当一个正在运行的Java程序试图调用函数foo时，Java解释器利用dlopen接口动态链接和加载foo.so，然后再调用foo。



28.现代系统以这样一种方式编译共享模块的代码段，使得可以把它们加载到内存的任何位置而无需链接器修改。这样子，无限多个进程可以共享一个共享模块的代码段的单一副本。（当然，每个进程仍然会有它自己的读/写数据库）。



可以加载而无需重定位的代码称为位置无关代码（PIC）。用户对GCC使用-fpic选项指示GNU编译系统生成PIC代码。共享库的编译必须总是使用该选项。



在一个x86-64系统中，对同一个目标模块中符号的引用是不需要特殊处理使之成为PIC。可以用PC相对寻址来编译这些引用，构造目标文件时由静态链接器重定位。



29.无论我们在内存中的何处加载一个目标模块（包括共享目标模块），数据段与代码段的距离总是保持不变。因此，代码段中任何指令和数据段中任何变量之间的距离都是一个运行时常量，与代码段和数据段的绝对内存位置是无关的。



30.延迟绑定：将过程地址的绑定推迟到第一次调用该过程时。



把函数地址的解析推迟到它实际被调用的地方，能避免动态链接器在加载时进行成百上千个其实并不需要的重定位。



延迟绑定是通过两个数据结构之间简洁但又有些复杂的交互来实现的，这两个数据结构是：GOT全局偏移量表和过程连接表PLT。



31.Linux链接器支持一个很强大的技术，称为库打桩，它允许你截获对共享库函数的调用，取而代之执行自己的代码。



其基本思想：给定一个需要打桩的目标函数，创建一个包函数，它的原型与目标函数完全一样。使用某种特殊的打桩机制，你就可以欺骗系统调用包装函数而不是目标函数了。包装函数通常会执行它自己的逻辑，然后调用目标函数，再将目标函数的返回值传递给调用者。



打桩可以发生在编译时、链接时或当程序被加载和执行的运行时。



32.可以使用C预处理器在编译时打桩。Linux静态链接器支持用—wrap f标志进行链接时打桩。



编译时打桩需要能够访问程序的源代码，链接时打桩需要能够访问程序的可重定位对象文件。基于动态链接器的LD_PRELOAD环境变量可以实现运行时打桩。



如果LD_PRELOAD环境变量被设置为一个共享库路径名的列表，那么当你加载和执行一个程序，需要解析未定义的引用时，动态链接器会先搜索LO_PRELOAD库，然后才搜索任何其他的库。



你可以用LD_PRELOAD对任何可执行程序的库函数调用打桩。

![](/images/2019/12/9.jpg) 



## 第八章 异常控制流



1.ECF异常控制流是操作系统用来实现I/O、进程和虚拟内存的基本机制。



应用程序通过使用一个叫做陷阱（trap）或者系统调用（system call）的ECF形式，向操作系统请求服务。比如，向磁盘写数据、从网络读取数据、创建一个新进程，以及终止当前进程，都是通过应用程序调用系统调用来实现的。



操作系统为应用程序提供了强大的ECF机制，用来创建新进程、等待进程终止、通知其他进程系统中的异常事件，以及检测和响应这些事件。



ECF是计算机系统中实现并发的基本机制。



像C++和Java这样的语言通过try、catch以及throw语句来提供软件异常机制。软件异常允许程序进行非本地跳转（即违反通常的调用/返回栈规则的跳转）来响应错误情况。非本地跳转是一种应用层ECF。



2.异常是异常控制流的一种形式，它一部分由硬件实现，一部分由操作系统实现。异常就是控制流中的突变，用来响应处理器状态中的某些变化。

在处理器中，状态被编码为不同的位和信号。状态变化称为事件。



3.在任何情况下，当处理器检测到有事件发生时，它就会通过一张叫做异常表的跳转表，进行一个间接过程调用（异常），到一个专门设计用来处理这类事件的操作系统子程序（异常处理程序exception handler）。



4.系统中可能的每种类型的异常都分配了一个唯一的非负整数的异常号。其中一些号码是由处理器的设计者分配的，其他号码是由操作系统内核的设计者分配的。



5.异常表的起始地址放在一个叫做异常表基址寄存器的特殊CPU寄存器里。



6.异常调用类似于过程调用，也有一些不同之处：



过程调用时，在跳转到处理程序之前，处理器将返回地址压入栈中。然而根据异常的类型，返回地址要么是当前指令（当事件发生时正在执行的指令），要么是下一条指令（如果事件不发生，将会在当前指令后执行的指令）。



处理器也把一些额外的处理器状态压到栈里，在处理程序返回时，重新开始执行被中断的程序会需要这些状态。



如果控制从用户程序转移到内核，所有这些项目都被压到内核栈中，而不是压到用户栈中。



异常处理程序运行在内核模式下，这意味着它们对所有的系统资源都有完全的访问权限。



7.一旦硬件触发了异常，剩下的工作就是由异常处理程序在软件中完成。在处理程序处理完事件之后，它通过执行一条特殊的“从中断返回”的指令，可选的返回到被中断的程序，该指令将适当的状态弹回到处理器的控制和数据寄存器中，如果异常中断的是一个用户程序，就将状态恢复为用户模式，然后将控制返回给被中断的程序。



8.异常可以分为四类：中断（interrupt）、陷阱（trap）、故障（fault）和终止（abort）。

![](/images/2019/12/10.jpg) 



需要注意的是，中断是在当前指令执行完成之后，才把控制转移给中断处理程序。剩下的异常类型（陷阱、故障和终止）是同步发生的，是执行当前指令的结果。我们把这类指令叫做故障指令。



9.陷阱是有意的异常，是执行一条指令的结果。就像中断处理程序一样，陷阱处理程序将控制返回到下一条指令。陷阱最重要的用途是在用户程序和内核之间提供一个像过程一样的接口，叫做系统调用。



10.用户程序经常需要向内核请求服务，比如读一个文件（read）、创建一个新的进程（fork）、加载一个新的程序（execve），或者终止当前进程（exit）。为了允许对这些内核服务的受控的访问，处理器提供了一条特殊的“syscall n”指令，当用户程序想要请求服务n时，可以执行这条指令。执行syscall指令会导致一个到异常处理程序的陷阱，这个处理程序解析参数，并调用适当的内核程序。



11.系统调用和普通的函数调用，它们的实现非常不同。普通函数运行在用户模式中，用户模式限制了函数可以执行的指令的类型，而且它们只能访问与调用函数相同的栈。系统调用运行在内核模式中，内核模式允许系统调用执行特权指令，并访问定义在内核中的栈。



12.故障由错误情况引起的，它可能能够被故障处理程序修正。如果处理程序能够修正这个错误情况，它就将控制返回到引起故障的指令，从而重新执行它。否则，处理程序返回到内核中的abort例程，abort会终止引起故障的应用程序。



一个经典的故障示例是缺页异常。



13.终止是不可恢复的致命错误造成的结果，通常是一些硬件错误。



14.Segment fault：通常是因为一个程序引用一个未定义的虚拟内存区域，或者因为程序试图写一个只读的文本段。



15.每个系统调用都有一个唯一的整数号，对应于一个到内核中跳转表的偏移量（注意：这个跳转表和异常表不一样）。



16.C程序用syscall 函数可以直接调用任何系统调用。



在 x86-64 系统上，系统调用是通过一条称为syscall 的陷阱指令来提供的。



所有到Linux系统调用的参数都是通过调用寄存器而不是栈传递的。按照惯例，寄存器%rax 包含系统调用号，寄存器%rdi、%rsi、%rdx、%r10、%r8和%r9包含最多6个参数。从系统调用返回时，寄存器%rcx和%r11 都会被破坏，%rax 包含返回值。



17.进程的经典定义就是一个执行中的程序的实例。系统中的每个程序都运行在某个进程的上下文context中。上下文是由程序正确运行所需的状态组成的。这个状态包括存放在内存中的程序的代码和数据，它的栈、通用目的寄存器的内容、程序计数器、环境变量以及打开文件描述符的集合。



进程提供给应用程序的关键抽象：



一个独立的逻辑控制流，它提供一个假象，好像我们的程序独占的使用处理器。



一个私有的地址空间，它提供一个假象，好像我们的程序独占的使用内存系统。



18.PC值的序列叫做逻辑控制流，或者简称逻辑流。



19.一个逻辑流的执行在时间上与另一个流重叠，称为并发流。多个流并发的执行的一般现象被称为并发。一个进程和其他进程轮流运行的概念称为多任务。一个进程执行它的控制流的一部分的每一时间段叫做时间片。因此，多任务也叫做时间分片。



并发流的思想与流运行的处理器核数或者计算机数无关。如果两个流在时间上重叠，那么它们就是并发的，即使它们是运行在同一个处理器上。



并行流是并发流的一个真子集。如果两个流并发的运行在不同的处理器核或者计算机上，那么我们称它们为并行流，它们并行的运行，且并行的执行。



20.为了使操作系统内核提供一个无懈可击的进程抽象，处理器必须提供一种机制，限制一个应用可以执行的指令以及它可以访问的地址空间范围。



处理器通常用某个控制寄存器中的一个模式位来提供这种功能的，该寄存器描述了进程当前享有的特权。当设置了模式位时，进程就运行在内核模式中。一个运行在内核模式的进程可以执行指令集中的任何指令，并且可以访问系统中的任何内存位置。



没有设置模式位时，进程就运行在用户模式中。用户模式中的进程不允许执行特权指令。用户程序必须通过系统调用接口间接的访问内核代码和数据。



进程从用户模式变为内核模式的唯一方法是通过诸如中断、故障或者陷入系统调用这样的异常。



21.操作系统内核使用一种称为上下文切换的较高层形式的异常控制流来实现多任务。



内核为每个进程维持一个上下文context。



上下文就是内核重新启动一个被抢占的进程所需的状态。它由一些对象的值组成，这些对象包括通用目的寄存器、浮点寄存器、程序计数器、用户栈、状态寄存器、内核栈和各种内核数据结构，比如描述地址空间的页表，包含有关当前进程信息的进程表，以及包含进程已打开文件的信息的文件表。



22.在进程执行的某些时刻，内核可以决定抢占当前进程，并重新开始一个先前被抢占了的进程。这种决策就叫做调度。



在内核调度了一个进程的运行后，它就抢占当前进程，并使用一种称为上下文切换的机制来将控制转移到新的进程。



上下文切换：保存当前进程的上下文；恢复某个先前被抢占的进程被保存的上下文；将控制传递给这个新恢复的进程。

![](/images/2019/12/11.jpg) 

![](/images/2019/12/12.jpg) 



23.sleep系统调用，它显式的请求让调用进程休眠。

pause函数让调用函数休眠，直到该进程收到一个信号。



24.每个进程都有一个唯一的正数进程ID（PID）。getpid函数返回调用进程的PID。getppid函数返回它的父进程的PID。



25.从程序员的角度，我们可以认为进程总是处于下面三种状态之一：



运行。进程要么在CPU上执行，要么在等待被执行且最终会被内核调度。



停止。进程的执行被挂起，且不会被调度。



终止。进程永远的停止了，进程会因为三种原因终止：收到一个信号，该信号的默认行为是终止进程；从主程序返回；调用exit函数。



exit函数以status退出状态来终止进程。



26.父进程通过调用fork函数创建一个新的运行的子进程。子进程得到与父进程用户级虚拟地址空间相同的（但是独立的）一份副本，包括代码和数据段、堆、共享库以及用户栈、子进程还获得与父进程任何打开文件描述符相同的副本，这就意味着当父进程调用fork时，子进程还获得与父进程中打开的任何文件。父进程和新创建的子进程之间最大的区别在于它们有不同的PID。



fork函数只被调用一次，却会返回两次：一次是在调用进程（父进程）中，一次是在新创建的子进程中。在父进程中，fork返回子进程的PID。在子进程中，fork返回0。因为子进程的PID总是为非零，返回值就提供一个明确的方法来分辨程序是在父进程还是在子进程中执行。



27.当一个进程由于某种原因终止时，内核并不是立即把它从系统中清楚。相反，进程被保持在一种已终止的状态中国，直到被它的父进程回收。一个终止了但还未被回收的进程称为僵死进程。



如果一个父进程终止了，进程会安排 init 进程成为它的孤儿进程的养父。init 进程的PID为1，是在系统启动时由内核创建的，它不会终止，是所有进程的祖先。如果父进程没有回收它的僵死子进程就终止了，那么内核会安排 init 进程去回收他们。 



一个进程可以通过调用 waitpid 函数来等待它的子进程终止或者停止。



28.execve 函数在当前进程的上下文中加载并运行一个新程序。 execve 加载并运行可执行目标文件，调用一次并从不返回。execve加载了filename之后，它调用启动代码，启动代码设置栈，并将控制传递给新程序的主函数。



29.程序和进程之间的区别：

程序是一堆代码合数据；程序可以作为目标文件存在于磁盘上，或者作为段存在于地址空间中。

进程是执行中程序的一个具体的实例；程序总是运行在某个进程的上下文中。



30.一种更高层的软件形式的异常，称为Linux信号，它允许进程和内核中断其他进程。



一个信号就是一条小消息，它通知进程系统发生了一个某种类型的事件。



每种信号类型都对应于某种系统事件。低层的硬件异常是由内核异常处理程序处理的，正常情况下，对用户进程而言是不可见的。信号提供了一种机制，通知用户进程发生了这些异常。

![](/images/2019/12/13.jpg) 



31.传送一个信号到目的进程是由两个不同步骤组成的：



①发送信号。内核通过更新目的进程上下文中的某个状态，发送一个信号给目的进程。发送信号可以有如下两种原因：内核检测到一个系统事件，比如除零错误或者子进程终止；一个进程调用了kill 函数，显式的要求内核发送一个信号给目的进程。一个进程可以发送信号给它自己。



②接收信号。当目的进程被内核强迫以某种方式对信号的发送做出反应时，它就接收了信号。进程可以忽略这个信号，终止或者通过执行一个称为信号处理程序的用户层函数捕获这个信号。



一个发出而没有被接收的信号叫做待处理信号。在任何时刻，一种类型至多只会有一个待处理信号。如果一个进程有一个类型为k 的待处理信号，那么任何接下来发送到这个进程的类型为k 的信号都不会排队等待，它们只是被简单的丢弃。



一个待处理信号最多只能被接收一次。



32.默认的，一个子进程和它的父进程同属于一个进程组。一个进程可以通过使用setpgid 函数来改变自己或者其他进程的进程组。



33.一个为负的PID会导致信号被发送到进程组PID中的每个进程，比如kill 函数。



进程通过调用kill函数发送信号给其他进程（包括它们自己）。kill（pid, sig）。



如果pid 大于零，那么kill函数发送信号号码sig 给进程pid。如果pid 等于零，那么kill 发送信号sig 给调用进程所在进程组中的每个进程，包括调用进程自己。如果pid 小于零，kill 发送信号sig 给进程组 |pid| （pid的绝对值）中的每个进程。



34.进程可以通过调用alarm 函数向它自己发送SIGALRM信号。在任何情况下，对alarm 的调用都将取消任何待处理的闹钟。



35.当内核把进程p从内核模式切换到用户模式时，它会检查进程p的未被阻塞的待处理信号的集合。如果集合是非空的，那么内核选择集合中的某个信号k（通常是最小的k），并且强制p接受信号k。



36.进程可以通过signal 函数修改和信号相关联的默认行为。唯一的例外就是SIGSTOP和SIGKILL，它们的默认行为是不能修改的。



37.可以用volatile 类型限定符来定义一个变量，告诉编译器不要缓存这个变量。例如 volatile int g；

volatile限定符强迫编译器每次在代码中引用g时，都要从内存中读取g的值。

C提供一种整形数据类型 sig_atomic_t，对它的读和写保证会是原子（不可中断）的。



38.因为pending位向量中每种类型的信号只对应有一位，所以每种类型最多只能有一个未处理的信号。



39.不可以用信号来对其他进程中发生的事件计数。



40.Posix标准定义了sigaction函数，它允许用户在设置信号处理时，明确指定他们想要的信号处理语义。



41.C语言提供了一种用户级异常控制流形式，称为非本地跳转，它将控制直接从一个函数转移到另一个当前正在执行的函数，而不需要经过正常的调用-返回序列。非本地跳转是通过segjmp 和longjmp 函数来提供的。



setjmp 函数在env 缓冲区中保存当前调用环境，以供后面的longjmp使用，调用环境包括程序计数器、栈指针和通用目的寄存器。setjmp返回的值不能被赋值给变量。



setjmp 函数只被调用一次，但返回多次；longjmp 函数被调用一次，但从不返回。



42.非本地跳转的一个重要应用就是允许从一个深层嵌套的函数调用中立即返回，通常是由检测到某个错误情况引起的。如果在一个深层嵌套的函数调用中发现一个错误情况，我们可以使用非本地跳转直接返回到一个普通的本地化的错误处理程序，而不是费力的解开调用栈。



longjmp 允许它跳过所有中间调用的特性可能产生意外的后果。例如，如果中间函数调用中分配了某些数据结构，本来预期在函数结尾处释放它们，那么这些释放代码会被跳过，因而会产生内存泄漏。



非本地跳转的另一个重要应用是使一个信号处理程序分支到一个特殊的代码位置，而不是返回到被信号到达中断了的指令的位置。



可以把try 语句中的catch 子句看做类似于setjmp 函数。相似的，throw 语句就类似于longjmp 函数。



43.STRACE：打印一个正在运行的程序和它的子进程调用的每个系统调用的轨迹。



PS：列出当前系统中的进程（包括僵死进程）。



TOP：打印出关于当前进程资源使用的信息。





## 第九章  虚拟内存



1.为了更加有效的管理内存并且少出错，现代系统提供了一种对主存的抽象概念，叫做虚拟内存（VM）。它为每个进程提供了一个大的、一致的和私有的地址空间。



虚拟内存提供了三个重要的能力：①它将主存看成是一个存储在磁盘上的地址空间的高速缓存，在主存中只保存活动区域，并根据需要在磁盘和主存之间来回传送数据，通过这种方式，它高效的使用了主存。②它为每个进程提供了一致的地址空间，从而简化了内存管理。③它保护了每个进程的地址空间不被其他进程破坏。



2.虚拟内存给予应用程序强大的能力，可以创建和销毁内存片，将内存片映射到磁盘文件的某个部分，以及与其他进程共享内存。可以加载一个文件的内容到内存中，而不需要进行任何显式的复制。



3.现代处理器使用的是一种称为虚拟寻址的寻址方式。



将一个虚拟地址转换为物理地址的任务叫做地址翻译，地址翻译需要CPU硬件和操作系统之间的紧密合作。CPU芯片上叫做内存管理单元（MMU）的专用硬件，利用存放在主存中的查询表来动态翻译虚拟地址，该表的内容由操作系统管理。



4.地址空间是一个非负整数地址的有序集合，如果地址空间中的证书是连续的，那么我们说它是一个线性地址空间。一个地址空间的大小是由表示最大地址所需要的位数来描述的。例如一个包含N=2^n个地址的虚拟地址空间就叫做一个n位地址空间。



5.概念上而言，虚拟内存被组织为一个由存放在磁盘上的N个连续的字节大小的单元组成的数组。磁盘上的数据被分割成块，这些块作为磁盘和主存之间的传输单元。VM系统通过将虚拟内存分割为称为虚拟页的大小固定的块来处理这个问题。每个虚拟页的大小为P=2^p字节，类似的，物理内存被分割为物理页，大小也为P字节（物理页也被称为页帧）。



6.因为大的不命中处罚和访问第一个字节的开销，虚拟页往往很大，通常是4KB-2MB。由于大的不命中处罚，DRAM缓存是全相联的，即任何虚拟页都可以放置在任何的物理页中。



与硬件对SRAM缓存相比，操作系统对DRAM缓存使用了更复杂精密的替换算法。



因为对磁盘的访问时间很长，DRAM缓存总是使用写回，而不是直写。



因为DRAM是全相联的，所以任意物理页都可以包含任意虚拟页。



7.页表将虚拟页映射到物理页。页表就是一个页表条目（PTE）的数组。虚拟地址空间中的每个页在页表中一个固定偏移量处都有一个PTE。



8.DRAM缓存不命中称为缺页。缺页异常调用内核中的缺页异常处理程序，该程序会选择一个牺牲页。



9.实际上，虚拟内存工作的相当好，这主要归功于我们的老朋友局部性。

局部性原则保证了在任意时刻，程序将趋向于在一个较小的活动页面集合上工作，这个集合叫做工作集或者常驻集合。在初始开销，也就是将工作集页面调度到内存中之后，接下来对这个工作集的引用将导致命中，而不会产生额外的磁盘流量。



如果工作集的大小超出了物理内存的大小，那么程序将产生一种不幸的状态，叫做抖动，这时页面将不断的换进换出。



多个虚拟页面可以映射到同一个共享物理页面上。



10.按需页面调度和独立的虚拟地址空间的结合，对系统中内存的使用和管理造成了深远的影响。特别的，VM简化了链接和加载、代码和数据共享，以及应用程序的内存分配。



简化链接。独立的地址空间允许每个进程的内存映像使用相同的基本格式，而不管代码和数据实际存放在物理内存的何处。这样的一致性极大的简化了链接器的设计和实现，允许链接器生成完全链接的可执行文件，这些可执行文件是独立于物理内存中代码和数据的最终位置的。



简化加载。加载器不从磁盘到内存实际复制任何数据。在每个页初次被引用时，要么是CPU取指令时引用的，要么是一条正在执行的指令引用一个内存位置时引用的，虚拟内存系统会按照需要自动的调入数据页。



将一组连续的虚拟页映射到任意一个文件中的任意位置的表示法称作内存映射。Linux提供一个称为mmap的系统调用，允许应用程序自己做内存映射。



简化共享。操作系统通过将不同进程中适当的虚拟页面映射到相同的物理页面，从而安排多个进程共享这部分代码的一个副本，而不是在每个进程中都包括单独的内核和C标准库的副本。



简化内存份额。当一个运行在用户进程中的程序要求额外的堆空间时，操作系统分配一个适当数字（例如k）个连续的虚拟内存页面，并且将它们映射到物理内存中任意位置的k个任意的物理页面。由于页表工作的方式，操作系统没有必要分配k个连续的物理内存页面。页面可以随机分散在物理内存中。



11.提供独立的地址空间使得区分不同进程的私有内存变得容易。但是，地址翻译机制可以以一种自然的方式扩展到提供更好的访问控制。因为每次CPU生成一个地址时，地址翻译硬件都会读一个PTE，所以通过在PTE上添加一些额外的许可位来控制对一个虚拟页面内容的访问十分简单。



如果一条指令违反了这些许可条件，那么CPU就触发一个一般保护故障，将控制传递给一个内核中的异常处理程序。Linux shell 一般将这种异常报告为 段错误 segment fault。



12.在任何既使用虚拟内存又使用SRAM高速缓存的系统中，都有应该使用虚拟地址还是使用物理地址来访问SRAM 高速缓存的问题。大多数系统是选择物理寻址的。使用物理寻址，多个进程同时在高速缓存中有存储块和共享来自相同虚拟页面的块成为很简单的事情。高速缓存无需处理保护问题，因为访问权限的检查是地址翻译过程的一部分。



13.内核虚拟内存包含内核中的代码和数据结构。有趣的是，Linux也将一组连续的虚拟页面（大小等于系统中DRAM的总量）映射到相应的一组连续的物理页面。这就为内核提供了一种便利的方法来访问物理内存中任何特定的位置。

![](/images/2019/12/14.jpg) 

PS：有兴趣可以了解下用户栈和内核的区别和联系，以及数量对比关系（一对一，一对多还是多对多？）。



14.内核为系统中的每个进程维护一个单独的任务结构（源代码中的task_struct）。任务结构中的元素包含或者指向内核运行该进程所需要的所有信息（例如，PID、指向用户栈的指针、可执行目标文件的名字，以及程序计数器）。



15.因为一个进程可以创建任意数量的新虚拟内存区域（使用mmap函数），所以顺序搜索区域结构的链表花销可能会很大。因为实际上，Linux在链表中构建了一棵树，并在这棵树上进行查找。



16.Linux通过将一个虚拟内存区域与一个磁盘上的对象关联起来，以初始化这个虚拟内存区域的内容，这个过程称为内存映射。



17.一旦一个虚拟页面被初始化了，它就在一个由内核维护的专门交换文件之间换来换去。交换文件也叫做交换空间或者交换区域。



18.一个对象可以被映射到虚拟内存的一个区域，要么作为共享对象，要么作为私有对象。



19.私有对象使用一种叫做写时复制的巧妙技术被映射到虚拟内存中。



20.关于execve函数加载和执行程序的过程见下图：

![](/images/2019/12/15.jpg) 



21.mmap(void* start, size_t length, int port, int flags, int fd, off_t offset)函数要求内核创建一个新的虚拟内存区域，最好是从地址start 开始的一个区域，并将文件描述符fd指定的对象的一个连续的偏映射到这个新的区域。连续的对象片大小为length字节，从距文件开始处偏移量为offset字节的地方开始。start地址仅仅是一个暗示，通常被定义为NULL。



munmap(void *start, size_t length) 函数删除从虚拟地址start 开始的，由接下来length 字节组成的区域。接下来对已删除区域的引用会导致段错误。



22.动态内存分配器维护着一个进程的虚拟内存区域，称为堆。对于每个进程，内核维护着一个变量brk，它指向堆的顶部。



23.显示分配器，要求应用显式的释放任何已分配的块。C++的new和delete操作符与C中的malloc和free相当。隐式分配器也叫做垃圾收集器，自动释放未使用的已分配的块的过程叫做垃圾收集。



24.malloc 不初始化它返回的内存。那些想要已初始化的动态内存的应用程序可以使用calloc，calloc 是一个基于malloc的瘦包装函数，它将分配的内存初始化为零。想要改变一个以前分配块的大小，可以使用realloc函数。



25.造成堆利用率很低的主要原因是一种称为碎片的现象，当虽然有未使用的内存但不能用来满足分配请求时，就发生这种现象。有两种形式的碎片：内部碎片和外部碎片。内部碎片是在一个已分配块比有效载荷大时发生的。外部碎片是当空闲内存合计起来足够满足一个分配请求，但是没有一个单独的空闲块足够大可以来处理这个请求时发生的。



因为外部碎片难以量化且不可能预测，所以分配器通常采用启发式策略来试图维持少量的大空闲块，而不是维持大量的小空闲块。



26.如果分配器不能为请求块找到合适的空闲块将发生什么呢？一个选择是通过合并那些在内存中物理上相邻的空闲块来创建一些更大的空闲块。如果还是不能生成一个足够大的块，那么分配器就会通过调用sbrk 函数，向内核请求额外的堆内存。



27.为了解决假碎片问题，任何实际的分配器都必须合并相邻的空闲块，这个过程称为合并。分配器可以选择立即合并，也就是在每次一个块被释放时，就合并所有的相邻块。或者它可以选择推迟合并，也就是等到某个稍晚的时候再合并空闲块。例如，分配器可以推迟合并，直到某个分配请求失败，然后扫描整个堆，合并所有的空闲块。



快速的分配器通常会选择某种形式的推迟合并。



28.C标准库中提供的GNU malloc 包就是采用的这种方法，因为这种方法既快速，对内存的使用也很有效率。



29.垃圾收集器是一种动态内存分配器，它自动释放程序不再需要的已分配块。在一个支持垃圾收集的系统中，应用显式分配堆块，但是从不显式的释放它们。



垃圾收集器将内存视为一张有向可达图。



30.像ML和Java这样的语言的垃圾收集器，对应用如何创建和使用指针有很严格的控制，能够维护可达图的一种精确的表示，因此也就能够回收所有垃圾。然而，诸如C和C++这样的语言的收集器通常不能维持可达图的精确表示。这样的收集器也叫做保守的垃圾收集器。



C程序的Mark & Sweep收集器必须是保守的，其根本原因是C语言不会用类型信息来标记内存位置。



31.收集器可以按需提供它们的服务，或者它们可以作为一个和应用并行的独立线程，不断的更新可达图和回收垃圾。



32.虽然bss内存位置（诸如未初始化的全局C变量）总是被加载器初始化为零，但是对于堆内存却并不是这样的。一个常见的错误就是假设堆内存被初始化为零。



33.虚拟内存是为主存的一个抽象。虚拟内存提供三个重要的功能。第一，它在主存中自动缓存最近使用的存放磁盘上的虚拟地址空间的内容。虚拟内存缓存中的块叫做页。第二，虚拟内存简化了内存管理，进而又简化了链接、在进程间共享数据、进程的内存分配以及程序加载。最后，虚拟内存通过在每条页表条目中加入保护位，从而简化了内存保护。



34.内存映射为共享数据、创建新的进程以及加载程序提供了一种高效的机制。应用可以使用mmap函数来手工的创建和删除虚拟地址空间的区域。





#### 第三部分  程序间的交互和通信



## 第十章  系统级I/O



1.输入/输出（I/O）是在主存和外部设备（例如磁盘驱动器、终端）之间复制数据的过程。



2.一个Linux文件就是一个m个字节的序列。所有的I/O设备（网络，磁盘和终端）都被模型化为文件，而所有的输入和输出都被当作对相应文件的读和写来执行。



3.打开文件时内核会返回一个小的非负整数，叫做描述符，它在后续对此文件的所有操作中标识这个文件。内核记录有关这个打开文件的所有信息。应用程序只需记住这个描述符。



Linux shell 创建的每个进程开始时都有三个打开的文件：标准输入（描述符为0）、标准输出（描述符为1）和标准错误（描述符为2）。



当应用完成了对文件的访问之后，它就通知内核关闭这个文件。作为响应，内核释放文件打开时创建的数据结构，并将这个描述符恢复到可用的描述符池中。无论一个进程因为何种原因终止时，内核都会关闭所有打开的文件并释放它们的内存资源。



4.文本文件是只含有ASCII或Unicode字符的普通文件，二进制文件是所有其他的文件。



5.目录是包含一组链接的文件，其中每个链接都将一个文件名映射到一个文件，这个文件可能是另一个目录。



套接字是用来与另一个进程进行跨网络通信的文件。



其他文件类型包含命名管道、符号链接，以及字符和块设备。



6.进程是通过调用open函数来打开一个已存在的文件或者创建一个新文件的。通过调用close函数关闭一个打开的文件。



关闭一个已关闭的描述符会出错。



通过调用lseek 函数，应用程序能够显示的修改当前文件的位置。



7.应用程序能够通过调用stat 和 fstat 函数，检索到关于文件的信息（有时也称为文件的元数据）。



8.内核用三个相关的数据结构来表示打开的文件：



描述符表。每个进程都有它独立的描述符表，它的表项是由进程打开的文件描述符来索引的。每个打开的描述符表项指向文件表中的一个表项。



文件表。打开文件的集合是由一张文件表来表示的，所有的进程共享这张表。每个文件表的表项组成包括当前的文件位置、引用计数，以及一个指向v-node表中对应表项的指针。关闭一个描述符会减少相应的文件表表项中的引用计数。内核不会删除这个文件表表项，直到它的引用计数为零。



v-node表。同文件表一样，所有的进程共享这张v-node表。每个表项包含stat结构中的大多数信息，包括st_mode和st_size 成员。

![](/images/2019/12/16.jpg) 



9.多个描述符也可以通过不同的文件表表项来引用同一个文件。例如，如果以同一个filename调用open函数两次，就会发生这种情况。

![](/images/2019/12/17.jpg) 



10.C语言定义了一组高级输入输出函数，称为标准I/O库。这个库libc 提供了打开和关闭文件的函数（fopen和fclose）、读和写字节的函数（fread和fwrite）、读和写字符串的函数（fgets和fputs），以及复杂的格式化的I/O函数（scanf和printf）。



标准I/O库将一个打开的文件模型化为一个流。每个ANSIC 程序开始时都有三个打开的流 stdin、stdout 和stderr ，分别对应于标准输入、标准输出和标准错误。



11.Unix I/O 模型是在操作系统内核中实现的。应用程序可以通过诸如open、close、lseek、read、write和stat这样的函数来访问Unix I/O。标准I/O函数提供了Unix I/O函数的一个更加完整的带缓冲的替代品。



12.对套接字使用lseek 函数是非法的。



13.建议在网络套接字上不要使用标准I/O 函数来进行输入和输出，而要使用健壮的RIO函数。如果你需要格式化的输出，使用sprintf函数在内存中格式化一个字符串，然后用 rio_writen把它发送到套接口。如果你需要格式化输入，使用rio_readlineb来读一个完整的文本行，然后用sscanf从文本行提取不同的字段。





## 第十一章  网络编程



1.客户端-服务器模型中的基本操作是事务。认识到客户端和服务器是进程，而不是常提到的机器或者主机，这是很重要的。



2.物理上而言，网络是一个按照地理远近组成的层次系统。最底层是LAN局域网，最流行的局域网是以太网。



在层次的更高级别中，多个不兼容的局域网可以通过叫做路由器的特殊计算机连接起来，组成一个Internet。每台路由器对于它所连接到的每个网络都有一个适配器（端口）。路由器也能连接高速点到点电话连接，这是称为WAN广域网的网络示例。



3.互联网络至关重要的特性是，它能由采用完全不同和不兼容技术的各种局域网和广域网组成。封装是互联网络的关键。



4.每台因特网主机都运行实现了TCP/IP协议的软件，几乎每个现代计算机系统都支持这个协议。因特网的客户端和服务器混合使用套接字接口函数和Unix I/O 函数来进行通信。通常将套接字函数实现为系统调用，这些系统调用会陷入内核，并调用各种内核模式的TCP/IP函数。



5.UDP 稍微扩展了IP协议，这样一来，包可以在进程间而不是在主机间传送。TCP 是一个构建在 IP 之上的复杂协议，提供了进程间可靠的全双工连接。



因特网主机上的进程能够通过连接和任何其他因特网主机上的进程通信。



6.TCP/IP 为任意整数数据项定义了统一的网络字节顺序（大端字节顺序）。在IP地址结构中存放的地址总是以（大端法）网络字节顺序存放的，即使主机字节顺序是小端法。



7.我们可以使用Linux的NSLOOKUP 程序来探究DNS 映射的一些属性。



8.因特网客户端和服务器通过在连接上发送和接收字节流来通信。从连接一对进程的意义上而言，连接是点对点的。从数据可以同时双向流动的角度来说，它是全双工的。



9.一个套接字是连接的一个端点。当客户端发起一个连接请求时，客户端套接字地址中的端口是由内核自动分配的，称为临时端口。然而，服务器套接字地址中的端口通常是某个知名端口，是和这个服务相对应的。例如，Web服务器通常使用端口80。



10.一个连接是由它两端的套接字地址唯一确定的。这对套接字地址叫做套接字对。

![](/images/2019/12/18.jpg) 

![](/images/2019/12/19.jpg) 



11.从Linux内核的角度来看，一个套接字就是通信的一个端点。从Linux程序的角度来看，套接字就是一个有相应描述符的打开文件。



12.客户端和服务器使用socket函数来创建一个套接字描述符。socket返回的clientfd描述符仅是部分打开的，还不能用于读写。客户端通过调用connect函数来建立和服务器的连接。connect函数会阻塞，一直到连接成功建立或是发生错误。



剩下的套接字函数——bind，listen和accept，服务器用它们来和客户端建立连接。



bind函数告诉内核将addr中的服务器套接字地址和套接字描述符sockfd联系起来。



服务器调用listen函数告诉内核，描述符是被服务器而不是客户端使用的。listen函数将socketfd从一个主动套接字转化为一个监听套接字。backlog 参数暗示了内核在开始拒绝连接请求之前，队列中要排队的未完成的连接请求的数量。通常我们会把它设置为一个较大的值，比如1024。



服务器通过调用accept函数来等待来自客户端的连接请求。



accept函数等待来自客户端的连接请求到达侦听描述符，然后在addr中填写客户端的套接字地址，并返回一个已连接描述符，这个描述符可被用来利用Unix I/O 函数与客户端通信。



监听描述符是作为客户端连接请求的一个端点。它通常被创建一次，并存在于服务器的整个生命周期。已连接描述符是客户端和服务器之间已经建立起来了的连接的一个端点。服务器每次接受连接请求时都会创建一次，它只存在于服务器为一个客户端服务的过程中。



在第一步中，服务器调用accept，等待连接请求到达监听描述符，具体的我们设定为描述符3。描述符0-2是预留给了标准文件的。



在第二步中，客户端调用connect函数，发送一个连接请求到listenfd。第三步，accept函数打开了一个新的已连接描述符connfd，在clientfd 和connfd 之间建立连接，并且随后返回connfd 给应用程序。



![](/images/2019/12/20.jpg) 



13.有监听描述符和已连接描述符之间的区别，可以使得我们可以建立并发服务器，它能够同时处理许多客户端连接。例如，每次一个连接请求到达监听描述符时，我们可以派生（fork）一个新的进程，它通过已连接描述符与客户端通信。



14.客户端关闭描述符（close），这会导致发送一个EOF通知到服务器，当服务器从它的reo_readlineb函数收到一个为零的返回码时，就会检测到这个结果。



15.其实并没有EOF字符这样的一个东西。进一步来说，EOF是由内核检测到的一种条件。应用程序在它接收到一个由read函数返回的零返回码时，它就会发现出EOF条件。对于磁盘文件，当前文件位置超出文件长度时，会发生EOF。对于因特网连接，当一个进程关闭连接它的那一端时，会发生EOF。连接另一端的进程在试图读取流中最后一个字节之后的字节时，会检测到EOF。



16.如果一个服务器写一个已经被客户端关闭了的连接，那么第一次这样的写会正常返回，但是第二次就会引起发送SIGPIPE 信号。





## 第十二章  并发编程





1.当一个应用正在等待来自慢速I/O设备的数据到达时，内核会运行其他进程，使CPU保持繁忙。每个应用都可以按照类似的方式，通过交替执行I/O请求和其他有用的工作来利用并发。



2.使用应用级并发的应用程序称为并发程序。现代操作系统提供了三种基本的构造并发程序的方法：



①进程。用这种方法，每个逻辑控制流都是一个进程，由内核来调度和维护。因为进程有独立的虚拟地址空间，想要和其他流通信，控制流必须使用某种显示的进程间通信（IPC）机制。



②I/O 多路复用。在这种形式的并发编程中，应用程序在一个进程的上下文中显示的调度它们自己的逻辑流。逻辑流被模型化为状态机，数据到达文件描述符后，主程序显式的从一个状态转换到另一个状态。



③线程。线程是运行在一个单一进程上下文中的逻辑流，由内核进行调度。你可以把线程看成是其他两种方式的混合体，像进程流一样由内核进行调度，而像I/O多路复用流一样共享同一个虚拟地址空间。



3.对于在父、子进程间共享状态信息，进程有一个非常清晰的模型：共享文件表，但是不共享用户地址空间。进程有独立的地址空间既是优点也是缺点。为了共享信息，它们必须使用显示的IPC机制。基于进程的设计的另一个缺点是，它们往往比较慢，因为进程控制和IPC的开销很高。



4.waitpid函数和信号是基本的IPC机制，它们允许进程发送小消息到同一主机上的其他进程。套接字接口是IPC的一种重要形式，它允许不同主机上的进程交换任意的字节流。



5.I/O多路复用技术的基本思想，就是使用select函数，要求内核挂起进程，只有在一个或多个I/O事件发生后，才将控制返回给应用程序。



6.现代高性能服务器（例如Node.js，nginx和Tornado）使用的都是基于I/O多路复用的事件驱动的编程方式，主要是因为相比于进程和线程的方式，它有明显的性能优势。



事件驱动设计的一个优点是，它比基于进程的设计给了程序员更多的对程序行为的控制。



一个基于I/O多路复用的事件驱动服务器是运行在单一进程上下文中的，因此每个逻辑流都能访问该进程的全部地址空间。这使得在流之间共享数据变得很容易。



事件驱动设计常常比基于进程的设计要高效得多，因为它们不需要进程上下文切换来调度新的流。



事件驱动设计一个明显的缺点就是编码复杂。修改事件驱动服务器来处理部分文本行不是一个简单的任务，但是基于进程的设计却能处理得很好，而且是自动处理的。基于事件的设计另一个重要的缺点是它们不能充分利用多核处理器。



7.线程就是运行在进程上下文中的逻辑流。线程由内核自动调度，每个线程都有它自己的线程上下文，包括一个唯一的整数线程ID、栈、程序计数器、通用目的寄存器和条件码。所有的运行在一个进程里的线程共享该进程的整个虚拟地址空间。



8.同进程一样，线程由内核自动调度，并且内核通过一个整数ID来识别线程。同基于I/O多路复用的流一样，多个线程运行在单一进程的上下文中，因此共享这个进程虚拟地址空间的所有内容，包括它的代码、数据、堆、共享库和打开的文件。



9.一个线程的上下文要比一个进程的上下文小得多，线程的上下文切换要比进程的上下文切换快得多。



和一个进程相关的线程组成一个对等（线程）池，独立于其他线程创建的线程。主线程和其他线程的区别仅在于它总是进程中第一个运行的线程。对等（线程）池概念的主要影响是，一个线程可以杀死它的任何对等线程，或者等待它的任意对等线程终止。另外，每个对等线程都能读写相同的共享数据。



10.Posix线程（Pthreads）是在C程序中处理线程的一个标准接口。



线程通过调用pthread_create函数来创建其他线程。



新线程可以通过调用pthread_self函数来获得它自己的线程ID。



通过调用pthread_exit函数，线程会显式的终止。如果主线程调用pthread_exit，它会等待所有其他对等线程终止，然后再终止主线程和整个进程。



某个对等线程调用Linux的exit函数，该函数终止进程以及所有与该进程相关的线程。



另一个对等线程通过以当前线程ID作为参数调用pthread_cancel函数来终止当前线程。



线程通过调用pthread_join函数等待其他线程终止。pthread_join函数会阻塞，直到其他线程终止。pthread_join函数只能等待一个指定的线程终止，没有办法让pthread_wait等待任意一个线程终止。



11.在任何一个时间点上，线程是可结合的或者是分离的。一个可结合的线程能够被其他线程收回和杀死。在被其他线程回收之前，它的内存资源（例如栈）是不释放的。相反，一个分离的线程是不能被其他线程回收或杀死的。它的内存资源在它终止时由系统自动释放。



12.在现实程序中，有很好的理由要使用分离的线程。例如，一个高性能Web服务器可能在每次收到Web浏览器的连接请求时都创建一个新的对等线程。在这种情况下，每个对等线程都应该在它开始处理请求之前分离它自身，这样就能在它终止后回收它的内存资源了。



13.一组并发线程运行在一个进程的上下文中。每个线程都有它自己独立的线程上下文，包括线程ID、栈、栈指针、程序计数器、条件码和通用目的寄存器值。每个线程和其他线程一起共享进程上下文的剩余部分。这包括整个用户虚拟地址空间，它是由只读文本（代码）、读/写数据、堆以及所有的共享库代码和数据区域组成的。线程也共享相同的打开文件的集合。



14.寄存器是从不共享的，而虚拟内存总是共享的。



15.各自独立的线程栈的内存模型不是那么整齐清楚的。这些栈被保存在虚拟地址空间的栈区域中，并且通常是被相应的线程独立的访问的。我们说通常而不是总是，是因为不同的线程栈是不对其他线程设防的。所以，如果一个线程以某种方式得到一个指向其他线程栈的指针，那么它就可以读写这个栈的任何部分。



16.以提供互斥为目的的二元信号量常常也称为互斥锁。



17.饥饿就是一个线程无限期的阻塞，无法进展。



18.并行程序是一个运行在多个处理器上的并发程序。因此，并行程序的集合是并发程序集合的真子集。



19.并行编程的一项重要教训：同步开销巨大，要尽可能避免。如果无可避免，必须要用尽可能多的有用计算弥补这个开销。



20.有一类重要的线程安全函数，叫做可重入函数，其特点在于它们具有这样一种属性：当它们被多个线程调用时，不会引用任何共享数据。

![](/images/2019/12/21.jpg) 