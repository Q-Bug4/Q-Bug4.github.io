title: 'nodejs手动安装过程出现"/usr/bin/env: "node" no such file"等问题'
date: '2021-08-24 21:25:55'
updated: '2021-08-24 21:25:55'
tags: [杂症]
permalink: /articles/2021/08/24/1629811555795.html
---
今天安装[nodejs](https://nodejs.org/en/)的时候，开开心心下好包，解压到`node`文件夹中，在终端跑了一下`node/bin/npm -v`，结果居然出现了`/usr/bin/env: "node" no such file`的报错，我擦列？

其实这个问题是因为node使用的是`/usr/bin/node`来跑npm，所以我们需要把解压后的`node/bin/node`建立软连接到`usr/bin/node`上。只需执行`ln -s /your path/node/bin/node /usr/bin/node`即可。

**但是这里有个小坑，建立连接的时候不要偷懒写./node，写全路径**，不然就会`/usr/bin/env: “node”: 符号连接的层数过多`的问题。
