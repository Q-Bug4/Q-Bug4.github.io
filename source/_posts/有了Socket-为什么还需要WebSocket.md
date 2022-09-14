---
title: '有了Socket, 为什么还需要WebSocket?'
date: 2022-08-20 21:45:46
tags: ['Socket', 'WebSocket']
---

## WebSocket和Socket
我们知道Socket(套接字)，其实就是协议+ip+端口, 是基于TCP协议的。不同于HTTP短暂的请求/响应，Socket可以保持一条TCP长链接几天，同时通信双方可以随时向对方发送消息。

而WebSocket看上去像是Socket的翻版，同样的可以为通信双方建立一条链接进行通信，双方也可以随时发送消息给对方。而WebSocket解决的问题就是服务器向客户端推送消息的问题，**那么问题来了**
1. Socket同样也可以解决服务器向客户端推送消息的问题，为什么会有WebSocket出现而不是直接用Socket呢？
2. WebSocket和Socket之间有什么关系吗？

在查阅了众多资料后发现实际上WebSocket和Socket并没有什么很明显或者说很强的关系，查阅资料后得出的一个结论就是WebSocket和Socket没什么关系。

**对以上两个问题的解答**：
1. WebSocket的一个使用场景是浏览器，而浏览器本身存在很多限制例如同源策略，所以浏览器其实不允许直接操作TCP。因此需要一个新的协议来实现这个功能。(实际上我们可以在很多地方使用WebSocket，例如在Java中就可以使用WebSocket客户端, autojs脚本中也可以使用)
2. WebSocket可以用Socket来实现。WebSocket与HTTP一样位于应用层，运行在TCP之上；而我们熟知的Java中的Socket相当于是TCP层的，直接操纵TCP，更加灵活。

实际上WebSocket最开始叫做TCPConnection，后来改名为WebSocket. 感觉就像JavaScript与Java之间的关系，不过WebSocket因为是应用层的协议，肯定是需要调用传输层的协议，所以多少还是需要依赖于Socket的。

## 参考资料
[What's the difference between WebSocket and plain socket communication? - StackOverflow](https://stackoverflow.com/questions/28480575/whats-the-difference-between-websocket-and-plain-socket-communication)