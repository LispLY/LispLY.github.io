---
title: "如何对类簇进行子类化"
date: 2019-01-10
comments: true
---

>本文的内容主要来源于 [Friday Q&A 2010-03-12: Subclassing Class Clusters](https://www.mikeash.com/pyblog/friday-qa-2010-03-12-subclassing-class-clusters.html) by [Mike Ash](https://www.mikeash.com/)，部分内容有增补和调整。


## 抽象类 - Abstract Class
要给一个类簇创建的子类，需要先知道类簇是什么。而要理解类簇，需要首先理解抽象类（Abstract Class）的概念。

抽象类是一个没有实现全部功能的类，它需要被子类化，由子类实现抽象类缺少的功能。抽象类不一定是个彻底的空壳，它依然可以包含很多功能，但只有在子类实现了抽象类没有实现的功能后，这个类才是完整的。

## 类簇 - Class Cluster
类簇（Class Cluster）是一个继承自公共抽象（基）类的层次结构。这个公共类提供了统一的接口和许多辅助功能，而核心功能由私有的子类来实现。公共类提供的实例化方法会返回私有子类的实例，使用户可以在不需要了解这些子类的情况下使用公共类。

例如，`NSArray`就是一个抽象类，需要其子类提供`count`和`objectAtIndex:`这两个方法的实现。抽象类`NSArray`提供了各种构建在这两者之上的方法，例如`indexOfObject:` ，`objectEnumerator` ， `makeObjectsPerformSelector:`等等。

`NSArray`的核心功能由私有子类（如`NSCFArray`） 实现。`NSArray`的实例化方法如`+arrayWithObjects:`或`-initWithContentsOfFile:`生成的是这些私有子类的实例。

从外部来看， 一般不容易发现`NSArray`是一个类簇。但是在对类对象进行自省（introspection）时（例如使用`isKindOfClass:`方法），可能会发现你得到的不是一个`NSArray`，而是一个`NSCFArray`或者别的子类；或者在调试时可能会发现虽然你创建了一个`NSArray`，但是调试信息上显示的是`NSCFArray`。除此之外，`NSArray`的行为常常与其他任何类一样，看不出什么特别的。

而在一种特定的情况下类簇的性质非常重要，那就是你自己将公共类子类化。

## 子类化 - Subclassing
对类簇进行子类化（这意味着对抽象类进行子类化）与子类化一个普通类完全不同。

子类化一个普通类时，超类已经提供了完整的功能。在这种情况下，子类可以不需要实现任何的方法，就已经是一个完全可用的类，此时它的行为与超类一样。然后，你可以根据需求为这个子类添加其他方法或覆盖现有方法。

当继承一个类簇时，超类不提供完整的功能。它提供了许多辅助功能，但你必须自己提供核心功能。这意味着，一个空的子类是无效的。你至少需要实现几个必要的方法。在关于类簇的术语中，必须实现的那些方法称为基本方法（primitive methods）。有两种简单的途径确定哪些方法是基本方法。

第一种途径是查看类簇的文档，在其中查看 "Subclassing Notes" 搜索单词 "primitive"，文档会告诉你需要覆盖（override）哪些方法。

第二种途径是打开类簇的头文件。原始方法会在类的主`@interface`块中声明。类簇提供的其他方法则会在分类（categories）中声明。

如果一个类簇本身是另一个类簇的子类，那你要多加小心。因为你需要实现这两个类簇所有的原始方法。例如，`NSMutableArray`本身有五个原始方法，它的超类`NSArray`有两个。如果你要子类化`NSMutableArray`，则必须为所有七个方法提供实现。

## 方法 - Techniques 
现在知道了要实现什么，但是如何实现？主要有三种方式。

方法一，你可以自己从头实现每个原始方法。举个例子，假设要编写一个专门用于保存两个元素的数组：
```
    @interface MyPairArray : NSArray {
        id _objs[2];
    }

    - (id)initWithFirst:(id)first second:(id)second;

    @end
    
    @implementation MyPairArray
    
    - (id)initWithFirst:(id)first second:(id)second
    {
        if((self = [self init])) {
            _objs[0] = first;
            _objs[1] = second;
        }
        return self;
    }
    
    - (NSUInteger)count {
        return 2;
    }
    
    - (id)objectAtIndex: (NSUInteger)index {
        if(index >= 2)
            [NSException raise: NSRangeException format: @"Index (%ld) out of bounds", (long)index];
        return _objs[index];
    }
    @end
```

当然，严格来讲，如何实现原始方法取决于你希望它们做什么。以上只是个例子，通过使用一个C数组实现了`MyPairArray`的功能。

方法二，你可以内置一个工作实例（working instance），并把对你的调用传递给它来处理：

```
    @interface MySpecialArray : NSArray

    @property (copy, nonatomic) NSArray *_realArray;
    
    - (id)initWithArray: (NSArray *)array;
    
    @end
    
    @implementation MySpecialArray
    
    - (id)initWithArray: (NSArray *)array {
        if((self = [self init])) {
            _realArray = [array copy];
        }
        return self;
    }
        
    - (NSUInteger)count {
        return [_realArray count];
    }
    
    - (id)objectAtIndex: (NSUInteger)index {
        id obj = [_realArray objectAtIndex: index];
        // do some processing with obj
        return obj;
    }
    
    // maybe implement more methods here
    @end
```

这种方法允许你重用原始方法的现有实现，然后添加更多功能。

方法三，给类簇添加一个分类（category），而不对其进行子类化。有时候只是需要添加新方法，而不需要修改现有功能。在Objective-C中，你可以在分类中添加新方法：

```
    @interface NSArray (FirstObjectAdditions)
    
    - (id)my_firstObject;
    
    @end
    
    @implementation NSArray (FirstObjectAdditions)
    
    - (id)my_firstObject {
        return [self count] ? [self objectAtIndex: 0] : nil;
    }
    
    @end
```

（这个方法名添加了前缀，以防与官方提供的方法造成冲突。）

## 结论 - Conclusion
类簇与普通类有所不同，但一旦理解了它们的含义，就很容易进行子类化。你需要实现类簇的基本方法 ，可以自己实现也可以通过内置一个类簇的实例来实现。最后，如果你创建子类的唯一目的是添加新方法，请改用分类来实现。

> 扩展阅读
[类簇-官方文档](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html#//apple_ref/doc/uid/TP40010810-CH4-SW1)

