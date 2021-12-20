title: fcitx-rime中州韻的安装
date: '2021-07-05 23:09:22'
updated: '2021-07-05 23:09:22'
tags: [fcitx-rime]
permalink: /articles/2021/07/05/1625497762712.html
---
![](https://b3logfile.com/bing/20200205.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

# 前记

在被deepin（deepin 20.2）自带的输入法折磨了2天，游荡于各个社区查看中文输入法的推荐，在ibus, fcitx这俩输入框架中选了fcitx，同时也选了一个在mac上叫做**鼠鬚管**， 在Windows上叫做**小狼毫**的输入法：**rime**. (Linux上叫做**中州韻** ~，我都佛了，怎么这么多名字啊~)。

# 开整

首先我的操作基本都是来自[安装中州韵 fcitx-rime 输入法](https://wiki.deepinos.org.cn/deepin%E6%8A%98%E8%85%BE%E7%AC%94%E8%AE%B0/%E7%AC%AC%E5%85%AD%E7%AB%A0/6.4.html)这篇文章。但是在用的过程还是出现了一些问题，因此在此把过程都记录下来。

### 卸载原本的fcitx & rime

之所以要干这一步，主要还是怕原来的fcitx和rime给你整出点幺蛾子，如果有搜狗输入法或者其他输入法的话，我的建议也是都删了把。

打开terminal（快捷键Ctrl Alt T），输入

`sudo apt purge fcitx* *rime*`
`sudo apt autoremove --purge`

操作完成后，其实电脑里的fcitx还是在跑着的，于是我们要把fcitx给`kill`掉, 输入

`sudo killall fcitx` （此处参考文章没有加上sudo，实测是需要sudo权限的）

然后查看一下是否还有漏网之鱼：`pgrep fcitx`，若出现结果（假设结果是1234，有多个就多kill几次），那就再执行：`kill 1234`

最后一步我们需要删除fcitx的配置文件，如果是新手的话建议用文件管理器进入该目录后将`~/.config/fcitx`删除。

知道`rm`的就直接 ： `rm -r ~/.config/fcitx`即可.

### 安装fcitx-rime

`sudo apt install fcitx-rime`

等待安装即可。

### 启动

此处按参考文章执行`fcitx-autostart`会出现Error以及一个要求配置`XMODIFIERS`的Warn. 并且我第一次尝试在此以失败告终， 在经历了2次重装后，我直接 : 托盘的输入法->配置->添加中州韵。

**！！！** 此处有巨坑。

在第1次重装的时候其实我就是直接在托盘处添加输入法的， 但是**奈何打不出中文**。在第2次重装的时候，将所有键盘都删除，然后添加了一个“键盘-汉语”以及“中州韵”。在操作都跟第一次重装一样的前提下，他成功了！

**后来猜想可能是我最开始的键盘是“键盘-英文”所导致的这个问题。** Whatever, 毕竟已经可以打字了，也就不管了。

### 自定义输入法

引用上面提到的文章的文段：

> 下载配置、词库和皮肤：[https://www.github.com/loaden/rime](https://www.github.com/loaden/rime)
> 覆盖到 ~/.config/fcitx/rime 目录下，托盘输入法图标右键菜单“重新部署”。
> 将 skin 目录移动到 ~/.config/fcitx 目录下可实现自定义皮肤。

大白话 ： 将上述的仓库clone下来（或者下载Zip包），将其中所有内容copy到`~/.config/fcitx/rime`中，配置文件为：`default.custom.yaml`，打开后就可以直接修改配置。修改完如下图重新部署即可。

![image.png](https://b3logfile.com/file/2021/07/image-198b42df.png)

此时你可能会发现，直接在输入法UI界面中无法直接设置，确实如此，我也不知道为什么。不过直接修改配置文件就可以满足需求了。

**给不知道怎么修改默认输入模式（五笔/拼音）的同学：**

打开配置文件后，找到`schema_list`，按照其他schema的格式，添加一个`schema: pinyin_simp`到第一个位置，保存即可。

# 尾声

至此，基本的安装设置已经做完了。但是还有词库，皮肤等内容还没开整， 等抽空有时间再开整。
