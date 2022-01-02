---
title: 'lutris 安装记录以及报错处理： TypeError: ''type'' object is not subscriptable'
date: 2021-12-05 20:45:41
tags:
---

# 前记
众所周知，Linux也是可以玩一些原本只能在Windows上玩的游戏的，通过wine可以让我们柔顺地游玩守望先锋，使用微信。不过毕竟Linux要打开Windows上的东西需要多一个步骤， 所以安装过程可能不会一帆风顺。

# 环境
- 系统： `5.13.19-2-MANJARO`
- 显卡： Nvidia 1050

# 问题与解决
我们在按照lutris的[安装指引](https://github.com/lutris/docs)将lutris和wine等依赖安装好之后，直接启动lutris。

这里就可能遇见第一个问题：网络问题。这里我自己由于环境变量`$http_proxy`没有加上HTTP的前缀，导致代理失效出现网络问题，而lutrix的GUI界面并没有提示网络有问题而只是一直卡在更新的步骤。这里建议从terminal启动lutrix, 这样你就可以通过lutrix的输出内容来排错.

当成功进入lutrix后我们搜索`battle.net`, 按照指引安装后出现错误：`TypeError: 'type' object is not subscriptable`. 

这是python的错误，在网上冲浪搜索了一番后发现有不少人出现这个问题，个人的解决方法都不一样，就我搜索到的解决方法有如下几种：
1. 重新安装python
2. 和Anaconda冲突，让lutrix使用本机其他python (我本机也有Anaconda, 但是lutrix用的python却不是Anaconda的python版本)
3. 重启电脑 (自己试出来的，有效)

最开始出现错误的时候我还以为是python版本太高的原因(3.9)，后来在lutrix官网看到有人建议将python更新到3.9，加上后来解决问题的时候3.9确实可以正常安装运行lutrix才确定不是版本问题。