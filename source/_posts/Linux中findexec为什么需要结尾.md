title: Linux中find -exec为什么需要"\;"结尾
date: '2021-05-15 11:26:00'
updated: '2021-05-16 00:51:46'
tags: [Linux, find]
permalink: /articles/2021/05/15/1621049160760.html
---
![](https://b3logfile.com/bing/20181209.jpg?imageView2/1/w/960/h/540/interlace/1/q/100) 

# 前记

在Linux中可以使用`find -exec`来查找相关目录/文件并做出cp, rm, mv等操作. 其格式一般为`find path [option] -exec mv {} /path/file \;`

# 需要以 \; 结尾的原因

首先, 我们先不以\;结尾来输入指令, 结果发现出现了如下的提示: **missing argument to exec.** 也就是说Linux没有正确识别到给`-exec`的参数.

而通过`man find`并且找到 `-exec command ;` 的解释可以发现有这么一句话:

> All following arguments to find are taken to be arguments to the command until an argument consisting of `;' is encountered.
> 翻译 : 以下所有要查找的参数都将被视为命令的参数，直到包含“；”的参数遇到。

而在Linux中, **分号;** 是有特殊含义的, 因此我们需要加上**转移符号\** 来进行转义.

因此当我们输入`find /home -name testfile -exec cp {} ~/testfile3 \;`后可以发现cp命令成功执行.


# 管道实现
除了使用`find -exec`来实现查找复制之外，我们还可以使用管道+xargs来实现。

首先，如果我们这样输入：`find ./ -name "*.log" | xargs cp ./logs/`，那铁定是不行的。这样输入的意思是将`./logs/`复制到`find`的结果中。即：`cp ./logs/ (find ./ -name "*.log")`。

要想像上述find -exec那样使用`{}`来代替find结果我们需要给xargs加上参数 -i. 最终结果：`find ./ -name "*.log" | xargs cp {} ./logs/`.

至于-i是什么意思，我们只需要查看说明书即可：`man xargs`查看即可，其实差不多就是占位符的意思。有意思的是里面的`-I`选项，我们可以使用`-I`来指定占位符的内容，例如：`find ./ -name "*.log" | xargs -I 0.o cp 0.o ./logs/`
