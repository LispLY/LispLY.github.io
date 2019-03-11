---
layout: post
title: Objective-C 引用计数的原理和内部实现
---

## 背景 - 内存管理
Objective-C 建立在 C 语言的基础上。C 语言程序的内存布局主要包括：
- 栈：由编译器自动分配释放，存放函数的参数值、局部变量的值等。
- 堆：通常存放程序运行中动态分配的存储空间。C语言中，使用malloc等函数分配的内存是从堆中分配的，在Objective-C中的对象也是从堆中分配内存的。
- BSS段：存放程序中未初始化的全局变量和静态变量。
- 数据段：存放程序中已初始化的全局变量和静态变量以及字符串常量。
- 代码段：存放程序执行代码。

如上所述，Objective-C 中的对象建立在堆上。Objective-C 使用引用计数管理内存，引用计数解决的是建立在堆上的 Objective-C 对象的内存管理问题。

## 什么是引用计数
>**引用计数**是计算机[编程语言](https://zh.wikipedia.org/wiki/%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80 "编程语言")中的一种**[内存管理](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86 "内存管理")技术**，是指将资源（可以是对象、内存或磁盘空间等等）的被引用次数保存起来，当被引用次数变为零时就将其释放的过程。使用引用计数技术可以实现自动资源管理的目的。同时引用计数还可以指使用引用计数技术回收未使用资源的[垃圾回收](https://zh.wikipedia.org/wiki/%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6 "垃圾回收")算法。
当创建一个对象的实例并在堆上申请内存时，对象的引用计数就为1，在其他对象中需要持有这个对象时，就需要把该对象的引用计数加1，需要释放一个对象时，就将该对象的引用计数减1，直至对象的引用计数为0，对象的内存会被立刻释放。
—— [维基百科](https://zh.wikipedia.org/wiki/%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0)

## 引用计数的原理
引用计数要解决的问题是何时释放对象，目标是立即释放不被使用的对象。
要避免的问题是：
- 不被使用的对象没有被释放（[内存泄漏](https://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)）。
- 错误地释放了还在使用中的对象（[悬垂指针](https://en.wikipedia.org/wiki/Dangling_pointer)）。

如果没有引用计数，用户需要自己控制销毁对象的时机。使用引用计数后，用户只要在需要一个对象时对其持有（**retain**， 引用数 +1），不需要时对其释放（**release**， 引用数 -1）。当一个对象已经没有人持有时（引用数为 0），这个对象会被自动销毁。

在引用计数的语境下，内存管理被抽象为四项简单的工作：
- 生成对象 `// 如 [Obj alloc] 或 [obj copy] 等。` 
- 持有对象 `// [obj retain];`
- 释放对象 `// [obj release];`
- 销毁对象 `// [obj dealloc];`

前三者由用户负责，第四项销毁对象由引用计数自动完成。

例子：
```
id obj = [[NSObject alloc] init];  // 引用计数 = 1
id ref = obj; // 引用计数 = 1
[ref retain]; // 引用计数 = 2
...... // some other code
[obj release]; // 引用计数 = 1
...... // some other code
[ref release]; // 引用计数 = 0，自动销毁
```
需要注意的一点：对于以 alloc、new、copy 和 mutableCopy 开头的方法，会自动持有创建的对象（相当于在创建对象语句的后面自动插入了一句 `[obj retain];`。如上面的例子，使用 `alloc` 创建对象后，不需要写 `[obj retain]`，引用计数就已经 +1 了。

其他情况如下面的例子：
```
NSArray *array = [NSArray array];  // 引用计数 = 0
[array retain]; // 引用计数 = 1
...... // some other code
[array release]; // 引用计数 = 0，自动销毁
```

## 引用计数的内部实现
系统在运行时维护一张散列表（引用计数表）来管理引用计数。散列表的键为各个对象内存地址的散列值，值为该对象的引用计数。
对应前面的生成对象、持有对象、释放对象、销毁对象四项工作的内部行为：
- 生成对象：在散列表中创建新的一行，记录新对象的引用计数。
- 持有对象：查找对象对应的行，该行的值 + 1。
- 释放对象：查找对象对应的行，该行的值 - 1。如果该行的值已经为 1，则销毁对象。
- 销毁对象：销毁对象。

## autorelease
autorelese （自动释放）的逻辑类似于 C 语言的自动变量，即超出作用域之后自动释放。
用法如下：
```
NSAutoreleasePool *pool =  [[NSAutoreleasePool alloc] init]; // 相当于自动变量作用域开始。
id obj = [[NSObject alloc] init];  // 引用计数 = 1；
[obj autorelease]; // 此时引用计数 = 1；
[pool drain]; // 相当于自动变量作用域结束，引用计数 = 0。
```
比较 release 和 autorelease：
release 会立即使引用计数 -1；autorelease 会将对象注册在 autoreleasePool 中，当 autoreleasePool 排空时引用计数 -1。


## autorelease 的内部实现

前面的例子中，`[obj autorelease]` 会调用 `[NSAutoreleasePool addObject: obj]`。autoreleasePool 会通过类似数组或链表的数据结构持有 `obj` 。当 autoreleasePool 被排空时，`obj`被释放。

前面将 autorelease 类比为自动变量。同样，autoreleasePool 的组织形式也可以类比为函数调用栈的形式。系统内部由名为 AutorelesePoolPage 的类来维护 autoreleasePool 的栈。当新建一个 autoreleasePool 时，相当于 push；排空一个 autoreleasePool 时，会进行 pop。








>  **参考**  
>[官方文档：Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011-SW1)  
>[坂本一树 / 古本智彦：Objective-C高级编程](https://book.douban.com/subject/24720270/)


