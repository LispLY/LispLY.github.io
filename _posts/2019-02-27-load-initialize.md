---
layout: post
title: Objective-C 笔记  关于 +load 和 +initialize
---
- `+load` 如果不实现的话不会被调用，`initialize` 如果不实现的话会沿继承关系向上查找父类的实现。
- 两者都会被自动调用，不要手动调用。

- 在 `+load` 方法中使用其他类是不安全的，因为其他类有可能尚未被设置好。

- 如果某个类没有实现  `+load` ，那么系统不会调用它的父类中的 `+load` 方法。这个是因为系统并不是通过 `msg_send()` 这一套来调用 `+load` ，而是使用内部的 `getLoadMethod()` 查找。

- `+load` 会阻塞整个应用程序，没必要的时候尽量不要使用。（可用于调试，比如判断 category 是否已载入。）

- `method swizzling` 一般在 `+load` 中使用

- `initialize` 是由 runtime 调用的，会在程序首次使用该类前调用，只调用一次。是线程安全的。会继承父类的实现。

- `initialize` 不应该调用其他方法，只应该设置内部数据。（例如设置不能在编译器创建的全局常量。）

- 一个对比图：
![图片来源：https://blog.csdn.net/qq_31810357/article/details/70622276](http://upload-images.jianshu.io/upload_images/110844-fc6301e3b08fa26d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 




> 参考： 
>  [Effective Objective-C 2.0](https://book.douban.com/subject/25829244/)
> [Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
> [iOS中 性能优化之浅谈load与initialize - 韩俊强的博客](https://blog.csdn.net/qq_31810357/article/details/70622276)
