title: You are in emergency mode ... Cannot open access to console, the root account
  is locked. 的一种解决方法
date: '2021-08-12 13:51:09'
updated: '2021-08-21 13:09:10'
tags: [Linux, solution]
permalink: /articles/2021/08/12/1628747469166.html
---
# 前记

前天（2021.08.10）电脑（deepin 20.2）在向某个disk写入数据的时候突然出现无法写入的错误，进入home目录一看，所有目录和文件都变成了只读状态，写入文件得到报错：read only file system。浏览器也无法使用`One Tab`拓展了，推测应该是写入数据的时候出了点状况导致整个disk变成只读了。

因为我的根目录是挂载在另外的硬盘上的，所以想着重启也不会出现什么大问题，而且不排除是bug的可能，重启一下试试，结果一重启，电脑干脆就卡在了开机界面不动了。

在出现问题的时候，因为我自己是很清楚是哪个disk出现问题了，所以估计开机卡在开机界面应该是开机的时候自动挂载了出问题的磁盘的原因。所以我们的思路就是开机不要挂载出问题的磁盘，然后我们把损坏的磁盘进行一个修复，再让系统开机挂载修复好的磁盘就可以了。

# 把出问题的disk取消开机挂载

由于我们现在连系统都进不去，我自己是直接掏出Live usb进入usb里面的系统（当然了你有其他方法进入tty或者修改grub启动参数进入系统，只要能修改到原本根目录下的文件`/etc/fstab`就可以）。

由于我是用Live usb的系统进入，所以此时的根目录并不是我们原本的根目录，这个时候就要手动将原本的根目录进行挂载，然后在终端输入`vim /xxxx/etc/fstab`，（具体方法可以看文章下面的**进入live USB修改原系统root密码**）将出问题的磁盘的挂载信息注释掉，例如下图中，我们需要在红色框内，UUID前加上`#`来将其注释掉。

![image.png](https://b3logfile.com/file/2021/08/image-46e584d9.png)

搞定之后保存，这个时候我们重启一下，就会发现电脑不再卡在开机界面的，但是会得到下面的提示：

![photo20210812133122.jpg](https://b3logfile.com/file/2021/08/photo_2021-08-12_13-31-22-48821942.jpg)

这个时候如果你的系统没有root密码，无论你按什么`Enter`, `Ctrl Alt F4`都无效，甚至按下电脑的开机键也不会关机，无奈只能将电脑强制断电了。

但是如果你可以进入`emergency mode`，那就直接跳转到**查看报错**吧

# 解决方法

可以看到上图中关键的信息是：`You are in emergency mode` 和 `Cannot open access to console, the root account is locked`。

我们先搜索第一个关键信息`emergency mode`，可以找到有很多人都有这种情况发生，但是大部分的人都是按下`Enter` or `Ctrl D`就可以跳过并且正常进入系统，和我们的情况不太相同，我们的情况是无论按什么都会重复出现这个提示信息。

搜索第一个信息无果，那我们搜第二个`Cannot open access to console, the root account is locked`. 几番查找后发现居然是因为没有root密码导致无法打开console，汗颜。这个时候我们只需要给root加上密码就行。

### 进入live USB修改原系统root密码

在我们安装deepin的时候做了一个类似于Windows的pe系统的安装工具在我们的U盘中，我们需要进入USB中的系统为我们原先的系统的root加上密码。首先电脑关机后插上U盘，开机后选择系统选项（这一点deepin做的真不错，不需要一直按F12或者其他键），在Boot处直接选择我们的U盘启动（跟重装系统是一样的）。

当界面到达安装界面的选择语言时，按下`Ctrl Alt F4`，输入`startx`后稍等一下就可以进入live usb中。进入系统后，输入`sudo fdisk -l`查看我们有的disk设备。

![image.png](https://b3logfile.com/file/2021/08/image-0e431565.png)

我们先将原先系统挂载在`/`上的disk, 在上图中我可以确定是`/dev/nvme0n1p3`，将其挂载到某个目录下（可以在~/下新建一个空目录）：`mount /dev/nvme0n1p3 ~/tmp`

> 这里如果无法确定哪个是我们要挂载的disk的话，deepin其实直接在文件管理器中双击打开分区就可以自动挂载到`/media/${user}/xxx`下，在文件管理器中也可以进入其中查看是否有`/bin`目录，如果有，那就是我们要的哪个disk了，直接在文件管理器中右键，打开终端即可。

然后我们需要在终端中切换到原系统中的root账户，将终端的当前目录切换到我们刚才挂载的目录`cd ~/tmp`中（如果是使用上述deepin的文件管理器右键进入终端的方法的话不需要输入上一条命令）

输入`sudo chroot .`

上面命令有个小数点不要漏掉，成功后我们再输入`passwd`进行密码修改。当我们修改完密码后其实就可以直接重启了，这个时候就可以正常进入到`emergency mode`了。

### 查看报错

当我们从`emergency mode`登录后，可以看到提示叫我们使用`journalctl -xb`查看启动日志。

![image.png](https://b3logfile.com/file/2021/08/image-74c064b2.png)

通过查看日志，在最后一页看到一个错误`Failed to start File System Check on /dev/disk/by-uuid/a2bfcdf2-ebe5-xxxxxxxxxx-xxxxx-xxxxxxxxx.`

> tips : 使用`f`可以翻页, `b`上一页，`G`到结尾。

可以看到，这里有个错误，检查某个disk的时候有错误，这个时候我们先拍照记下这个uuid，然后输入`cat /etc/fstab`查看对应的uuid是哪个。

![image.png](https://b3logfile.com/file/2021/08/image-46e584d9.png)

### 修复磁盘

我自己发生问题的时候对应的是上图红圈处对应的设备`/dev/sdb1`。我们运行`fsck -a /dev/sdb1`进行修复，稍微等待一下就可以修复成功了。

> 参数`-a`是让fsck发现错误后直接修复而不需要询问
>
> 我自己的磁盘是`/dev/sdb1`，但是对应到你自己的电脑中就要按照自己的来了。

修复完毕后千万记得，我们前面是修改过`/etc/fstab`的，所以要将其修改回来，把我们之前加上的注释去掉就可以重启了。

# 后记

在搜索过程中，发现很多出现这个问题的人都是对分区作出一些修改，修改了`/etc/fstab`文件导致的问题。但是我自己上一次对`fstab`文件修改已经是很多次开关机之前的事情了，导致问题的原因可能还是写入数据的后死后出现了某些问题导致整个file system出错。

同时中间遇到的root没有密码居然无法进入`emergency mode`也是让人非常汗颜，甚至最开始并不知道是这个原因，以为是磁盘损坏了，扫描了半天并没有发现磁盘损坏。

在使用deepin的live usb的时候，遇到一个bug，扫描磁盘的时候我出门，习惯性按下`win L`进行锁屏，然后我使用那个用户也没有密码，所以很悲催，我无法解锁，不输入任何内容直接点解锁也无法解锁，甚至关机也需要密码，按下笔记本的开机键关机也需要密码，无奈只能直接断电（但是我的磁盘还在扫描阿！！！淦）
