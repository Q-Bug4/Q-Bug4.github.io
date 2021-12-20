title: Vim 映射Esc位置的方案
date: '2021-07-06 22:50:47'
updated: '2021-08-21 13:33:47'
tags: [Vim, xmodmap, keyboard]
permalink: /articles/2021/07/06/1625583047644.html
---
![](https://b3logfile.com/bing/20200212.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

由于在用Vim的时候需要频繁去按Esc，为了我们的手指健康以及效率提高，于是就有很多映射Esc的方案出现了。

### jj

使用`jj`来代替`Esc`是配置起来最容易的方案，只需要在`~/.vimrc`中输入`imap jj <Esc>`，退出Vim再进入就可以轻敲2次`j`键来代替`Esc`了。

但是这里可能就会有一个疑问了，**在Vim的command mode下会不会出现快速按下j被识别为Esc导致不会移动光标呢？**

答案是**不会**，此处引用一下[fandom](https://vim.fandom.com/wiki/Avoid_the_escape_key)的说明。

> The `:imap` command is used to create the mapping so that it **only** applies while in insert mode

但是会导致一个新问题 ： **在insert mode下输入`jj`会直接`Esc`**

### 使用`Ctrl [`代替Esc

此处引用一下[fandom](https://vim.fandom.com/wiki/Avoid_the_escape_key)的说明。

> If you have an American English keyboard, pressing **Ctrl-[** (control plus left square bracket) is equivalent to pressing Esc. This provides an easy way to exit from insert mode.

简单来说，你只需要在Vim中按下两个键就可以实现Esc的功能：`Ctrl` + `[`。

### CapsLock大小写锁定键

`CapsLock`这个键占据了一个非常有利的位置，只需要伸出左手小拇指就可以按到，我自己也是用`Caps`当作我的游戏麦克风开关快捷键的。

然而这个键说实话基本上没什么用，唯一功能就是切换大小写，然而切换大小写我们可以按住`Shift`键来实现，只要你习惯了用`Shift`切换大小写输入，那么这个`Caps`键就显得有些食之有肉，弃之有味了。

将`Caps`这个占了高效位置的键交给其他高频使用的按键来使用，说起来也算是很符合计算机思想了，同时网上也有很多人推荐将`CapsLock`映射到`Esc`。

# 使用系统的键盘映射工具修改 - 推荐

由于下面两个方法`xmodmap` & `setxkbmap`都是启动后就会失效，所以我Google了一下，发现了这么一句话：**Isn't there a “keyboard settings” GUI in ubuntu?**. 这倒是提醒了我，于是直接在Google搜“deepin 键盘映射”，还真让我找到了这个[修改键盘映射 for deepin](https://wiki.deepin.org/index.php?title=%E4%BF%AE%E6%94%B9%E9%94%AE%E7%9B%98%E6%98%A0%E5%B0%84&language=zh), 那对于其他系统，我们直接如法炮制按照系统来搜索修改键盘映射的方法即可。

经测试，重启登录后，我们的修改依然有效，终于是解决了这个问题。

# xmodmap - 不推荐

一开始是打算直接修改`vimrc`来完成键位映射，结果发现vim中似乎无法映射`Caps`键，那没办法， 只能请出我们的`xmodmap`来修改用户全局的键盘映射了。

首先，**查看要更换的两个键的keycode**。

在terminal中输入`xev`， 按下`Caps`键，可以看到输出了如下信息

```bash
KeyPress event, serial 38, synthetic NO, window 0x8400001,
    root 0x205, subw 0x0, time 25407643, (895,586), root:(895,616),
    state 0x12, keycode 66 (keysym 0xff1b, Escape), same_screen YES,
    XLookupString gives 1 bytes: (1b) "
    XmbLookupString gives 1 bytes: (1b) "
    XFilterEvent returns: False
```

重点信息在于按下的`Caps`键的keycode，记录下来。同理，记录下`Esc`的keycode.

接下来只需要执行`xmodmap -e 'clear Lock' -e 'keycode 66 = Escape' -e 'keycode 9 = Caps_Lock'`即可。

ps : 可以使用`xmodmap -pke`来查看所有"keycode = key"的信息。

> 如果使用`xmodmap`来改变映射，在看Youtube视频的时候还是需要按下键盘上的`Esc`键才能退出。

# setxkbmap - 不是很推荐

使用`xmodmap`改变映射会出现一个问题**有点繁琐，而且重启就没了**.

而虽然我们也可以用`setxkbmap -option caps:swapescape`来交换两者（`caps` & `escape`），但是一样的，重启就没了。

在终端中输入`setxkbmap -option caps:swapescape`即可交换(只能在本次连接内生效)，我试过将该命令放在`~/.profile`，`crontab`，`~/.zshrc`中，没有一个生效的，每次重启还是需要手动进行修改。

### 我的推荐

实际上，我更加推荐使用`Caps`作为`Esc`的方法，同时也推荐使用系统的键盘映射工具进行修改，一劳永逸，即使重启修改也依然存在。
