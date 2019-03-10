> 本文内容主要来自于[坂本一树 / 古本智彦：Objective-C高级编程](https://book.douban.com/subject/24720270/)


Block - 带有自动变量的匿名函数。

## Block 的实质

Block 实质上是一个 Objective-C 对象。和其它对象一样，Block 在内部是由一个结构体实现的。结构体中主要包含：
结构体成员 | 意义 
- |-
void *isa |  isa 指针 
int Flags |  标志位
void *FuncPtr |  函数指针，指向实现 Block 功能的 C 语言函数
unsigned long Block_size |  Block 的大小
...... | 捕获的变量

其中，void *FuncPtr 指针指向的函数参数中包含一个指向 Block 结构体自身的 self 指针。
- Block 的定义，就是定义一个表示这个 Block 的新的结构体。
- Block 的创建，就是调用这个结构体的构造函数。
- Block 的调用，就是以 Block 结构体为参数（之一），调用 Block 成员变量中的函数指针。

## 捕获自动变量
在捕获自动变量的情况下：
- Block 定义时，将捕获的变量加入结构体。
- Block 创建时，使用被捕获变量的值来初始化结构体（传值）。
- Block 调用时，Block 内部的 C 函数会通过 self 指针访问 Block 结构体成员中的变量（传值）。

## __block 说明符
#### Block 可以修改被捕获的变量吗？
正常情况下，Block 无法修改被捕获的原始的自动变量，只能修改保存在自己内部的那一份。
Block 当然可以访问并修改全局变量和静态全局变量。
Block 也可以访问并修改所属作用域中的静态变量。Block 在定义时，会将指向静态变量的指针存入结构体使用，这样就可以在静态变量的作用域外正常使用它，并且修改它的值。

#### __block 变量的内部实现
__block 说明符是为了支持 Block 语法而新增加的存储类说明符，用来指定可以被 Block 修改的自动变量。
带有 __block 说明符的变量，在内部被实现为一个结构体，由这个结构体持有变量的值。
捕获了 __block 变量的 Block 会把指向 __block变量（结构体）的指针放到自己的结构体中。

## Block 存储域
共有 3 种 Block：
- **_NSConcreteGlobalBlock**：建立在内存数据段的 Block。定义在全局作用域下的 Block 或者不捕获任何自动变量的 Block 会将结构体的 isa 指针指向 _NSConcreteGlobalBlock。因为 _NSConcreteGlobalBlock 内部不保存任何状态，不会变化，所以只需要一个实例。
- **_NSConcreteStackBlock**：捕获自动变量的 Block，建立在栈上。
- **_NSConcreteMallocBlock**：建立在堆上。

当超出作用域时，建立在栈上的 Block 会被自动销毁。为了避免销毁，可以使用 `copy`实例方法将 Block 复制到堆上。ARC 会自动管理堆上的 Block。
cocoa 框架中名字包含 *usingBlock:* 的方法，以及 GCD 中的 API，会自动将 Block 复制到堆上。其他情况下如果需要，应该手动复制 Block。（例如将栈上的 Block 加入到集合类中时。）

#### __block 变量存储域
__block 变量建立在栈上，当使用 __block 变量的 Block 被复制到堆上时，__block 变量也会自动复制到堆上。
当 __block 变量被复制到堆上后，会被 Block 所持有。ARC 会自动管理堆上的 __block 变量。

#### Block 被复制到堆上的时机
以下情况，Block 会被自动复制到堆上：
- Block 作为函数返回值时。
- 将 Block 赋值给类的成员变量，变量类型为 id __strong 或者是 Block 类型时。
- 如前所述，在 cocoa 框架中名字包含 *usingBlock:* 的方法，以及 GCD 中的 API 中传递 Block 时。

注意：栈上的 Block 不会持有变量。如果 Block 要超出作用域使用作为自动变量的对象，一定要使 `copy` 方法将 Block 复制到堆上，除非前述 Block 会被自动复制的情况。 （否则对象会被提前销毁。）






