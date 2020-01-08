---
title: iOS崩溃日志ips文件解析
layout: posts
categories: iOS
tag: objc
---

## 解析步骤

1. 下载 `xcarchive` 文件，找到对应的符号文件：`xxx.dSYM`
 
2. 查找 crash 分析工具 `symbolicatecrash`

```
➜  github_io git:(master) find /Applications/Xcode.app -name symbolicatecrash -type f
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/iOSSupport/Library/PrivateFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/WatchSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/AppleTVSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```

3. 将 `symbolicatecrash` 工具拷贝到 crash 分析文件夹，与 `.app` 和 `.app.dSYM` 放一起

```
➜  github_io git:(master) cp /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/iOSSupport/Library/PrivateFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash /Users/rodgerjluo/Downloads/CrashDump
```

4. 执行 `symbolicatecrash`

```
➜  CrashDump ./symbolicatecrash xxx.ips xxx.app.dSYM > xxx.crash
```

5. 如出现 `Error: "DEVELOPER_DIR" is not defined at ./symbolicatecrash line 69.` 错误则执行以下命令

```
➜  CrashDump export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"
```

6. 执行完成后会在 crash 文件夹里面多一个 `xxx.crash` 文件，可以查看崩溃堆栈信息。