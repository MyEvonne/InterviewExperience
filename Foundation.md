# Foundation
## 1.NSTimer怎么处理内存泄漏
知道了这种隐藏的内存问题是什么，那我么接下来看一下为什么这种情况下会产生循环引用。

```objective-c
[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timeStart) userInfo:nil repeats:YES];
```

在这个方法中，如果repeats传入的参数是YES，那么NSTimer对象会保留target传入的对象，也就是强持有它，这种情形会保持到NSTimer对象失效为止。如果是一次性的定时器，不需要手动去invalidate就会失效，但是重复性的定时器，需要主动去调用invalidate方法才会失效。
综上看来，viewController强持有了NSTimer对象，而NSTimer对象在其``[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timeStart) userInfo:nil repeats:YES]``;方法中对传入的target参数也进行了强引用，这也保证了NSTimer对象调用时的正确性。此时如果传入target的是self，那么就会形成viewController和NSTimer对象的循环引用环，如果循环引用环没有被打破，那么viewController在弹栈的时候就不会被释放，也就不会走dealloc方法，也就不会将NSTimer对象invalidate，这个时候viewController和NSTimer对象都得不到释放，也就产生了内存泄漏。  
https://www.dazhuanlan.com/2019/10/02/5d94062bacb0e/  
解决方法：
https://juejin.im/post/6844903622031966222

### NSTimer与runloop的关系
NSTimer必须添加到一个runloop里面才能运行，不然会无效。
https://www.jianshu.com/p/f9999b5958f8

### NSTimer与CADisplayLink以及GCD Timer的对比
* NSTimer依赖于runloop，当runloop在执行一个比价费时的任务时，NSTimer会不准。
* CADisplayLink是根据屏幕刷新频率来的，所以出发间隔设置比较死板，并且在每次处罚时不能执行比较费时的任务，一般用来处理一些界面重绘的任务。
* GCD Timer不依赖runloop，比较精确，就是代码比较不美观。
https://www.jianshu.com/p/b90754c033fc

### 如何检测系统卡顿
1. FPS,使用CADisplayLink。
2. runloop 在runloop上添加observer，然后监听kCFRunLoopBeforeSources跟kCFRunLoopBeforeWaiting两个状态之间的时间。

https://juejin.cn/post/6844903944867545096

https://blog.csdn.net/u014795020/article/details/72084735?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

https://blog.ibireme.com/2015/05/18/runloop/

## 2.delegate在什么情况下会出现内存泄漏，怎么解决？
在delegate被声明为strong的时候，会形成循环引用。
https://www.jianshu.com/p/1d2f0cb0c2bf  
https://www.jianshu.com/p/b2f5e0429712

## 3.delegate和Notification的区别。
delegate是一对一，notification可以看做是一种广播。
delegate在调用后，receiver还可以返回高速delegate结果，而notification则不管结果。

https://www.jianshu.com/p/dfb86899ea44

## 4.iOS中有哪些多线程技术
pThread,GCD,NSOperation,NSThread
NSOperation是GCD的封装。
https://juejin.cn/post/6844903951431647246
https://www.jianshu.com/p/a110d5038a2d
https://blog.jerrychu.top/2018/03/25/MainQueue/
https://www.jianshu.com/p/d6c94f773b34

### 主队列和主线程的关系，全局并发队列一定在主线程上运行么？
主队列的任务一般都是在主线程上运行，除非主线程阻塞，那么系统会将任务调配到其他线程进行。

While doing some research for this post I found a commit to libdispatch that ensures that blocks dispatched with dispatch_sync are always executed on the current thread. This means if you use dispatch_sync to dispatch a block from the main queue to a concurrent background queue, the code executing on the background queue will actually be executed on the main thread. While this might not be entirely intuitive, it makes sense: since the main queue needs to wait until the dispatched block completed, the main thread will be available to process blocks from queues other than the main queue.

https://www.jianshu.com/p/d6c94f773b34

## 5.如果有两个同步任务嵌套会怎样？常见的锁，为什么要加锁？C依赖AB任务执行完成才执行，你怎么设计？
1. 程序会卡住  

2. @synchronized  
NSLock 对象锁  
NSRecursiveLock 递归锁  
NSConditionLock 条件锁  
pthread_mutex 互斥锁（C语言）  
dispatch_semaphore 信号量实现加锁（GCD）  
OSSpinLock （暂不建议使用，原因参见这里）

作者：怪小喵
链接：https://www.jianshu.com/p/1e59f0970bf5
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

3. 为了防止两个不同的任务同时操作一个数据，造成错误。
4. 可以使用dispatch_group来执行AB任务，AB任务都完成后通过回调执行C任务。  
或者可以使用dispatch_semaphore来通知C任务是否AB任务都完成了。

## 6.读写锁底层怎么实现
https://juejin.im/post/6844903902530240526

## 7.GCD中的常见操作
dispatch_async  
dispatch_sync  
dispatch_group dispatch_group_wait  
dispatch_barrier_async  
dispath_after  
dispatch_semaphore_t  
>Dispatch semaphore是持有计数的信号，该计数是多线程编程中的计数类型信号。所谓信号，类似于过马路时常用的手旗，可以通过时举起手旗，不可通过时放下手旗。而在Dispatch semaphore，使用计数来实现该功能。计数为0时等待，计数为1或大于1时，减去1而不等待。
通过``dispatch_semaphore_create(<#long value#>)``函数生成Dispatch Semaphore。参数表示计数的初始值，

dispatch_source

作者：zziazm
链接：https://www.jianshu.com/p/435122351078
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 8.动态库与静态库的区别
动态库形式：.dylib和.framework

静态库形式：.a和.framework

静态库：链接时，静态库会被完整地复制到可执行文件中，被多次使用就有多份冗余拷贝（图1所示）

系统动态库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存（图2所示）

作者：齐滇大圣
链接：https://www.jianshu.com/p/42891fb90304
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 9.iOS中线程间通信
Mach Port
https://juejin.cn/post/6844904152082939917

## 10.iOS中检测方法执行时间
CFAbsoluteTimeGetCurrent
time profiler

## 11.Autoreleasepool的底层实现机制
* 自动释放池是一个个 AutoreleasePoolPage 组成的一个page是4096字节大小,每个 AutoreleasePoolPage 以双向链表连接起来形成一个自动释放池
* 当对象调用 autorelease 方法时，会将对象加入 AutoreleasePoolPage 的栈中
* pop 时是传入边界对象,然后对page 中的对象发送release 的消息

作者：jackyshan_
链接：https://juejin.cn/post/6844903609428115470
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
