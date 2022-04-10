---
title: Manjaro构建WinPE镜像
date: 2022-04-11 00:26:20
tags: [winpe, linux, manjaro]
---

## 问题
最近显卡不知道出什么问题了，Manjaro偶尔开机才能检测到，导致我想玩Overwatch的时候需要重复开机直到显卡出现，最近给我整烦了，于是在排查是不是系统问题过程中，就需要我们的WinPE出场了。

## 环境
GPU: Nvidia GTX 1050 + Intel集成显卡

## 动手
根据[Windows PE - ArchWiki](https://wiki.archlinux.org/title/Windows_PE#Creating_a_bootable_Windows_PE_image)里面的流程，我们需要：
1. 确保你有类似安装`Ventoy`之类的USB设备（因为本文主要是构建winPE镜像，因此不过多赘述这一块）
2. `sudo pacman -S wimlib`，因为下面用到的`mkwinpeimg`命令由这个lib提供
3. 下载个Windows镜像iso，我们可以去itellyou下载，很方便也很快，如果去官网下载又慢又不好找。
4. 新建一个文件`start.cmd`，里面的内容为
```cmd
cmd.exe
pause
```
5. 将下载的iso镜像和第4步的脚本放在同一个文件夹下（比较方便）
6. 将下载的iso挂载到某目录下，可以直接在当前目录下创建新目录来挂载：`sudo mount 下载的镜像名.iso 目录`。执行`mkwinpeimg --iso --windows-dir=你的目录 --start-script=start.cmd 输出winPE镜像名.iso`. （请替换目录和镜像名）。例子：`mkwinpeimg --iso --windows-dir=/media/winimg --start-script=start.cmd winpe.iso`
7. 得到的winpe.iso就是我们要的pe镜像了，然后记得取消挂载`sudo umount 你的目录`

## 可能出现的问题
1. 挂载的时候出现`mount: /xxx/xxx: failed to setup loop device for /xxx/xxx/windows.iso.`，我个人是使用sudo进行挂载就可以了。


## 参考文档
[Windows PE - ArchWiki](https://wiki.archlinux.org/title/Windows_PE#Creating_a_bootable_Windows_PE_image)