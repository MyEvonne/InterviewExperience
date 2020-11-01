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