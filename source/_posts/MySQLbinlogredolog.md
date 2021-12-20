title: MySQL - binlog & redo log
date: '2021-07-10 20:24:37'
updated: '2021-07-10 20:24:37'
tags: [待分类]
permalink: /articles/2021/07/10/1625919877898.html
---
![](https://b3logfile.com/bing/20171202.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

我们知道，MySQL常用的存储引擎InnoDB使用了两个日志模块：`binlog` 和 `redo log`来保证数据库有灾难恢复的能力（crash-safe）。本文主要记录关于两个日志模块相关的一些问题。

### 简单介绍一下binlog和redo log

**binlog:** MySQL自带的日志模块，用于记录数据库发生了什么改变。例如其他可能有一条记录：表user中，给id=4的地方的字段grade加1. 

**redo log:** InnoDB特有的日志模块，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来。

### 为什么需要redo log

因为数据库有数据变更时，就要向磁盘进行IO操作。最开始没有redo log的时候，每次有一条更新就需要将其更新到数据库中，而这个更新过程需要在庞大的数据库中找到该条数据。

查找过程如果频繁的话会占用大量IO，此时就出现了redo log，**redo log提高了MySQL的效率**。

redo log像是一个缓冲区，每次有数据更新就将记录写入redo log，然后更新内存中的数据，等到磁盘空闲或者MySQL觉得合适的时机才将redo log中的更新全部真正写入到数据库（磁盘）中。

另外一点，**redo log赋予了MySQL crash-safe的能力**.

### 为什么只用binlog没有crash-safe的能力

首先我们要知道，binlog可以用来主从同步和数据恢复。

如果MySQL只使用binlog日志模块，那我们分两种情况讨论：

1. 先提交事务，后写入binlog
2. 先写入binlog，后提交事务

如果先提交事务后写入binlog，那么可能会出现事务提交后数据库崩了，或者机器宕机，那么就会出现binlog中没有对应的记录，那么就会出现数据不一致的问题。

那先写入binlog呢？先写入binlog会出现binlog存在记录，但是事务没提交，一样是数据不一致。

因此只用binlog无法保证能够crash-safe。

### 怎么用两个日志模块保证crash-safe

MySQL使用了redo log + binlog，而这两个结合起来就可以保证crash-safe了，其流程如下。

1. 事务准备提交时，向redo log写入一条prepare记录使得事务为prepare状态
2. 写入binlog
3. 将redo log中的prepare状态改写为commit

这套流程也叫做**两阶段提交。**

同样的，我们假设在每个步骤之间会出现宕机的情况。

1. **redo log为prepare后立即宕机**：此时只有redo log中有prepare的记录，重启数据库后可以发现redo log中有一条prepare记录，binlog中无该prepare对应的记录，于是可以继续提交该事务。
2. **写入binlog后宕机**：重启数据库后，binlog中存在记录，而redo log中记录为prepare，此时就可以直接将prepare改为commit进行提交。

因此，无论在哪个步骤出现了异常错误，MySQL都可以通过两阶段提交的机制恢复数据。
