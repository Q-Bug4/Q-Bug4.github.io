---
title: Chrome无法同步密码
date: 2022-03-08 20:24:45
tags:
---

## 问题
最近重装了一次Manjaro, 然后重新在AUR安装Chrome之后发现Chrome同步的时候不会读取密码，输入密码后点击Save也不会保存密码，在设置中查看Saved passwords也是空空如也，试过重新登录但是并没有用。

在CLI打开Chrome，在打开一些原本会自动填充密码的网站时会发现CLI中出现了如下日志：
```text
[223901:223901:0308/201429.790834:ERROR:password_sync_bridge.cc(340)] Passwords datatype error was encountered: Failed to load entries from password store. Encryption service failure.
```
按照这个去网上搜索，发现了一个可以完美解决的方法。

## 猜想
因为问题是出现在重装之后，而我的`home`目录在重装的时候没有格式化，是另外的硬盘，所以上次登录的数据还存在，可能是存在的数据在校验的时候不匹配，导致无法同步密码。但是具体原因也不清楚，在搜索过程中并没有找到对应的解释。期间我还安装了Dev版本发现并没有这个问题再加上删除登录数据就可以解决这个问题，因此排除了是系统或是机器的问题。


## 删除Login Data与Login Data-journal
1. 退出Chrome
2. 进入Chrome对应用户资料目录
   1. Mac： `~/Library/Application Support/Google/Chrome`
   2. Linux：`~/.config/google-chrome`
   3. Windows `%UserProfile%\AppData\Local\Google\Chrome\User Data`
3. 如果你没有添加其他配置文件只有一份默认配置文件的话，那么`cd Default; rm Login\ Data; rm Login\ Data-journal` 
4. 如果有多份配置文件那么就重复对这多份配置文件做下面同样的操作: `rm <profile folder name>/Login\ Data; rm <profile folder name>/Login\ Data-journal`. (替换<>以及其中的内容为配置文件夹名)

## 参考
[Fixing a Google Chrome failure to save passwords](http://plasmasturm.org/log/chromepwstore/)
