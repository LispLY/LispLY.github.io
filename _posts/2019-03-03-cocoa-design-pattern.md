---
layout: post
title: Cocoa Touch 框架与设计模式
---

> Any customer can have a car painted any color that he wants so long as it is black. 
——**Henry Ford**, the founder of the Ford Motor Company

对于一个 iOS 开发者来说，学习设计模式的意义主要有两个。一是理解 Cocoa Touch 框架所使用的各种设计模式，以便更好地使用系统框架；二是学习各个设计模式的原理和实现方式，以便在自己的代码中应用设计模式，提高工作效率和代码质量，改善代码可读性。这篇短文主要关注前者，讲讲 Cocoa Touch 都应用了哪些设计模式。



## 抽象工厂模式 & 类簇
> **抽象工厂 Abstract Factory：**提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

抽象工厂可以将使用者与各种具体的实体类的细节进行解耦。
Cocoa 框架中大量使用的类簇模式 *Class Cluster* ，通过一个公共的抽象父类来封装私有的实体子类，是抽象工厂的一种形式。

#### 类簇的例子：
- NSString, NSData, NSDictionary, NSSet, NSArray 以及它们的可变形式。
- NSNumber 在内部被实现为 NSCFNumber、NSCFBoolean 等私有子类，而使用者通常不需要了解这些信息。

Cocoa 框架的类簇模式是抽象工厂的一种形式。抽象父类（例如 NSNumber ）声明了创建它的私有子类（例如 NSCFBoolean）的方法，根据不同的方法（例如 numberWithBool: ）来创建各种子类。

## 适配器模式 & 协议 protocol
> **适配器 Adapter：**将一个类的接口转换成用户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

适配器模式中的三种角色：
target：用户需要的接口，用户只能够和 target 交流。
adaptee：能实际实现功能的类。
adapter：用来将 target 和 adaptee 适配。

#### 协议的例子：UITableViewDelegate protocol
1. 在 UITableViewDelegate protocol 的使用中，**用户**是 UITableView，**target** 是 protocol，**adaptee** 是能够为 UITableView 提供具体功能的 UITableView。
2. UITableView 和其他对象通过 UITableViewDelegate protocol 进行交流，所以一个普通的 UIViewController 和 UITableView 无法进行顺畅的交流。
3. 运用适配器模式，可以创建一个 UIViewController 的子类 MYViewController，由这个子类来实现 UITableViewDelegate protocol。这样 MYViewController 既能够使用 UIViewController 的完整功能，又能够通过 protocol 和 UITableView 进行交流了。

### 责任链模式 & 响应链
> **责任链 Chain of Responsibility：**使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间发生耦合。此模式将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

一看到上面的定义你马上会想到， UIKit 中处理屏幕事件的响应链 *Responder Chain* 显然是责任链模式的一个典型实现。
响应链由 UIResponder 的各个子类或后代（UIApplication、UIWindow、UIView）的实例组成。一旦一个 UIView 的实例无法处理特定事件，它会将事件交给它的 next responder 继续处理。

响应链的介绍：[Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events?language=objc)

## 命令模式：NSInvocation 和 Target-Action 机制
> **命令：**将请求封装为一个对象，从而可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。

命令模式将操作的生成和执行拆分开。

#### NSInvocation
命令模式的典型例子。
NSInvocation 的实例可以封装一条 Objective-C 消息，包含目标对象、方法和方法参数。通过创建 NSInvocation 对象可以得到一个封装好的请求，可以通过修改对象的属性来修改请求的各个方面，此后可以通过 `- invoke` 方法执行，甚至反复执行；还可以将请求对象再次封装到 NSInvocationOperation 对象内，加入 NSOperationQueue 队列中执行、暂停和取消。

#### Target-Action 机制
命令模式的又一个例子，相比 NSInvocation 更轻量化。
给一个 UIControl 对象设置一个 Target-Action 不会导致任务的立即执行，而是当满足条件时执行。

## 组合模式 & 视图层级 View Hierarchy
> **组合 Composite**：将对象组合成树形结构以表示 **部分 - 整体** 的层次结构。组合使得用户对单个对象和组合对象的使用具有一致性。

组合模式便于统一处理组合结构中的所有对象。

#### 视图层级 View Hierarchy
Cocoa 的视图层级是组合模式的一个例子，每个视图既是一个显示内容的对象，又是一个可以容纳子视图的容器。操作一个视图时（比如移动或者删除），不需要了解视图层级的所有对象，只需要对最顶层的视图进行操作。

![图片来源：https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW6](http://upload-images.jianshu.io/upload_images/6448579-498127a512c5be99.gif?imageMogr2/auto-orient/strip)

## 装饰模式：delegate 和 category
> **装饰 decorator：**动态地给一个对象添加一些额外的职责。就扩展功能来说，装饰模式比生成子类更为灵活。

像子类化一样，装饰模式可以不用改变原有代码。它传达了这样一条设计原则：对于类，应该更多地扩展它们，而不是常常修改它们。
装饰模式的特点是：不改变原有类增加功能、和原有类共用一套接口。

delegate 和 category 都不是装饰模式的严格应用，但是体现了这个模式的部分思想。

## 外观模式 & UIImage
> **外观 Facade：**为系统中的一组接口提供一个统一的接口。外观定义一个高层接口，隐藏了子系统间复杂的通信和依赖关系，让子系统更易于使用。

UIImage 是应用外观模式的一个例子。UIImage 是对图片的一个高层的接口，而图片的具体实现格式被隐藏起来。UIImage 甚至没有提供直接获取底层原始图片数据的接口，但是用户可以通过  CGImage 或 CIImage 属性，或者UIImagePNGRepresentation() 和 UIImageJPEGRepresentation() 函数来获取合适的图片表示方式。

## 迭代器模式 & 枚举
> **迭代器 Iterator：**提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。

这个比较容易理解。Cocoa 框架中各个集合类提供的 `enumerateObjectsUsingBlock: ` 方法，以及 for-in 快速枚举都是迭代器的例子。

## 中介者模式 & ViewController
> **中介者 Mediator：**用一个对象来封装一系列对象的交互方式。中介者使各对象不需要显示地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

UITabController 和 UINavigationController 是中介者模式的两个比较轻量级的应用，它们把不同的 ViewController 的交互集中到自己身上，便于开发者应对大量复杂的交互。如果需要更进一步的解耦合，也可以选择自己来实现中介者模式。例如：[iOS 组件化方案探索](https://blog.cnbang.net/tech/3080/)。

## 备忘录模式： 归档和序列化

> **备忘录 Memento：**在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。

[官方文档：归档和序列化](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Archiving/Archiving.html#//apple_ref/doc/uid/10000047i)

## 观察者模式：NSNotification 和 KVO

> **观察者 Observer：**定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

使用场景：一个对象需要通知其他对象，但不知道有多少其他对象，或者其他对象都是什么。

#### NSNotification
Cocoa Touch 框架的 NSNotification 机制依照观察者模式实现了一种一对多的消息广播。在程序中，可以把对象加入某个通知的观察者列表。通知发布者会创建一条通知，并把它发布到通知中心。通知中心会通过消息传递把通知发送给观察者列表中的每个对象。
发布消息是一个同步的过程，如果要异步地发布消息，可以把消息加入消息队列中。
相对于委托（可以在事件发生前决定是否允许事件发生），通知的观察者只能在事件发生后得到通知，不能对事件造成影响。
Cocoa Touch 框架中的类大量地使用到了通知中心。

#### KVO（Key-Value Observing）
KVO 允许一个对象通知另一个对象自己特定属性发生变化，KVO 机制基于 NSKeyValueObserving 非正式协议。被观察的属性可以是简单值、一对一关系或者一对多关系。对于遵从 NSKeyValueObserving 的对象，KVO 的各种方法是由 Cocoa Touch 框架自动实现的。（当然也可以手动实现。）


NSNotification 和 KVO 比较而言，NSNotification 是中心化的，由通知中心统一发送消息，以事件为单位；KVO 则由被观测对象直接发送消息，通知内容绑定于被观测对象特定的 property。

## 代理模式 & NSProxy
> **代理 Proxy**：为其他对象提供一种代理（替代者或者占位符）以控制对这个对象的访问。

代理一般用于远程对象、创建成本高昂的对象，或者因安全原因不应该被直接访问的对象。客户端可以透明地使用代理。
代理模式和装饰模式有些相似，但两者目的不同，代理引入的功能是控制对原对象的访问。

#### NSProxy
NSProxy 是一个抽象类，实际上它和 NSObject 一样是个根类（root class）。NSProxy 定义了作为一个对象的代理所需要的接口。NSProxy 对象一般会把收到的消息直接转发给它代理的对象，通过子类化可以赋予它更多功能。

例子：[YYWeakProxy：使用弱代理解除 retain circle。](https://github.com/ibireme/YYKit/blob/master/YYKit/Utility/YYWeakProxy.h)

## 单例模式
> **单例 Singleton：**保证一个类仅有一个实例，并提供一个访问它的全局访问点。

不多解释了。

例子：UIApplication 等。
不严格的例子：NSFileManager、NSUserDefaults（可以自己新建实例，严格来讲不算是单例）。


## 模板方法模式

> **模板方法 Template Method：**定义一个操作中算法的骨架，而将一些步骤延迟到子类中。模板方法使子类可以重新定义算法的某些特定步骤而不改变该算法的结构。

父类一次性实现算法的不变部分，将可变的部分留给子类来实现。模板方法模式中的控制结构流程是反转的，父类的模板方法调用子类的操作，而不是子类调用父类的操作。

#### 应用
模板方法模式是 Cocoa Touch 乃至各种面向对象框架的基础。Cocoa Touch 框架中各个类的编程接口经常包含留给子类来覆盖的方法。典型的例子，UIView 的 `-drawRect：` 默认什么也不做，你可以覆盖它来自定义绘制。

其他例子：
例如 UIViewController 的：`prepareForSegue:sender: `。


















> **参考：** 
> [设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/1052241/) 
[Objective-C编程之道：iOS设计模式解析](https://book.douban.com/subject/6920082/)
[已过期文档： Cocoa Design Patterns](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW6)


