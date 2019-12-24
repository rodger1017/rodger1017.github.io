---
layout: post
title: NSArray & NSMutableArray 基本用法
---

## [NSArray](https://developer.apple.com/documentation/foundation/nsarray?language=objc) & [NSMutableArray](https://developer.apple.com/documentation/foundation/nsmutablearray?language=objc)

- `NSArray`: 不可变数组，是一个对象，是任意类型对象地址的集合，不能存放简单的数据类型`(int, float, NSInteger…)`，除非通过一些方法把简单数据类型变成对象。`NSArray` 建立之后就不可修改。
需要注意的是 `NSArray` 下标越界不会有警告，但是运行会直接报错，报错信息如下：
```
Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndexedSubscript:]: index 3 beyond bounds [0 .. 2]'
```

- `NSMutableArray`: 可变数组，是 `NSArray` 的子类，可以对数组进行增，删，改，查操作，使用比`NSArray` 更加方便，但缺少了 `NSArray` 的安全性。 

- 创建静态数组

```objc
+ (void)createArray {
    NSLog(@"***Creating an Array***");
    // 创建空数组
    NSArray *arr1 = [NSArray array];
    NSLog(@"arr1: %@", arr1);
    
    // 使用字面量创建数组
    // 注意不能用 nil, 编译报错：Collection element of type 'void *' is not an Objective-C object
    NSArray *arr2 = @[@"test", @1, @2.0];
    NSLog(@"arr2: %@", arr2);
    
    // 通过数组创建新数组
    NSArray *arr3 = [NSArray arrayWithArray:arr2];
    NSLog(@"arr3: %@", arr3);
    
    // 从文件获取
    NSString *filePath = @"./test.data";
    NSArray *arr4 = [NSArray arrayWithContentsOfFile:filePath];
    NSLog(@"arr4: %@", arr4);
    NSArray *arr5 = [NSArray arrayWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"arr5: %@", arr5);
    
    // 通过对象创建数组
    NSArray *arr6 = [NSArray arrayWithObject:@"test"];
    NSLog(@"arr6: %@", arr6);
    
    NSDate *aDate = [NSDate distantFuture];
    NSValue *aValue = @(5);
    NSString *aString = @"hello";
    // 不能缺少 nil， 告警提示：Missing sentinel in method dispatch
    // 运行时崩溃：Thread 1: EXC_BAD_ACCESS (code=1, address=0x40)
    NSArray *arr7 = [NSArray arrayWithObjects:aDate, aValue, aString, nil];
    NSLog(@"arr7: %@", arr7);
    
    // 通过c数组创建，取前count个元素
    NSString *strs[3];
    strs[0] = @"First";
    strs[1] = @"Second";
    strs[2] = @"Third";
    NSArray *arr8 = [NSArray arrayWithObjects:strs count:2];
    NSLog(@"arr8: %@", arr8);
}
```
输出：
```
2019-12-22 14:56:34.121122+0800 ObjcBaseDemo[39313:680680] ***Creating an Array***
2019-12-22 14:56:34.121275+0800 ObjcBaseDemo[39313:680680] arr1: (
)
2019-12-22 14:56:34.121398+0800 ObjcBaseDemo[39313:680680] arr2: (
    test,
    1,
    2
)
2019-12-22 14:56:34.121499+0800 ObjcBaseDemo[39313:680680] arr3: (
    test,
    1,
    2
)
2019-12-22 14:56:34.121598+0800 ObjcBaseDemo[39313:680680] arr4: (null)
2019-12-22 14:56:34.121780+0800 ObjcBaseDemo[39313:680680] arr5: (null)
2019-12-22 14:56:34.121884+0800 ObjcBaseDemo[39313:680680] arr6: (
    test
)
2019-12-22 14:56:34.123160+0800 ObjcBaseDemo[39313:680680] arr7: (
    "4001-01-01 00:00:00 +0000",
    5,
    hello
)
2019-12-22 14:56:34.123282+0800 ObjcBaseDemo[39313:680680] arr8: (
    First,
    Second
)
```

- 初始化静态数组

```objc
+ (void)initArray
{
    NSLog(@"***Initializing an Array***");
    // 空数组
    NSArray *arr1 = [[NSArray alloc]init];
    NSLog(@"arr1: %@", arr1);
    // 根据数组创建数组
    NSArray *arr2 = [[NSArray alloc] initWithArray:arr1];
    NSLog(@"arr2: %@", arr2);
    NSArray *arr3 = [[NSArray alloc] initWithArray:arr2 copyItems:YES];
    NSLog(@"arr3: %@", arr3);
    
    // 从文件获取
    NSString *filePath = [self testData];
    NSArray *arr4 = [[NSArray alloc] initWithContentsOfFile:filePath];
    NSLog(@"arr4: %@", arr4);
    NSArray *arr5 = [[NSArray alloc] initWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"arr5: %@", arr5);
    
    // 包含多个数据的数组
    NSArray *arr6 = [[NSArray alloc] initWithObjects:@"Hello", @"World", nil];
    NSLog(@"arr6: %@", arr6);
}
```
输出：
```
2019-12-22 14:56:34.123357+0800 ObjcBaseDemo[39313:680680] ***Initializing an Array***
2019-12-22 14:56:34.123433+0800 ObjcBaseDemo[39313:680680] arr1: (
)
2019-12-22 14:56:34.123523+0800 ObjcBaseDemo[39313:680680] arr2: (
)
2019-12-22 14:56:34.123596+0800 ObjcBaseDemo[39313:680680] arr3: (
)
2019-12-22 14:56:34.124749+0800 ObjcBaseDemo[39313:680680] writeToFile: 1
2019-12-22 14:56:34.124918+0800 ObjcBaseDemo[39313:680680] arr4: (
    One,
    Two
)
2019-12-22 14:56:34.125088+0800 ObjcBaseDemo[39313:680680] arr5: (
    One,
    Two
)
2019-12-22 14:56:34.125193+0800 ObjcBaseDemo[39313:680680] arr6: (
    Hello,
    World
)
```

- 查询与遍历元素

```objc
+ (void)queryArray
{
    NSLog(@"***Querying an Array***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", @"Four", @"Five", nil];
    // 判断是否存在
    NSString *str = @"Two";
    BOOL contain = [array containsObject:str];
    NSLog(@"array%@ containsObject %@", contain?@"":@" not", str);
    
    // 元素个数
    NSLog(@"array count:%lu", (unsigned long)[array count]);
    
    // 读取第一个元素
    NSLog(@"[array firstObject]: %@", [array firstObject]);
    NSLog(@"array[0]: %@", array[0]);
    
    // 读取最后一个元素
    NSLog(@"[array lastObject]: %@", [array lastObject]);
    
    // 读取第0个位置元素
    NSLog(@"[array objectAtIndex:1]: %@", [array objectAtIndex:0]);
    NSLog(@"[array objectAtIndexedSubscript:1]: %@", [array objectAtIndexedSubscript:0]);
    NSLog(@"[array objectsAtIndexes:]: %@", [array objectsAtIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(0,2)]]);
    // 枚举遍历获取
    NSEnumerator *enumerator = [array objectEnumerator];
    id anObject;
    while (anObject = [enumerator nextObject]) {
        NSLog(@"[array objectEnumerator]: %@", anObject);
    }
    
    enumerator = [array reverseObjectEnumerator];
    while (anObject = [enumerator nextObject]) {
        NSLog(@"[array reverseObjectEnumerator]: %@", anObject);
    }
    
    // 快速遍历
    for (anObject in array) {
        NSLog(@"[array for in]: %@", anObject);
    }
    
    // 获取C组数
    id *objects;
    NSRange range = NSMakeRange(1, 2);
    objects = malloc(sizeof(id) * range.length);
    [array getObjects:objects range:range];
    for (int i = 0; i < range.length; i++) {
        NSLog(@"[array getObjects:range:][%d]: %@", i, objects[i]);
    }
    free(objects);
}
```
输出：
```
2019-12-22 14:56:34.127973+0800 ObjcBaseDemo[39313:680680] ***Querying an Array***
2019-12-22 14:56:34.128067+0800 ObjcBaseDemo[39313:680680] array containsObject Two
2019-12-22 14:56:34.128149+0800 ObjcBaseDemo[39313:680680] array count:5
2019-12-22 14:56:34.128226+0800 ObjcBaseDemo[39313:680680] [array firstObject]: One
2019-12-22 14:56:34.128303+0800 ObjcBaseDemo[39313:680680] array[0]: One
2019-12-22 14:56:34.128377+0800 ObjcBaseDemo[39313:680680] [array lastObject]: Five
2019-12-22 14:56:34.128446+0800 ObjcBaseDemo[39313:680680] [array objectAtIndex:1]: One
2019-12-22 14:56:34.128522+0800 ObjcBaseDemo[39313:680680] [array objectAtIndexedSubscript:1]: One
2019-12-22 14:56:34.128691+0800 ObjcBaseDemo[39313:680680] [array objectsAtIndexes:]: (
    One,
    Two
)
2019-12-22 14:56:34.128935+0800 ObjcBaseDemo[39313:680680] [array objectEnumerator]: One
2019-12-22 14:56:34.129074+0800 ObjcBaseDemo[39313:680680] [array objectEnumerator]: Two
2019-12-22 14:56:34.129261+0800 ObjcBaseDemo[39313:680680] [array objectEnumerator]: Three
2019-12-22 14:56:34.129438+0800 ObjcBaseDemo[39313:680680] [array objectEnumerator]: Four
2019-12-22 14:56:34.129613+0800 ObjcBaseDemo[39313:680680] [array objectEnumerator]: Five
2019-12-22 14:56:34.129826+0800 ObjcBaseDemo[39313:680680] [array reverseObjectEnumerator]: Five
2019-12-22 14:56:34.130027+0800 ObjcBaseDemo[39313:680680] [array reverseObjectEnumerator]: Four
2019-12-22 14:56:34.130253+0800 ObjcBaseDemo[39313:680680] [array reverseObjectEnumerator]: Three
2019-12-22 14:56:34.130409+0800 ObjcBaseDemo[39313:680680] [array reverseObjectEnumerator]: Two
2019-12-22 14:56:34.130600+0800 ObjcBaseDemo[39313:680680] [array reverseObjectEnumerator]: One
2019-12-22 14:56:34.130826+0800 ObjcBaseDemo[39313:680680] [array for in]: One
2019-12-22 14:56:34.131068+0800 ObjcBaseDemo[39313:680680] [array for in]: Two
2019-12-22 14:56:34.131292+0800 ObjcBaseDemo[39313:680680] [array for in]: Three
2019-12-22 14:56:34.131511+0800 ObjcBaseDemo[39313:680680] [array for in]: Four
2019-12-22 14:56:34.131730+0800 ObjcBaseDemo[39313:680680] [array for in]: Five
2019-12-22 14:56:34.131951+0800 ObjcBaseDemo[39313:680680] [array getObjects:range:][0]: Two
2019-12-22 14:56:34.132172+0800 ObjcBaseDemo[39313:680680] [array getObjects:range:][1]: Three
```

- 查询元素位置

```objc
+ (void)findObjectInArray
{
    NSLog(@"***Finding Objects in an Array***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", @"Four", @"Five", nil];
    NSString *two = @"Two";
    NSString *three = @"Three";
    NSRange range = NSMakeRange(1, 2);
    NSLog(@"[array indexOfObjcet:%@]:%lu", two, (unsigned long)[array indexOfObject:two]);
    NSLog(@"[array indexOfObject:%@ inRange:range]:%lu", two, (unsigned long)[array indexOfObject:two inRange:range]);
    NSLog(@"[array indexOfObjectIdenticalTo:%@]:%lu", two, (unsigned long)[array indexOfObjectIdenticalTo:two]);
    NSLog(@"[array indexOfObjectIdenticalTo:%@ inRange:range]:%lu", two, (unsigned long)[array indexOfObjectIdenticalTo:two inRange:range]);
    // 串行查找单个
    NSInteger index = [array indexOfObjectPassingTest:^BOOL(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:two]) {
            *stop = YES;
            return YES;
        }
        return NO;
    }];
    NSLog(@"[array indexOfObjectPassingTest:...]: %lu", (unsigned long)index);
    
    // 并发查找单个
    index = [array indexOfObjectWithOptions:NSEnumerationConcurrent passingTest:^BOOL(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:two]) {
            *stop = YES;
            return YES;
        }
        return NO;
    }];
    NSLog(@"[array indexOfObjectWithOptions:passingTest:]: %lu", (unsigned long)index);
    
    // 串行查找多个
    NSIndexSet *set = [array indexesOfObjectsPassingTest:^BOOL(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:two] || [obj isEqualToString:three]) {
            return YES;
        }
        return NO;
    }];
    NSLog(@"[array indexesOfObjectsPassingTest:]: %@", set);
    
    // 并发查找多个
    set = [array indexesOfObjectsWithOptions:NSEnumerationConcurrent passingTest:^BOOL(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:two] || [obj isEqualToString:three]) {
            return YES;
        }
        return NO;
    }];
    NSLog(@"[array indexesOfObjectsWithOptions:passingTest:]: %@", set);
}
```
输出：
```
2019-12-22 14:56:34.132408+0800 ObjcBaseDemo[39313:680680] ***Finding Objects in an Array***
2019-12-22 14:56:34.132685+0800 ObjcBaseDemo[39313:680680] [array indexOfObjcet:Two]:1
2019-12-22 14:56:34.132932+0800 ObjcBaseDemo[39313:680680] [array indexOfObject:Two inRange:range]:1
2019-12-22 14:56:34.133139+0800 ObjcBaseDemo[39313:680680] [array indexOfObjectIdenticalTo:Two]:1
2019-12-22 14:56:34.133291+0800 ObjcBaseDemo[39313:680680] [array indexOfObjectIdenticalTo:Two inRange:range]:1
2019-12-22 14:56:34.133498+0800 ObjcBaseDemo[39313:680680] [array indexOfObjectPassingTest:...]: 1
2019-12-22 14:56:34.133735+0800 ObjcBaseDemo[39313:680680] [array indexOfObjectWithOptions:passingTest:]: 1
2019-12-22 14:56:34.134028+0800 ObjcBaseDemo[39313:680680] [array indexesOfObjectsPassingTest:]: <NSIndexSet: 0x6000030b56c0>[number of indexes: 2 (in 1 ranges), indexes: (1-2)]
2019-12-22 14:56:34.134297+0800 ObjcBaseDemo[39313:680680] [array indexesOfObjectsWithOptions:passingTest:]: <NSIndexSet: 0x6000030a8f40>[number of indexes: 2 (in 1 ranges), indexes: (1-2)]
```

- 向数组元素发送消息

```objc

@interface Person : NSObject

@property (nonatomic,strong)NSString * name;
@property (nonatomic,assign)int age;

- (id)initWithName:(NSString *)name andAge:(int)age;
- (void)printInfo;
- (void)say:(id)hello;

@end

@implementation Person

- (id)initWithName:(NSString *)name andAge:(int)age
{
    if (self = [super init])
    {
        _name = name;
        _age = age;
    }
    return self;
}

-(void)printInfo {
    NSLog(@"%@", [self description]);
}

- (void)say:(id)hello {
    NSLog(@"%@ say %@", _name, hello);
}

- (NSString *)description
{
    return [NSString stringWithFormat:@"age = %d name = %@", _age, _name];
}

@end

+ (void)sendMessageToArrayElements
{
    NSLog(@"***Sending Messages to Elements***");
    Person *p1 = [[Person alloc] initWithName:@"Tom" andAge:20];
    Person *p2 = [[Person alloc] initWithName:@"Jim" andAge:18];
    Person *p3 = [[Person alloc] initWithName:@"Kim" andAge:38];

    NSArray * array = [[NSArray alloc] initWithObjects:p1, p2, p3, nil];

    // 通知数组中的每个元素执行方法
    [array makeObjectsPerformSelector:@selector(printInfo)];
    
    // 携带参数发出通知
    [array makeObjectsPerformSelector:@selector(say:) withObject:@"hello"];
    
    // 串行自定义通知
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"[array enumerateObjectsUsingBlock:]: %lu", (unsigned long)idx);
        [obj say:@(idx)];
        
    }];
    
    // 并行自定义通知
    [array enumerateObjectsWithOptions:NSEnumerationConcurrent usingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"[array enumerateObjectsWithOptions:usingBlock:]: %lu", (unsigned long)idx);
        [obj say:@(idx)];
    }];
    
    // 根据索引发出通知
    NSIndexSet *indexSet = [[NSIndexSet alloc] initWithIndex:0];
    [array enumerateObjectsAtIndexes:indexSet options:NSEnumerationConcurrent usingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"[array enumerateObjectsAtIndexes:options:usingBlock:]: %lu", (unsigned long)idx);
        [obj say:@(idx)];
    }];
}
```
输出：
```
2019-12-22 14:56:34.134526+0800 ObjcBaseDemo[39313:680680] ***Sending Messages to Elements***
2019-12-22 14:56:34.134788+0800 ObjcBaseDemo[39313:680680] age = 20 name = Tom
2019-12-22 14:56:34.134903+0800 ObjcBaseDemo[39313:680680] age = 18 name = Jim
2019-12-22 14:56:34.135122+0800 ObjcBaseDemo[39313:680680] age = 38 name = Kim
2019-12-22 14:56:34.135375+0800 ObjcBaseDemo[39313:680680] Tom say hello
2019-12-22 14:56:34.135625+0800 ObjcBaseDemo[39313:680680] Jim say hello
2019-12-22 14:56:34.135846+0800 ObjcBaseDemo[39313:680680] Kim say hello
2019-12-22 14:56:34.136047+0800 ObjcBaseDemo[39313:680680] [array enumerateObjectsUsingBlock:]: 0
2019-12-22 14:56:34.136232+0800 ObjcBaseDemo[39313:680680] Tom say 0
2019-12-22 14:56:34.136390+0800 ObjcBaseDemo[39313:680680] [array enumerateObjectsUsingBlock:]: 1
2019-12-22 14:56:34.136567+0800 ObjcBaseDemo[39313:680680] Jim say 1
2019-12-22 14:56:34.136767+0800 ObjcBaseDemo[39313:680680] [array enumerateObjectsUsingBlock:]: 2
2019-12-22 14:56:34.136951+0800 ObjcBaseDemo[39313:680680] Kim say 2
2019-12-22 14:56:34.137129+0800 ObjcBaseDemo[39313:680680] [array enumerateObjectsWithOptions:usingBlock:]: 0
2019-12-22 14:56:34.137136+0800 ObjcBaseDemo[39313:680885] [array enumerateObjectsWithOptions:usingBlock:]: 1
2019-12-22 14:56:34.137140+0800 ObjcBaseDemo[39313:680879] [array enumerateObjectsWithOptions:usingBlock:]: 2
2019-12-22 14:56:34.137360+0800 ObjcBaseDemo[39313:680680] Tom say 0
2019-12-22 14:56:34.137496+0800 ObjcBaseDemo[39313:680879] Kim say 2
2019-12-22 14:56:34.137931+0800 ObjcBaseDemo[39313:680885] Jim say 1
2019-12-22 14:56:34.138068+0800 ObjcBaseDemo[39313:680680] [array enumerateObjectsAtIndexes:options:usingBlock:]: 0
2019-12-22 14:56:34.138217+0800 ObjcBaseDemo[39313:680680] Tom say 0
```

- 数组比较

```objc
+ (void)compareArray
{
    NSLog(@"***Comparing Arrays***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    NSArray<NSString *> *array2 = [NSArray arrayWithObjects:@"One", @"Two", nil];
    
    // 返回第一个相同的数据
    NSString *str = [array firstObjectCommonWithArray:array2];
    NSLog(@"[array firstObjectCommonWithArray:]: %@", str);
    
    // 数组内的内容是否相同
    BOOL isEqual = [array isEqualToArray:array2];
    NSLog(@"[array isEqualToArray:]: %d", isEqual);
}
```
输出：
```
2019-12-22 14:56:34.138363+0800 ObjcBaseDemo[39313:680680] ***Comparing Arrays***
2019-12-22 14:56:34.138555+0800 ObjcBaseDemo[39313:680680] [array firstObjectCommonWithArray:]: One
2019-12-22 14:56:34.138781+0800 ObjcBaseDemo[39313:680680] [array isEqualToArray:]: 0
```

- 生成新数组

```objc
+ (void)deriveNewArray
{
    NSLog(@"***Deriving New Arrays***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", nil];
    
    // 添加单个数据，并生成一个新的数组
    array = [array arrayByAddingObject:@"Three"];
    NSLog(@"[array arrayByAddingObject:]: %@", array);
    
    // 添加多个数据，并返回一个新的数组
    array = [array arrayByAddingObjectsFromArray:array];
    NSLog(@"[array arrayByAddingObjectsFromArray:]: %@", array);
    
    // 通过过滤器筛选数组
    NSPredicate *predicate = [NSPredicate predicateWithValue:YES];
    array = [array filteredArrayUsingPredicate:predicate];
    NSLog(@"[array filteredArrayUsingPredicate:]: %@", array);
    
    // 通过范围生成数组
    NSRange range = {0, 2};
    array = [array subarrayWithRange:range];
    NSLog(@"[array subarrayWithRange:]: %@", array);
}
```
输出：
```
2019-12-22 14:56:34.139033+0800 ObjcBaseDemo[39313:680680] ***Deriving New Arrays***
2019-12-22 14:56:34.139305+0800 ObjcBaseDemo[39313:680680] [array arrayByAddingObject:]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.139468+0800 ObjcBaseDemo[39313:680680] [array arrayByAddingObjectsFromArray:]: (
    One,
    Two,
    Three,
    One,
    Two,
    Three
)
2019-12-22 14:56:34.139704+0800 ObjcBaseDemo[39313:680680] [array filteredArrayUsingPredicate:]: (
    One,
    Two,
    Three,
    One,
    Two,
    Three
)
2019-12-22 14:56:34.139921+0800 ObjcBaseDemo[39313:680680] [array subarrayWithRange:]: (
    One,
    Two
)
```

- 数组排序

```objc
NSInteger sortByFunction(NSString * obj1, NSString * obj2, void * context)
{
    return [obj1 compare:obj2];
}


+ (void)sortArray
{
    NSLog(@"***Sorting***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    
    // Function 排序
    array = [array sortedArrayUsingFunction:sortByFunction context:nil];
    NSLog(@"[array sortedArrayUsingFunction:]: %@", array);
    NSData *sortedArrayHint = array.sortedArrayHint;
    array = [array sortedArrayUsingFunction:sortByFunction context:nil hint:sortedArrayHint];
    NSLog(@"[array sortedArrayUsingFunction:context:hint]: %@", array);
    
    // Selector 排序
    array = [array sortedArrayUsingSelector:@selector(compare:)];
    NSLog(@"[array sortedArrayUsingSelector:]: %@", array);
    
    // Block排序
    array = [array sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[array sortedArrayUsingComparator:^]: %@", array);
    
    // 并发block排序
    array = [array sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[array sortedArrayWithOptions:usingComparator^]: %@", array);
}
```
输出：
```
2019-12-22 14:56:34.140136+0800 ObjcBaseDemo[39313:680680] ***Sorting***
2019-12-22 14:56:34.140386+0800 ObjcBaseDemo[39313:680680] [array sortedArrayUsingFunction:]: (
    One,
    Three,
    Two
)
2019-12-22 14:56:34.140650+0800 ObjcBaseDemo[39313:680680] [array sortedArrayUsingFunction:context:hint]: (
    One,
    Three,
    Two
)
2019-12-22 14:56:34.140867+0800 ObjcBaseDemo[39313:680680] [array sortedArrayUsingSelector:]: (
    One,
    Three,
    Two
)
2019-12-22 14:56:34.141012+0800 ObjcBaseDemo[39313:680680] [array sortedArrayUsingComparator:^]: (
    One,
    Three,
    Two
)
2019-12-22 14:56:34.141209+0800 ObjcBaseDemo[39313:680680] [array sortedArrayWithOptions:usingComparator^]: (
    One,
    Three,
    Two
)
```

- 处理字符串数组

```objc
+ (void)workWithStringArrayElements
{
    NSLog(@"***Working with String Elements***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    
    // 数组中的NSString元素拼接
    NSString *str = [array componentsJoinedByString:@","];
    NSLog(@"[array componentsJoinedByString:]: %@", str);
}
```
输出：
```
2019-12-22 14:56:34.141433+0800 ObjcBaseDemo[39313:680680] ***Working with String Elements***
2019-12-22 14:56:34.141644+0800 ObjcBaseDemo[39313:680680] [array componentsJoinedByString:]: One,Two,Three
```

- 创建描述信息

```objc
+ (void)creatArrayDescription {
    NSLog(@"***Creating a Description***");
    NSArray<NSString *> *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    
    // 描述信息
    NSString *description = [array description];
    NSLog(@"[array description]: %@", description);
    description = [array descriptionWithLocale:nil];
    NSLog(@"[array descriptionWithLocale:]: %@", description);
    description = [array descriptionWithLocale:nil indent:1];
    NSLog(@"[array descriptionWithLocale:indent:]: %@", description);
    
    // 获取Document文件夹
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    // 存储的路径
    NSString *filePath = [documentsDirectory stringByAppendingPathComponent:@"test.plist"];
    
    // 写入
    BOOL write = [array writeToFile:filePath atomically:YES];
    write = [array writeToURL:[NSURL fileURLWithPath:filePath] atomically:YES];
}
```
输出：
```
2019-12-22 14:56:34.141875+0800 ObjcBaseDemo[39313:680680] ***Creating a Description***
2019-12-22 14:56:34.142017+0800 ObjcBaseDemo[39313:680680] [array description]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.142251+0800 ObjcBaseDemo[39313:680680] [array descriptionWithLocale:]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.142484+0800 ObjcBaseDemo[39313:680680] [array descriptionWithLocale:indent:]:     (
        One,
        Two,
        Three
    )
```

- 测试数据

```objc
+ (NSString *)testData {
    // 获取应用中Document文件夹
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    
    // 测试数据
    NSArray *array = [NSArray arrayWithObjects:@"One", @"Two", nil];
    NSString *filePath = [documentsDirectory stringByAppendingPathComponent:@"test.plist"];
    BOOL write = [array writeToFile:filePath atomically:YES]; // 输入写入
    NSLog(@"writeToFile: %d", write);
    
    return filePath;
}
```

- 动态数组初始化

```objc
+ (void)creatAndInitMutalbeArray {
    NSLog(@"***Creating and Initializing a Mutable Array***");
    NSString *filePath = [self testData];
    
    // (+)创建
    NSMutableArray *mArray = [NSMutableArray arrayWithCapacity:1];
    NSLog(@"[NSMutableArray arrayWithCapacity:]: %@", mArray);
    // 根据文件路径创建数组
    mArray = [NSMutableArray arrayWithContentsOfFile:filePath];
    NSLog(@"[NSMutableArray arrayWithContentsOfFile:]: %@", mArray);
    mArray = [NSMutableArray arrayWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"[NSMutableArray arrayWithContentsOfURL:]: %@", mArray);
    
    // (-)创建
    mArray = [[NSMutableArray alloc] initWithCapacity:1];
    NSLog(@"[[NSMutableArray alloc] initWithCapacity:]: %@", mArray);
    mArray = [[NSMutableArray alloc] initWithContentsOfFile:filePath];
    NSLog(@"[[NSMutableArray alloc] initWithContentsOfFile:]: %@", mArray);
    mArray = [[NSMutableArray alloc] initWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"[[NSMutableArray alloc] initWithContentsOfURL:[NSURL fileURLWithPath:]]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.143999+0800 ObjcBaseDemo[39313:680680] ***Creating and Initializing a Mutable Array***
2019-12-22 14:56:34.145403+0800 ObjcBaseDemo[39313:680680] writeToFile: 1
2019-12-22 14:56:34.145515+0800 ObjcBaseDemo[39313:680680] [NSMutableArray arrayWithCapacity:]: (
)
2019-12-22 14:56:34.145678+0800 ObjcBaseDemo[39313:680680] [NSMutableArray arrayWithContentsOfFile:]: (
    One,
    Two
)
2019-12-22 14:56:34.145845+0800 ObjcBaseDemo[39313:680680] [NSMutableArray arrayWithContentsOfURL:]: (
    One,
    Two
)
2019-12-22 14:56:34.145934+0800 ObjcBaseDemo[39313:680680] [[NSMutableArray alloc] initWithCapacity:]: (
)
2019-12-22 14:56:34.146075+0800 ObjcBaseDemo[39313:680680] [[NSMutableArray alloc] initWithContentsOfFile:]: (
    One,
    Two
)
2019-12-22 14:56:34.146251+0800 ObjcBaseDemo[39313:680680] [[NSMutableArray alloc] initWithContentsOfURL:[NSURL fileURLWithPath:]]: (
    One,
    Two
)
```

- 动态数组增加元素

```objc
+ (void)addMutableArray {
    NSLog(@"***Adding Objects***");
    NSMutableArray *mArray = [NSMutableArray array];
    NSLog(@"[mArray]: %@", mArray);
    // 增加单一数据
    [mArray addObject:@"One"];
    NSLog(@"[mArray addObject:]: %@", mArray);
    
    // 批量添加数据
    NSArray *array = [NSArray arrayWithObjects:@"Two", @"Three", nil];
    [mArray addObjectsFromArray:array];
    NSLog(@"[mArray addObjectsFromArray:]: %@", mArray);
    
    // 指定位置插入单一数据
    [mArray insertObject:@"Zero" atIndex:0];
    NSLog(@"[mArray insertObject:]: %@", mArray);
    
    // 指定位置插入多个数据
    NSRange range = {1, array.count};
    NSIndexSet *indexSet = [[NSIndexSet alloc] initWithIndexesInRange:range];
    [mArray insertObjects:array atIndexes:indexSet];
    NSLog(@"[mArray insertObjects:atIndexes]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.146331+0800 ObjcBaseDemo[39313:680680] ***Adding Objects***
2019-12-22 14:56:34.146403+0800 ObjcBaseDemo[39313:680680] [mArray]: (
)
2019-12-22 14:56:34.146473+0800 ObjcBaseDemo[39313:680680] [mArray addObject:]: (
    One
)
2019-12-22 14:56:34.146570+0800 ObjcBaseDemo[39313:680680] [mArray addObjectsFromArray:]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.146646+0800 ObjcBaseDemo[39313:680680] [mArray insertObject:]: (
    Zero,
    One,
    Two,
    Three
)
2019-12-22 14:56:34.146734+0800 ObjcBaseDemo[39313:680680] [mArray insertObjects:atIndexes]: (
    Zero,
    Two,
    Three,
    One,
    Two,
    Three
)
```

- 动态数组删除数据

```objc
+ (void)removMutalbeArray {
    NSLog(@"***Removing Objects***");
    NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"One", @"Two", @"Three", @"Four", @"Five", nil];
    NSLog(@"[mArray]: %@", mArray);
    // 删除所有元素
    [mArray removeAllObjects];
    NSLog(@"[mArray removeAllObjects]: %@", mArray);
    
    // 删除最后一个元素
    mArray = [NSMutableArray arrayWithObjects:@"Zero", @"One", @"Two", @"Three", @"Four", @"Five", nil];
    NSLog(@"[mArray]: %@", mArray);
    [mArray removeLastObject];
    NSLog(@"[mArray removeLastObject]: %@", mArray);
    
    // 根据位置删除对象
    [mArray removeObjectAtIndex:0];
    NSLog(@"[mArray removeObjectAtIndex]: %@", mArray);
    
    // 根据数组删除对象
    NSArray *array = [NSArray arrayWithObjects:@"Three", @"Four", nil];
    [mArray removeObjectsInArray:array];
    NSLog(@"[mArray removeObjectAtIndex]: %@", mArray);
    
    // 根据对象删除
    [mArray removeObject:@"Two"];
    NSLog(@"[mArray removeObject]: %@", mArray);
    [mArray removeObjectIdenticalTo:@"One"];
    NSLog(@"[mArray removeObjectIdenticalTo]: %@", mArray);
    
    // 删除指定范围内的对象
    NSRange range = {0, 2};// 第0个位置开始，连续2个
    mArray = [NSMutableArray arrayWithObjects:@"Zero", @"One", @"Two", @"Three", @"Four", @"Five", nil];
    NSLog(@"[mArray]: %@", mArray);
    [mArray removeObject:@"One" inRange:range];
    NSLog(@"[mArray removeObject:inRange:]: %@", mArray);
    [mArray removeObjectIdenticalTo:@"Two" inRange:range];
    NSLog(@"[mArray removeObjectIdenticalTo:inRange:]: %@", mArray);
    
    // 删除指定NSRange范围内的对象，批量删除
    [mArray removeObjectsInRange:range];
    NSLog(@"[mArray removeObjectsInRange:]: %@", mArray);
    NSIndexSet *indexSet = [NSIndexSet indexSetWithIndexesInRange:range];
    [mArray removeObjectsAtIndexes:indexSet];
    NSLog(@"[mArray removeObjectsAtIndexes:]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.146885+0800 ObjcBaseDemo[39313:680680] ***Removing Objects***
2019-12-22 14:56:34.147142+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    One,
    Two,
    Three,
    Four,
    Five
)
2019-12-22 14:56:34.147347+0800 ObjcBaseDemo[39313:680680] [mArray removeAllObjects]: (
)
2019-12-22 14:56:34.147532+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    Zero,
    One,
    Two,
    Three,
    Four,
    Five
)
2019-12-22 14:56:34.147745+0800 ObjcBaseDemo[39313:680680] [mArray removeLastObject]: (
    Zero,
    One,
    Two,
    Three,
    Four
)
2019-12-22 14:56:34.147984+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectAtIndex]: (
    One,
    Two,
    Three,
    Four
)
2019-12-22 14:56:34.148228+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectAtIndex]: (
    One,
    Two
)
2019-12-22 14:56:34.148420+0800 ObjcBaseDemo[39313:680680] [mArray removeObject]: (
    One
)
2019-12-22 14:56:34.148598+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectIdenticalTo]: (
)
2019-12-22 14:56:34.148793+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    Zero,
    One,
    Two,
    Three,
    Four,
    Five
)
2019-12-22 14:56:34.149002+0800 ObjcBaseDemo[39313:680680] [mArray removeObject:inRange:]: (
    Zero,
    Two,
    Three,
    Four,
    Five
)
2019-12-22 14:56:34.149252+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectIdenticalTo:inRange:]: (
    Zero,
    Three,
    Four,
    Five
)
2019-12-22 14:56:34.149505+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectsInRange:]: (
    Four,
    Five
)
2019-12-22 14:56:34.149744+0800 ObjcBaseDemo[39313:680680] [mArray removeObjectsAtIndexes:]: (
)
```

- 动态数组替换元素

```objc
+ (void)replacMutableArray{
    NSLog(@"***Replacing Objects***");
    NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"[mArray]: %@", mArray);
    NSArray *array = [NSArray arrayWithObjects:@"1", @"2", @"3", nil];
    NSRange range = {0, array.count};// 第0个位置开始，连续count个
    NSIndexSet *indexSet = [NSIndexSet indexSetWithIndexesInRange:range];
    
    // 指定位置替换对象
    [mArray replaceObjectAtIndex:0 withObject:@"I"];
    NSLog(@"[mArray replaceObjectAtIndex:]: %@", mArray);
    [mArray setObject:@"II" atIndexedSubscript:1];
    
    // 数组替换
    [mArray setArray:array];
    NSLog(@"[mArray setArray:]: %@", mArray);
    
    // 用array替换数组中指定位置的所有元素
    mArray = [NSMutableArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"[mArray]: %@", mArray);
    [mArray replaceObjectsInRange:range withObjectsFromArray:array];
    NSLog(@"[mArray replaceObjectsInRange:withObjectsFromArray:]: %@", mArray);
    mArray = [NSMutableArray arrayWithObjects:@"I", @"II", @"III", nil];
    NSLog(@"[mArray]: %@", mArray);
    [mArray replaceObjectsAtIndexes:indexSet withObjects:array];
    NSLog(@"[mArray replaceObjectsAtIndexes:withObjects:]: %@", mArray);
    
    // 局部替换，使用array中的部分元素替换目标数组指定位置的元素
    mArray = [NSMutableArray arrayWithObjects:@"Un", @"Deux", @"Trois", nil];
    NSLog(@"[mArray]: %@", mArray);
    range.length = 2;
    [mArray replaceObjectsInRange:range withObjectsFromArray:array range:range];
    NSLog(@"[mArray replaceObjectsInRange:withObjectsFromArray:]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.149942+0800 ObjcBaseDemo[39313:680680] ***Replacing Objects***
2019-12-22 14:56:34.150162+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.150399+0800 ObjcBaseDemo[39313:680680] [mArray replaceObjectAtIndex:]: (
    I,
    Two,
    Three
)
2019-12-22 14:56:34.150578+0800 ObjcBaseDemo[39313:680680] [mArray setArray:]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.150802+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.150976+0800 ObjcBaseDemo[39313:680680] [mArray replaceObjectsInRange:withObjectsFromArray:]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.151186+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    I,
    II,
    III
)
2019-12-22 14:56:34.151361+0800 ObjcBaseDemo[39313:680680] [mArray replaceObjectsAtIndexes:withObjects:]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.151606+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    Un,
    Deux,
    Trois
)
2019-12-22 14:56:34.151826+0800 ObjcBaseDemo[39313:680680] [mArray replaceObjectsInRange:withObjectsFromArray:]: (
    1,
    2,
    Trois
)
```

- 动态数组过滤

```objc
+ (void)filterMutalbeArray {
    NSLog(@"***Filtering Content***");
    NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"[mArray]: %@", mArray);
    
    // 使用过滤器过滤数组中的元素
    NSPredicate *predicate = [NSPredicate predicateWithBlock:^BOOL(id  _Nonnull evaluatedObject, NSDictionary<NSString *,id> * _Nullable bindings) {
        BOOL filter = NO;
        if ([@"Two" isEqualToString:evaluatedObject]) {
            filter = YES;
        }
        NSLog(@"evaluatedObject: %@ predicate: %d", evaluatedObject, filter);
        return filter;
    }];
    [mArray filterUsingPredicate:predicate];
    NSLog(@"[mArray filterUsingPredicate]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.152049+0800 ObjcBaseDemo[39313:680680] ***Filtering Content***
2019-12-22 14:56:34.152268+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    One,
    Two,
    Three
)
2019-12-22 14:56:34.152493+0800 ObjcBaseDemo[39313:680680] evaluatedObject: One predicate: 0
2019-12-22 14:56:34.152707+0800 ObjcBaseDemo[39313:680680] evaluatedObject: Two predicate: 1
2019-12-22 14:56:34.152945+0800 ObjcBaseDemo[39313:680680] evaluatedObject: Three predicate: 0
2019-12-22 14:56:34.153183+0800 ObjcBaseDemo[39313:680680] [mArray filterUsingPredicate]: (
    Two
)
```

- 动态数组排序

```objc
NSInteger mSortByFunction(NSString * obj1, NSString * obj2, void * context) {
    return [obj1 compare:obj2];
}


+ (void)sortMutableArray {
    NSLog(@"***Rearranging Content***");
    
    NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"1", @"2", @"3", nil];
    NSLog(@"[mArray]: %@", mArray);
    
    // 交换两个位置的数据
    [mArray exchangeObjectAtIndex:0 withObjectAtIndex:1];
    NSLog(@"[mArray exchangeObjectAtIndex:withObjectAtIndex]: %@", mArray);
    
    // 对象自带的方法排序
    [mArray sortUsingSelector:@selector(compare:)];
    NSLog(@"[mArray sortUsingSelector]: %@", mArray);
    
    // Function排序
    [mArray sortUsingFunction:mSortByFunction context:nil];
    NSLog(@"[mArray sortUsingFunction]: %@", mArray);
    
    // block排序
    [mArray sortUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[mArray sortUsingComparator:^]: %@", mArray);
    
    // 并发排序
    [mArray sortWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[mArray sortWithOptions:usingComparator^]: %@", mArray);
}
```
输出：
```
2019-12-22 14:56:34.153411+0800 ObjcBaseDemo[39313:680680] ***Rearranging Content***
2019-12-22 14:56:34.153619+0800 ObjcBaseDemo[39313:680680] [mArray]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.153833+0800 ObjcBaseDemo[39313:680680] [mArray exchangeObjectAtIndex:withObjectAtIndex]: (
    2,
    1,
    3
)
2019-12-22 14:56:34.154013+0800 ObjcBaseDemo[39313:680680] [mArray sortUsingSelector]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.154231+0800 ObjcBaseDemo[39313:680680] [mArray sortUsingFunction]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.154490+0800 ObjcBaseDemo[39313:680680] [mArray sortUsingComparator:^]: (
    1,
    2,
    3
)
2019-12-22 14:56:34.154715+0800 ObjcBaseDemo[39313:680680] [mArray sortWithOptions:usingComparator^]: (
    1,
    2,
    3
)
```

[demo地址](https://github.com/rodger1017/Study/tree/master/ObjcBaseDemo)