在运行时创建新类的流程：
- 使用 objc_allocateClassPair 创建新类（和元类，这就是函数名叫 objc_allocateClass**Pair** 的原因吧）。
- 使用 class_addIvar 添加实例变量。
- 使用 class_addMethod 添加方法。
- 使用 objc_registerClassPair 将设置好的新类注册到运行时。👌

## objc_allocateClassPair 
```
Class objc_allocateClassPair(Class superclass, const char *name, size_t extraBytes);
```
- 参数：超类、新建类的类名、类对象的额外空间（一般不需要）
- 返回：新类的类对象。如失败返回 `nil`。
- 可以通过调用 `object_getClass(newClass)` 获得 metaclass 的对象。

## class_addIvar
```
BOOL class_addIvar(Class cls, const char *name, size_t size, uint8_t alignment, const char *types);
```
- 只能在 `objc_allocateClassPair` 和 `objc_registerClassPair` 之间调用。已经存在的类不能够再添加实例变量。
- 不能对 `metaclass` 使用。

## class_addMethod
```
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types);
```
- 可以对已经存在的类使用（例如用在方法解析）。
- 如果需要添加类方法，可以对 `metaclass `使用。
- 可以覆盖超类的实现，但不能覆盖自己类中已有的实现。
- 如果想覆盖自己类中已有的实现，可以使用 `method_setImplementation`。

 
> 参考： [Objective-C Runtime](https://developer.apple.com/documentation/objectivec/objective-c_runtime?language=objc )（官方文档）
