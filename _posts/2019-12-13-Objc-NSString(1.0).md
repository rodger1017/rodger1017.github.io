---
title: NSString & NSMutableString 基本用法(1.0)
layout: posts
categories: iOS
tag: objc
---

### NSString
- 创建字符串

```objc
NSString *str = @"test";
NSString *str1 = [NSString new];
NSString *str2 = [[NSString alloc] initWithString:@"test"];
NSString *str3 = [NSString stringWithFormat:@"this is %@", @"test"];
NSString *str4 = [[NSString alloc] initWithUTF8String:"测试"];
```

- 获取字符串长度

```objc
NSUInteger length = str.length;
NSLog(@"%@ length:%lu", str, (unsigned long)length);
```
输出：
```
2019-12-13 12:47:53.089504+0800 ObjcBaseDemo[7119:3821229] test length:4
```

- 获取字符串某个位置字符

```objc
unichar c = [str characterAtIndex:1];
NSLog(@"%c", c);
```
输出：
```
2019-12-13 12:47:53.089639+0800 ObjcBaseDemo[7119:3821229] e
```

- 截取字符串

```objc
NSRange range = {1, 2};  //location（索引开始的位置）、length（截取的长度）
NSString *subStr1 = [str substringWithRange:range];
NSString *subStr2 = [str substringFromIndex:1];
NSString *subStr3 = [str substringToIndex:2];
NSLog(@"substr1:%@ substr2:%@ substr3:%@", subStr1, subStr2, subStr3);
```
输出：
```
2019-12-13 12:47:53.089726+0800 ObjcBaseDemo[7119:3821229] substr1:es substr2:est substr3:te
```

- 获取子字符串在字符串中的索引位置和长度

```objc
NSRange range1 = [str rangeOfString:@"es"];
NSRange range2 = [str rangeOfString:@"ha"]; // 如果未找到 返回{-1, 0}
NSLog(@"range1:[%d, %d] range2:[%d, %d]", (int)range1.location, (int)range1.length, (int)range2.location, (int)range2.length);
```
输出：
```
2019-12-13 12:47:53.089792+0800 ObjcBaseDemo[7119:3821229] range1:[1, 2] range2:[-1, 0]
```

- 判断字符串内容是否相同（内容，不是地址）

```objc
BOOL isEqual1 = [str isEqualToString:str2];
NSLog(@"%@ is%@ equal to %@", str, isEqual1?@"":@" not", str2);
BOOL isEqual2 = [str2 isEqualToString:str4];
NSLog(@"%@ is%@ equal to %@", str2, isEqual2?@"":@" not", str4);
```
输出：
```
2019-12-13 12:47:53.089857+0800 ObjcBaseDemo[7119:3821229] test is equal to test
2019-12-13 12:47:53.089929+0800 ObjcBaseDemo[7119:3821229] test is not equal to 测试
```

- 替换字符串中的子字符串为给定的字符串

```objc
NSString * newStr = [str stringByReplacingOccurrencesOfString: @"e" withString: @"a"];
NSLog(@"newstr: %@", newStr);
```
输出：
```
2019-12-13 12:47:53.089996+0800 ObjcBaseDemo[7119:3821229] newstr: tast
```

- 字符串拼接

```objc
NSString *joinStr = [str stringByAppendingString: @" objc"];
NSLog(@"joinstr: %@", joinStr);
```
输出：
```
2019-12-13 12:47:53.090069+0800 ObjcBaseDemo[7119:3821229] joinstr: test objc
```

- 计算子字符串出现次数
  
```objc
NSUInteger count = 0;
NSString *str5 = @"testistestestask";
NSString *str6 = @"test";
// i=0 的时候比较123和123，i=1的时候比较23a和123，i=2的时候比较3as和123...以此类推，直到string1遍历完成
for(int i=0; i < str5 .length - str6 .length + 1; i++) {
    // 截取字符串与之比较是否相同
    if([[str5 substringWithRange:NSMakeRange(i, str6.length)] isEqualToString:str6 ]) {
        count++;
    }
}

NSLog(@"%@ contains %d %@",str5, count, str6);
```
输出：
```
2019-12-13 12:47:53.090131+0800 ObjcBaseDemo[7119:3821229] testistestestask contains 3 test
```

-  根据分隔符进行拆分

```objc
NSString *str7 = @"test1 test2";
NSArray *array = [str7 componentsSeparatedByString:@" "];
NSLog(@"array: %@", array);
```
输出：
```
2019-12-13 12:47:53.090249+0800 ObjcBaseDemo[7119:3821229] array: (
    test1,
    test2
)
```

- 数字转换

```objc
NSString *str8 = @"10";
BOOL b = [str8 boolValue];
int i = [str8 intValue];
NSLog(@"b: %d i: %d", b, i);
```
输出：
```
2019-12-13 12:47:53.090320+0800 ObjcBaseDemo[7119:3821229] b: 1 i: 10
```

- 大小写切换

```objc
NSString* str9 =@"hello WORLD";
NSString *ucstr = [str9 uppercaseString];   //转成大写
NSString *lcstr = [str9 lowercaseString];   //转成小写
NSString *cstr = [str9 capitalizedString];  //首字母大写，其余小写
NSLog(@"ucstr: %@ lcstr: %@ cstr: %@", ucstr, lcstr, cstr);
```
输出：
```
2019-12-13 12:47:53.090393+0800 ObjcBaseDemo[7119:3821229] ucstr: HELLO WORLD lcstr: hello world cstr: Hello World
```

- 编码 & 解码

```objc
NSString *str10 =@"你好啊";
NSString *encStr = [str10 stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
NSLog(@"encstr: %@", encStr);
NSString *str11 =@"\u5982\u4f55\u8054\u7cfb\u5ba2\u670d\u4eba\u5458\uff1f";
NSString *decStr = [str11 stringByRemovingPercentEncoding];
NSLog(@"decstr: %@", decStr);
```
输出：
```
2019-12-13 12:47:53.090470+0800 ObjcBaseDemo[7119:3821229] encstr: %E4%BD%A0%E5%A5%BD%E5%95%8A
2019-12-13 12:47:53.090598+0800 ObjcBaseDemo[7119:3821229] decstr: 如何联系客服人员？

```

- 验证

```objc
NSString *str12 = @"http:www.baidu.com";
BOOL prefix = [str12 hasPrefix:@"http"]; //是否是以http开头
BOOL suffix = [str12 hasSuffix:@"com"];  //文件路径是否以com结尾
NSLog(@"prefix: %d  suffix: %d", prefix, suffix);
```
输出：
```
2019-12-13 12:47:53.090708+0800 ObjcBaseDemo[7119:3821229] prefix: 1  suffix: 1
```

### NSMutableString

- 字符串拼接

```objc
NSMutableString *mstr1 = [[NSMutableString alloc] init];
NSLog(@"before append mstr1: %@", mstr1);
[mstr1 appendString: @"hello"];
NSLog(@"after append mstr1: %@", mstr1);
```
输出：
```
2019-12-13 12:47:53.090850+0800 ObjcBaseDemo[7119:3821229] before append mstr1:
2019-12-13 12:47:53.093384+0800 ObjcBaseDemo[7119:3821229] after append mstr1: hello
```

- 在指定的索引位置插入字符串

```objc
[mstr1 insertString:@" world" atIndex:5];
NSLog(@"after insert mstr1: %@", mstr1);
```
输出：
```
2019-12-13 12:47:53.093446+0800 ObjcBaseDemo[7119:3821229] after insert mstr1: hello world
```

-  删除指定范围的字符串

```objc
NSRange range3 = {0, 6};
[mstr1 deleteCharactersInRange:range3];
NSLog(@"after delete mstr1: %@", mstr1);
```
输出：
```
2019-12-13 12:47:53.093498+0800 ObjcBaseDemo[7119:3821229] after delete mstr1: world
```

[demo地址](https://github.com/rodger1017/Study/tree/master/ObjcBaseDemo)