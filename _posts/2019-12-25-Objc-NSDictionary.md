---
title: NSDictionary & NSMutalbeDictionary 基本用法
layout: posts
categories: iOS
tag: objc
---

## [NSDictionary](https://developer.apple.com/documentation/foundation/nsdictionary?language=objc) & [NSMutalbeDictionary](https://developer.apple.com/documentation/foundation/nsmutabledictionary?language=objc)

- 创建字典

```objc
+ (void)creatDictionary
{
    NSLog(@"***Creating a Dictionary***");
    //空字典
    NSDictionary *dictionary = [NSDictionary dictionary];
    NSLog(@"[NSDictionary dictionary]: %@", dictionary);
    // 测试数据
    NSString *filePath = [self testData];
    
    // 通过文件路径创建字典
    dictionary = [NSDictionary dictionaryWithContentsOfFile:filePath];
    NSLog(@"[NSDictionary dictionaryWithContentsOfFile:]: %@", dictionary);
    dictionary = [NSDictionary dictionaryWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"[NSDictionary dictionaryWithContentsOfURL:]: %@", dictionary);
    
    //通过字典生成一个新的字典
    dictionary = [NSDictionary dictionaryWithDictionary:dictionary];
    NSLog(@"[NSDictionary dictionaryWithDictionary:]: %@", dictionary);
    
    // 生成只有一个键-值对的字典
    dictionary = [NSDictionary dictionaryWithObject:@"value" forKey:@"key"];
    NSLog(@"[NSDictionary dictionaryWithObject:forKey:]: %@", dictionary);
    
    // 根据两个数组合并生成包含多个键-值对的字典
    NSArray *values = [NSArray arrayWithObjects:@"value1",@"value2",nil];
    NSArray *keys = [NSArray arrayWithObjects:@"key1",@"key2",nil];
    dictionary = [NSDictionary dictionaryWithObjects:values forKeys:keys];
    NSLog(@"[NSDictionary dictionaryWithObjects:forKeys:]: %@", dictionary);
    
    // 生成包含多个键-值对的字典。数据的顺序为value，key，nil
    dictionary = [NSDictionary dictionaryWithObjectsAndKeys:@"value1", @"key1", @"value2", @"Key2", nil];
    NSLog(@"[NSDictionary dictionaryWithObjectsAndKeys:]: %@", dictionary);
    
    // 字面量初始化
    dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    NSLog(@"dictionary: %@", dictionary);
}
```
输出：
```
2019-12-24 21:15:26.521566+0800 ObjcBaseDemo[82036:4308944] ***Creating a Dictionary***
2019-12-24 21:15:26.521707+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionary]: {
}
2019-12-24 21:15:26.524625+0800 ObjcBaseDemo[82036:4308944] writeToFile: 1
2019-12-24 21:15:26.524865+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithContentsOfFile:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.525058+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithContentsOfURL:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.525163+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithDictionary:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.525247+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithObject:forKey:]: {
    key = value;
}
2019-12-24 21:15:26.525364+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithObjects:forKeys:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.525491+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithObjectsAndKeys:]: {
    Key2 = value2;
    key1 = value1;
}
2019-12-24 21:15:26.525577+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
```

- 初始化字典

```objc
+ (void)initDictionary
{
    NSLog(@"***Initializing a Dictionary***");
    // 空字典
    NSDictionary *dictionary = [[NSDictionary alloc] init];
    NSLog(@"[[NSDictionary alloc] init]: %@", dictionary);
    
    // 测试数据
    NSString *filePath = [self testData];
    
    // 通过文件路径创建字典
    dictionary = [[NSDictionary alloc] initWithContentsOfFile:filePath];
    NSLog(@"[[NSDictionary alloc] initWithContentsOfFile]: %@", dictionary);
    dictionary = [[NSDictionary alloc] initWithContentsOfURL:[NSURL fileURLWithPath:filePath]];
    NSLog(@"[[NSDictionary alloc] initWithContentsOfURL]: %@", dictionary);
    
    
    // 通过字典生成一个新的字典
    dictionary = [[NSDictionary alloc] initWithDictionary:dictionary];
    NSLog(@"[[NSDictionary alloc] initWithDictionary]: %@", dictionary);
    dictionary = [[NSDictionary alloc] initWithDictionary:dictionary copyItems:YES];
    NSLog(@"[[NSDictionary alloc] initWithDictionary:copyItems:]: %@", dictionary);

    // 根据两个数组合并生成包含多个键-值对的字典
    NSArray *values = [NSArray arrayWithObjects:@"value1", @"value2", nil];
    NSArray *keys = [NSArray arrayWithObjects:@"key1", @"key2", nil];
    dictionary = [[NSDictionary alloc] initWithObjects:values forKeys:keys];
    NSLog(@"[[NSDictionary alloc] initWithObjects:forKeys:]: %@", dictionary);
    
    // 生成包含多个键-值对的字典。数据的顺序为value，key，nil
    dictionary = [[NSDictionary alloc] initWithObjectsAndKeys:@"value1", @"key1", @"value2", @"key2", nil];
    NSLog(@"[[NSDictionary alloc] initWithObjectsAndKeys:]: %@", dictionary);
}
```
输出：
```
2019-12-24 21:15:26.526127+0800 ObjcBaseDemo[82036:4308944] ***Initializing a Dictionary***
2019-12-24 21:15:26.526213+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] init]: {
}
2019-12-24 21:15:26.527185+0800 ObjcBaseDemo[82036:4308944] writeToFile: 1
2019-12-24 21:15:26.527349+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithContentsOfFile]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.527550+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithContentsOfURL]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.527649+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithDictionary]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.527752+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithDictionary:copyItems:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.527828+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithObjects:forKeys:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.527915+0800 ObjcBaseDemo[82036:4308944] [[NSDictionary alloc] initWithObjectsAndKeys:]: {
    key1 = value1;
    key2 = value2;
}
```

- 计算个数

```objc
+ (void)countEntries
{
    NSLog(@"***Counting Entries***");
    NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:@"value1", @"key1", @"value2", @"key2", nil];
    NSLog(@"[NSDictionary dictionaryWithObjectsAndKeys:]: %@", dictionary);
    
     // 字典内的key-value个数,
    NSUInteger count = [dictionary count];
    NSLog(@"[dictionary count]: %lu", (unsigned long)count);
     count = dictionary.count;
    NSLog(@"dictionary.count: %lu", (unsigned long)count);
}
```
输出：
```
2019-12-24 21:15:26.527988+0800 ObjcBaseDemo[82036:4308944] ***Counting Entries***
2019-12-24 21:15:26.528069+0800 ObjcBaseDemo[82036:4308944] [NSDictionary dictionaryWithObjectsAndKeys:]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.528139+0800 ObjcBaseDemo[82036:4308944] [dictionary count]: 2
2019-12-24 21:15:26.528207+0800 ObjcBaseDemo[82036:4308944] dictionary.count: 2
```

- 比较

```objc
+ (void)compareDictionaries
{
    NSLog(@"***Comparing Dictionaries***");
    NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:@"value1", @"key1", @"value2", @"key2", nil];
    NSLog(@"dictionary: %@", dictionary);
    NSDictionary *dictionary1 = [NSDictionary dictionaryWithObjectsAndKeys:@"value1", @"key1", @"value2", @"key2", nil];
    NSLog(@"dictionary1: %@", dictionary1);
    
    // 比较字典中的数据是否一致
    BOOL isEqual = [dictionary isEqualToDictionary:dictionary1];
    NSLog(@"[dictionary isEqualToDictionary:]: %d", isEqual);
}
```
输出：
```
2019-12-24 21:15:26.528322+0800 ObjcBaseDemo[82036:4308944] ***Comparing Dictionaries***
2019-12-24 21:15:26.528558+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.528791+0800 ObjcBaseDemo[82036:4308944] dictionary1: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.529029+0800 ObjcBaseDemo[82036:4308944] [dictionary isEqualToDictionary:]: 1
```

- 访问键和值

```objc
+ (void)accessKeysAndValues
{
    NSLog(@"***Accessing Keys and Values***");
    NSDictionary *dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    NSLog(@"dictionary: %@", dictionary);
    
    // 所有的keys
    NSArray *keys = dictionary.allKeys;
    NSLog(@"dictionary.allKeys: %@", keys);
    
    // 根据value获取keys，可能多个key指向
    keys = [dictionary allKeysForObject:@"value1"];
    NSLog(@"[dictionary allKeysForObject:]: %@", keys);
    
    // 所有的values
    NSArray *values = dictionary.allValues;
    NSLog(@"dictionary.values:%@", values);
    
    // 根据key提取value
    NSString *value = [dictionary objectForKey:@"key1"];
    NSLog(@"[dictionary objectForKey:]: %@", value);
    value = [dictionary valueForKey:@"key2"];
    NSLog(@"[dictionary valueForKey:]: %@", value);
    value = [dictionary objectForKeyedSubscript:@"key1"];
    NSLog(@"[dictionary objectForKeyedSubscript:]: %@", value);
    
    // 根据多个key获取多个value,如果没找到
    values = [dictionary objectsForKeys:keys notFoundMarker:@"NotFound"];
    NSLog(@"[dictionary objectsForKeys:notFoundMarker:]: %@", values);
}
```
输出：
```
2019-12-24 21:15:26.529208+0800 ObjcBaseDemo[82036:4308944] ***Accessing Keys and Values***
2019-12-24 21:15:26.529445+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.529749+0800 ObjcBaseDemo[82036:4308944] dictionary.allKeys: (
    key1,
    key2
)
2019-12-24 21:15:26.529936+0800 ObjcBaseDemo[82036:4308944] [dictionary allKeysForObject:]: (
    key1
)
2019-12-24 21:15:26.530093+0800 ObjcBaseDemo[82036:4308944] dictionary.values:(
    value1,
    value2
)
2019-12-24 21:15:26.530242+0800 ObjcBaseDemo[82036:4308944] [dictionary objectForKey:]: value1
2019-12-24 21:15:26.530458+0800 ObjcBaseDemo[82036:4308944] [dictionary valueForKey:]: value2
2019-12-24 21:15:26.530679+0800 ObjcBaseDemo[82036:4308944] [dictionary objectForKeyedSubscript:]: value1
2019-12-24 21:15:26.530858+0800 ObjcBaseDemo[82036:4308944] [dictionary objectsForKeys:notFoundMarker:]: (
    value1
)
```

- 遍历

```objc
+ (void)enumerateDictionary
{
    NSLog(@"***Enumerating Dictionaries***");
    NSDictionary *dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    
    //遍历keys
    NSEnumerator *enumerator = [dictionary keyEnumerator];
    id key;
    while (key = [enumerator nextObject]) {
        NSLog(@"[enumerator nextObject] key: %@", key);
    }
    
    // 遍历values
    enumerator = [dictionary objectEnumerator];
    id value;
    while (value = [enumerator nextObject]) {
        NSLog(@"[enumerator nextObject] value: %@", value);
    }
    
    // 快速遍历key,value
    for (id key in dictionary) {
        id value = [dictionary objectForKey:key];
        NSLog(@"for in: %@=%@", key, value);
    }
    
    // key-value遍历
    [dictionary enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        NSLog(@"[dictionary enumerateKeysAndObjectsUsingBlock:] key: %@ value: %@", key, obj);
        // 当stop设为YES时，会停止遍历
        //*stop = YES;
    }];
    
    [dictionary enumerateKeysAndObjectsWithOptions:NSEnumerationReverse usingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        NSLog(@"[dictionary enumerateKeysAndObjectsWithOptions:usingBlock:] key:%@ value:%@", key, obj);
        // 当stop设为YES时，会停止遍历,必须设NSEnumerationReverse
        // *stop = YES;
    }];
}
```
输出：
```
2019-12-24 21:15:26.531058+0800 ObjcBaseDemo[82036:4308944] ***Enumerating Dictionaries***
2019-12-24 21:15:26.531264+0800 ObjcBaseDemo[82036:4308944] [enumerator nextObject] key: key1
2019-12-24 21:15:26.531449+0800 ObjcBaseDemo[82036:4308944] [enumerator nextObject] key: key2
2019-12-24 21:15:26.531665+0800 ObjcBaseDemo[82036:4308944] [enumerator nextObject] value: value1
2019-12-24 21:15:26.531889+0800 ObjcBaseDemo[82036:4308944] [enumerator nextObject] value: value2
2019-12-24 21:15:26.532066+0800 ObjcBaseDemo[82036:4308944] for in: key1=value1
2019-12-24 21:15:26.532208+0800 ObjcBaseDemo[82036:4308944] for in: key2=value2
2019-12-24 21:15:26.532385+0800 ObjcBaseDemo[82036:4308944] [dictionary enumerateKeysAndObjectsUsingBlock:] key: key1 value: value1
2019-12-24 21:15:26.532598+0800 ObjcBaseDemo[82036:4308944] [dictionary enumerateKeysAndObjectsUsingBlock:] key: key2 value: value2
2019-12-24 21:15:26.532845+0800 ObjcBaseDemo[82036:4308944] [dictionary enumerateKeysAndObjectsWithOptions:usingBlock:] key:key1 value:value1
2019-12-24 21:15:26.533065+0800 ObjcBaseDemo[82036:4308944] [dictionary enumerateKeysAndObjectsWithOptions:usingBlock:] key:key2 value:value2
```

- 排序

```objc
+ (void)sortDictionary
{
    NSLog(@"***Sorting Dictionaries***");
    NSDictionary *dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    NSLog(@"dictionary: %@", dictionary);
    
    // 使用Selector比较获取key, compare:由对象去实现
    NSArray *array = [dictionary keysSortedByValueUsingSelector:@selector(compare:)];
    NSLog(@"[dictionary keysSortedByValueUsingSelector:]: %@", array);
    
    // 使用block比较value,排序key
    array = [dictionary keysSortedByValueUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[dictionary keysSortedByValueUsingComparator:]: %@", array);
    
    // 并发排序
    array = [dictionary keysSortedByValueWithOptions:NSSortStable usingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
        return [obj1 compare:obj2];
    }];
    NSLog(@"[dictionary keysSortedByValueWithOptions:usingComparator:]: %@", array);
}
```
输出：
```
2019-12-24 21:15:26.533273+0800 ObjcBaseDemo[82036:4308944] ***Sorting Dictionaries***
2019-12-24 21:15:26.533617+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.533860+0800 ObjcBaseDemo[82036:4308944] [dictionary keysSortedByValueUsingSelector:]: (
    key1,
    key2
)
2019-12-24 21:15:26.534116+0800 ObjcBaseDemo[82036:4308944] [dictionary keysSortedByValueUsingComparator:]: (
    key1,
    key2
)
2019-12-24 21:15:26.534402+0800 ObjcBaseDemo[82036:4308944] [dictionary keysSortedByValueWithOptions:usingComparator:]: (
    key1,
    key2
)
```

- 过滤

```objc
+ (void)filterDictionary
{
    NSLog(@"***Filtering Dictionaries***");
    NSDictionary *dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    NSLog(@"dictionary: %@", dictionary);
    
    // 返回过滤后的keys
    NSSet *set = [dictionary keysOfEntriesPassingTest:^BOOL(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        // stop 是否停止过滤
        // return 通过yes，禁止no
        return YES;
    }];
    NSLog(@"[dictionary keysOfEntriesPassingTest:]: %@", set);
    
    // 并发过滤
    set = [dictionary keysOfEntriesWithOptions:NSEnumerationConcurrent passingTest:^BOOL(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        // stop 是否停止过滤
        // return 通过yes，禁止no
        return YES;
    }];
    NSLog(@"[dictionary keysOfEntriesWithOptions:passingTest:]: %@", set);
}
```
输出：
```
2019-12-24 21:15:26.534775+0800 ObjcBaseDemo[82036:4308944] ***Filtering Dictionaries***
2019-12-24 21:15:26.535062+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.535336+0800 ObjcBaseDemo[82036:4308944] [dictionary keysOfEntriesPassingTest:]: {(
    key1,
    key2
)}
2019-12-24 21:15:26.535659+0800 ObjcBaseDemo[82036:4308944] [dictionary keysOfEntriesWithOptions:passingTest:]: {(
    key1,
    key2
)}
```

- 存储到文件

```objc
+ (void)storeDictionary
{
    NSLog(@"***Storing Dictionaries***");
    NSDictionary *dictionary = @{@"key1":@"value1", @"key2":@"value2"};
    NSLog(@"dictionary: %@", dictionary);
    NSString *filePath = [self testData];
    NSLog(@"filePath: %@", filePath);
    
    // 根据路径存储字典
    BOOL write = [dictionary writeToFile:filePath atomically:YES];
    NSLog(@"[dictionary writeToFile:atomically:]: %d", write);
    
    write = [dictionary writeToURL:[NSURL fileURLWithPath:filePath] atomically:YES];
    NSLog(@"[dictionary writeToFile:atomically:]: %d", write);
}
```
输出：
```
2019-12-24 21:15:26.535883+0800 ObjcBaseDemo[82036:4308944] ***Storing Dictionaries***
2019-12-24 21:15:26.536173+0800 ObjcBaseDemo[82036:4308944] dictionary: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.537285+0800 ObjcBaseDemo[82036:4308944] writeToFile: 1
2019-12-24 21:15:26.537384+0800 ObjcBaseDemo[82036:4308944] filePath: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/5EB285DA-AD4E-4F45-AAA7-C07BE28C330E/Documents/test.plist
2019-12-24 21:15:26.538282+0800 ObjcBaseDemo[82036:4308944] [dictionary writeToFile:atomically:]: 1
2019-12-24 21:15:26.539173+0800 ObjcBaseDemo[82036:4308944] [dictionary writeToFile:atomically:]: 1
```

- 访问文件属性

```objc
+ (void)accessFileAttributes {
    NSLog(@"***Accessing File Attributes***");
    
    NSString *filePath = [self testData];
    NSError *error;
    NSDictionary *dictionary = [[NSFileManager defaultManager] attributesOfItemAtPath:filePath error:&error];
    
    // 创建时间
    NSLog(@"dictionary.fileCreationDate: %@", dictionary.fileCreationDate);
    // 是否可见
    NSLog(@"dictionary.fileExtensionHidden: %d", dictionary.fileExtensionHidden);
    // 用户组ID
    NSLog(@"dictionary.fileGroupOwnerAccountID: %@", dictionary.fileGroupOwnerAccountID);
    // 用户组名
    NSLog(@"[dictionary fileGroupOwnerAccountName]: %@", [dictionary fileGroupOwnerAccountName]);
    // HFS编码
    NSLog(@"(unsigned int)dictionary.fileHFSCreatorCode: %u", (unsigned int)dictionary.fileHFSCreatorCode);
    // HFS类型编码
    NSLog(@"(unsigned int)dictionary.fileHFSTypeCode: %u", (unsigned int)dictionary.fileHFSTypeCode);
    // 是否只读
    NSLog(@"dictionary.fileIsAppendOnly: %d", dictionary.fileIsAppendOnly);
    // 是否可修改
    NSLog(@"dictionary.fileIsImmutable: %d", dictionary.fileIsImmutable);
    // 修改时间
    NSLog(@"dictionary.fileModificationDate: %@", dictionary.fileModificationDate);
    // 所有者ID
    NSLog(@"dictionary.fileOwnerAccountID: %@", dictionary.fileOwnerAccountID);
    // 所有者名
    NSLog(@"dictionary.fileOwnerAccountName:%@", dictionary.fileOwnerAccountName);
    // Posix权限
    NSLog(@"(unsigned long)dictionary.filePosixPermissions: %lu", (unsigned long)dictionary.filePosixPermissions);
    // 文件大小
    NSLog(@"dictionary.fileSize:%llu", dictionary.fileSize);
    // 系统文件数量
    NSLog(@"(unsigned long)dictionary.fileSystemFileNumber: %lu", (unsigned long)dictionary.fileSystemFileNumber);
    // 文件系统的数量
    NSLog(@"(long)dictionary.fileSystemNumber: %ld", (long)dictionary.fileSystemNumber);
    // 文件类型
    NSLog(@"dictionary.fileType: %@", dictionary.fileType);
}
```
输出：
```
2019-12-24 21:15:26.539376+0800 ObjcBaseDemo[82036:4308944] ***Accessing File Attributes***
2019-12-24 21:15:26.540320+0800 ObjcBaseDemo[82036:4308944] writeToFile: 1
2019-12-24 21:15:26.609711+0800 ObjcBaseDemo[82036:4308944] dictionary.fileCreationDate: Wed Dec 24 21:15:26 2019
2019-12-24 21:15:26.609812+0800 ObjcBaseDemo[82036:4308944] dictionary.fileExtensionHidden: 0
2019-12-24 21:15:26.609900+0800 ObjcBaseDemo[82036:4308944] dictionary.fileGroupOwnerAccountID: 20
2019-12-24 21:15:26.610680+0800 ObjcBaseDemo[82036:4308944] [dictionary fileGroupOwnerAccountName]: staff
2019-12-24 21:15:26.610783+0800 ObjcBaseDemo[82036:4308944] (unsigned int)dictionary.fileHFSCreatorCode: 0
2019-12-24 21:15:26.610850+0800 ObjcBaseDemo[82036:4308944] (unsigned int)dictionary.fileHFSTypeCode: 0
2019-12-24 21:15:26.610932+0800 ObjcBaseDemo[82036:4308944] dictionary.fileIsAppendOnly: 0
2019-12-24 21:15:26.611004+0800 ObjcBaseDemo[82036:4308944] dictionary.fileIsImmutable: 0
2019-12-24 21:15:26.611168+0800 ObjcBaseDemo[82036:4308944] dictionary.fileModificationDate: Wed Dec 24 21:15:26 2019
2019-12-24 21:15:26.611245+0800 ObjcBaseDemo[82036:4308944] dictionary.fileOwnerAccountID: 501
2019-12-24 21:15:26.611557+0800 ObjcBaseDemo[82036:4308944] dictionary.fileOwnerAccountName:(null)
2019-12-24 21:15:26.611631+0800 ObjcBaseDemo[82036:4308944] (unsigned long)dictionary.filePosixPermissions: 420
2019-12-24 21:15:26.611740+0800 ObjcBaseDemo[82036:4308944] dictionary.fileSize:272
2019-12-24 21:15:26.611824+0800 ObjcBaseDemo[82036:4308944] (unsigned long)dictionary.fileSystemFileNumber: 45052071
2019-12-24 21:15:26.611911+0800 ObjcBaseDemo[82036:4308944] (long)dictionary.fileSystemNumber: 16777220
2019-12-24 21:15:26.611983+0800 ObjcBaseDemo[82036:4308944] dictionary.fileType: NSFileTypeRegular
```

- 测试数据

```objc
+ (NSString *)testData {
    // 获取应用中Document文件夹
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    
    // 测试数据
    NSArray *values = [NSArray arrayWithObjects:@"value1", @"value2", nil];
    NSArray *keys = [NSArray arrayWithObjects:@"key1", @"key2", nil];
    NSDictionary *dictionary = [NSDictionary dictionaryWithObjects:values forKeys:keys];
    NSString *filePath = [documentsDirectory stringByAppendingPathComponent:@"test.plist"];
    BOOL write = [dictionary writeToFile:filePath atomically:YES];
    NSLog(@"writeToFile: %d", write);
    
    return filePath;
}
```

- 动态字典创建与初始化

```objc
+ (void)createAndInitDictionary
{
    NSLog(@"***Creating and Initializing a Mutable Dictionary***");
    // 创建包含一个key-value的可变字典。
    NSMutableDictionary *mDictionary = [NSMutableDictionary dictionaryWithCapacity:1];
    NSLog(@"[NSMutableDictionary dictionaryWithCapacity:]: %@", mDictionary);
    mDictionary = [[NSMutableDictionary alloc] initWithCapacity:1];
    NSLog(@"[[NSMutableDictionary alloc] dictionaryWithCapacity:]: %@", mDictionary);
    
    // 空字典
    mDictionary = [NSMutableDictionary dictionary];
    NSLog(@"[NSMutableDictionary dictionary:]: %@", mDictionary);
    mDictionary = [[NSMutableDictionary alloc] init];
    NSLog(@"[[NSMutableDictionary alloc] init:]: %@", mDictionary);
}
```
输出：
```
2019-12-24 21:15:26.612054+0800 ObjcBaseDemo[82036:4308944] ***Creating and Initializing a Mutable Dictionary***
2019-12-24 21:15:26.612163+0800 ObjcBaseDemo[82036:4308944] [NSMutableDictionary dictionaryWithCapacity:]: {
}
2019-12-24 21:15:26.612372+0800 ObjcBaseDemo[82036:4308944] [[NSMutableDictionary alloc] dictionaryWithCapacity:]: {
}
2019-12-24 21:15:26.612554+0800 ObjcBaseDemo[82036:4308944] [NSMutableDictionary dictionary:]: {
}
2019-12-24 21:15:26.612723+0800 ObjcBaseDemo[82036:4308944] [[NSMutableDictionary alloc] init:]: {
}
```

- 动态字典添加元素

```objc
+ (void)addEntries
{
    NSLog(@"***Adding Entries to a Mutable Dictionary***");
    NSMutableDictionary *mDictionary = [NSMutableDictionary dictionary];
    NSLog(@"[NSMutableDictionary dictionary]: %@", mDictionary);
    
    //增加单一记录
    [mDictionary setObject:@"value1" forKey:@"key1"];
    NSLog(@"[mDictionary setObject:forKey]: %@", mDictionary);
    [mDictionary setObject:@"value2" forKeyedSubscript:@"key2"];
    NSLog(@"[mDictionary setObject:forKeyedSubscript]: %@", mDictionary);
    [mDictionary setValue:@"value3" forKey:@"key3"];
    NSLog(@"[mDictionary setValue:forKey]: %@", mDictionary);
    
    // 从字典中增加数据
    NSDictionary *dictionary = [NSDictionary dictionaryWithObject:@"value4" forKey:@"key4"];
    [mDictionary addEntriesFromDictionary:dictionary];
    NSLog(@"[mDictionary addEntriesFromDictionary:]: %@", mDictionary);
    
    // 用新的字典数据覆盖原有字典数据
    [mDictionary setDictionary:dictionary];
    NSLog(@"[mDictionary setDictionary:]: %@", mDictionary);
}
```
输出：
```
2019-12-24 21:15:26.612915+0800 ObjcBaseDemo[82036:4308944] ***Adding Entries to a Mutable Dictionary***
2019-12-24 21:15:26.613155+0800 ObjcBaseDemo[82036:4308944] [NSMutableDictionary dictionary]: {
}
2019-12-24 21:15:26.613405+0800 ObjcBaseDemo[82036:4308944] [mDictionary setObject:forKey]: {
    key1 = value1;
}
2019-12-24 21:15:26.613632+0800 ObjcBaseDemo[82036:4308944] [mDictionary setObject:forKeyedSubscript]: {
    key1 = value1;
    key2 = value2;
}
2019-12-24 21:15:26.613795+0800 ObjcBaseDemo[82036:4308944] [mDictionary setValue:forKey]: {
    key1 = value1;
    key2 = value2;
    key3 = value3;
}
2019-12-24 21:15:26.613981+0800 ObjcBaseDemo[82036:4308944] [mDictionary addEntriesFromDictionary:]: {
    key1 = value1;
    key2 = value2;
    key3 = value3;
    key4 = value4;
}
2019-12-24 21:15:26.614090+0800 ObjcBaseDemo[82036:4308944] [mDictionary setDictionary:]: {
    key4 = value4;
}
```

- 动态字典删除元素

```objc
+ (void)removeEntries
{
    NSLog(@"***Removing Entries From a Mutable Dictionary***");
    NSMutableDictionary *mDictionary = [NSMutableDictionary dictionary];
    NSLog(@"[NSMutableDictionary dictionary]: %@", mDictionary);

    // 添加元素
    [mDictionary setObject:@"value1" forKey:@"key1"];
    [mDictionary setObject:@"value2" forKeyedSubscript:@"key2"];
    [mDictionary setValue:@"value3" forKey:@"key3"];
    [mDictionary setValue:@"value4" forKey:@"key4"];
    NSLog(@"mDictionary: %@", mDictionary);
    
    // 根据key删除单一记录
    [mDictionary removeObjectForKey:@"key1"];
    
    NSLog(@"[mDictionary removeObjectForKey:]: %@", mDictionary);
    
    // 批量删除多个key对应的记录
    NSArray *keys = [NSArray arrayWithObjects:@"key2", @"key3", nil];
    [mDictionary removeObjectsForKeys:keys];
    NSLog(@"[mDictionary removeObjectsForKeys:]: %@", mDictionary);
    
    // 删除所有记录
    [mDictionary removeAllObjects];
    NSLog(@"[mDictionary removeAllObjects]: %@", mDictionary);
}
```
输出：
```
2019-12-24 21:15:26.614221+0800 ObjcBaseDemo[82036:4308944] ***Removing Entries From a Mutable Dictionary***
2019-12-24 21:15:26.614348+0800 ObjcBaseDemo[82036:4308944] [NSMutableDictionary dictionary]: {
}
2019-12-24 21:15:26.631125+0800 ObjcBaseDemo[82036:4308944] mDictionary: {
    key1 = value1;
    key2 = value2;
    key3 = value3;
    key4 = value4;
}
2019-12-24 21:15:26.631235+0800 ObjcBaseDemo[82036:4308944] [mDictionary removeObjectForKey:]: {
    key2 = value2;
    key3 = value3;
    key4 = value4;
}
2019-12-24 21:15:26.631333+0800 ObjcBaseDemo[82036:4308944] [mDictionary removeObjectsForKeys:]: {
    key4 = value4;
}
2019-12-24 21:15:26.631412+0800 ObjcBaseDemo[82036:4308944] [mDictionary removeAllObjects]: {
}
```

[demo地址](https://github.com/rodger1017/Study/tree/master/ObjcBaseDemo)