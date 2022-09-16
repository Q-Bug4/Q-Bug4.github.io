---
title: Flutter安卓使用外部存储
date: 2022-09-16 22:04:56
tags: ['Flutter', 'Android']
---

## 环境
```bash
flutter --version
# Flutter 3.0.5 • channel stable • https://github.com/flutter/flutter.git  
# Framework • revision f1875d570e (9 周前) • 2022-07-13 11:24:16 -0700  
# Engine • revision e85ea0e79c  
# Tools • Dart 2.17.6 • DevTools 2.12.2  

# Android 12
```

## 问题
最近在写Flutter的应用，这个应用会生成一些文件，但我在使用`file.saveTo('/storage/emulated/0/tmp.csv')`的时候会报错，提示没有权限，即使在手机上赋予应用对文件拥有完全控制权也无法保存。

搜索之后发现可以使用`path_provider`库的`getExternalStorageDirectory()`方法获取路径，再使用获取到的路径进行保存就可以成功了。

同时需要在`android/app/src/main/AndroidManifest.xml`文件中加入以下的权限请求。（有一些是不必要的，但是因为不熟悉安卓，搜索到的答案提到的权限都一股脑全加了）然后就大功告成了！
```xml
// AndroidManifest.xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_INTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.MANAGE_DOCUMENTS" />
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.MEDIA_CONTENT_CONTROL" />

<application
// ...
android:requestLegacyExternalStorage="true"
// ...
```