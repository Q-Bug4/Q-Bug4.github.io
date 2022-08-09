---
title: Flutter dart_vlc库踩坑
date: 2022-08-09 23:19:58
tags: ['Flutter']
---

## dart_vlc
[dart_vlc](https://pub.dev/packages/dart_vlc)是`Flutter`中的一个库，用于播放视频。基本上这个库就是调用vlc来完成视频的播放。但是官方的文档似乎是有点缺失的，所以踩了一些坑。

## 环境
```bash
$ uname -a
Linux hasee 5.15.50-1-MANJARO #1 SMP PREEMPT Sun Jun 26 07:06
:30 UTC 2022 x86_64 GNU/Linux

$ flutter --version
Flutter 3.0.3 • channel stable •
https://github.com/flutter/flutter.git
Framework • revision 676cefaaff (7 周前) • 2022-06-22 11:34:4
9
-0700
Engine • revision ffe7b86a1e
Tools • Dart 2.17.5 • DevTools 2.12.2
```

## 使用前建议
dart_vlc虽然文档可能更新不太及时，但是他提供了一个可运行的example。克隆[dart_vlc的代码仓库](https://github.com/alexmercerind/dart_vlc)，运行`example/main.dart`后可以将例子跑起来，如果跑得起来说明在当前平台dart_vlc是没有问题的。

## 坑1：Readme中的版本是旧版本
在`pub.dev`上，最新版本是`3.0.0`，但是文档中的依赖引入写的却是`1.9.0`，出问题的时候搜了一下Issue才发现这件事情。

## 坑2：vlc 和 libvlc 冲突
最开始引入dart_vlc的时候，我电脑上是已经安装了vlc了。但是看文档中说还需要安装libvlc，一安装，好家伙，和vlc冲突了，需要先卸载vlc。于是后面我把vlc卸载了，然后安装了libvlc。然后当我想要重新把vlc安装回来的时候，elisa依赖libvlc了，无奈只能把elisa和libvlc全删了，再安装回vlc。

## 坑3：vlc.hpp not found
这个坑是最开始没有安装libvlc的时候发现的，但当时我没有在main方法中进行`await DartVLC.initialize(useFlutterNativeView: true);`. 后面在处理坑2之后发现了这件事所以补上了初始化语句，然后一路顺畅就没报错了。也不清楚是哪一步的问题。
