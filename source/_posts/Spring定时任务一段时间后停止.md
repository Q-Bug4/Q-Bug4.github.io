title: Spring 定时任务一段时间后停止
date: '2021-05-19 21:36:47'
updated: '2021-05-19 21:39:49'
tags: [http, 超时]
permalink: /articles/2021/05/19/1621431407601.html
---
# 前记

我在项目中需要定时使用http请求数据, 在本地部署到tomcat使用过程中无问题, 在部署到vultr的服务器后一段时间后发现出现了问题 : **定时任务完全不工作了.**

先说结论 : 由于网络不佳导致http请求超时, 在代码中加上timeout即可. (我还以为超时会直接抛出异常，也可能是我哪里姿势不对)

# 参数环境

定时任务使用spring的`@scheduled`注解, 每30s执行一次.

`tomcat9`

`spring 5.0.2.RELEASE`

工具库`Hutool 5.6.3`

# 思考问题所在

首先在本地部署时定时任务连续执行了一天, 正常运行没有出现问题.

然而一旦部署到vultr就会时不时出现定时任务不运行, 考虑到vultr的网络问题, 猜想**应该是http请求失败**导致定时任务完全不执行.

# 解决

首先到tomcat的日志中查看是否存在异常抛出, 在tomcat的logs目录中, 查看所有定时任务停止的时间段的日志. 可惜, 并没有发现错误. 考虑到vultr是外国服务器, 可能请求超时, 于是**给所有http请求加上timeout.**

Java中使用Hutool库如下 :

```java
int timeout = 2000;
String body = HttpRequest.post(host).timeout(timeout).header().body().execute().body(); // POST请求
String body2 = HttpUtil.get(url,timeout); // GET请求
```

来加上timeout进行请求. 问题解决了.
