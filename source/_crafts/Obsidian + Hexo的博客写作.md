---
title: Obsidian + Hexo的博客写作
date: 2021-12-26 17:43
updated: 2021-12-26 17:44
tags: [tool,obsidian,hexo]
---
## Obsidian
最近Obsidian开启了所见即所得的feature, 在上手了短短几分钟后笔者就爱上了这款编辑器，一是因为颜值确实高（连图标都很好看），二是因为居然支持使用Vim. 当场就弃暗投明投身Obsidian. 

同时Obsidian有着看上去还不错的插件社区，所以想了一下打算用Obsidian来构建一个Obsidian + Hexo + Git的“写作IDE”，而要完成一体化的整合，那我们需要能做到下面几件事情：
1. 拥有一个写作体验良好的编辑器
2. 编辑器可以直接调用`hexo`命令，方便部署
3. 编辑器可以直接调用`git push`和`git pull`， 方便同步写作内容

能做到上面3件事情那么就算大功告成了。

## Obsidian的插件支持
目前看来Obsidian不太方便直接调用hexo，等有时间再整一整。