---
title: Objc Runtime 总结
layout: posts
categories: iOS
tag: objc
---

## 简介
Objc Runtime 使得C具有了面向对象能力，在程序运行时创建，检查，修改类、对象和它们的方法。Runtime 是C和汇编编写的，本文对应源码是[objc4-756.2](https://opensource.apple.com/source/objc4/objc4-756.2/)。

Runtime系统是由一系列的函数和数据结构组成的公共接口动态共享库，在/usr/include/objc目录下可以看到头文件，可以用其中一些函数通过C语言实现objectivec中一样的功能。

## 基础数据结构
### Class
`objc/runtime.h` 中 `objc_class` 结构体定义如下：

```c
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */
```
其中 `Class` 就是 `struct objc_class`：

```ojbc
typedef struct objc_class *Class;
```

- `isa`：指向 Meta Class，因为 Objc 的类的本身也是一个 Object，为了处理这个关系，runtime 创造了 Meta Class，当给类发送 [NSObject alloc] 这样消息时，实际上是把这个消息发给了 Class Object;
- `super_class`：父类；
- `name`：类名；
- `version`：版本信息，默认为0；
- `info`：类信息，供运行时使用的标志信息；
- `instance_size`：该类的实例变量大小；
- `ivars`：该类的成员变量链表；
- `methodLists`：方法定义链表
- `cache`：方法缓存，提高常用方法调用效率；
- `protocols`：协议链表；

#### objc_ivar_list

```c
struct objc_ivar {
    char * _Nullable ivar_name                               OBJC2_UNAVAILABLE;
    char * _Nullable ivar_type                               OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}                                                            OBJC2_UNAVAILABLE;

struct objc_ivar_list {
    int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
```

#### objc_method_list

```c
struct objc_method {
    SEL _Nonnull method_name                                 OBJC2_UNAVAILABLE;
    char * _Nullable method_types                            OBJC2_UNAVAILABLE;
    IMP _Nonnull method_imp                                  OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

struct objc_method_list {
    struct objc_method_list * _Nullable obsolete             OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
```

#### objc_cache
objc_class 结构体中的 cache 字段用于缓存调用过的 method。

```c
struct objc_cache {
    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method _Nullable buckets[1]                              OBJC2_UNAVAILABLE;
};
```

- `mask`：指定分配缓存 bucket 的总数。runtime 使用这个字段确定线性查找数组的索引位置；
- `occupied`：实际占用缓存bucket总数；
- `buckets`：指向 Method 数据结构指针的数组，这个数组的总数不能超过 mask+1，但是指针是可能为空的，这就表示缓存 bucket 没有被占用，数组会随着时间增长。

#### objc_object
`objc_object` 是一个类的实例结构体，定义在 `objc/objc.h` 中：

```c
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
向 object 发送消息时，Runtime 库会根据 object 的 isa 指针找到这个实例 object 所属于的类，然后在类的方法列表以及父类方法列表寻找对应的方法运行。id 是一个 objc_object 结构类型的指针，这个类型的对象能够转换成任何一种对象。

#### Meta Class
meta class 是一个类对象的类，当向对象发消息，runtime 会在这个对象所属类方法列表中查找发送消息对应的方法，但当向类发送消息时，runtime 就会在这个类的 meta class 方法列表里查找。所有的 meta class，包括 Root class，Superclass，Subclass 的 isa 都指向 Root class 的 meta class，这样能够形成一个闭环。
![MetaClass]({{ site.baseurl }}/images/objc-runtime/metaclass.jpg)

```objc
void TestMetaClass(id self, SEL _cmd) {
     NSLog(@"This objcet is %p", self);
     NSLog(@"Class is %@, super class is %@", [self class], [self superclass]);
     Class currentClass = [self class];

     for (int i = 0; i < 4; i++) {
          NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
          // 通过objc_getClass获得对象isa，这样可以回溯到Root class及NSObject的meta class，可以看到最后指针指向的是0x0和NSObject的meta class类地址一样。
          currentClass = objc_getClass((__bridge void *)currentClass);
     }
     NSLog(@"NSObject's class is %p", [NSObject class]);
     NSLog(@"NSObject's meta class is %p", objc_getClass((__bridge void *)[NSObject class]));
}


- (void)runtimeMetaClassDemo {
     Class newClass = objc_allocateClassPair([NSError class], "TestClass", 0);
     class_addMethod(newClass, @selector(testMetaClass), (IMP)TestMetaClass, "v@:");
     objc_registerClassPair(newClass);
     id instance = [[newClass alloc] initWithDomain:@"some domain" code:0 userInfo:nil];
     [instance performSelector:@selector(testMetaClass)];
}
```

执行结果：

```
2019-12-05 17:27:55.837821+0800 ObjcRuntimeDemo[60949:5828523] This objcet is 0x6000005e6430
2019-12-05 17:27:55.837949+0800 ObjcRuntimeDemo[60949:5828523] Class is TestClass, super class is NSError
2019-12-05 17:27:55.838022+0800 ObjcRuntimeDemo[60949:5828523] Following the isa pointer 0 times gives 0x6000005e65e0
2019-12-05 17:27:55.838086+0800 ObjcRuntimeDemo[60949:5828523] Following the isa pointer 1 times gives 0x0
2019-12-05 17:27:55.838140+0800 ObjcRuntimeDemo[60949:5828523] Following the isa pointer 2 times gives 0x0
2019-12-05 17:27:55.838194+0800 ObjcRuntimeDemo[60949:5828523] Following the isa pointer 3 times gives 0x0
2019-12-05 17:27:55.838251+0800 ObjcRuntimeDemo[60949:5828523] NSObject's class is 0x10a476200
2019-12-05 17:27:55.838310+0800 ObjcRuntimeDemo[60949:5828523] NSObject's meta class is 0x0
```

## 类相关操作函数
类相关操作函数以 class 为前缀。
### 基本信息

```c
// 创建类实例
id _Nullable class_createInstance(Class _Nullable cls, size_t extraBytes);

// 获取类的类名
const char * _Nonnull class_getName(Class _Nullable cls);

// 获取类的父类
Class _Nullable class_getSuperclass(Class _Nullable cls);

// 判断是否为 meta class
BOOL class_isMetaClass(Class _Nullable cls);

// 获取示例大小
size_t class_getInstanceSize(Class _Nullable cls);

// 获取版本号
int class_getVersion(Class _Nullable cls);

// 设置版本号
void class_setVersion(Class _Nullable cls, int version);
```

### 成员变量(ivars)及属性

```c
// 获取类中指定名称实例成员变量的信息
Ivar _Nullable class_getInstanceVariable(Class _Nullable cls, const char * _Nonnull name);

// 获取类成员变量的信息
Ivar _Nullable class_getClassVariable(Class _Nullable cls, const char * _Nonnull name);

// 获取整个成员变量列表
Ivar _Nonnull * _Nullable class_copyIvarList(Class _Nullable cls, unsigned int * _Nullable outCount);

// 添加成员变量
BOOL class_addIvar(Class _Nullable cls, const char * _Nonnull name, size_t size, uint8_t alignment, const char * _Nullable types);

// 获取属性信息
objc_property_t _Nullable class_getProperty(Class _Nullable cls, const char * _Nonnull name)

// 获取属性列表
objc_property_t _Nonnull * _Nullable class_copyPropertyList(Class _Nullable cls, unsigned int * _Nullable outCount);

// 添加属性信息
BOOL class_addProperty(Class _Nullable cls, const char * _Nonnull name, const objc_property_attribute_t * _Nullable attributes, unsigned int attributeCount);

// 替换属性
void class_replaceProperty(Class _Nullable cls, const char * _Nonnull name, const objc_property_attribute_t * _Nullable attributes, unsigned int attributeCount);
```

### Method 相关函数

```c
// 添加方法
BOOL class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types);

// 获取实例方法
Method _Nullable class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name);

// 获取类方法
Method _Nullable class_getClassMethod(Class _Nullable cls, SEL _Nonnull name);

// 获取方法列表
Method _Nonnull * _Nullable class_copyMethodList(Class _Nullable cls, unsigned int * _Nullable outCount);

// 替换方法实现
IMP _Nullable class_replaceMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types);

// 返回方法的具体实现
IMP _Nullable class_getMethodImplementation(Class _Nullable cls, SEL _Nonnull name);
IMP _Nullable class_getMethodImplementation_stret(Class _Nullable cls, SEL _Nonnull name);

// 类实例是否响应指定的selector
BOOL class_respondsToSelector(Class _Nullable cls, SEL _Nonnull sel);
```

### Protocol 相关函数

```c
// 添加协议
BOOL class_addProtocol(Class _Nullable cls, Protocol * _Nonnull protocol);

// 返回类是否实现指定的协议
BOOL class_conformsToProtocol(Class _Nullable cls, Protocol * _Nullable protocol);

// 返回类实现的协议列表
Protocol * __unsafe_unretained _Nonnull * _Nullable class_copyProtocolList(Class _Nullable cls, unsigned int * _Nullable outCount);
```

#### isKindOfClass vs isMemberOfClass

```objc
@interface ORDClassDemo : NSObject
@end
@implementation ORDClassDemo
@end
- (void)runtimeClassDemo {
    BOOL res1 = [(id)[NSObject class] isKindOfClass:[NSObject class]];
    BOOL res2 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];
    BOOL res3 = [(id)[ORDClassDemo class] isKindOfClass:[ORDClassDemo class]];
    BOOL res4 = [(id)[ORDClassDemo class] isMemberOfClass:[ORDClassDemo class]];
    BOOL res5 = [(id)[[NSObject alloc] init] isKindOfClass:[NSObject class]];
    BOOL res6 = [(id)[[NSObject alloc] init] isMemberOfClass:[NSObject class]];
    BOOL res7 = [(id)[[ORDClassDemo alloc] init] isKindOfClass:[ORDClassDemo class]];
    BOOL res8 = [(id)[[ORDClassDemo alloc] init] isMemberOfClass:[ORDClassDemo class]];
    NSLog(@"%d %d %d %d %d %d %d %d", res1, res2, res3, res4, res5, res6, res7, res8);
}
```

输出结果：

```
2020-01-04 12:17:33.048033+0800 ObjcRuntimeDemo[54041:6941164] 1 0 0 0 1 1 1 1
```

首先看下 `isKindOfClass` 与 `isMemberOfClass` 的定义：
- isKindOfClass: Returns a Boolean value that indicates whether the receiver is an instance of given class or an instance of any class that inherits from that class.
- isMemberOfClass: Returns a Boolean value that indicates whether the receiver is an instance of a given class.

翻译一下：
- isKindOfClass：判断该对象是否是该类以及该类子类的实例。
- isMemberOfClass：判断该对象是否是该类的实例。
> 注意：Class 是 Meta Class 的实例。

```objc
Class object_getClass(id obj)
{
    if (obj) return obj->getIsa();
    else return Nil;
}

+ (Class)class {
    return self;
}

- (Class)class {
    return object_getClass(self);
}

+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}

- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}

+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = object_getClass((id)self); tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}

- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}
```

从源码中可以看出 `isKindOfClass` 类方法与实例方法最终都是通过 `object_getClass` 获取 `isa` 后进行比较，`isKindOfClass` 相比于 `isMemberOfClass` 多了一次 `superclass` 比较；
第一个用例 `[NSObject class]` 的 `isa` 指向 `NSObject` 的 `Meta Class`，根据前面提到的闭环原则，对照图中右上角部分，`NSObject` 的 `Meta Class` 的 `superclass` 就是 `NSObjcet` 本身，所以`res1=1`。

res2：`[NSObjce class]` 与 `Meta class` 不同，返回false；

res3：第一次是 `ORDClassDemo Meta class`， 第二次是 `NSObjcet Meta class`，所以返回false；

后面就不一一分析了，补充说明一下，1-4中左边对象对应的是图中中间那一列，5-8中左边对象对应的是图中左侧那一列。

#### 类相关函数示例

```objc
// ORDClassDemo.h
@interface ORDClassDemo : NSObject <NSCopying, NSCoding>
@property (nonatomic, strong) NSArray *array;
@property (nonatomic, copy) NSString *string;

+ (void)classMethod1;
- (void)method1;
- (void)method2;
@end

// --------------
// ORDClassDemo.m
#import "ORDClassDemo.h"

@interface ORDClassDemo () {
    NSInteger _instance1;
    NSString * _instance2;
}

@property (nonatomic, assign) NSUInteger integer;
- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2;
@end

@implementation ORDClassDemo

+ (void)classMethod1 {
}

- (void)method1 {
     NSLog(@"call method method1");
}

- (void)method2 {
}

- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2 {
     NSLog(@"arg1 : %ld, arg2 : %@", arg1, arg2);
}

@end

// -------------
// ViewController.m
#import "ViewController.h"
#import "ORDClassDemo.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    [self runtimeClassMethodDemo];
}

- (void)runtimeClassMethodDemo {
    ORDClassDemo *ordClassDemo = [[ORDClassDemo alloc] init];
     unsigned int outCount = 0;
     Class cls = ordClassDemo.class;
     // 类名
     NSLog(@"class name: %s", class_getName(cls));
     NSLog(@"==========================================================");
     // 父类
     NSLog(@"super class name: %s", class_getName(class_getSuperclass(cls)));
     NSLog(@"==========================================================");
     // 是否是元类
     NSLog(@"ORDClassDemo is %@ a meta-class", (class_isMetaClass(cls) ? @"" : @"not"));
     NSLog(@"==========================================================");
     Class meta_class = objc_getMetaClass(class_getName(cls));
     NSLog(@"%s's meta-class is %s", class_getName(cls), class_getName(meta_class));
     NSLog(@"==========================================================");
     // 变量实例大小
     NSLog(@"instance size: %zu", class_getInstanceSize(cls));
     NSLog(@"==========================================================");
     // 成员变量
     Ivar *ivars = class_copyIvarList(cls, &outCount);
     for (int i = 0; i < outCount; i++) {
          Ivar ivar = ivars[i];
          NSLog(@"instance variable's name: %s at index: %d", ivar_getName(ivar), i);
     }
     free(ivars);
     Ivar string = class_getInstanceVariable(cls, "_string");
     if (string != NULL) {
          NSLog(@"instace variable %s", ivar_getName(string));
     }
     NSLog(@"==========================================================");
     // 属性操作
     objc_property_t * properties = class_copyPropertyList(cls, &outCount);
     for (int i = 0; i < outCount; i++) {
          objc_property_t property = properties[i];
          NSLog(@"property's name: %s", property_getName(property));
     }
     free(properties);
     objc_property_t array = class_getProperty(cls, "array");
     if (array != NULL) {
          NSLog(@"property %s", property_getName(array));
     }
     NSLog(@"==========================================================");
     // 方法操作
     Method *methods = class_copyMethodList(cls, &outCount);
     for (int i = 0; i < outCount; i++) {
          Method method = methods[i];
          NSLog(@"method's signature: %s", method_getName(method));
     }
     free(methods);
     Method method1 = class_getInstanceMethod(cls, @selector(method1));
     if (method1 != NULL) {
          NSLog(@"method %s", method_getName(method1));
     }
     Method classMethod = class_getClassMethod(cls, @selector(classMethod1));
     if (classMethod != NULL) {
          NSLog(@"class method : %s", method_getName(classMethod));
     }
     NSLog(@"ORDClassDemo is%@ responsd to selector: method3WithArg1:arg2:", class_respondsToSelector(cls, @selector(method3WithArg1:arg2:)) ? @"" : @" not");
     IMP imp = class_getMethodImplementation(cls, @selector(method1));
     imp();
     NSLog(@"==========================================================");
     // 协议
     Protocol * __unsafe_unretained * protocols = class_copyProtocolList(cls, &outCount);
     Protocol * protocol;
     for (int i = 0; i < outCount; i++) {
          protocol = protocols[i];
          NSLog(@"protocol name: %s", protocol_getName(protocol));
     }
     NSLog(@"ORDClassDemo is%@ responsed to protocol %s", class_conformsToProtocol(cls, protocol) ? @"" : @" not", protocol_getName(protocol));
     NSLog(@"==========================================================");
}

@end
```

执行结果：

```
2020-01-04 16:19:07.200363+0800 ObjcRuntimeDemo[76065:7619697] class name: ORDClassDemo
2020-01-04 16:19:07.200484+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.200570+0800 ObjcRuntimeDemo[76065:7619697] super class name: NSObject
2020-01-04 16:19:07.200642+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.200717+0800 ObjcRuntimeDemo[76065:7619697] ORDClassDemo is not a meta-class
2020-01-04 16:19:07.200781+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.200842+0800 ObjcRuntimeDemo[76065:7619697] ORDClassDemo's meta-class is ORDClassDemo
2020-01-04 16:19:07.200899+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.201004+0800 ObjcRuntimeDemo[76065:7619697] instance size: 48
2020-01-04 16:19:07.201152+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.201368+0800 ObjcRuntimeDemo[76065:7619697] instance variable's name: _instance1 at index: 0
2020-01-04 16:19:07.201584+0800 ObjcRuntimeDemo[76065:7619697] instance variable's name: _instance2 at index: 1
2020-01-04 16:19:07.329429+0800 ObjcRuntimeDemo[76065:7619697] instance variable's name: _array at index: 2
2020-01-04 16:19:07.329534+0800 ObjcRuntimeDemo[76065:7619697] instance variable's name: _string at index: 3
2020-01-04 16:19:07.329612+0800 ObjcRuntimeDemo[76065:7619697] instance variable's name: _integer at index: 4
2020-01-04 16:19:07.329692+0800 ObjcRuntimeDemo[76065:7619697] instace variable _string
2020-01-04 16:19:07.329769+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.329851+0800 ObjcRuntimeDemo[76065:7619697] property's name: integer
2020-01-04 16:19:07.329912+0800 ObjcRuntimeDemo[76065:7619697] property's name: array
2020-01-04 16:19:07.329988+0800 ObjcRuntimeDemo[76065:7619697] property's name: string
2020-01-04 16:19:07.330060+0800 ObjcRuntimeDemo[76065:7619697] property array
2020-01-04 16:19:07.330130+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.482054+0800 ObjcRuntimeDemo[76065:7619697] method's signature: method1
2020-01-04 16:19:07.482138+0800 ObjcRuntimeDemo[76065:7619697] method's signature: method2
2020-01-04 16:19:07.482290+0800 ObjcRuntimeDemo[76065:7619697] method's signature: method3WithArg1:arg2:
2020-01-04 16:19:07.482379+0800 ObjcRuntimeDemo[76065:7619697] method's signature: .cxx_destruct
2020-01-04 16:19:07.482455+0800 ObjcRuntimeDemo[76065:7619697] method's signature: string
2020-01-04 16:19:07.482517+0800 ObjcRuntimeDemo[76065:7619697] method's signature: array
2020-01-04 16:19:07.482645+0800 ObjcRuntimeDemo[76065:7619697] method's signature: setString:
2020-01-04 16:19:07.482905+0800 ObjcRuntimeDemo[76065:7619697] method's signature: setArray:
2020-01-04 16:19:07.483131+0800 ObjcRuntimeDemo[76065:7619697] method's signature: setInteger:
2020-01-04 16:19:07.483352+0800 ObjcRuntimeDemo[76065:7619697] method's signature: integer
2020-01-04 16:19:07.483550+0800 ObjcRuntimeDemo[76065:7619697] method method1
2020-01-04 16:19:07.483817+0800 ObjcRuntimeDemo[76065:7619697] class method : classMethod1
2020-01-04 16:19:07.483910+0800 ObjcRuntimeDemo[76065:7619697] ORDClassDemo is responsd to selector: method3WithArg1:arg2:
2020-01-04 16:19:07.484148+0800 ObjcRuntimeDemo[76065:7619697] call method method1
2020-01-04 16:19:07.484378+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
2020-01-04 16:19:07.484565+0800 ObjcRuntimeDemo[76065:7619697] protocol name: NSCopying
2020-01-04 16:19:07.484746+0800 ObjcRuntimeDemo[76065:7619697] protocol name: NSCoding
2020-01-04 16:19:07.484916+0800 ObjcRuntimeDemo[76065:7619697] ORDClassDemo is responsed to protocol NSCoding
2020-01-04 16:19:07.485105+0800 ObjcRuntimeDemo[76065:7619697] ==========================================================
```

### 动态创建类

```objc
// 创建一个新类和元类
Class _Nullable objc_allocateClassPair(Class _Nullable superclass, const char * _Nonnull name, size_t extraBytes);

// 在应用中注册由 objc_allocateClassPair 创建的类
// 注意：为新类添加方法，实例变量和属性后再调用这个来注册类
void objc_registerClassPair(Class _Nonnull cls);

// 复制一个类，用于 Foundation 的 KVO, 不要主动调用该函数
Class _Nonnull objc_duplicateClass(Class _Nonnull original, const char * _Nonnull name, size_t extraBytes);

// 销毁一个类及其相关的元类
void objc_disposeClassPair(Class _Nonnull cls);
```

示例代码：

```objc
void imp_submethod1(id self, SEL _cmd) {
    NSLog(@"run sub method 1");
}

- (void)runtimeCreatClassDemo {
    Class cls = objc_allocateClassPair(ORDClassDemo.class, "ORDSubClassDemo", 0);
    class_addMethod(cls, @selector(submethod1), (IMP)imp_submethod1, "v@:");
    class_replaceMethod(cls, @selector(method1), (IMP)imp_submethod1, "v@:");
    class_addIvar(cls, "_ivar1", sizeof(NSString *), log(sizeof(NSString *)), "i");
    objc_property_attribute_t type = {"T", "@\"NSString\""};
    objc_property_attribute_t ownership = { "C", "" };
    objc_property_attribute_t backingivar = { "V", "_ivar1"};
    objc_property_attribute_t attrs[] = {type, ownership, backingivar};
    class_addProperty(cls, "property2", attrs, 3);
    objc_registerClassPair(cls);
    id instance = [[cls alloc] init];
    [instance performSelector:@selector(submethod1)];
    [instance performSelector:@selector(method1)];
}
```

执行结果：

```
2020-01-04 17:13:37.160846+0800 ObjcRuntimeDemo[80498:7824924] run sub method 1
2020-01-04 17:13:37.160986+0800 ObjcRuntimeDemo[80498:7824924] run sub method 1
```

### 动态创建对象

```objc
// 创建类实例
id _Nullable class_createInstance(Class _Nullable cls, size_t extraBytes);

// 在指定位置创建实例
id _Nullable objc_constructInstance(Class _Nullable cls, void * _Nullable bytes);

// 销毁实例
void * _Nullable objc_destructInstance(id _Nullable obj);
```

示例代码：

```objc
- (void)runtimeCreateInstance {
    //可以看出 class_createInstance 和 alloc 的不同
    id theObject = class_createInstance(NSString.class, sizeof(unsigned));
    id str1 = [theObject init];
    NSLog(@"%@", [str1 class]);
    id str2 = [[NSString alloc] initWithString:@"test"];
    NSLog(@"%@", [str2 class]);
}
```

执行结果：

```
2020-01-04 18:51:52.812889+0800 ObjcRuntimeDemo[88509:8134997] NSString
2020-01-04 18:51:52.813006+0800 ObjcRuntimeDemo[88509:8134997] __NSCFConstantString
```

## 实例相关函数
### 整个对象操作

```objc
// 复制指定对象
id _Nullable object_copy(id _Nullable obj, size_t size);

// 释放指定对象占用的内存
id _Nullable object_dispose(id _Nullable obj);
```

示例代码：

```objc
NSObject *a = [[NSObject alloc] init];
id newB = object_copy(a, class_getInstanceSize(ORDClassDemo.class));
object_setClass(newB, ORDClassDemo.class);
object_dispose(a);
```

### 对象实例变量相关函数

```objc
// 设置实例变量的值
Ivar _Nullable object_setInstanceVariable(id _Nullable obj, const char * _Nonnull name, void * _Nullable value);

// 获取实例变量的值
Ivar _Nullable object_getInstanceVariable(id _Nullable obj, const char * _Nonnull name, void * _Nullable * _Nullable outValue);

// 返回指向给定对象分配的任何额外字节的指针
// 额外字节就是 class_createInstance 中的 extraBytes 参数
void * _Nullable object_getIndexedIvars(id _Nullable obj);

// 返回实例变量的值
id _Nullable object_getIvar(id _Nullable obj, Ivar _Nonnull ivar);

// 设置实例变量的值
void object_setIvar(id _Nullable obj, Ivar _Nonnull ivar, id _Nullable value);
```

### 对象的类操作

```objc
// 返回给定对象的类名
const char * _Nonnull object_getClassName(id _Nullable obj);

// 返回对象的类
Class _Nullable object_getClass(id _Nullable obj);

// 设置对象的类
Class _Nullable object_setClass(id _Nullable obj, Class _Nonnull cls);
```

### 获取类定义操作

```objc
// 获取已注册类定义列表
int objc_getClassList(Class _Nonnull * _Nullable buffer, int bufferCount);

// 创建并返回一个指向所有已注册类的指针列表
Class _Nonnull * _Nullable
objc_copyClassList(unsigned int * _Nullable outCount);
// 返回指定类的类定义
Class _Nullable objc_lookUpClass(const char * _Nonnull name);
Class _Nullable objc_getClass(const char * _Nonnull name);
Class _Nonnull objc_getRequiredClass(const char * _Nonnull name);
// 返回指定类的元类
Class objc_getMetaClass ( const char *name );
```

示例代码：

```objc
- (void)runtimeObjectMethodDemo {
    int numClasses;
    Class * classes = NULL;
    numClasses = objc_getClassList(NULL, 0);
    if (numClasses > 0) {
         classes = malloc(sizeof(Class) * numClasses);
         numClasses = objc_getClassList(classes, numClasses);
         NSLog(@"number of classes: %d", numClasses);
         for (int i = 0; i < numClasses; i++) {
              Class cls = classes[i];
              NSLog(@"class name: %s", class_getName(cls));
         }
         free(classes);
    }
}
```

执行结果

```
2020-01-05 12:27:55.773037+0800 ObjcRuntimeDemo[10896:517938] number of classes: 21761
2020-01-05 12:27:55.773176+0800 ObjcRuntimeDemo[10896:517938] class name: JSExport
2020-01-05 12:27:55.773272+0800 ObjcRuntimeDemo[10896:517938] class name: NSLeafProxy
2020-01-05 12:27:55.773340+0800 ObjcRuntimeDemo[10896:517938] class name: NSProxy
2020-01-05 12:27:55.773441+0800 ObjcRuntimeDemo[10896:517938] class name: WKObject
2020-01-05 12:27:55.773519+0800 ObjcRuntimeDemo[10896:517938] class name: WKNSURLAuthenticationChallenge
2020-01-05 12:27:55.773616+0800 ObjcRuntimeDemo[10896:517938] class name: WKNSURLRequest
...
```

## 成员变量与属性
### Ivar
实例变量类型，指向objc_ivar结构体的指针，ivar指针地址是根据class结构体的地址加上基地址偏移字节得到的。

```c
/// An opaque type that represents an instance variable.
typedef struct objc_ivar *Ivar;

struct objc_ivar {
    char * _Nullable ivar_name                               OBJC2_UNAVAILABLE;
    char * _Nullable ivar_type                               OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}      
```

- `ivar_name`：变量名
- `ivar_type`：变量类型
- `ivar_offset`：基地址偏移字节

### objc_property_t

```c
/// An opaque type that represents an Objective-C declared property.
typedef struct objc_property *objc_property_t;

// 获取类的属性
objc_property_t _Nonnull * _Nullable class_copyPropertyList(Class _Nullable cls, unsigned int * _Nullable outCount);

// 获取协议的属性
objc_property_t _Nonnull * _Nullable protocol_copyPropertyList(Protocol * _Nonnull proto, unsigned int * _Nullable outCount);
objc_property_t _Nonnull * _Nullable protocol_copyPropertyList2(Protocol * _Nonnull proto, unsigned int * _Nullable outCount, BOOL isRequiredProperty, BOOL isInstanceProperty);

// 获取属性名称
const char * _Nonnull property_getName(objc_property_t _Nonnull property);

// 通过给出的名称在类和协议中获取属性的引用
objc_property_t _Nullable class_getProperty(Class _Nullable cls, const char * _Nonnull name);
objc_property_t _Nullable protocol_getProperty(Protocol * _Nonnull proto, const char * _Nonnull name, BOOL isRequiredProperty, BOOL isInstanceProperty);

// 获取属性信息
const char * _Nullable property_getAttributes(objc_property_t _Nonnull property);
```

示例代码：

```objc
- (void)runtimePropertyMethodDemo {
    // 获取属性列表
    id ordClassDemo = objc_getClass("ORDClassDemo");
    unsigned int outCount;
    objc_property_t *properties = class_copyPropertyList(ordClassDemo, &outCount);

    unsigned int i;
    for (i = 0; i < outCount; i++) {
         objc_property_t property = properties[i];
         fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
    }
}
```

执行结果：

```
integer TQ,N,V_integer
array T@"NSArray",&,N,V_array
string T@"NSString",C,N,V_string
```

### objc_property_attribute_t

```c
/// Defines a property attribute
typedef struct {
    const char * _Nonnull name;           /**< The name of the attribute */
    const char * _Nonnull value;          /**< The value of the attribute (usually empty) */
} objc_property_attribute_t;
```

示例代码：

```objc
@interface ORDPropertyDemo : NSObject
@property (nonatomic, copy) NSString *name;
@end
@implementation ORDPropertyDemo
- (void)speak
{
     unsigned int numberOfIvars = 0;
     Ivar *ivars = class_copyIvarList([self class], &numberOfIvars);
     for(const Ivar *p = ivars; p < ivars+numberOfIvars; p++) {
          Ivar const ivar = *p;
          ptrdiff_t offset = ivar_getOffset(ivar);
          const char *name = ivar_getName(ivar);
          NSLog(@"ORDPropertyDemo ivar name = %s, offset = %td", name, offset);
     }
     NSLog(@"my name is %p", &_name);
     NSLog(@"my name is %@", *(&_name));
}
@end

@interface ORDPropertyAttributeDemo : NSObject
@end
@implementation ORDPropertyAttributeDemo
- (instancetype)init
{
     self = [super init];
     if (self) {
          NSLog(@"ORDPropertyAttributeDemo instance = %@", self);
          void *self2 = (__bridge void *)self;
          NSLog(@"ORDPropertyAttributeDemo instance pointer = %p", &self2);
          id cls = [ORDPropertyDemo class];
          NSLog(@"Class instance address = %p", cls);
          void *obj = &cls;
          NSLog(@"Void *obj = %@", obj);
          [(__bridge id)obj speak];
     }
     return self;
}
```

执行结果：

```
2020-01-05 12:53:14.600896+0800 ObjcRuntimeDemo[13013:629047] ORDPropertyAttributeDemo instance = <ORDPropertyAttributeDemo: 0x6000008b01a0>
2020-01-05 12:53:14.601003+0800 ObjcRuntimeDemo[13013:629047] ORDPropertyAttributeDemo instance pointer = 0x7ffee59b2088
2020-01-05 12:53:14.601088+0800 ObjcRuntimeDemo[13013:629047] Class instance address = 0x10a251be8
2020-01-05 12:53:14.601180+0800 ObjcRuntimeDemo[13013:629047] Void *obj = <ORDPropertyDemo: 0x7ffee59b2080>
2020-01-05 12:53:14.601270+0800 ObjcRuntimeDemo[13013:629047] ORDPropertyDemo ivar name = _name, offset = 8
2020-01-05 12:53:14.601365+0800 ObjcRuntimeDemo[13013:629047] my name is 0x7ffee59b2088
2020-01-05 12:53:14.601451+0800 ObjcRuntimeDemo[13013:629047] my name is <ORDPropertyAttributeDemo: 0x6000008b01a0>
```

obj为指向 ORDPropertyDemo Class 的指针，相当于 ORDPropertyDemo 的实例对象，根据objc_msgSend 流程，obj 指针能够在方法列表中找到 speak 方法。

ORDPropertyDemo 中 Property name 会被转换成 ivar 到类的结构里，runtime 会计算 ivar 的地址偏移来找 ivar 的最终地址，根据输出可以看出 ORDPropertyDemo class 的指针地址加上 ivar 的偏移量正好跟 ORDPropertyAttributeDemo 对象指针地址。

### 关联对象
相关函数：

```c
// 以键值形式添加关联对象
void objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssociationPolicy policy);

// 根据 key 获取关联对象
id _Nullable objc_getAssociatedObject(id _Nonnull object, const void * _Nonnull key);

// 移除所有关联对象
void objc_removeAssociatedObjects(id _Nonnull object);

// 上面方法以键值对的形式动态的向对象添加，获取或者删除关联值。其中关联政策是一组枚举常量。这些常量对应着引用关联值机制，也就是Objc内存管理的引用计数机制。
enum {
     OBJC_ASSOCIATION_ASSIGN = 0,
     OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,
     OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
     OBJC_ASSOCIATION_RETAIN = 01401,
     OBJC_ASSOCIATION_COPY = 01403
};
```

示例代码：

```objc
@interface ORDPropertyDemo (Category)
@property (nonatomic, strong) NSString *categoryProperty;
@end
@implementation ORDPropertyDemo (Category)

- (NSString *)categoryProperty {
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setCategoryProperty:(NSString *)categoryProperty {
    objc_setAssociatedObject(self, @selector(categoryProperty), categoryProperty, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
```

### 成员变量与属性相关函数
#### 成员变量

```c
// 获取成员变量名
const char * _Nullable ivar_getName(Ivar _Nonnull v);

// 获取成员变量类型编码
const char * _Nullable ivar_getTypeEncoding(Ivar _Nonnull v);

// 获取成员变量偏移量
ptrdiff_t ivar_getOffset(Ivar _Nonnull v);
```

#### 属性

```c
// 获取属性名
const char * _Nonnull property_getName(objc_property_t _Nonnull property);

// 获取属性特性描述字符串
const char * _Nullable property_getAttributes(objc_property_t _Nonnull property);

// 获取属性中指定的特性
char * _Nullable property_copyAttributeValue(objc_property_t _Nonnull property, const char * _Nonnull attributeName);

// 获取属性的特性列表
objc_property_attribute_t * _Nullable property_copyAttributeList(objc_property_t _Nonnull property, unsigned int * _Nullable outCount);
```

示例代码:

```objc
//定义一个映射字典（全局）
static NSMutableDictionary *map = nil;

@interface ORDPropertyDemo : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString * status;
- (void)setDataWithDict: (NSDictionary *)dict;
@end
@implementation ORDPropertyDemo
+ (void)load
{
    map = [NSMutableDictionary dictionary];
    map[@"name1"] = @"name";
    map[@"status1"] = @"status";
    map[@"name2"] = @"name";
    map[@"status2"] = @"status";
}

- (void)setDataWithDict: (NSDictionary *)dict  {
    [dict enumerateKeysAndObjectsUsingBlock:^(NSString *key, id obj, BOOL *stop) {
         NSString *propertyKey = [self propertyForKey:key];
         if (propertyKey)
         {
              [self setValue:obj forKey:propertyKey];
         }
        NSLog(@"name:%@ status:%@", self.name, self.status);
    }];
}

- (NSString *)propertyForKey: (NSString*) key {
    return map[key];
}

@end

- (void)runtimePropteryIvarDemo {
    ORDPropertyDemo *ordPropertyDemo = [[ORDPropertyDemo alloc] init];
    
    // @{@"name1": "test", @"status1": @"start"}
    [ordPropertyDemo setDataWithDict:@{@"name1": @"test", @"status1": @"start"}];
    
    // @{@"name1": "test", @"status1": @"end"}
    [ordPropertyDemo setDataWithDict:@{@"name1": @"test", @"status1": @"end"}];
}
```

执行结果：

```
2020-01-05 15:02:07.060195+0800 ObjcRuntimeDemo[23754:983962] name:test status:(null)
2020-01-05 15:02:07.060336+0800 ObjcRuntimeDemo[23754:983962] name:test status:start
2020-01-05 15:02:07.060422+0800 ObjcRuntimeDemo[23754:983962] name:test status:start
2020-01-05 15:02:07.060497+0800 ObjcRuntimeDemo[23754:983962] name:test status:end
```

## Method 与消息相关函数
### 基本数据结构
- SEL
选择器表示一个方法的 selector 的指针，可以理解为 Method 中的 ID 类型。

```c
/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;
```

获取SEL的三个方法：
1. sel_registerName 函数
2. objectivec 编译器提供的 @selector
3. NSSelectorFromString 方法

示例代码：

```objc
//objc_selector 编译时会根据每个方法名字参数序列生成唯一标识
SEL sel1 = @selector(method1);
NSLog(@"sel : %p", sel1);
```

- IMP
`IMP` 是函数指针，指向方法的首地址，通过 `SEL` 快速得到对应 `IMP`，这时可以跳过 Runtime 消息传递机制直接执行函数，比直接向对象发消息高效。定义如下:

```c
/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```

- Method
`Method` 用于表示类定义中的方法。

```c
/// An opaque type that represents a method in a class definition.
typedef struct objc_method *Method;

struct objc_method {
    SEL _Nonnull method_name                                 OBJC2_UNAVAILABLE;
    char * _Nullable method_types                            OBJC2_UNAVAILABLE;
    IMP _Nonnull method_imp                                  OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
```

### Method 相关操作函数

```c
// 调用指定方法的实现，返回的是方法实现时的返回，参数 receiver 不能为空，这个比method_getImplementation 和 method_getName 快
void method_invoke(void /* id receiver, Method m, ... */ );

// 调用返回一个数据结构的方法的实现
void method_invoke_stret(void /* id receiver, Method m, ... */ );

// 获取方法名，希望获得方法明的C字符串，使用 sel_getName(method_getName(method))
SEL _Nonnull method_getName(Method _Nonnull m);

// 返回方法的实现
IMP _Nonnull method_getImplementation(Method _Nonnull m);

// 获取描述方法参数和返回值类型的字符串
const char * _Nullable method_getTypeEncoding(Method _Nonnull m);

// 获取方法的返回值类型的字符串
char * _Nonnull method_copyReturnType(Method _Nonnull m);

// 获取方法的指定位置参数的类型字符串
unsigned int method_getNumberOfArguments(Method _Nonnull m);

// 通过引用返回方法的返回值类型字符串
void method_getReturnType(Method _Nonnull m, char * _Nonnull dst, size_t dst_len);

// 返回方法的参数的个数
unsigned int method_getNumberOfArguments(Method _Nonnull m);

// 通过引用返回方法指定位置参数的类型字符串
void method_getArgumentType(Method _Nonnull m, unsigned int index, char * _Nullable dst, size_t dst_len) ;

// 返回指定方法的方法描述结构体
struct objc_method_description * _Nonnull method_getDescription(Method _Nonnull m);

// 设置方法的实现
IMP _Nonnull method_setImplementation(Method _Nonnull m, IMP _Nonnull imp);

// 交换两个方法的实现
void method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2);
```

- Method 的 SEL

```c
// 返回给定选择器指定的方法的名称
const char * _Nonnull sel_getName(SEL _Nonnull sel);

// 在 objc Runtime 系统中注册一个方法，将方法名映射到一个选择器，并返回这个选择器
SEL _Nonnull sel_registerName(const char * _Nonnull str);

// 在 objc Runtime 系统中注册一个方法
SEL _Nonnull sel_getUid(const char * _Nonnull str);

// 比较两个选择器
BOOL sel_isEqual(SEL _Nonnull lhs, SEL _Nonnull rhs);
```

### Method 调用流程
消息转发相关函数

```c
// 发送消息并返回基本数据类型
void objc_msgSend(void /* id self, SEL op, ... */ );
void objc_msgSendSuper(void /* struct objc_super *super, SEL op, ... */ );

// 发送消息并返回数据结构类型
void objc_msgSend_stret(void /* id self, SEL op, ... */ );
void objc_msgSendSuper_stret(void /* struct objc_super *super, SEL op, ... */ );

// 发送消息并返回浮点类型
void objc_msgSend_fpret(void /* id self, SEL op, ... */ );
```

在iOS中函数的调用，实质就是给对象发消息。而在程序的运行过程中，函数调用的实现是不确定的，只有在运行时才去确定函数的实现。在程序运行时，编译器会把函数的调用转换成 `objc_msgsend`。这个函数会动态的寻找下一个要执行的方法。

1. 编译阶段：[receiver selector]方法调用被编译为：
- `objc_msgSend(receiver, selector)`(不带参数方法)；
- `objc_msgSend(receiver, selector, org1, org2)`(带参数方法)；
2. 运行时阶段：
- 通过 `receiver` 的 `isa` 指针，找到 `receiver` 所属的 `Class`(类)；
- 在 `receiver` 所属类的 `method list`(方法列表)中找对应的 `selector`(先找方法缓存列表再找方法列表);
- 如果在 `Class` 中没有找到 `selector` 对应的实现，就继续去 `superClass`(父类)方法列表中查找；
- 如果找到对应 `selector`，直接执行 `receiver` 中 `selector` 方法对应的 `IMP`(实现)；
- 若找不到对应的 `selector`，消息将被转发或者临时向 `receiver` 添加这个 `selector` 对应的实现，否则会 crash。

程序如果找不到对应的方法实现就会 crash，可以通过重写以下方法进行处理：
![resolvemethod]({{ site.baseurl }}/images/objc-runtime/resolvemethod.png)

```objc
// 当调用一个不存在的类方法时调用
+ (BOOL)resolveClassMethod:(SEL)sel;

// 当调用一个不存在的实例方法时调用
+ (BOOL)resolveInstanceMethod:(SEL)sel;

// 将这个不存在的方法重定向到其他类进行处理，返回一个类的实例
- (id)forwardingTargetForSelector:(SEL)aSelector;

// 将这个不存在的方法打包成NSInvocation丢进来，需要调用invokeWithTarget：给某个能执行方法的实例
- (void)forwardInvocation:(NSInvocation *)anInvocation;
```

动态方法解析示例：

```objc
@interface ORDMethodHelper : NSObject
- (void)forwardTargetMethod;
- (void)forwardInvocationMethod;
@end

@interface ORDMethodDemo : NSObject
@property (nonatomic, strong)NSString *propertyName;
- (void)dynamicMethod;
- (void)forwardTargetMethod;
- (void)forwardInvocationMethod;
@end

void dynamicMethodIMP(id self, SEL _cmd) {
    NSLog(@"%@, %p", self, _cmd);
}

@implementation ORDMethodHelper

- (void)forwardTargetMethod {
    NSLog(@"%@, %p", self, _cmd);
}

- (void)forwardInvocationMethod {
    NSLog(@"%@, %p", self, _cmd);
}

@end

@interface ORDMethodDemo () {
    ORDMethodHelper *_helper;
}
@end

@implementation ORDMethodDemo
@dynamic propertyName;

- (instancetype)init {
    self = [super init];
    if (self != nil) {
        _helper = [[ORDMethodHelper alloc] init];
    }
    return self;
}

+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    NSLog(@"resolveInstanceMethod");
    NSString *selStr = NSStringFromSelector(sel);
    if ([selStr isEqualToString:@"dynamicMethod"]) {
        class_addMethod([self class], sel, (IMP) dynamicMethodIMP, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"forwardingTargetForSelector");
    NSString *selStr = NSStringFromSelector(aSelector);
    // 将消息转发给_helper来处理
    if ([selStr isEqualToString:@"forwardMethod"]) {
        return _helper;
    }
    return [super forwardingTargetForSelector:aSelector];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    NSLog(@"methodSignatureForSelector");
    NSMethodSignature *signature = [super methodSignatureForSelector:sel];
    if (!signature) {
        if ([ORDMethodHelper instancesRespondToSelector:sel]) {
            signature = [ORDMethodHelper instanceMethodSignatureForSelector:sel];
        }
    }
    return signature;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    NSLog(@"forwardInvocation");
    if ([ORDMethodHelper instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:_helper];
    }
}
@end

- (void)runtimeDynamicMethodDemo {
    ORDMethodDemo *ordMethodDemo = [[ORDMethodDemo alloc] init];
    [ordMethodDemo performSelector:@selector(dynamicMethod)];
    [ordMethodDemo performSelector:@selector(forwardTargetMethod)];
    [ordMethodDemo performSelector:@selector(forwardInvocationMethod)];
}
```

执行结果：

```
2020-01-05 17:09:15.618676+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.618801+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.618883+0800 ObjcRuntimeDemo[34405:1535121] <ORDMethodDemo: 0x6000017dc440>, 0x1069594ea
2020-01-05 17:09:15.618963+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.619041+0800 ObjcRuntimeDemo[34405:1535121] forwardingTargetForSelector
2020-01-05 17:09:15.619121+0800 ObjcRuntimeDemo[34405:1535121] methodSignatureForSelector
2020-01-05 17:09:15.619223+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.619311+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.619390+0800 ObjcRuntimeDemo[34405:1535121] forwardInvocation
2020-01-05 17:09:15.619482+0800 ObjcRuntimeDemo[34405:1535121] <ORDMethodHelper: 0x6000017dc410>, 0x106959323
2020-01-05 17:09:15.619555+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.619644+0800 ObjcRuntimeDemo[34405:1535121] forwardingTargetForSelector
2020-01-05 17:09:15.619789+0800 ObjcRuntimeDemo[34405:1535121] methodSignatureForSelector
2020-01-05 17:09:15.619954+0800 ObjcRuntimeDemo[34405:1535121] resolveInstanceMethod
2020-01-05 17:09:15.620114+0800 ObjcRuntimeDemo[34405:1535121] forwardInvocation
2020-01-05 17:09:15.620340+0800 ObjcRuntimeDemo[34405:1535121] <ORDMethodHelper: 0x6000017dc410>, 0x106959337
```

### Method Swizzling
Method Swizzling 是改变一个 `selector` 实际实现的技术，可以在运行时修改 `selector` 对应的函数来修改 `Method` 的实现。

需要注意的点：
- Swizzling 应该总在 `+load` 中执行：`objc` 在运行时会自动调用类的两个方法 `+load` 和 `+initialize`。`+load` 会在类初始加载时调用，和 `+initialize` 比较 `+load` 能保证在类的初始化过程中被加载;
- Swizzling 应该总是在 `dispatch_once` 中执行：swizzling 会改变全局状态，所以在运行时采取一些预防措施，使用 `dispatch_once` 就能够确保代码不管有多少线程都只被执行一次。这将成为 method swizzling 的最佳实践。
- Selector，Method和Implementation：这几个之间关系可以这样理解，一个类维护一个运行时可接收的消息分发表，分发表中每个入口是一个 `Method`，其中 key 是一个特定的名称，即 `SEL`，与其对应的实现是 `IMP` 即指向底层 C 函数的指针。

示例代码：

```objc
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // 通过 methodswizzling 修改了 UIViewController 的 @selecto(viewWillAppear:) 的指针使其指向了自定义的 xxx_viewWillAppear
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
        
        // 如果类中不存在要替换的方法，就先用 class_addMethod 和 class_replaceMethod 函数添加和替换两个方法现。
        // 但如果已经有了要替换的方法，就调用 method_exchangeImplementations 函数交换两个方法的 Implemenation。
        if (didAddMethod) {
            class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling
- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
```

执行结果：

```
2020-01-05 17:47:42.661831+0800 ObjcRuntimeDemo[37682:1696071] viewWillAppear: <ViewController: 0x7fedcee095d0>
```

- method_exchangeImplementations 等价实现：

```objc
IMP imp1 = method_getImplementation(m1);
IMP imp2 = method_getImplementation(m2);
method_setImplementation(m1, imp2);
method_setImplementation(m2, imp1);
```

Method Swizzling 另一种实现：

```objc
- (void)replacementReceiveMessage:(const struct BInstantMessage *)arg1 {
    NSLog(@"arg1 is %@", arg1);
    [self replacementReceiveMessage:arg1];
}
+ (void)load {
    SEL originalSelector = @selector(ReceiveMessage:);
    SEL overrideSelector = @selector(replacementReceiveMessage:);
    Method originalMethod = class_getInstanceMethod(self, originalSelector);
    Method overrideMethod = class_getInstanceMethod(self, overrideSelector);
    if (class_addMethod(self, originalSelector, method_getImplementation(overrideMethod), method_getTypeEncoding(overrideMethod))) {
        class_replaceMethod(self, overrideSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
    } else {
         method_exchangeImplementations(originalMethod, overrideMethod);
    }
}
```

## Protocol 和 Category
### Category
`Category` 是指向分类的结构体的指针。

```c
/// An opaque type that represents a category.
typedef struct objc_category *Category;

struct objc_category {
    char * _Nonnull category_name                            OBJC2_UNAVAILABLE;
    char * _Nonnull class_name                               OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable instance_methods     OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable class_methods        OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
}

// objc-runtime-new.h
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

从 `Category` 的结构可见：
1. 它可以添加实例方法，类方法，甚至可以实现协议，添加属性(但是这里的属性不会自动生成实例变量和对应的 set、get 方法需要通过关联对象实现);
2. 不可以添加实例变量。

libobjc.order

```
__objc_init
_environ_init
_tls_init
_lock_init
_recursive_mutex_init
_exception_init
_map_images
_map_images_nolock
```

`Category` 加载过程：

```
objc-os.mm
    -_objc_init
objc-runtime.mm
    -map_images
    -map_images_nolock
    -_read_images
    -remethodizeClass
    -attachCategories
```

入口方法 _objc_init:

```c
void _objc_init(void)
{
    static bool initialized = false;
    if (initialized) return;
    initialized = true;
    
    // fixme defer initialization until an objc-using image is found?
    environ_init();
    tls_init();
    static_init();
    lock_init();
    exception_init();

    _dyld_objc_notify_register(&map_images, load_images, unmap_image);
}
```

_objc_init 内调用 map_images，最终调用 _read_images，Category 相关代码如下：

```c
// Discover categories. 
    for (EACH_HEADER) {
        category_t **catlist = 
            _getObjc2CategoryList(hi, &count);
        bool hasClassProperties = hi->info()->hasCategoryClassProperties();

        for (i = 0; i < count; i++) {
            category_t *cat = catlist[i];
            Class cls = remapClass(cat->cls);

            if (!cls) {
                // Category's target class is missing (probably weak-linked).
                // Disavow any knowledge of this category.
                catlist[i] = nil;
                if (PrintConnecting) {
                    _objc_inform("CLASS: IGNORING category \?\?\?(%s) %p with "
                                 "missing weak-linked target class", 
                                 cat->name, cat);
                }
                continue;
            }

            // Process this category. 
            // First, register the category with its target class. 
            // Then, rebuild the class's method lists (etc) if 
            // the class is realized. 
            bool classExists = NO;
            if (cat->instanceMethods ||  cat->protocols  
                ||  cat->instanceProperties) 
            {
                addUnattachedCategoryForClass(cat, cls, hi);
                if (cls->isRealized()) {
                    remethodizeClass(cls);
                    classExists = YES;
                }
                if (PrintConnecting) {
                    _objc_inform("CLASS: found category -%s(%s) %s", 
                                 cls->nameForLogging(), cat->name, 
                                 classExists ? "on existing class" : "");
                }
            }

            if (cat->classMethods  ||  cat->protocols  
                ||  (hasClassProperties && cat->_classProperties)) 
            {
                addUnattachedCategoryForClass(cat, cls->ISA(), hi);
                if (cls->ISA()->isRealized()) {
                    remethodizeClass(cls->ISA());
                }
                if (PrintConnecting) {
                    _objc_inform("CLASS: found category +%s(%s)", 
                                 cls->nameForLogging(), cat->name);
                }
            }
        }
    }

    ts.log("IMAGE TIMES: discover categories");
    // Category discovery MUST BE LAST to avoid potential races 
    // when other threads call the new category code before 
    // this thread finishes its fixups.
```

这里主要做：
1. 把 `Category` 的实例方法、协议以及属性添加到类上；
2. 把 `Category` 的类方法和协议添加到类的 `MetaClass` 上；

`addUnattachedCategoryForClass` 只是把类和 `Category` 做一个关联映射，而`remethodizeClass` 负责添加各个列表到类上。

```c
/***********************************************************************
* remethodizeClass
* Attach outstanding categories to an existing class.
* Fixes up cls's method list, protocol list, and property list.
* Updates method caches for cls and its subclasses.
* Locking: runtimeLock must be held by the caller
**********************************************************************/
static void remethodizeClass(Class cls)
{
    category_list *cats;
    bool isMeta;

    runtimeLock.assertLocked();

    isMeta = cls->isMetaClass();

    // Re-methodizing: check for more categories
    if ((cats = unattachedCategoriesForClass(cls, false/*not realizing*/))) {
        if (PrintConnecting) {
            _objc_inform("CLASS: attaching categories to class '%s' %s", 
                         cls->nameForLogging(), isMeta ? "(meta)" : "");
        }
        
        attachCategories(cls, cats, true /*flush caches*/);        
        free(cats);
    }
}
```

从上面代码中可以看出，最后真正执行添加操作是 `attachCategories`:

```c
// Attach method lists and properties and protocols from categories to a class.
// Assumes the categories in cats are all loaded and sorted by load order, 
// oldest categories first.
static void 
attachCategories(Class cls, category_list *cats, bool flush_caches)
{
    if (!cats) return;
    if (PrintReplacedMethods) printReplacements(cls, cats);

    bool isMeta = cls->isMetaClass();

    // fixme rearrange to remove these intermediate allocations
    method_list_t **mlists = (method_list_t **)
        malloc(cats->count * sizeof(*mlists));
    property_list_t **proplists = (property_list_t **)
        malloc(cats->count * sizeof(*proplists));
    protocol_list_t **protolists = (protocol_list_t **)
        malloc(cats->count * sizeof(*protolists));

    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int propcount = 0;
    int protocount = 0;
    int i = cats->count;
    bool fromBundle = NO;
    while (i--) {
        auto& entry = cats->list[i];

        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
            fromBundle |= entry.hi->isBundle();
        }

        property_list_t *proplist = 
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            proplists[propcount++] = proplist;
        }

        protocol_list_t *protolist = entry.cat->protocols;
        if (protolist) {
            protolists[protocount++] = protolist;
        }
    }

    auto rw = cls->data();

    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);

    rw->properties.attachLists(proplists, propcount);
    free(proplists);

    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}
```

需要注意的有两点：
1. `Category` 的方法没有“完全替换掉”原来类已经有的方法，也就是说如果 `Category` 和原来类都有methodA，那么 `Category` 附加完成之后，类的方法列表里会有两个 `methodA`。
2 `Category` 的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的 `Category` 的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，只要一找到对应名字的方法就会返回。

Category 与 Extension 的区别:
- Category
1. 是运行期决议的;
2. 类扩展可以添加实例变量，分类不能添加实例变量
-Extension
1. 在编译期决议，是类的一部分，在编译器和头文件的@interface和实现文件里的@implement一起形成了一个完整的类;
2. 伴随着类的产生而产生，也随着类的消失而消失。 Extension一般用来隐藏类的私有消息，你必须有一个类的源码才能添加一个类的Extension，所以对于系统一些类，如NSString，就无法添加类扩展。

示例代码：

```objc
#pragma mark - Catetory Demo
@interface NSObject (ORDCategoryDemo)
+ (void)hello;
@end
@implementation NSObject (ORDCategoryDemo)
- (void)hello
{
     NSLog(@"IMP: -[NSObject(ORDCategoryDemo) hello]");
}
@end

- (void)runtimeCategoryDemo {
    [NSObject hello];
    [[NSObject new] hello];
}
```

执行结果：

```
2020-01-05 18:54:36.584411+0800 ObjcRuntimeDemo[19513:2658648] IMP: -[NSObject(ORDCategoryDemo) hello]
2020-01-05 18:54:36.584519+0800 ObjcRuntimeDemo[19513:2658648] IMP: -[NSObject(ORDCategoryDemo) hello]
```

`objc runtime` 加载后 `NSObject` 的 `ORDCategoryDemo Category` 被加载，类方法 `+(void)hello` 没有 `IMP`，只会出现一个 `warning`。`-(void)hello` 被加到 `Class` 的 `Method list` 里，`Meta Class` 的方法列表里没有。

执行 `[NSObject hello]` 时，会在 `Meta Class` 的 `Method list` 里找，找不着就继续往 `super class` 里找，`NSObject Meta Clas` 的 `super class` 是 `NSObject` 本身，这时在  `NSObject` 的 `Method list` 里就有 `hello` 这个方法了，能够正常输出。

执行 `[[NSObject new] hello]` 就简单的多了，`[NSObject new]` 生成一个实例，实例的 `Method list` 是有 `hello` 方法的，于是正常输出。

### Protocol
`Protocol` 其实就是一个对象结构体:

```c
typedef struct objc_object Protocol;
```

### 相关操作函数
`Category` 操作函数信息都包含在 `objc_class` 中，我们可以通过 `objc_class` 的操作函数来获取分类的操作函数信息。
`Protocol` 相关操作函数如下：
```c
// 返回指定的协议
Protocol * _Nullable objc_getProtocol(const char * _Nonnull name);

// 获取运行时所知道的所有协议的数组
Protocol * __unsafe_unretained _Nonnull * _Nullable objc_copyProtocolList(unsigned int * _Nullable outCount);

// 创建新的协议实例
Protocol * _Nullable objc_allocateProtocol(const char * _Nonnull name);

// 在运行时中注册新创建的协议, 创建一个新协议后必须使用这个进行注册这个新协议，但是注册后不能够再修改和添加新方法。
void objc_registerProtocol(Protocol * _Nonnull proto);

// 为协议添加方法
void protocol_addMethodDescription(Protocol * _Nonnull proto, SEL _Nonnull name, const char * _Nullable types, BOOL isRequiredMethod, BOOL isInstanceMethod);
                              
// 添加一个已注册的协议到协议中
void protocol_addProtocol(Protocol * _Nonnull proto, Protocol * _Nonnull addition);

// 为协议添加属性
void
protocol_addProperty(Protocol * _Nonnull proto, const char * _Nonnull name, const objc_property_attribute_t * _Nullable attributes, unsigned int attributeCount, BOOL isRequiredProperty, BOOL isInstanceProperty);

// 返回协议名
const char * _Nonnull protocol_getName(Protocol * _Nonnull proto);

// 测试两个协议是否相等
BOOL protocol_isEqual(Protocol * _Nullable proto, Protocol * _Nullable other);

// 获取协议中指定条件的方法的方法描述数组
struct objc_method_description * _Nullable
protocol_copyMethodDescriptionList(Protocol * _Nonnull proto, BOOL isRequiredMethod, BOOL isInstanceMethod, unsigned int * _Nullable outCount);

// 获取协议中指定方法的方法描述
struct objc_method_description protocol_getMethodDescription(Protocol * _Nonnull proto, SEL _Nonnull aSel, BOOL isRequiredMethod, BOOL isInstanceMethod);

// 获取协议中的属性列表
objc_property_t _Nonnull * _Nullable protocol_copyPropertyList(Protocol * _Nonnull proto, unsigned int * _Nullable outCount);

// 获取协议的指定属性
objc_property_t _Nullable protocol_getProperty(Protocol * _Nonnull proto, const char * _Nonnull name, BOOL isRequiredProperty, BOOL isInstanceProperty);

// 获取协议采用的协议
Protocol * __unsafe_unretained _Nonnull * _Nullable protocol_copyProtocolList(Protocol * _Nonnull proto, unsigned int * _Nullable outCount);

// 查看协议是否采用了另一个协议
BOOL protocol_conformsToProtocol(Protocol * _Nullable proto, Protocol * _Nullable other);
```

## Block
`Block` 先关操作函数：

```c
// 创建一个指针函数的指针，该函数调用时会调用特定的block
IMP _Nonnull imp_implementationWithBlock(id _Nonnull block);

// 返回与IMP(使用imp_implementationWithBlock创建的)相关的block
id _Nullable imp_getBlock(IMP _Nonnull anImp);

// 解除block与IMP(使用imp_implementationWithBlock创建的)的关联关系，并释放block的拷贝
BOOL imp_removeBlock(IMP _Nonnull anImp);
```

示例代码：

```objc
@interface ORDBlockDemo : NSObject
@end
@implementation ORDBlockDemo
@end

- (void)runtimeBlockDemo {
    IMP imp = imp_implementationWithBlock(^(id obj, NSString *str) {
        NSLog(@"%@", str);
    });
    class_addMethod(ORDBlockDemo.class, @selector(testBlock:), imp, "v@:@");
    ORDBlockDemo *ordBlock = [[ORDBlockDemo alloc] init];
    [ordBlock performSelector:@selector(testBlock:) withObject:@"hello world!"];
}
```

执行结果：

```
2020-01-05 19:07:57.276405+0800 ObjcRuntimeDemo[25422:2857881] hello world!
```

## Runtime 应用
### 获取系统库信息
相关函数：

```objc
// 获取所有加载的objectivec框架和动态库的名称
const char * _Nonnull * _Nonnull objc_copyImageNames(unsigned int * _Nullable outCount);

// 获取指定类所在动态库
const char * _Nullable class_getImageName(Class _Nullable cls);

// 获取指定库或框架中所有类的类名
const char * _Nonnull * _Nullable objc_copyClassNamesForImage(const char * _Nonnull image, unsigned int * _Nullable outCount);
```

示例代码：

```objc
- (void)runtimeLibraryDemo {
    NSLog(@"获取指定类所在动态库");
    NSLog(@"UIView's Framework: %s", class_getImageName(NSClassFromString(@"UIView")));
    NSLog(@"获取指定库或框架中所有类的类名");
    int outCount = 0;
    const char ** classes = objc_copyClassNamesForImage(class_getImageName(NSClassFromString(@"UIView")), &outCount);
    for (int i = 0; i < outCount; i++) {
         NSLog(@"class name: %s", classes[i]);
    }
}
```

执行结果:

```
2020-01-05 19:18:31.781855+0800 ObjcRuntimeDemo[26323:2899299] 获取指定类所在动态库
2020-01-05 19:18:31.781977+0800 ObjcRuntimeDemo[26323:2899299] UIView's Framework: /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Library/Developer/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/System/Library/PrivateFrameworks/UIKitCore.framework/UIKitCore
2020-01-05 19:18:31.782050+0800 ObjcRuntimeDemo[26323:2899299] 获取指定库或框架中所有类的类名
2020-01-05 19:18:31.782517+0800 ObjcRuntimeDemo[26323:2899299] class name: UIContinuousPathIntroductionView
2020-01-05 19:18:31.782591+0800 ObjcRuntimeDemo[26323:2899299] class name: _UIContextMenuStyle
2020-01-05 19:18:31.782659+0800 ObjcRuntimeDemo[26323:2899299] class name: _UIPlatterSoftShadowView
2020-01-05 19:18:31.782713+0800 ObjcRuntimeDemo[26323:2899299] class name: UIAccessibilityContainerReferenceHolder
2020-01-05 19:18:31.782777+0800 ObjcRuntimeDemo[26323:2899299] class name: UIAccessibilityCustomAction
```


## 参考资料
- [Objective-C Runtime](https://developer.apple.com/documentation/objectivec/objective-c_runtime?language=objc)
- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html?language=objc#//apple_ref/doc/uid/TP40008048)
- [Objc Runtime 总结](http://www.starming.com/2015/04/01/objc-runtime/)

- [关联对象 AssociatedObject 完全解析](https://draveness.me/ao)
- [iOS 界的毒瘤：Method Swizzle](https://juejin.im/entry/5a1fceddf265da43310d9985)
- [How do I implement method swizzling?](http://stackoverflow.com/questions/5371601/how-do-i-implement-method-swizzling)
- [Method Swizzling](http://nshipster.com/method-swizzling/)
- [What are the Dangers of Method Swizzling in Objective C?](http://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objectivec)
- [深入理解Objective-C：Category](https://tech.meituan.com/2015/03/03/diveintocategory.html)