---
title: MySQL8 内存占用高
date: 2022-03-11 14:10:15
tags:
---

## 问题
xwash服务部署在一个内存只有1G的服务器上，最近突然发现服务器偶尔无法ssh登录，服务也停止了，重启服务器后发现内存居然占用高达95%，`docker stats`发现MySQL占用了300+MB的内存。

## 猜想
这里有3个问题需要找到答案：
1. xwash调用MySQL的频率并不高，数据量很小，但是为什么MySQL占用这么高？
2. 内存占用高为什么我的服务不可用了？
3. 在个人电脑上同样跑`docker compose up`后，`docker stats`中MySQL只占用了20+MB而服务器上却占用了300+MB？

猜想：
1. MySQL是一个磁盘IO数据库，为了加速应该是默认用内存做了一些缓存，导致内存占用高。
2. 内存占用高导致系统需要将数据从内存和虚拟内存之间频繁置换，导致CPU飙升无法进行服务。
3. 可能是系统、docker版本、拉取的镜像的原因。

## 解决
在一番搜索后发现只需要修改`/etc/mysql/conf.d/docker.cnf`中的内容（如果不是用docker那可以修改mysql.cnf效果应该是一样的），添加下面的配置后重启MySQL服务即可
```c
performance_schema_max_table_instances=400  
table_definition_cache=400  
table_open_cache=256
performance_schema=off
```
我们大概可以从上述配置中看出来，这些配置应该是降低了缓存的空间，禁用了`performance`这个功能，设置了`performance`的实例数量之类的操作。缓存比较好理解，毕竟缓存是需要内存空间的，但是这个`performance`是个什么东西？

在[MySQL :: MySQL 8.0 Reference Manual :: 27.19 Using the Performance Schema to Diagnose Problems](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-examples.html)中是这样描述的：
> The Performance Schema is a tool to help a DBA do performance tuning by taking real measurements instead of “wild guesses.”

也就是说这个`Performance Schema`是一个用于帮助提高数据库性能的工具，但是对于我们这种小机器来说，内存不足才是重要的问题，这个工具对我们来说可能是雪上加霜而不是雪中送炭，所以将其禁用掉。

而`performance_schema_max_table_instances`大致意思应该是检测的表的最大数量，在官方文档中是这样描述的：
> The maximum number of instrumented table objects

## 未解
1. 我们在配置文件中已经将`performance`关掉了为什么还需要设置`performance_schema_max_table_instances`为400？
2. 为什么个人电脑上的MySQL和服务器上的MySQL在使用同一个`docker-compose.xml`跑相同的服务时内存占用相差300MB？
```c
CONTAINER ID   NAME             CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
// 服务器docker stats（优化后，优化前占用为300+MB）
a41da2fc2d2   xwash-backend-mysql-1    0.10%     160.6MiB / 980.8MiB   16.38%    43.3kB / 483kB    70.2MB / 69.6MB   42
// 个人电脑docker stats
5e47f78866f4   xwash-backend-mysql-1    0.31%     24.84MiB / 7.646GiB   0.32%     2.93MB / 14.1MB   256MB / 70.7MB    46
```

## 参考资料
[docker 安装 MySQL 8，并减少内存占用 记录](https://www.cnblogs.com/WNpursue/p/10617217.html)  
[MySQL :: MySQL 8.0 Reference Manual :: 27.19 Using the Performance Schema to Diagnose Problems](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-examples.html)
