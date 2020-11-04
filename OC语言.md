# Objective-C语言相关
## 1.weak的实现原理
1、weak的原理在于底层维护了一张weak_table_t结构的hash表，key是所指对象的地址，value是weak指针的地址数组。
2、weak 关键字的作用是弱引用，所引用对象的计数器不会加1，并在引用对象被释放的时候自动被设置为 nil。
3、对象释放时，调用clearDeallocating函数根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。
4、文章中介绍了SideTable、weak_table_t、weak_entry_t这样三个结构，它们之间的关系如下链接所示。



作者：夜幕降临耶
链接：https://juejin.im/post/6844904101839372295
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
## 2.weak的弱引用表是可变还是不可变的
可变的

## 3.weak是在什么时候置为nil，如果同时有很多对象性能影响大怎么办？
对象释放时，调用clearDeallocating函数根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。  
性能考虑。使用 weak 对性能有一些影响，因此对性能要求高的地方可以考虑使用 unsafe_unretained 替换 weak。一个例子是 YYModel 的实现，为了追求更高的性能，其中大量使用 unsafe_unretained 作为变量标识符。  
https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/MM.html

## 4.struct中成员变量占多少字节

## 5.一个OC对象在iOS中所占的内存字节数
* OC对象 最少占用 16 个字节内存 .
* 当对象中包含属性, 会按属性占用内存开辟空间. 在结构体内存分配原则下自动偏移和补齐 .
* 对象最终满足 16 字节对齐标准 .
* 属性最终满足 8 字节对齐标准 .
* 可以通过 #pragma pack() 自定义对齐方式 .

作者：李斌同学
链接：https://juejin.im/post/6844903939985391629
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 6.dealloc的整个过程
https://www.jianshu.com/p/1be2526f5879

## 7.+load与initialize的区别
### +load，已经调用时机。
* OC文件在编译的时候，类相关数据结构已经被加载到目标文件夹中，在程序启动的时候，首先会调用load方法将这些类全都加载到内存中。  
* 不管这些类在代码中是否有被调用甚至引用，这些类的load方法都会被调用。  
* load方法的调用不是消息传递，是直接内存地址的调用，所以不管是父类还是子类，每个被实现了的load方法都会被调用一次。  
* load方法的调用顺序是哪个OC先编译先调用，然后是category，所以可以再compile source里面去修改编译顺序。

### initialize
* initialize是在这个类第一次收到消息的时候调用，而且一个类只会调用一次这个方法。
* initialize方法也是走消息传递的方式进行调用。

https://juejin.im/post/6844904040703197191

## 8.category的实现原理。
category有一个结构体：
```
// 定义在objc-runtime-new.h文件中
struct category_t {
    const char *name; // 比如给Student添加分类，name就是Student的类名
    classref_t cls;
    struct method_list_t *instanceMethods; // 分类的实例方法列表
    struct method_list_t *classMethods; // 分类的类方法列表
    struct protocol_list_t *protocols; // 分类的协议列表
    struct property_list_t *instanceProperties; // 分类的实例属性列表
    struct property_list_t *_classProperties; // 分类的类属性列表
};

作者：一叶知秋0830
链接：https://juejin.im/post/6844904039671398407
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
runtime阶段，这些方法都会并入这个类的methodlist里面，先编译的会被调用到。

## 9.category与extension的区别。
1. 类别中原则上只能增加方法（能添加属性的的原因只是通过runtime解决无setter/getter的问题而已）；
2. 类扩展不仅可以增加方法，还可以增加实例变量（或者属性），只是该实例变量默认是@private类型的（
用范围只能在自身类，而不是子类或其他地方）；
3. 类扩展中声明的方法没被实现，编译器会报警，但是类别中的方法没被实现编译器是不会有任何警告的。这是因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中。
4. 类扩展不能像类别那样拥有独立的实现部分（@implementation部分），也就是说，类扩展所声明的方法必须依托对应类的实现部分来实现。
5. 定义在 .m 文件中的类扩展方法为私有的，定义在 .h 文件（头文件）中的类扩展方法为公有的。类扩展是在 .m 文件中声明私有方法的非常好的方式。

作者：CoderDancer
链接：https://www.jianshu.com/p/9e827a1708c6
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
