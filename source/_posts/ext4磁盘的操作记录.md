---
title: ext4磁盘的操作记录
date: 2022-10-04 22:01:35
tags: ['Linux', '磁盘']
---

## 磁盘信息查看
`sudo fdisk -l`: 查看当前磁盘的信息

## 格式化
`sudo mkfs -t ext4 <device>`: 格式化为ext4格式（注：格式化操作会让磁盘内容丢失）
> 例子：`sudo mkfs -t ext4 /dev/sdc1`

## 修改磁盘标识
`e2label <device> <name>`: 修改磁盘标识（磁盘名）
> 例子：`e2label /dev/sdc1 new_disk`

## 挂载/取消挂载
`sudo mount <device> <directory>`: 挂载
`sudo umount <device>`：取消挂载

## 注意事项
1. 格式化后的磁盘可能没有权限读写，需要在挂载之后修改权限。`sudo chown -R <directory> <user>:<group>`
> 例子：`sudo chown -R /mnt/new_disk qbug:qbug`
