---
layout: post
title: NSString & NSMutableString 基本用法
---

## [NSString](https://developer.apple.com/documentation/foundation/nsstring?language=objc) & [NSMutableString](https://developer.apple.com/documentation/foundation/nsmutablestring?language=objc)

- 初始化字符串

```objc
+ (void)creatAndInitStrings
{
    NSLog(@"***Creating and Initializing Strings***");
    //空字符串
    NSString *string = [NSString string];
    NSLog(@"[NSString string]: %@", string);
    string = [[NSString alloc]init];
    NSLog(@"[[NSString alloc]init]: %@", string);
    string = @"";
    NSLog(@"string: %@", string);
    
    //通过字符串生成字符串
    string = @"Test";
    string = [NSString stringWithString:string];
    NSLog(@"[NSString stringWithString:]: %@", string);
    string = [[NSString alloc]initWithString:string];
    NSLog(@"[[NSString alloc]initWithString:]: %@", string);
    string = string;
    NSLog(@"string: %@", string);
    
    // 组合生成NSString
    string = [NSString stringWithFormat:@"This is %@", @"Test"];
    NSLog(@"[NSString stringWithFormat:]: %@", string);
    string = [[NSString alloc]initWithFormat:@"This is %@ too", @"Test"];
    NSLog(@"[[NSString alloc]initWithFormat:]: %@", string);
    
    // 通过utf-8字符串
    string = [NSString stringWithUTF8String:"测试1"];
    NSLog(@"[NSString stringWithUTF8String:]: %@", string);
    string = [[NSString alloc]initWithUTF8String:"测试2"];
    NSLog(@"[[NSString alloc]initWithUTF8String:]: %@", string);
    
    //通过C字符串
    const char *cStr = "This is 测试";
    //const char *cStr = [string cStringUsingEncoding:NSUTF8StringEncoding];
    string = [NSString stringWithCString:cStr encoding:NSUTF8StringEncoding];
    NSLog(@"[NSString stringWithCString:encoding:]: %@", string);
    string = [[NSString alloc]initWithCString:cStr encoding:NSUTF8StringEncoding];
    NSLog(@"[[NSString alloc]initWithCString:encoding:]: %@", string);
}
```
输出：
```
2019-12-22 14:13:22.280441+0800 ObjcBaseDemo[32533:501266] ***Creating and Initializing Strings***
2019-12-22 14:13:22.280552+0800 ObjcBaseDemo[32533:501266] [NSString string]:
2019-12-22 14:13:22.280625+0800 ObjcBaseDemo[32533:501266] [[NSString alloc]init]:
2019-12-22 14:13:22.280727+0800 ObjcBaseDemo[32533:501266] string:
2019-12-22 14:13:22.280798+0800 ObjcBaseDemo[32533:501266] [NSString stringWithString:]: Test
2019-12-22 14:13:22.280876+0800 ObjcBaseDemo[32533:501266] [[NSString alloc]initWithString:]: Test
2019-12-22 14:13:22.280947+0800 ObjcBaseDemo[32533:501266] string: Test
2019-12-22 14:13:22.281010+0800 ObjcBaseDemo[32533:501266] [NSString stringWithFormat:]: This is Test
2019-12-22 14:13:22.281093+0800 ObjcBaseDemo[32533:501266] [[NSString alloc]initWithFormat:]: This is Test too
2019-12-22 14:13:22.281167+0800 ObjcBaseDemo[32533:501266] [NSString stringWithUTF8String:]: 测试1
2019-12-22 14:13:22.281252+0800 ObjcBaseDemo[32533:501266] [[NSString alloc]initWithUTF8String:]: 测试2
2019-12-22 14:13:22.281457+0800 ObjcBaseDemo[32533:501266] [NSString stringWithCString:encoding:]: This is 测试
2019-12-22 14:13:22.281722+0800 ObjcBaseDemo[32533:501266] [[NSString alloc]initWithCString:encoding:]: This is 测试
```

- 测试数据

```objc
+ (NSString *)testData {
    // 获取应用中Document文件夹
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    
    // 测试数据
    NSString *string = @"This is test";
    NSString *filePath = [documentsDirectory stringByAppendingPathComponent:@"test.plist"];
    NSError *error;
    [string writeToFile:filePath atomically:YES encoding:NSUTF8StringEncoding error:&error];
    if (error) {
        NSLog(@"错误：%@", error.localizedDescription);
    }
    return filePath;
}
```

- 通过NSString路径读取文件内容

```objc
+ (void)createAndInitFromFile
{
    NSLog(@"***Creating and Initializing a String from a File***");
    NSString *filePath = [self testData];
    NSError *error;
    
    // 指定编码格式读取
    NSString *string = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:&error];
    NSLog(@"[NSString stringWithContentsOfFile:encoding:error]: %@", string);
    string = [[NSString alloc]initWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:&error];
    NSLog(@"[[NSString alloc] initWithContentsOfFile:encoding:error]: %@", string);
    
    // 自动打开，并返回解析的编码模式
    NSStringEncoding enc;
    string = [NSString stringWithContentsOfFile:filePath usedEncoding:&enc error:&error];
    NSLog(@"[NSString stringWithContentsOfFile:usedEncoding:error]: %@", string);
    string = [[NSString alloc] initWithContentsOfFile:filePath usedEncoding:&enc error:&error];
    NSLog(@"[[NSString alloc] initWithContentsOfFile:usedEncoding:error]: %@", string);
    if (error) {
        NSLog(@"错误：%@", error.localizedDescription);
    }
}
```
输出：
```
2019-12-22 14:13:22.281939+0800 ObjcBaseDemo[32533:501266] ***Creating and Initializing a String from a File***
2019-12-22 14:13:22.283128+0800 ObjcBaseDemo[32533:501266] [NSString stringWithContentsOfFile:encoding:error]: This is test
2019-12-22 14:13:22.283280+0800 ObjcBaseDemo[32533:501266] [[NSString alloc] initWithContentsOfFile:encoding:error]: This is test
2019-12-22 14:13:22.283457+0800 ObjcBaseDemo[32533:501266] [NSString stringWithContentsOfFile:usedEncoding:error]: This is test
2019-12-22 14:13:22.283607+0800 ObjcBaseDemo[32533:501266] [[NSString alloc] initWithContentsOfFile:usedEncoding:error]: This is test
```

- 通过NSURL路径读取文件内容

```objc
+ (void)createAndInitFromURL
{
    NSLog(@"***Creating and Initializing a String from an URL***");
    NSString *filePath = [self testData];
    NSURL *fileUrl = [NSURL fileURLWithPath:filePath];
    NSError *error;
    
    // 指定编码格式读取
    NSString *string = [NSString stringWithContentsOfURL:fileUrl encoding:NSUTF8StringEncoding error:&error];
    NSLog(@"[NSString stringWithContentsOfURL:encoding:error]: %@", string);
    string = [[NSString alloc] initWithContentsOfURL:fileUrl encoding:NSUTF8StringEncoding error:&error];
    NSLog(@"[[NSString alloc] initWithContentsOfURL:encoding:error]: %@", string);
    
    // 自动打开，并返回解析的编码模式
    NSStringEncoding enc;
    string = [NSString stringWithContentsOfURL:fileUrl usedEncoding:&enc error:&error];
    NSLog(@"[NSString stringWithContentsOfURL:usedEncoding:error]: %@", string);
    string = [[NSString alloc] initWithContentsOfURL:fileUrl usedEncoding:&enc error:&error];
    NSLog(@"[[NSString alloc] initWithContentsOfFile:usedEncoding:error]: %@", string);
    
    if (error) {
        NSLog(@"错误：%@", error.localizedDescription);
    }
}
```
输出：
```
2019-12-22 14:13:22.283679+0800 ObjcBaseDemo[32533:501266] ***Creating and Initializing a String from an URL***
2019-12-22 14:13:22.284614+0800 ObjcBaseDemo[32533:501266] [NSString stringWithContentsOfURL:encoding:error]: This is test
2019-12-22 14:13:22.284719+0800 ObjcBaseDemo[32533:501266] [[NSString alloc] initWithContentsOfURL:encoding:error]: This is test
2019-12-22 14:13:22.284840+0800 ObjcBaseDemo[32533:501266] [NSString stringWithContentsOfURL:usedEncoding:error]: This is test
2019-12-22 14:13:22.284980+0800 ObjcBaseDemo[32533:501266] [[NSString alloc] initWithContentsOfFile:usedEncoding:error]: This is test
```

- 通过NSString或NSURL路径写入NSString

```objc
+ (void)writeFileOrURL
{
    NSLog(@"***Writing to a File or URL***");
    NSString *filePath = [self testData];
    NSURL *fileURL = [NSURL fileURLWithPath:filePath];
    NSError *error;
    NSString *string = @"One_Two_Three";
    
    BOOL write = [string writeToFile:filePath atomically:YES encoding:NSUTF8StringEncoding error:&error];
    
    write = [string writeToURL:fileURL atomically:YES encoding:NSUTF8StringEncoding error:&error];
    
    if (error) {
        NSLog(@"错误：%@", error.localizedDescription);
    } else {
        string = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:&error];
        NSLog(@"[NSString stringWithContentsOfFile:encoding:error]:%@", string);
    }
}
```
输出：
```
2019-12-22 14:13:22.285051+0800 ObjcBaseDemo[32533:501266] ***Writing to a File or URL***
2019-12-22 14:13:22.287220+0800 ObjcBaseDemo[32533:501266] [NSString stringWithContentsOfFile:encoding:error]:One_Two_Three
```

- 获取字符串长度

```objc
+ (void)getLength
{
    NSLog(@"***Getting a String’s Length***");
    NSString *string = @"Hello World";
    
    //长度
    NSInteger length = string.length;
    NSLog(@"[string.length]: %ld", length);
    
    // 指定编码格式后的长度
    length = [string lengthOfBytesUsingEncoding:NSUTF8StringEncoding];
    NSLog(@"[string lengthOfBytesUsingEncoding:]: %ld", length);
    
    // 返回存储时需要指定的长度
    length  = [string maximumLengthOfBytesUsingEncoding:NSUTF8StringEncoding];
    NSLog(@"[string maximumLengthOfBytesUsingEncoding:]: %ld",length);
}
```
输出：
```
2019-12-22 14:13:22.287329+0800 ObjcBaseDemo[32533:501266] ***Getting a String’s Length***
2019-12-22 14:13:22.287390+0800 ObjcBaseDemo[32533:501266] [string.length]: 11
2019-12-22 14:13:22.287465+0800 ObjcBaseDemo[32533:501266] [string lengthOfBytesUsingEncoding:]: 11
2019-12-22 14:13:22.287537+0800 ObjcBaseDemo[32533:501266] [string maximumLengthOfBytesUsingEncoding:]: 33
```

- 获取Characters和Bytes

```objc
+ (void)getCharactersAndBytes
{
    NSLog(@"***Getting Characters and Bytes***");
    NSString *string = @"This is a test string";
    
    // 提取指定位置的character
    unichar uch = [string characterAtIndex:3];
    NSLog(@"[string characterAtIndex:]: %c", uch);
    
    // 提取Bytes，并返回使用的长度
    NSUInteger length = [string maximumLengthOfBytesUsingEncoding:NSUTF8StringEncoding];
    NSLog(@"[string maximumLengthOfBytesUsingEncoding:]: %lu", (unsigned long)length);
    const void *bytes;
    NSRange range = {0,5};
    BOOL gByte = [string getBytes:&bytes maxLength:length usedLength:&length encoding:NSUTF8StringEncoding options:NSStringEncodingConversionAllowLossy range:range remainingRange:nil];
    if(gByte) {
        NSLog(@"string getBytes:maxLength:usedLength:encoding:options:range:remainingRange:]: %lu", (unsigned long)length);
    }
}
```
输出：
```
2019-12-22 14:13:22.287600+0800 ObjcBaseDemo[32533:501266] ***Getting Characters and Bytes***
2019-12-22 14:13:22.287671+0800 ObjcBaseDemo[32533:501266] [string characterAtIndex:]: s
2019-12-22 14:13:22.287736+0800 ObjcBaseDemo[32533:501266] [string maximumLengthOfBytesUsingEncoding:]: 63
2019-12-22 14:13:22.287812+0800 ObjcBaseDemo[32533:501266] string getBytes:maxLength:usedLength:encoding:options:range:remainingRange:]: 5
```

- 识别和比较字符串

```objc
+ (void)identifyAndCompareStrings
{
    NSLog(@"***Identifying and Comparing Strings***");
    NSString *string = @"Hello,world";
    NSString *compareStr = @"Hello,Apple";
    
    NSRange searchRange = {0, string.length};
    
    //比较大小
    NSComparisonResult result = [string compare:compareStr];
    NSLog(@"[string compare:]: %ld", result);
    
    //前缀比较
    NSLog(@"[string hasPrefix:]: %d", [string hasPrefix:@"Hello"]);
    
    // 后缀比较
    NSLog(@"[string hasSuffix:]: %d", [string hasSuffix:@"Hello"]);
    
    //比较两个字符串是否相同:
    NSLog(@"[string isEqualToString:]: %d", [string isEqualToString:compareStr]);
    
    // 通过指定的比较模式，比较字符串
    result = [string compare:compareStr options:NSCaseInsensitiveSearch];
    NSLog(@"[string compare:options:]: %ld", (long)result);
    result = [string caseInsensitiveCompare:compareStr];
    NSLog(@"[string caseInsensitiveCompare:]: %ld", (long)result);
    
    // 添加比较范围
    result = [string compare:compareStr options:NSCaseInsensitiveSearch range:searchRange];
    NSLog(@"[string compare:options:range:]: %ld", (long)result);
    
    // 增加比较地域
    result = [string compare:compareStr options:NSCaseInsensitiveSearch range:searchRange locale:nil];
    NSLog(@"[string compare:options:range:locale:]: %ld", (long)result);
    
    // 本地化字符串，再比较
    result = [string localizedCompare:compareStr];
    NSLog(@"[string localizedCompare:]: %ld", (long)result);
    result = [string localizedStandardCompare:compareStr];
    NSLog(@"[string localizedStandardCompare:]: %ld", (long)result);
    
    // NSCaseInsensitiveSearch模式
    result = [string localizedCaseInsensitiveCompare:compareStr];
    NSLog(@"[string localizedCaseInsensitiveCompare:]: %ld", (long)result);
    
    // hash值
    NSUInteger hash = string.hash;
    NSLog(@"string.hash: %lu", (unsigned long)hash);
}
```
输出：
```
2019-12-22 14:13:22.287880+0800 ObjcBaseDemo[32533:501266] ***Identifying and Comparing Strings***
2019-12-22 14:13:22.287975+0800 ObjcBaseDemo[32533:501266] [string compare:]: 1
2019-12-22 14:13:22.288194+0800 ObjcBaseDemo[32533:501266] [string hasPrefix:]: 1
2019-12-22 14:13:22.288394+0800 ObjcBaseDemo[32533:501266] [string hasSuffix:]: 0
2019-12-22 14:13:22.288591+0800 ObjcBaseDemo[32533:501266] [string isEqualToString:]: 0
2019-12-22 14:13:22.288808+0800 ObjcBaseDemo[32533:501266] [string compare:options:]: 1
2019-12-22 14:13:22.288942+0800 ObjcBaseDemo[32533:501266] [string caseInsensitiveCompare:]: 1
2019-12-22 14:13:22.289184+0800 ObjcBaseDemo[32533:501266] [string compare:options:range:]: 1
2019-12-22 14:13:22.289417+0800 ObjcBaseDemo[32533:501266] [string compare:options:range:locale:]: 1
2019-12-22 14:13:22.289878+0800 ObjcBaseDemo[32533:501266] [string localizedCompare:]: 1
2019-12-22 14:13:22.292041+0800 ObjcBaseDemo[32533:501266] [string localizedStandardCompare:]: 1
2019-12-22 14:13:22.292162+0800 ObjcBaseDemo[32533:501266] [string localizedCaseInsensitiveCompare:]: 1
2019-12-22 14:13:22.292247+0800 ObjcBaseDemo[32533:501266] string.hash: 16243880619944621395
```

- 获取C字符串

```objc
+ (void)getCString
{
    NSLog(@"***Getting C Strings***");
    NSString *string = @"This is a test string";
    
    // 指定编码格式,获取C字符串
    const char *c = [string cStringUsingEncoding:NSUTF8StringEncoding];
    NSLog(@"[string cStringUsingEncoding:]: %s", c);
    
    // 获取通过UTF-8转码的字符串
    const char *UTF8String = [string UTF8String];
    NSLog(@"[string UTF8String]: %s", UTF8String);
}
```
输出：
```
2019-12-22 14:13:22.292318+0800 ObjcBaseDemo[32533:501266] ***Getting C Strings***
2019-12-22 14:13:22.292394+0800 ObjcBaseDemo[32533:501266] [string cStringUsingEncoding:]: This is a test string
2019-12-22 14:13:22.292491+0800 ObjcBaseDemo[32533:501266] [string UTF8String]: This is a test string
```

- 字符串拼接

```objc
+ (void)combineString
{
    NSLog(@"***Combining Strings***");
    NSString *string = @"This is a test ";
    
    // string后增加字符串并生成一个新的字符串
    string = [string stringByAppendingString:@"string"];
    NSLog(@"[string stringByAppendingString:]: %@", string);
    
    // string后增加组合字符串并生成一个新的字符串
    string = [string stringByAppendingFormat:@"%@",@";lalala"];
    NSLog(@"[string stringByAppendingFormat:]: %@", string);
    
    // string后增加循环字符串，stringByPaddingToLength：完毕后截取的长度；startingAtIndex：从增加的字符串第几位开始循环增加。
    string = [string stringByPaddingToLength:30 withString:@";heihei" startingAtIndex:4];
    NSLog(@"[string stringByPaddingToLength:withString:startingAtIndex]: %@", string);
}
```
输出：
```
2019-12-22 14:13:22.292555+0800 ObjcBaseDemo[32533:501266] ***Combining Strings***
2019-12-22 14:13:22.292621+0800 ObjcBaseDemo[32533:501266] [string stringByAppendingString:]: This is a test string
2019-12-22 14:13:22.292710+0800 ObjcBaseDemo[32533:501266] [string stringByAppendingFormat:]: This is a test string;lalala
2019-12-22 14:13:22.292878+0800 ObjcBaseDemo[32533:501266] [string stringByPaddingToLength:withString:startingAtIndex]: This is a test string;lalalahe
```

- 大小写变化

```objc
+ (void)changeCase
{
    NSLog(@"***Changing Case***");
    NSString *string = @"Hello,中国;hello,world;";
    NSLocale *locale = [NSLocale currentLocale];
    
    // 单词首字母变大写
    NSString *result = string.capitalizedString;
    NSLog(@"string.capitalizedString: %@",result);
    // 指定系统环境变化
    result =  [string capitalizedStringWithLocale:locale];
    NSLog(@"[string capitalizedStringWithLocale:]: %@",result);
    
    //全变大写
    result = string.uppercaseString;
    NSLog(@"string.uppercaseString: %@",result);
    result = [string uppercaseStringWithLocale:locale];
    NSLog(@"[string uppercaseStringWithLocale:]: %@",result);
    
    // 全变小写
    result = string.lowercaseString;
    NSLog(@"string.lowercaseString: %@",result);
    result = [string lowercaseStringWithLocale:locale];
    NSLog(@"[string lowercaseStringWithLocale:]: %@",result);
}
```
输出：
```
2019-12-22 14:13:22.293088+0800 ObjcBaseDemo[32533:501266] ***Changing Case***
2019-12-22 14:13:22.293328+0800 ObjcBaseDemo[32533:501266] string.capitalizedString: Hello,中国;Hello,World;
2019-12-22 14:13:22.293531+0800 ObjcBaseDemo[32533:501266] [string capitalizedStringWithLocale:]: Hello,中国;Hello,World;
2019-12-22 14:13:22.293786+0800 ObjcBaseDemo[32533:501266] string.uppercaseString: HELLO,中国;HELLO,WORLD;
2019-12-22 14:13:22.293995+0800 ObjcBaseDemo[32533:501266] [string uppercaseStringWithLocale:]: HELLO,中国;HELLO,WORLD;
2019-12-22 14:13:22.294224+0800 ObjcBaseDemo[32533:501266] string.lowercaseString: hello,中国;hello,world;
2019-12-22 14:13:22.294454+0800 ObjcBaseDemo[32533:501266] [string lowercaseStringWithLocale:]: hello,中国;hello,world;
```

- 字符串分割

```objc
+ (void)divideString
{
    NSLog(@"***Dividing Strings***");
    NSString *string = @"Hello,World;Hello,iOS;Hello,Objective-C";
    
    // 根据指定的字符串分割成数组
    NSArray<NSString *> *array = [string componentsSeparatedByString:@";"];
    NSLog(@"[string componentsSeparatedByString:]: %@", array);
    
    // 通过系统自带的分割方式分割字符串
    NSCharacterSet *set = [NSCharacterSet uppercaseLetterCharacterSet];
    array = [string componentsSeparatedByCharactersInSet:set];
    NSLog(@"[string componentsSeparatedByCharactersInSet:]: %@",array);
    
    // 返回指定位置后的字符串
    NSLog(@"[string substringFromIndex:]: %@", [string substringFromIndex:3]);
    
    // 返回指定范围的字符串
    NSRange range = {1,3};
    NSLog(@"[string substringWithRange:]: %@", [string substringWithRange:range]);
    
    // 返回指定位置前的字符串
    NSLog(@"[string substringToIndex:]: %@", [string substringToIndex:3]);
}
```
输出：
```
2019-12-22 14:13:22.294661+0800 ObjcBaseDemo[32533:501266] ***Dividing Strings***
2019-12-22 14:13:22.294935+0800 ObjcBaseDemo[32533:501266] [string componentsSeparatedByString:]: (
    "Hello,World",
    "Hello,iOS",
    "Hello,Objective-C"
)
2019-12-22 14:13:22.295057+0800 ObjcBaseDemo[32533:501266] [string componentsSeparatedByCharactersInSet:]: (
    "",
    "ello,",
    "orld;",
    "ello,i",
    "",
    ";",
    "ello,",
    "bjective-",
    ""
)
2019-12-22 14:13:22.295259+0800 ObjcBaseDemo[32533:501266] [string substringFromIndex:]: lo,World;Hello,iOS;Hello,Objective-C
2019-12-22 14:13:22.295469+0800 ObjcBaseDemo[32533:501266] [string substringWithRange:]: ell
2019-12-22 14:13:22.295679+0800 ObjcBaseDemo[32533:501266] [string substringToIndex:]: Hel
```

- 替换字符串

```objc
+ (void)replaceSubstrings
{
    NSLog(@"***Replacing Substrings***");
    NSString *string = @"One Two Thre Four";
    NSRange searchRange = {0, string.length};
    
    //全局替换
    string = [string stringByReplacingOccurrencesOfString:@"One" withString:@"111"];
    NSLog(@"string stringByReplacingOccurrencesOfString:withString]: %@", string);
    
    // 设置替换的模式，并设置范围
    string = [string stringByReplacingOccurrencesOfString:@"Two" withString:@"222" options:NSCaseInsensitiveSearch range:searchRange];
    NSLog(@"[string stringByReplacingOccurrencesOfString:withString:options:]: %@", string);
    
    // 将指定范围的字符串替换为指定的字符串
    string = [string stringByReplacingCharactersInRange:searchRange withString:@"0"];
    NSLog(@"[string stringByReplacingCharactersInRange:withString:]: %@", string);
}
```
输出：
```
2019-12-22 14:13:22.295888+0800 ObjcBaseDemo[32533:501266] ***Replacing Substrings***
2019-12-22 14:13:22.296037+0800 ObjcBaseDemo[32533:501266] string stringByReplacingOccurrencesOfString:withString]: 111 Two Thre Four
2019-12-22 14:13:22.296278+0800 ObjcBaseDemo[32533:501266] [string stringByReplacingOccurrencesOfString:withString:options:]: 111 222 Thre Four
2019-12-22 14:13:22.296503+0800 ObjcBaseDemo[32533:501266] [string stringByReplacingCharactersInRange:withString:]: 0
```

- 获得共享的前缀

```objc
+ (void)getSharedPrefix
{
    NSLog(@"***Getting a Shared Prefix***");
    NSString *string = @"Hello,World";
    NSString *compareStr = @"Hello,iOS";
    
    // 返回两个字符串相同的前缀
    NSString *prefix = [string commonPrefixWithString:compareStr options:NSCaseInsensitiveSearch];
    NSLog(@"[string commonPrefixWithString:options:]: %@", prefix);
}
```
输出：
```
2019-12-22 14:13:22.296740+0800 ObjcBaseDemo[32533:501266] ***Getting a Shared Prefix***
2019-12-22 14:13:22.296992+0800 ObjcBaseDemo[32533:501266] [string commonPrefixWithString:options:]: Hello,
```

- 获取数值

```objc
+ (void)getNumberValues
{
    NSLog(@"***Getting Numeric Values***");
    NSString *string = @"123";
    
    NSLog(@"doubleValue:%.3f", string.doubleValue);
    NSLog(@"floatValue:%.2f", string.floatValue);
    NSLog(@"intValue:%d", string.intValue);
    NSLog(@"integerValue:%ld", (long)string.integerValue);
    NSLog(@"longLongValue:%lld", string.longLongValue);
    NSLog(@"boolValue:%d", string.boolValue);
}
```
输出：
```
2019-12-22 14:13:22.297177+0800 ObjcBaseDemo[32533:501266] ***Getting Numeric Values***
2019-12-22 14:13:22.297393+0800 ObjcBaseDemo[32533:501266] doubleValue:123.000
2019-12-22 14:13:22.297594+0800 ObjcBaseDemo[32533:501266] floatValue:123.00
2019-12-22 14:13:22.297781+0800 ObjcBaseDemo[32533:501266] intValue:123
2019-12-22 14:13:22.297971+0800 ObjcBaseDemo[32533:501266] integerValue:123
2019-12-22 14:13:22.298162+0800 ObjcBaseDemo[32533:501266] longLongValue:123
2019-12-22 14:13:22.298363+0800 ObjcBaseDemo[32533:501266] boolValue:1
```

- 使用路径

```objc
+ (void)workWithPaths {
    NSLog(@"***Working with Paths***");
    // 获取应用中Document文件夹
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSLog(@"paths: %@", paths);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    NSLog(@"documentsDirectory: %@", documentsDirectory);
    
    // 路径拆分为数组中的元素
    NSArray<NSString *> *pathComponents = documentsDirectory.pathComponents;
    // 将数组中的元素拼接为路径
    documentsDirectory = [NSString pathWithComponents:pathComponents];
    NSLog(@"[NSString pathWithComponents:]: %@", documentsDirectory);
    // 加载测试数据
    NSString *string = @"This is a test string";
    NSString *filePath = [documentsDirectory stringByAppendingPathComponent:@"test.plist"];
    NSLog(@"[documentsDirectory stringByAppendingPathComponent:]: %@", filePath);
    [string writeToFile:filePath atomically:YES encoding:NSUTF8StringEncoding error:nil];
    filePath = [documentsDirectory stringByAppendingPathComponent:@"test1.plist"];
    NSLog(@"[documentsDirectory stringByAppendingPathComponent:]: %@", filePath);
    [string writeToFile:filePath atomically:YES encoding:NSUTF8StringEncoding error:nil];
    // 寻找文件夹下包含指定扩展名的文件路径
    NSString *outputName;// 相同的前缀
    NSArray *filterTypes = [NSArray arrayWithObjects:@"txt", @"plist", nil];
    NSUInteger matches = [documentsDirectory completePathIntoString:&outputName caseSensitive:YES matchesIntoArray:&pathComponents filterTypes:filterTypes];
    NSLog(@"找到：%lu", (unsigned long)matches);
    
    // 添加路径
    filePath = [documentsDirectory stringByAppendingPathComponent:@"test"];
    NSLog(@"[documentsDirectory stringByAppendingPathComponent:]: %@", filePath);
    // 添加扩展名
    filePath = [filePath stringByAppendingPathExtension:@"plist"];
    
    // 是否绝对路径
    NSLog(@"filePath.absolutePath:%d", filePath.absolutePath);
    
    // 最后一个路径名
    NSLog(@"filePath.lastPathComponent:%@", filePath.lastPathComponent);
    
    // 扩展名
    NSLog(@"filePath.pathExtension:%@", filePath.pathExtension);
    
    // 去掉扩展名
    string = filePath.stringByDeletingPathExtension;
    NSLog(@"filePath.stringByDeletingPathExtension: %@", string);
    
    // 去掉最后一个路径
    string = filePath.stringByDeletingLastPathComponent;
    NSLog(@"filePath.stringByDeletingLastPathComponent: %@", string);
    
    // 批量增加路径，并返回生成的路径
    pathComponents = [filePath stringsByAppendingPaths:pathComponents];
    NSLog(@"[filePath stringsByAppendingPaths:]: %@", pathComponents);
    
    // 没啥用
    string = filePath.stringByExpandingTildeInPath;
    NSLog(@"filePath.stringByExpandingTildeInPath: %@", string);
    string = filePath.stringByResolvingSymlinksInPath;
    NSLog(@"filePath.stringByResolvingSymlinksInPath: %@", string);
    string = filePath.stringByStandardizingPath;
    NSLog(@"filePath.stringByStandardizingPath: %@", string);
}
```
输出：
```
2019-12-22 14:13:22.298524+0800 ObjcBaseDemo[32533:501266] ***Working with Paths***
2019-12-22 14:13:22.298782+0800 ObjcBaseDemo[32533:501266] paths: (
    "/Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents"
)
2019-12-22 14:13:22.299026+0800 ObjcBaseDemo[32533:501266] documentsDirectory: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents
2019-12-22 14:13:22.299270+0800 ObjcBaseDemo[32533:501266] [NSString pathWithComponents:]: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents
2019-12-22 14:13:22.299504+0800 ObjcBaseDemo[32533:501266] [documentsDirectory stringByAppendingPathComponent:]: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist
2019-12-22 14:13:22.300583+0800 ObjcBaseDemo[32533:501266] [documentsDirectory stringByAppendingPathComponent:]: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test1.plist
2019-12-22 14:13:22.302410+0800 ObjcBaseDemo[32533:501266] 找到：2
2019-12-22 14:13:22.302503+0800 ObjcBaseDemo[32533:501266] [documentsDirectory stringByAppendingPathComponent:]: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test
2019-12-22 14:13:22.302579+0800 ObjcBaseDemo[32533:501266] filePath.absolutePath:1
2019-12-22 14:13:22.302653+0800 ObjcBaseDemo[32533:501266] filePath.lastPathComponent:test.plist
2019-12-22 14:13:22.302759+0800 ObjcBaseDemo[32533:501266] filePath.pathExtension:plist
2019-12-22 14:13:22.302842+0800 ObjcBaseDemo[32533:501266] filePath.stringByDeletingPathExtension: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test
2019-12-22 14:13:22.302928+0800 ObjcBaseDemo[32533:501266] filePath.stringByDeletingLastPathComponent: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents
2019-12-22 14:13:22.303026+0800 ObjcBaseDemo[32533:501266] [filePath stringsByAppendingPaths:]: (
    "/Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist/Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test1.plist",
    "/Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist/Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist"
)
2019-12-22 14:13:22.303103+0800 ObjcBaseDemo[32533:501266] filePath.stringByExpandingTildeInPath: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist
2019-12-22 14:13:22.303281+0800 ObjcBaseDemo[32533:501266] filePath.stringByResolvingSymlinksInPath: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist
2019-12-22 14:13:22.303387+0800 ObjcBaseDemo[32533:501266] filePath.stringByStandardizingPath: /Users/rodgerjluo/Library/Developer/CoreSimulator/Devices/B06EE11B-2BE1-418F-85EA-5BB72CF7C2A6/data/Containers/Data/Application/925BF22B-C562-4622-BE3F-0612D4A814CF/Documents/test.plist
```

- 使用URL

```objc
+ (void)workWithURLs {
    NSLog(@"***Working with URL Strings***");
    NSString *path = @"hello/world";
    
    NSCharacterSet *set = [NSCharacterSet controlCharacterSet];
    
    // 转%格式码
    NSString *string = [path stringByAddingPercentEncodingWithAllowedCharacters:set];
    NSLog(@"[path stringByAddingPercentEncodingWithAllowedCharacters:]: %@",string);
    
    // 转可见
    string = string.stringByRemovingPercentEncoding;
    NSLog(@"string.stringByRemovingPercentEncoding: %@",string);
}
```
输出：
```
2019-12-22 14:13:22.303584+0800 ObjcBaseDemo[32533:501266] ***Working with URL Strings***
2019-12-22 14:13:22.303805+0800 ObjcBaseDemo[32533:501266] [path stringByAddingPercentEncodingWithAllowedCharacters:]: %68%65%6C%6C%6F%2F%77%6F%72%6C%64
2019-12-22 14:13:22.304040+0800 ObjcBaseDemo[32533:501266] string.stringByRemovingPercentEncoding: hello/world
```

- 初始化动态字符串

```objc
+ (void)createAndInitMutableString {
    NSLog(@"***Creating and Initializing a Mutable String***");
    // 创建空字符串
    NSMutableString *mString = [NSMutableString stringWithCapacity:3];
    NSLog(@"[NSMutableString stringWithCapacity:]: %@", mString);
    
    mString = [[NSMutableString alloc] initWithCapacity:3];
    NSLog(@"[[NSMutableString alloc] initWithCapacity:]: %@", mString);
}
```
输出：
```
2019-12-22 14:13:22.304271+0800 ObjcBaseDemo[32533:501266] ***Creating and Initializing a Mutable String***
2019-12-22 14:13:22.304524+0800 ObjcBaseDemo[32533:501266] [NSMutableString stringWithCapacity:]:
2019-12-22 14:13:22.304738+0800 ObjcBaseDemo[32533:501266] [[NSMutableString alloc] initWithCapacity:]:
```

- 修改动态字符串

```objc
+ (void)modifyMutableString
{
    NSLog(@"***Modifying a String***");
    NSMutableString *mString = [NSMutableString stringWithCapacity:10];
    
    // Format添加
    [mString appendFormat:@"%@",@"Hello,"];
    NSLog(@"[mString appendFormat:]: %@", mString);
    
    //添加单一字符串
    [mString appendString:@"World"];
    NSLog(@"[mString appendString:]: %@", mString);
    
    // 删除指定范围的字符串
    NSRange range = {0,3};
    [mString deleteCharactersInRange:range];
    NSLog(@"[mString deleteCharactersInRange:]: %@", mString);
    
    // 指定位置后插入字符串
    [mString insertString:@"***" atIndex:0];
    NSLog(@"[mString insertString:]: %@", mString);
    
    // 替换指定范围的字符串
    [mString replaceCharactersInRange:range withString:@"111"];
    NSLog(@"[mString replaceCharactersInRange:withString:]: %@", mString);
    
    // 将指定范围内的字符串替换为指定的字符串，并返回替换了几次
    NSUInteger index =  [mString replaceOccurrencesOfString:@"1" withString:@"23" options:NSCaseInsensitiveSearch range:range];
    NSLog(@"[mString replaceOccurrencesOfString:withString:options:range:]: %@ index: %lu", mString, (unsigned long)index);
    
    // 字符串全替换
    [mString setString:@"lalalalala"];
    NSLog(@"[mString setString:]: %@", mString);
}
```
输出：
```
2019-12-22 14:13:22.304916+0800 ObjcBaseDemo[32533:501266] ***Modifying a String***
2019-12-22 14:13:22.305134+0800 ObjcBaseDemo[32533:501266] [mString appendFormat:]: Hello,
2019-12-22 14:13:22.305352+0800 ObjcBaseDemo[32533:501266] [mString appendString:]: Hello,World
2019-12-22 14:13:22.305598+0800 ObjcBaseDemo[32533:501266] [mString deleteCharactersInRange:]: lo,World
2019-12-22 14:13:22.305735+0800 ObjcBaseDemo[32533:501266] [mString insertString:]: ***lo,World
2019-12-22 14:13:22.305962+0800 ObjcBaseDemo[32533:501266] [mString replaceCharactersInRange:withString:]: 111lo,World
2019-12-22 14:13:22.306181+0800 ObjcBaseDemo[32533:501266] [mString replaceOccurrencesOfString:withString:options:range:]: 232323lo,World index: 3
2019-12-22 14:13:22.306413+0800 ObjcBaseDemo[32533:501266] [mString setString:]: lalalalala
```


[demo地址](https://github.com/rodger1017/Study/tree/master/ObjcBaseDemo)