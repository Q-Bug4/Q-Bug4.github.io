title: localhost 和 127.0.0.1
date: '2021-05-15 11:22:24'
updated: '2021-05-15 11:22:24'
tags: [localhost]
permalink: /articles/2021/05/15/1621048944039.html
---
# 前记

环境 : Windows10 + WSL2
今天在WSL2中搭建Halo博客的时候发现无法通过127.0.0.1进行访问, 在防火墙开放端口, 将ip更换为WSL2的ip依旧不能访问. 随后搜了一下"WSL2 访问内部服务"后发现原来是需要使用localhost进行访问, 使用localhost后就可以正常访问了.

# localhost 和 127.0.0.1 的区别

虽然localhost和127.0.0.1都可以用来访问到本地主机, 但这两者在细节之处还是有一些不同的.

1. localhost指"本机服务器", 不会被解析成ip, 而是直接访问到本机的服务, 同时因为是"本机服务器"的身份, 因此也不会受一些权限的限制.
2. 127.0.0.1指"本地地址", 相当于本机向127.0.0.1这个地址发起请求, 对于本机的相对服务来说, 这个请求不知道是谁发来的, 因此可能有权限的限制.
3. 将hosts中的关于127.0.0.1的解析行注释掉后, 通过Tracert查看两者经过的路由器可以看到, 两者走的路由器并不和baidu.com路由器不一样, 也就是说127.0.0.1和localhost都不会发送出去, 应该是在IP层就被拦截然后发送回应用层了.![image86580aa9d8114aaf88cd097300a0936f.png](https://b3logfile.com/file/2021/05/solo-fetchupload-8174272754117245921-8ed2fce5.png)

> 注:  127开头的ip地址一般用于环回, 也就是说不会发送出去, 只是在本机中传输, 其所在的回环接口一般被理解为虚拟网卡，并不是真正的路由器接口.

# 验证是否走网卡

在hosts文件中有这样的说明:"localhost name resolution is handled within DNS itself."即localhost直接交由DNS进行解析, 同时hosts中的localhost解析行也被注释了.

为了验证localhost和127.0.0.1究竟走不走网卡, 我把所有网卡都禁用了(连带WSL2的网卡也禁用了), 然后进行ping后发现localhost和127.0.0.1依然可以直接ping通, 且WSL2中的服务也可以很流畅的访问. 因此**localhost和127.0.0.1是不走网卡的.**

# 相关文章

[彻底明白ip地址，区分localhost、127.0.0.1和0.0.0.0](https://www.jianshu.com/p/ad7cd1d5be45)
[Windows访问WSL2的网络应用 - 微软官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/compare-versions#accessing-network-applications)