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
