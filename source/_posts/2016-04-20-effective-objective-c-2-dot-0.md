---
layout: post
title: "《Effective Objective-C 2.0》"
date: 2016-04-20 23:14:27 +0800
comments: true
categories: 读书笔记 iOS开发
keywords: Objective-C iOS开发 Effective Objective-C 2.0 
description: 
---

本书是iOS开发进阶的必读书籍之一。文中部分名词的中文翻译略坑，比如对block和GCD的翻译。其他整体还好，原作者写的比较用心。代码规范讲了不少，底层原理讲了一点点，且主要集中在第二章。另第六章对GCD的讲解还算不错。作者原文写了52条编码建议，不过本人在整理读书笔记时并未按照原来的条数来做区分，只是把自己认为比较重要的做了标记，并记录了下来。


<!--more-->

###第一章
1.对于NSString *someString = @“the string”;
someString是一个分配在栈上的指针，@“the string”是分配在堆上的内存块。Objective-C的对象所占内存总是分配在堆上。
一个指针在32位架构的手机上占4个字节，在64位架构的手机上占8个字节。

2.在类的头文件中尽量少引入其他类的头文件，可以使用@class进行前向声明。协议如果在多个类里使用，则最好单独放在一个文件。目的都是为了解耦。

3.用字面量语法创建数组时，若数组元素中有对象为nil则会抛出异常，反而有助于排查bug，程序更安全。字典中的键和值都必须是Objective-C对象。使用字面量创建出来的字符串、数组、字典都是不可变的。

4.多用类型常量，少用#define预处理指令。且变量一定要同时用static和const声明，避免同名冲突的同时还可避免被无意修改。

5.凡是需要以按位或操作来组合的枚举都应使用NS_OPTIONS定义，若是枚举不需要互相结合，则应使用NS_ENUM来定义。如果用枚举来定义状态机，比如switch-case，则最好不要有default分支，避免加一个枚举值时漏掉状态机，若没有default则编译器会给出警告。

###第二章

1.属性的getter和setter方法是由编译器在编译期合成的。@dynamic关键字告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法。

2.atomic并不能保证线程安全，若要实现线程安全的操作，还需要采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为atomic，也还是会读到不同的属性值。——用dispatch_queue可以解决

3.在初始化方法和dealloc方法中，总是应该直接通过实例变量来读写数据。

——具体原因见[这里](http://www.jianshu.com/p/c684803fcf08)

4.关于NSObject的isEqual:方法和hash方法，NSObject类对这两个方法的默认实现是：当且仅当其指针值完全相等时，这两个对象才相等。如果“isEqual:”方法判定两个对象相等，那么其hash方法也必须返回同一个值。但是，如果两个对象的hash方法返回同一个值，那么“isEqual:”方法未必会认为两者相等。

5.当对象接收到无法解读的消息后，就会启动消息转发机制。整个的流程为：动态方法解析-》备援接收者-》完整的消息转发。
resolveInstanceMethod-》forwardingTargetForSelector-》forwardInvocation-》消息未能处理。

6.“isMemberOfClass:”能够判断出对象是否为某个特定类的实例，而“isKindOfClass:”则能够判断出对象是否为某类或其派生类的实例。

——两个方法的具体实现细节见[这里](http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective%5Bnil%5Dc-runtime-(2)%5Bnil%5D-object-and-class-and-meta-class/)

###第三章

1.用前缀避免命名空间冲突，尤其是要打包成库给第三方使用的代码。——可参考SDWebImage这个库，所有方法均添加sd_前缀。

2.description和debugDescription方法在NSLog打印数据时会调用类的这两个方法，如果想打印时输出更多细节，可自己实现这两个方法。

3.应该尽量把对外公布出来的属性设为只读，而且只在确有必要时才将属性对外公布。当然即使变量声明为只读，也可以通过KVC或计算指针偏移量后更改指针值的方式来改变变量。

4.方法名里不要使用缩略后的类型名称，给方法起名时的第一要务就是确保其风格与你自己的代码或所要集成的框架相符。

####5.在ARC下使用try-catch捕获异常要注意内存泄露的问题，打开-fobjc-arc-exceptions编译器标志可以确保编译器添加异常安全的代码，以避免内存泄露。

6.无论当前实例是否可变，若需获取其可变版本的拷贝，均应该调用mutableCopy方法。同理，若需要不可变的拷贝，则总应通过copy方法来获取。

7.Foundation框架中的所有collection类在默认情况下都执行浅拷贝，也就是说，只拷贝容器对象本身，而不复制其中的数据。

###第四章

1.将类的实现代码分散到便于管理的多个类别中去。——这一点在一个类功能较为复杂时常用，把某个功能点抽出来作为一个类别方便管理，也方便阅读。

2.为第三方类的类别名称加前缀。将类别方法加入类中这一操作是在运行期系统加载类别时完成的。如果类中本来就有该方法的实现，而类别又实现了一次，那么类别中的方法会覆盖原来类中的那一份代码实现。如果多个类别都实现了同一个方法，多次覆盖的结果以最后一个加载的类别为准。

3.一般说来，类别无法把实现属性所需的实例变量合成出来，但是使用关联对象可以解决这个问题，不过除非不得已，尽量避免这样设计。

4.使用类扩展是隐藏类的实现细节，包括一些方法和属性，实例变量，能够不暴露给外界的尽量隐藏在类扩展里。

5.Objective-C动态消息系统的工作方式决定了其不可能实现真正的私有方法或私有实例变量。因为在运行期总可以调用某些方法绕过此限制（比如KVC）。不过从一般意义上来说，它们还是私有的。

6.我们通常不直接访问实例变量，而是通过设置访问方法来做，因为这样能够触发KVO通知。

7.若观察者正在读取属性值而内部又在写入该属性时，则有可能引发竞争条件，合理使用同步机制能缓解此问题（dispatch_queue）。

###第五章

1.ARC几乎把所有内存管理事宜都交给编译器来决定，内存管理的代码也是编译的时候由编译器在合适的地方插入retain和release等操作。

2.使用ARC时一定要记住，引用计数实际上还是要执行的，只不过保留与释放操作现在是由ARC自动为你添加。ARC在调用release、retain这些方法时，并不通过普通的Objective-C消息派发机制，而是直接调用其底层C语言版本。

3.__autoreleasing符号表示把对象“按引用传递”给方法时，使用这个特殊的修饰符。此值在方法返回时自动释放。

4.使用ARC则不必要再编写dealloc方法了（仅指Objective-C对象，CoreFoundation对象还是需要自己管理内存的），ARC会借助C++对象的析构函数在.cxx_destruct方法中生成清理内存所需的代码。

5.ARC只负责管理Objective-C对象的内存，CoreFoundation等非Objective-C对象（比如CoreText，CoreGraphics）的内存需要自己管理。

6.在dealloc方法中移除通知以及KVO观察。还需注意，不要在dealloc里随意调用其他方法，包括getter和setter。

7.自动释放池可以嵌套使用，嵌套的好处在于可以控制程序的内存峰值。

8.僵尸对象（Zombie Object）是调试内存管理问题的最佳方式。将NSZombieEnabled环境变量设置为YES即可开启此功能。——通过选择edit scheme->run->Diagnostics，然后勾选Enable Zombie Objects即可。

9.单例对象的引用计数值不会改变，这种对象的retain和release操作都是空操作。尽量避免依赖对象的引用计数值来编码，因为其不可靠（autorelease）。

###第六章

1.在block的内存布局中，最重要的就是invoke变量，这是个函数指针，指向block的实现代码。GCD是纯C语言的API。

2.block会把它所捕获的所有变量都拷贝一份。不过拷贝的并不是对象本身，而是指向这些对象的指针变量。

3.block一旦复制到堆上，就成了带引用计数的对象了。后续的复制操作都不会真的执行复制，而只是递增block对象的引用计数。

4.全局的block（NSConcreteGlobalBlock）的拷贝操作是个空操作，因为全局block不可能为系统所回收。这种block实际上相当于单例。

5.对于通知来说，若没有指定队列，则按默认方式执行，也就是说，将由发出通知的那个线程来执行。

6.尽量多的使用dispatch_queue，少用同步锁，包括@synchronized和NSLock。

7.dispatch_barrier_async称之为栅栏方法，栅栏block必须单独执行，不能与其他block并行执行。这支队并发队列有意义，因为串行队列的块总是按顺序逐个来执行的。并发队列如果发现接下来要处理的块是个栅栏块，那么就一直要等当前所有并发块都执行完毕，才会单独执行这个栅栏块。

8.dispatch_group机制可以把任务分组，等组内任务全部完成后再做其他操作。如果是网络请求，比如需要等待好几个网络请求回调之后才做刷新UI的操作，可以使用dispatch_group_notify函数，需要注意的是，dispatch_group_notify的block里的代码的执行时机是等group里的所有任务都完成之后才执行，但是网络请求又是异步的，GCD默认网络请求发出后该任务即执行完成，所以如果是需要等网络回调之后才算任务完成的话，可以使用dispatch_group_enter和dispatch_group_leave来对分组里要执行的任务数进行递增和递减操作（在网络发出之前递增，在网络回调后递减）。

9.不要使用dispatch_get_current_queue，因为容易出现死锁。

###第七章

1.Cocoa本身并不是框架，但是里面集成了一批创建应用程序时经常会用到的框架。

2.iOS8以后应用程序可以使用动态库，也就是说，如果你的app仅支持iOS8及以上系统，可以添加自己的动态库。——注：书里说iOS程序不能包含动态库，这一规则已经在2014年WWDC过时。

3.关于桥接技术。__bridge本身的意思是：ARC仍然具备这个Objective-C对象的所有权。而__bridge_retained则与之相反，意味着ARC将交出对象的所有权。与此相似，反向转换可以通过__bridge_transfer来实现。

4.构建缓存时选用NSCache而不是NSDictionary。NSCache胜过NSDictionary之处在于，当系统资源将要耗尽时，它可以自动删减缓存。另外NSCache是线程安全的，多个线程可以同时访问NSCache。此外，它与字典不同，并不会拷贝键。

5.对于类的+load方法，必定会调用而且只调用一次。如果类别和类里都定义了load方法，则先调用类里的，再调用类别里的。

6.因为无法判断出各个类的加载顺序，所以在load方法里使用其他类是不安全的，因为很可能此时其他类还未加载。

7.如果某个类本身没有实现load方法，那么不管其各级父类是否实现了此方法，系统都不会调用。此外，类别和其所属的类里，都可能出现load方法。此时两种实现代码都会调用，类的实现要比类别的实现先执行。

8.load方法务必实现的精简一些，因为整个应用程序在执行load方法时都会阻塞。

9.+initialize方法：对每个类来说，该方法会在程序首次使用该类之前调用，且只调用一次。

10.+initialize方法只应该用来设置内部数据。不应该在其中调用其他方法，即便是本类自己的方法，也最好别调用。

11.只有把NSTimer放在NSRunLoop里，它才能正常出发任务。

12.NSTimer会保留其目标对象，直到自身失效时再释放此对象。调用invalidate方法可令计时器失效。