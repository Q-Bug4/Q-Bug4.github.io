title: 乱谈select, poll, epoll
date: '2021-06-21 00:52:15'
updated: '2021-08-05 17:47:28'
tags: [设计, IO]
permalink: /articles/2021/06/21/1624207935238.html
---
![](https://b3logfile.com/bing/20200421.jpg?imageView2/1/w/960/h/540/interlace/1/q/100) 

# 前记

select, poll, epoll 这三者是学习IO模型绕不开的三座山, 比较常见的是这三者为多路复用IO模型服务, 同时学习这三者对自身编程思想页多有裨益.

其工作流程大概是:

1. 为socket创建fd并声明动作(write, read, ...)
2. 将要监视(watched)的fd从用户态拷贝到内核态
3. 交由内核帮我们监视fd (此时函数为阻塞状态)
4. 当内核发现fd有数据到来时通知函数继续执行

# 前置知识

### file descriptor (简称 fd)

file descriptor我个人理解为一个索引, 一个非负数, 用于定位一个在操作系统打开的文件. 也就是说利用fd, 我们可以对fd对应的文件做一些监视, 写入, 读取等操作.

"Linux中一切皆文件", 这句话很多人应该都耳熟能详. socket连接在Linux中也是一个文件, 因此我们可以通过fd来对socket进行操作.

### 用户态与内核态

在地址空间中，对于每个进程都会有4G（32位）/ 256T（64位下）的虚拟内存空间，其中用户空间:内核空间=3:1。用户运行程序时一般是在用户态进行一些业务处理，当用户调用了系统调用后就会“陷入”内核态，交由内核完成一些操作后再返回到用户态。

### 多路复用IO

--------

# select

```scpp
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`
```

假如操作系统中有如下fd集合0-13, 红色数字为我们通过select交给内核进行监视的fd: 1, 6, 7, 11, 12

![image.png](https://b3logfile.com/file/2021/06/image-c2e2af30.png)

当select接收到我们给的fd集合时, 会逐个从0到12遍历监视, 即使我们只需要监视红色的fd, select也会从0开始扫描直到扫描到我们给出的最大值为止. 因此select执行过程会有很多无效的性能消耗.

接下来直接看代码example :

```cpp
... // 省略一些声明以及不重要的代码
for (i=0;i<5;i++) {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    // fds为要监视的fd集合, 通过accept来获取socket对应的fd
    fds[i] = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    if(fds[i] > max)
    	max = fds[i];   // max的作用为告诉select我们需要监视的fd的最大值(fd是一个非负整数)
			// 防止select遍历完操作系统内的所有fd
  }
  
  while(1){
// 每次select都需要初始化一次rset
	FD_ZERO(&rset);  // 将rset置为0, rset为select操作后修改结果
  	for (i = 0; i< 5; i++ ) {
  		FD_SET(fds[i],&rset);
  	}
 
   	puts("round again");
// int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
// nfds : fd的最大值, 即max+1
// readfds, writefds, exceptfds : 需要监视哪个fd集合发生什么动作就将该fd集合放在对应的参数位置, 若传入0
// 则说明不需要监视该动作
// timeout : 超时时间
	select(max+1, &rset, NULL, NULL, NULL);  // 调用后阻塞等待内核通知或者超时
 
	for(i=0;i<5;i++) {
		if (FD_ISSET(fds[i], &rset)){
			memset(buffer,0,MAXBUF);
			read(fds[i], buffer, MAXBUF);
			puts(buffer);
		}
	}
  }
```

参考如上例子, 我们需要向select传入fd的最大值以及一个rset引用对象, select会将结果存放在rset中. 而我们需要做的就是确定我们需要监视的每一个fd是否在rset中被置位了, 简单来说就是通过rset来判断fd是否有数据到来, 有就读取.

这是select性能低的另一个点, 每次都需要遍历我们的fd集合才能知道有多少有哪些socket有数据可以读写.

总的来说, select存在的缺点有:

1. select实现低效, 需要从0开始判断fd, 其中有很多是我们不需要判断的fd
2. select每次执行之前都要初始化传入的rset数组, 否则上一次的执行结果会影响到下一次
3. select的执行结果需要一个一个遍历, 不能直接确定有多少个有哪些socket可以进行操作

但是select也有一个优点 : 够"便携"(一些文章说的是portable), 基本上Unix平台都会有select支持.

# poll

话不多说, 先摆出poll的方法体

```cpp
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

可以看到, poll比select少了很多参数, 这里面timeout和select是一样的就不看了. 虽然poll中依然有一个和select一样的参数nfds, 但是这里的nfds指的是要监视的fd的集合长度, 也就是传入的fds数组的长度.

我们可以看到poll中出现了一个结构体`pollfd`, 其定义如下

```cpp
struct pollfd {
      int fd;		// fd
      short events; 	// 希望发生的事件
      short revents;	// 实际发生的事件
};
```

其中events为输入的参数, 而revents为poll返回的结果. events与revents实际上除了含义不一样, 其设计结构和取值是一样的, 均可以取如下表内值. ( 后三个为出现错误return的值, 所以events取后三个没有意义 )


| **event**        | **description**                                           |
| ------------------ | ----------------------------------------------------------- |
| ***POLLIN***     | **Data may be read without blocking.**                    |
| ***POLLPRI***    | **High-priority(OOB) data may be read without blocking.** |
| ***POLLOUT***    | **Data may be written without blocking.**                 |
| ***POLLWRNORM*** | **Equivalent to POLLOUT.**                                |
| ***POLLRDNORM*** | **Equivalent to POLLIN.**                                 |
| ***POLLNORM***   | **Equivalent to POLLIN.**                                 |

直接看代码:

```cpp
for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    pollfds[i].fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    pollfds[i].events = POLLIN; 
  }
  sleep(1);
  while(1){
  	puts("round again");
	poll(pollfds, 5, 50000);
 
	for(i=0;i<5;i++) {
		if (pollfds[i].revents & POLLIN){
			pollfds[i].revents = 0;
			memset(buffer,0,MAXBUF);
			read(pollfds[i].fd, buffer, MAXBUF);
			puts(buffer);
		}
	}
  }
```

可以看到, 传入poll的参数是一个pollfd数组以及其长度5, 这和select需要传入fd值已经有不小的差别的.

再看判断fd是否准备好进行IO操作的部分:`if (pollfds[i].revents & POLLIN)`, 这个条件判断最开始我看的时候有点懵圈, 怎么与操作一下就可以判断了, 后来查看poll的说明, 发现[poll(2) - Linux manual page (man7.org)](https://man7.org/linux/man-pages/man2/poll.2.html)对events字段是这么解释的:

> The field events is an input parameter, **a bit mask** specifying the
> events the application is interested in for the file descriptor
> fd.

yes, events是用某一位取1而其他位取0来表示其值的. 也就是说events以及其宏定义只能取:

1. 0001
2. 0010
3. 0100
4. 1000
5.  ... (short在不同机器似乎有不同位数? 这里不重要所以只用了4位来表示)

假设events取值为0100, 那么只有当events & 0100才会出现非0值, 即只有和自身相等的值做&操作才能得到一个非0值, 即真值.

判断成立后接下来的代码就跟select差不多了, 重置revents, 进行IO操作.

**select vs poll:**

1. poll 不需要计算fd的最大值, 因为poll的实现是只监视传入的fd而不需要从头开始遍历操作系统所有fd
2. poll 实现更高效简洁

然而poll依然存在一个本质上的问题，需要自己轮训所有fd来确定是否有事件发生。而接下来的epoll，则是采用注册call_back()的方式，来让事件发生后主动通知epoll_wait()，这种本质上的改变也带来了效率上的提高

# epoll

select和poll每次都会在用户态下准备好fd集合, 并将集合发送给内核. 因此每次需要更改fd集合时都需要重新修改集合, 再发送给内核后重新调用select/poll.

而epoll在这一点上做出了改变, epoll通过`epoll_create`在内核中创建了一块和用户态共享的区域, 再通过`epoll_ctl`对这块区域进行操作, 免去了从用户态拷贝至内核态的操作. 同时，epoll为每一个fd都注册了一个回调函数call_back()，当fd发生事件时可以通过call_back()响应给应用程序。

上代码:

```cpp
struct epoll_event events[5];
  int epfd = epoll_create(10);
  ...
  ...
  for (i=0;i<5;i++) 
  {
    static struct epoll_event ev;
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    ev.data.fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    ev.events = EPOLLIN;
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev); 
  }
  
  while(1){
  	puts("round again");
  	nfds = epoll_wait(epfd, events, 5, 10000);

	for(i=0;i<nfds;i++) {
			memset(buffer,0,MAXBUF);
			read(events[i].data.fd, buffer, MAXBUF);
			puts(buffer);
	}
  }
```

可以看到epoll的使用流程:

1. 用`epoll_create`在内核中新建一块区域存放要监视的fd集合, 这里的参数会被忽略, 但是也要写上非负数
2. 用`epoll_ctl`新增fd
3. 调用`epoll_wait`交由内核监视
4. 有事件发生后，遍历events集合, 进行IO操作.

细心的小伙伴可能会发现, 上述代码用到了一个`epoll_wait`的返回值作为遍历的上界. 这点和poll select有很明显的不同.

epoll执行后会将发生预期变化的fd集合存放在传入的events数组的数组头部, 并将最后准备好进行IO操作的fd数量作为返回值返回

如此我们在判断哪些fd需要进行IO操作的时候就可以用返回值来进行遍历操作, 所遍历到的元素均为需要进行IO操作的元素, 在这一点上epoll的性能相较于poll是得到了很大的提升的.

实际上，epoll的执行方式是当回调函数被调用后，对应的fd就会被加入到就绪队列中，`epoll_wait()`就是不断轮训就绪队列查看是否存在就绪的fd。而这里就需要提到epoll的两种工作方式：LT模式，ET模式。

LT模式主要在于稳定，每次通知用户处理fd后，如果用户没有处理，那么LT模式下epoll在下一次通知还是会通知用户处理，同时会将该fd重新加入就绪队列。

而ET模式是高效，只通知一次，不管用户处不处理都只会通知一次，因此不会出现同一个fd一次事件发生而通知多次的情况发生。

epoll vs poll :

1. epoll_wait 阻塞过程中可以通过epoll_ctl对fd集合进行修改
2. epoll性能表现更好
3. epoll只返回发生预期变化的fd

### 细节问题

**epoll_create的参数**

[epoll - epoll_create and epoll_wait - Stack Overflow](https://stackoverflow.com/questions/9577230/epoll-create-and-epoll-wait)

**epoll_create的区域真的在内核态里吗**

是也不是，是和用户态共享的区域。
