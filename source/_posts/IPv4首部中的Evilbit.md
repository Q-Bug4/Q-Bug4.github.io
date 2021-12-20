title: IPv4首部中的Evil bit
date: '2021-08-15 17:44:34'
updated: '2021-08-15 18:58:33'
tags: [彩蛋, Internet]
permalink: /articles/2021/08/15/1629020674611.html
---
![](https://b3logfile.com/bing/20200807.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

今天突然了解到IPv4中有一个“彩蛋”：Evil bit. 觉得很有意思所以在这里记下来。

### 首部结构

我们先看一下[维基百科 IPv4](https://en.wikipedia.org/wiki/IPv4#Header)给出的IPv4首部结构：

![image.png](https://b3logfile.com/file/2021/08/image-6669a4e2.png)

可以看到其中有一个3位的Flags标志，flags的第一位就是我们的**Evil bit**, 第二位是DF(不分片)，第三位是MF（还有分片）。

### Evil bit来源

在[维基百科 Evil bit](https://en.wikipedia.org/wiki/Evil_bit)中说到了**Evil bit**来源于[RFC 3514](https://www.ietf.org/rfc/rfc3514.txt), 是一个愚人节的幽默玩笑，用于指出ip数据包是不是有恶意的，因为有了这个字段，所以接收方只需要检查Evil bit就可以知道数据包是否是恶意数据包。

> The **evil bit** is a fictional [IPv4 packet header](https://en.wikipedia.org/wiki/IPv4#Header "IPv4") field proposed in RFC 3514, a humorous [April Fools&#39; Day RFC](https://en.wikipedia.org/wiki/April_Fools%27_Day_RFC "April Fools' Day RFC") from 2003 authored by [Steve Bellovin](https://en.wikipedia.org/wiki/Steven_M._Bellovin "Steven M. Bellovin"). The [RFC](https://en.wikipedia.org/wiki/Request_for_Comments "Request for Comments") recommended that the last remaining unused bit, the "Reserved Bit"^[[1]](https://en.wikipedia.org/wiki/Evil_bit#cite_note-1)^ in the [IPv4](https://en.wikipedia.org/wiki/IPv4) packet header, be used to indicate whether a packet had been sent with malicious intent, thus making [computer security](https://en.wikipedia.org/wiki/Computer_security "Computer security") engineering an easy problem – simply ignore any messages with the evil bit set and trust the rest.

当然了，都说了是愚人节玩笑了，那怎么可能有用嘛，不过倒是真的有这个字段，就是没啥用，一般博客或者文章也不会提及这个字段是干嘛的。

而在[RFC 3514](https://www.ietf.org/rfc/rfc3514.txt)中煞有其事地提到：

> Currently-assigned values are defined as follows:
>
> 0x0 If the bit is set to 0, the packet has no evil intent. Hosts, network elements, etc., SHOULD assume that the packet is harmless, and SHOULD NOT take any defensive measures. (We note that this part of the spec is already implemented by many common desktop operating systems.)
>
> 0x1 If the bit is set to 1, the packet has evil intent. Secure systems SHOULD try to defend themselves against such packets. Insecure systems MAY chose to crash, be penetrated, etc.

### 后记
除了本文所说到的`Evil bit`彩蛋之外， 其实有很多IT项目都是有彩蛋的。例如redis的`LOLWUT`（甚至还能加参数），也有让人离职的Antd圣诞彩蛋（bushi）. 

