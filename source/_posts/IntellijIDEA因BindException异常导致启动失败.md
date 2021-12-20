title: Intellij IDEA因BindException异常导致启动失败
date: '2021-06-22 12:14:28'
updated: '2021-06-22 12:14:28'
tags: [IDE]
permalink: /articles/2021/06/22/1624335267925.html
---
![](https://b3logfile.com/bing/20191119.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

# 前记

最近几天电脑启动Pycharm和IDEA都偶尔会出现异常并且无法打开, 最开始是只出现一两次重启解决了也就没管, 今天电脑刚开机就遇见IDEA无法启动了, 这可就不能忍了.

注 : 题主电脑是Windows

# 解决

由于上官网很快就解决了就忘记截图了, 不过栈顶的异常如下 : 

`java.net.BindException: Address already in use: bind`

看到这个异常的时候大概推测应该是端口被占用了, 刚好异常提醒很贴心的给出了官网的处理连接[Start Failed, Internal error: recovering IDE to the working state after the critical startup error – IDEs Support (IntelliJ Platform) | JetBrains](https://intellij-support.jetbrains.com/hc/en-us/articles/360007568559)

在官网中找到了下面这段话

> If you get "**java.net.BindException: Address already in use: bind** " exception, please refer to [IDEA-238995](https://youtrack.jetbrains.com/issue/IDEA-238995) for the workaround.

进入上面的超链接就有解决方案了, 解决方案我贴在下面

### 官网给出的解决方案

**应变方法** : 在具有管理员权限的Powershell或者cmd中运行如下命令 :

( 如果是Win10可以直接右键最下角的开始就可以找到带管理员的powershell )

```
netsh int ipv4 set dynamicport tcp start=49152 num=16383
netsh int ipv4 set dynamicport udp start=49152 num=16383
```

如果上面的命令运行了之后还是打不开IDEA, 那么试一下下面的命令

```
net stop winnat
net start winnat
```

**PROBLEM**

If all the 50 ports between 6942 and 6991 are reserved, taken by the other apps or firewall doesn't allow IDE to bind on them, startup fails with the following exception:

```
java.util.concurrent.CompletionException: java.net.BindException: Address already in use: bind
    at java.base/java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:314)
    at java.base/java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:319)
    at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1702)
    at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
    at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
    at java.base/java.lang.Thread.run(Thread.java:834)
Caused by: java.net.BindException: Address already in use: bind
    at java.base/sun.nio.ch.Net.bind0(Native Method)
    at java.base/sun.nio.ch.Net.bind(Net.java:455)
    at java.base/sun.nio.ch.Net.bind(Net.java:447)
    at java.base/sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:227)
    at io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:134)
    at io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:550)
    at io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1334)
    at io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:506)
    at io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:491)
    at io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:973)
    at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:248)
    at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:356)
    at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:164)
    at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:472)
    at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500)
    at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989)
    at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    ... 1 more

-----
JRE 11.0.6+8-b765.25 amd64 by JetBrains s.r.o
C:\Users\yatwi\AppData\Local\JetBrains\Toolbox\apps\Rider\ch-0\201.6668.197\jbr
```
