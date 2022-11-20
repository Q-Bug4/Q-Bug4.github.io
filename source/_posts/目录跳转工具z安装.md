---
title: 目录跳转工具z安装
date: '2022-11-20 18:09:46'
tags: ["CLI"]
---

## 简介
[z](https://github.com/rupa/z)是一个在命令行中非常好用的工具，他的作用也非常简单，就是让你可以用简短的文本快速跳转到你访问过的目录。例如我在目录`/home/qbug/proj/work/java/backend/myWork`中工作，这个时候我需要切换到`/usr/local/share`目录下做其他事情，等到我想要跳转回第一个目录时，只需要输入`z myWork`甚至`z my`即可。

## 安装
`z`的Readme界面如果没有细看的话就很容易遗漏掉安装方法，同时官方提供的安装方法对于新手来说也是太过简单了，因此我在这里提供一个相对比较详细的安装方法。

1. 克隆仓库 `git clone https://github.com/rupa/z && cd z`
2. 将克隆下来的`z.sh`文件放入`/usr/local/bin`目录下: `sudo mv z.sh /usr/local/bin/z`（或者也可以放在其他目录中，只要在`$PATH`中即可）
3. 将说明书移入说明书目录：`sudo mv z.1 /usr/local/share/man/man1`
4. 在你使用的shell配置文件（例如`zsh.rc`, `bash.rc`）中添加`. /usr/local/bin/z`. （如果你在第2步的时候修改了z的位置，那么这里也要相应修改）

大功告成了，现在`z`会记住你访问过的目录，只要是你访问过的目录都可以用`z`进行快速跳转了！
