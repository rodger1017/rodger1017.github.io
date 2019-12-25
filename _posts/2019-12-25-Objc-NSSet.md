---
title: NSSet & NSMutalbeSet & NSCountedSet 基本用法
layout: posts
category: iOS
tag: objc
---

## [NSSet](https://developer.apple.com/documentation/foundation/nsset?language=objc) & [NSMutableSet](https://developer.apple.com/documentation/foundation/nsmutableset?language=objc) & [NSCountedSet](https://developer.apple.com/documentation/foundation/nscountedset?language=objc)

- 创建集合

```objc
+ (void)createSet
{
    NSLog(@"***Creating a Set***");
    // 空集合
    NSSet *set = [NSSet set];
    NSLog(@"[NSSet set]: %@", set);
    
    // 通过数组
    NSArray *array = [NSArray arrayWithObjects:@"One", @"Two", @"Three", nil];
    set = [NSSet setWithArray:array];
    NSLog(@"[NSSet setWithArray:]: %@", set);
    
    // 只包含一个元素
    set = [NSSet setWithObject:@"One"];
    NSLog(@"[NSSet setWithObject:]: %@", set);
    
    //多个初始化
    set = [NSSet setWithObjects:@"One", @"Two",nil];
    NSLog(@"[NSSet setWithObjects:]: %@", set);
    
    //通过集合创建
    set = [NSSet setWithSet:set];
    NSLog(@"[NSSet setWithSet:]: %@", set);
    
    // 通过集合增加一个元素
    NSSet *set1 = [set setByAddingObject:@"Three"];
    NSLog(@"[set setByAddingObject:]: %@", set1);
    
    // 通过集合增加元素
    NSSet *set2 = [set setByAddingObjectsFromSet:set1];
    NSLog(@"[set setWithArray:]: %@", set2);
    
     // 通过数组增加元素
    set2 = [set setByAddingObjectsFromArray:array];
    NSLog(@"[set setWithArray:]: %@", set2);
}
```
输出:
```
2019-12-25 13:56:28.034212+0800 ObjcBaseDemo[6720:4895481] ***Creating a Set***
2019-12-25 13:56:28.034360+0800 ObjcBaseDemo[6720:4895481] [NSSet set]: {(
)}
2019-12-25 13:56:28.034444+0800 ObjcBaseDemo[6720:4895481] [NSSet setWithArray:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.034524+0800 ObjcBaseDemo[6720:4895481] [NSSet setWithObject:]: {(
    One
)}
2019-12-25 13:56:28.034607+0800 ObjcBaseDemo[6720:4895481] [NSSet setWithObjects:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.034683+0800 ObjcBaseDemo[6720:4895481] [NSSet setWithSet:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.034763+0800 ObjcBaseDemo[6720:4895481] [set setByAddingObject:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.034836+0800 ObjcBaseDemo[6720:4895481] [set setWithArray:]: {(
    One,
    Three,
    Two
)}
2019-12-25 13:56:28.034919+0800 ObjcBaseDemo[6720:4895481] [set setWithArray:]: {(
    One,
    Three,
    Two
)}
```

- 初始化集合

```objc
+ (void)initSet
{
    NSLog(@"***Initializing a Set***");
    NSArray *array = [NSArray arrayWithObjects:@"One", @"Two", @"One", nil];
    
    //空set
    NSSet *set = [[NSSet alloc] init];
    NSLog(@"[[NSSet alloc] init]: %@", set);
    
    // 通过数组
    set = [[NSSet alloc] initWithArray:array];
    NSLog(@"[[NSSet alloc] initWithArray:]: %@", set);
    
    // 多个初始化
    set = [[NSSet alloc] initWithObjects:@"One", @"Two", nil];
    NSLog(@"[[NSSet alloc] initWithObjects:]: %@", set);
    
    // 通过集合初始化
    set = [[NSSet alloc] initWithSet:set];
    NSLog(@"[[NSSet alloc] initWithSet:]: %@", set);
    set = [[NSSet alloc] initWithSet:set copyItems:YES];
    NSLog(@"[[NSSet alloc] initWithSet:copyItems:]: %@", set);
}
```
输出:
```
2019-12-25 13:56:28.034992+0800 ObjcBaseDemo[6720:4895481] ***Initializing a Set***
2019-12-25 13:56:28.035075+0800 ObjcBaseDemo[6720:4895481] [[NSSet alloc] init]: {(
)}
2019-12-25 13:56:28.035174+0800 ObjcBaseDemo[6720:4895481] [[NSSet alloc] initWithArray:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.035390+0800 ObjcBaseDemo[6720:4895481] [[NSSet alloc] initWithObjects:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.035584+0800 ObjcBaseDemo[6720:4895481] [[NSSet alloc] initWithSet:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.038543+0800 ObjcBaseDemo[6720:4895481] [[NSSet alloc] initWithSet:copyItems:]: {(
    One,
    Two
)}
```

- 计算元素数量

```objc
+ (void)countEntries
{
    NSLog(@"***Counting Entries***");
    NSSet *set = [NSSet setWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"set: %@ count: %lu", set, (unsigned long)set.count);
}
```
输出:
```
2019-12-25 13:56:28.038654+0800 ObjcBaseDemo[6720:4895481] ***Counting Entries***
2019-12-25 13:56:28.038750+0800 ObjcBaseDemo[6720:4895481] set: {(
    One,
    Two,
    Three
)} count: 3
```

- 访问集合元素

```objc
+ (void)accessSetMembers
{
    NSLog(@"***Accessing Set Members***");
    NSSet<NSString *> *set = [NSSet setWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"set: %@", set);
    // 所有成员
    NSArray *array = set.allObjects;
    NSLog(@"set.allObjects:%@", array);

    // 随机获取
    NSString *anyObject = [set anyObject];
    NSLog(@"set.anyObject: %@", anyObject);
    
    // 是否存在某个对象
    BOOL containsObject = [set containsObject:anyObject];
    NSLog(@"[set containsObject:]: %d", containsObject);

    // 通过过滤器过滤
    NSPredicate *predicate = [NSPredicate predicateWithBlock:^BOOL(id  _Nonnull evaluatedObject, NSDictionary<NSString *,id> * _Nullable bindings) {
        return YES;
    }];
    set = [set filteredSetUsingPredicate:predicate];
    NSLog(@"[set filteredSetUsingPredicate:]: %@", set);
    
    // block 过滤
    set = [set objectsPassingTest:^BOOL(NSString * _Nonnull obj, BOOL * _Nonnull stop) {
        return YES;
    }];
    NSLog(@"[set objectsPassingTest:^]: %@", set);
    
    // 并发 block 过滤
    set = [set objectsWithOptions:NSEnumerationConcurrent passingTest:^BOOL(NSString * _Nonnull obj, BOOL * _Nonnull stop) {
        return YES;
    }];
    NSLog(@"[set objectsWithOptions:passingTest:^]: %@", set);
    
    // 给每个成员发送消息
    [set makeObjectsPerformSelector:@selector(length)];
    
    // 给每个成员发送消息并携带数据
    [set makeObjectsPerformSelector:@selector(hasPrefix:) withObject:@"Three"];
    
    // 当存在这个对象时，返回这个对象
    NSString *member = @"Three";
    member = [set member:member];
    NSLog(@"[set member:]: %@", member);
    
    // for in 遍历
    for (id value in set) {
        NSLog(@"for in value: %@", value);
    }
    
    // enum 遍历
    NSEnumerator *enumerator = [set objectEnumerator];
    id value;
    while (value = [enumerator nextObject]) {
        /* code that acts on the set’s values */
        NSLog(@"[enumerator nextObject]: %@", value);
    }
    
    // block 遍历
    [set enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, BOOL * _Nonnull stop) {
        NSLog(@"[set enumerateObjectsUsingBlock:^]: %@", obj);
    }];
    
    // 并发 block 遍历
    [set enumerateObjectsWithOptions:NSEnumerationConcurrent usingBlock:^(NSString * _Nonnull obj, BOOL * _Nonnull stop) {
        NSLog(@"[set enumerateObjectsWithOptions:usingBlock:^]: %@", obj);
    }];
}
```
输出:
```
2019-12-25 13:56:28.038816+0800 ObjcBaseDemo[6720:4895481] ***Accessing Set Members***
2019-12-25 13:56:28.038874+0800 ObjcBaseDemo[6720:4895481] set: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.038973+0800 ObjcBaseDemo[6720:4895481] set.allObjects:(
    One,
    Two,
    Three
)
2019-12-25 13:56:28.039046+0800 ObjcBaseDemo[6720:4895481] set.anyObject: One
2019-12-25 13:56:28.039110+0800 ObjcBaseDemo[6720:4895481] [set containsObject:]: 1
2019-12-25 13:56:28.039337+0800 ObjcBaseDemo[6720:4895481] [set filteredSetUsingPredicate:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.039517+0800 ObjcBaseDemo[6720:4895481] [set objectsPassingTest:^]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.039772+0800 ObjcBaseDemo[6720:4895481] [set objectsWithOptions:passingTest:^]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.040016+0800 ObjcBaseDemo[6720:4895481] [set member:]: Three
2019-12-25 13:56:28.040185+0800 ObjcBaseDemo[6720:4895481] for in value: One
2019-12-25 13:56:28.040354+0800 ObjcBaseDemo[6720:4895481] for in value: Two
2019-12-25 13:56:28.040557+0800 ObjcBaseDemo[6720:4895481] for in value: Three
2019-12-25 13:56:28.040697+0800 ObjcBaseDemo[6720:4895481] [enumerator nextObject]: One
2019-12-25 13:56:28.040871+0800 ObjcBaseDemo[6720:4895481] [enumerator nextObject]: Two
2019-12-25 13:56:28.041064+0800 ObjcBaseDemo[6720:4895481] [enumerator nextObject]: Three
2019-12-25 13:56:28.041305+0800 ObjcBaseDemo[6720:4895481] [set enumerateObjectsUsingBlock:^]: One
2019-12-25 13:56:28.041527+0800 ObjcBaseDemo[6720:4895481] [set enumerateObjectsUsingBlock:^]: Two
2019-12-25 13:56:28.041760+0800 ObjcBaseDemo[6720:4895481] [set enumerateObjectsUsingBlock:^]: Three
2019-12-25 13:56:28.041957+0800 ObjcBaseDemo[6720:4895481] [set enumerateObjectsWithOptions:usingBlock:^]: One
2019-12-25 13:56:28.041967+0800 ObjcBaseDemo[6720:4895688] [set enumerateObjectsWithOptions:usingBlock:^]: Two
2019-12-25 13:56:28.041967+0800 ObjcBaseDemo[6720:4895687] [set enumerateObjectsWithOptions:usingBlock:^]: Three
```

- 集合比较

```objc
+ (void)compareSets
{
    NSLog(@"***Comparing Sets***");
    NSSet<NSString *> *set = [NSSet setWithObjects:@"One", @"Two", nil];
    NSLog(@"set: %@", set);
    NSSet<NSString *> *set2 = [NSSet setWithObjects:@"One", @"Two", @"Three", nil];
    NSLog(@"set2: %@", set2);
    
    // 是否子集合
    BOOL isBool = [set isSubsetOfSet:set2];
    NSLog(@"[set isSubsetOfSet:]: %d", isBool);
    
    // 是否有交集
    isBool = [set intersectsSet:set2];
    NSLog(@"[set intersectsSet:]: %d", isBool);
    
    // 是否相等
    isBool = [set isEqualToSet:set2];
    NSLog(@"[set isEqualToSet:]: %d", isBool);
}
```
输出:
```
2019-12-25 13:56:28.042452+0800 ObjcBaseDemo[6720:4895481] ***Comparing Sets***
2019-12-25 13:56:28.042584+0800 ObjcBaseDemo[6720:4895481] set: {(
    One,
    Two
)}
2019-12-25 13:56:28.042801+0800 ObjcBaseDemo[6720:4895481] set2: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.042965+0800 ObjcBaseDemo[6720:4895481] [set isSubsetOfSet:]: 1
2019-12-25 13:56:28.043199+0800 ObjcBaseDemo[6720:4895481] [set intersectsSet:]: 1
2019-12-25 13:56:28.043421+0800 ObjcBaseDemo[6720:4895481] [set isEqualToSet:]: 0
```

- 集合排序

```objc
+ (void)sortSet
{
    NSLog(@"***Creating a Sorted Array***");
    // 降序排列
    NSSet<NSString *> *set = [NSSet setWithObjects:@"1", @"3", @"2", nil];
    NSSortDescriptor *sortDes = [NSSortDescriptor sortDescriptorWithKey:nil ascending:NO];
    NSArray<NSSortDescriptor *> *sArray = [NSArray arrayWithObjects:sortDes, nil];
    NSArray<NSString *> *array = [set sortedArrayUsingDescriptors:sArray];
    NSLog(@"[set sortedArrayUsingDescriptors:]: %@", array);
}
```
输出:
```
2019-12-25 13:56:28.043691+0800 ObjcBaseDemo[6720:4895481] ***Creating a Sorted Array***
2019-12-25 13:56:28.043895+0800 ObjcBaseDemo[6720:4895481] set: {(
    1,
    3,
    2
)}
2019-12-25 13:56:28.044005+0800 ObjcBaseDemo[6720:4895481] [set sortedArrayUsingDescriptors:]: (
    3,
    2,
    1
)
```

- 创建动态集合

```objc
+ (void)createMutableSet
{
    NSLog(@"***Creating a mutable set***");
    // 创建可变集合，并设置初始的内部元素个数
    NSMutableSet *mSet = [NSMutableSet setWithCapacity:10];
    NSLog(@"[NSMutableSet setWithCapacity:]: %@", mSet);
    mSet = [[NSMutableSet alloc] initWithCapacity:10];
    NSLog(@"[[NSMutableSet alloc] initWithCapacity:]: %@", mSet);
}
```
输出:
```
2019-12-25 13:56:28.044247+0800 ObjcBaseDemo[6720:4895481] ***Creating a mutable set***
2019-12-25 13:56:28.044490+0800 ObjcBaseDemo[6720:4895481] [NSMutableSet setWithCapacity:]: {(
)}
2019-12-25 13:56:28.044737+0800 ObjcBaseDemo[6720:4895481] [[NSMutableSet alloc] initWithCapacity:]: {(
)}
```

- 动态集合添加删除元素

```objc
+ (void)addAndremoveEntries
{
    NSLog(@"***Adding and removing entries***");
    NSMutableSet *mSet = [NSMutableSet set];
    NSLog(@"mset: %@", mSet);
    //增加
    [mSet addObject:@"One"];
    NSLog(@"[mSet addObject:]: %@", mSet);
    
    // 同时增加多个对象
    [mSet addObjectsFromArray:[NSArray arrayWithObjects:@"Two", @"Three", nil]];
    NSLog(@"[mSet addObjectsFromArray:]: %@", mSet);
    
    // 过滤
    NSPredicate *predicate = [NSPredicate predicateWithValue:YES];
    [mSet filterUsingPredicate:predicate];
    NSLog(@"[mSet filterUsingPredicate:]: %@", mSet);
    
    // 删除单一对象
    [mSet removeObject:@"Three"];
    NSLog(@"[mSet removeObjcet:]: %@", mSet);
    
    // 删除全部
    [mSet removeAllObjects];
    NSLog(@"[mSet removeAllObjects]: %@", mSet);
}
```
输出:
```
2019-12-25 13:56:28.044859+0800 ObjcBaseDemo[6720:4895481] ***Adding and removing entries***
2019-12-25 13:56:28.045075+0800 ObjcBaseDemo[6720:4895481] mset: {(
)}
2019-12-25 13:56:28.045311+0800 ObjcBaseDemo[6720:4895481] [mSet addObject:]: {(
    One
)}
2019-12-25 13:56:28.045510+0800 ObjcBaseDemo[6720:4895481] [mSet addObjectsFromArray:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.045756+0800 ObjcBaseDemo[6720:4895481] [mSet filterUsingPredicate:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.046015+0800 ObjcBaseDemo[6720:4895481] [mSet removeObjcet:]: {(
    One,
    Two
)}
2019-12-25 13:56:28.046202+0800 ObjcBaseDemo[6720:4895481] [mSet removeAllObjects]: {(
)}
```

- 动态集合结合和重组

```objc
+ (void)combineAndRecombine
{
    NSLog(@"***Combining and recombining sets***");
    NSMutableSet *mSet = [NSMutableSet setWithObjects:@"One", @"Two", nil];
    NSLog(@"mSet: %@", mSet);
    NSMutableSet *mSet2 = [NSMutableSet setWithObjects:@"One", @"Three", nil];
    NSLog(@"mSet2: %@", mSet2);
    
    // 并集
    [mSet unionSet:mSet2];
    NSLog(@"[mSet unionSet:]: %@", mSet);
    
    // 差集
    [mSet minusSet:mSet2];
    NSLog(@"[mSet minusSet:]: %@", mSet);
    
    // 交集
    [mSet intersectSet:mSet2];
    NSLog(@"[mSet intersectSet:]: %@", mSet);
    
    // 全替换
    [mSet setSet:mSet2];
    NSLog(@"[mSet setSet:]: %@", mSet);
}
```
输出:
```
2019-12-25 13:56:28.046385+0800 ObjcBaseDemo[6720:4895481] ***Combining and recombining sets***
2019-12-25 13:56:28.046603+0800 ObjcBaseDemo[6720:4895481] mSet: {(
    One,
    Two
)}
2019-12-25 13:56:28.046835+0800 ObjcBaseDemo[6720:4895481] mSet2: {(
    One,
    Three
)}
2019-12-25 13:56:28.047074+0800 ObjcBaseDemo[6720:4895481] [mSet unionSet:]: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.047310+0800 ObjcBaseDemo[6720:4895481] [mSet minusSet:]: {(
    Two
)}
2019-12-25 13:56:28.047548+0800 ObjcBaseDemo[6720:4895481] [mSet intersectSet:]: {(
)}
2019-12-25 13:56:28.047799+0800 ObjcBaseDemo[6720:4895481] [mSet setSet:]: {(
    One,
    Three
)}
```

- 计数集合获取指定元素个数

```objc
+ (void)examineCountedSet {
    NSLog(@"***Examining a Counted Set***");
    NSCountedSet *cSet = [[NSCountedSet alloc] initWithCapacity:5];
    [cSet addObject:@"One"];
    [cSet addObject:@"Two"];
    [cSet addObject:@"Three"];
    [cSet addObject:@"Two"];
    [cSet addObject:@"Three"];
    NSLog(@"mSet: %@", cSet);
    
    NSUInteger count = [cSet countForObject:@"Two"];
    NSLog(@"[cSet countForObject:]: %lu", (unsigned long)count);
}
```
输出:
```
2019-12-25 13:56:28.048046+0800 ObjcBaseDemo[6720:4895481] ***Examining a Counted Set***
2019-12-25 13:56:28.048277+0800 ObjcBaseDemo[6720:4895481] mSet: {(
    One,
    Two,
    Three
)}
2019-12-25 13:56:28.048485+0800 ObjcBaseDemo[6720:4895481] [cSet countForObject:]: 2
```

[demo地址](https://github.com/rodger1017/Study/tree/master/ObjcBaseDemo)