---
title: Hexo Github Action配置
date: 2022-03-04 15:00:27
tags: [Hexo, Github Action]
---

## 概括
 在本文中我们将会实现下列目标：
1. 编写完博客后将博客 push 上 Github，由 Github Action 自动帮我们构建，并且部署到 Github page 上
2. 大概聊一下整个过程的简单原理

## 原理
Github Action 相当于在我们 push 代码或是执行其他动作的时候自动执行一些操作，Github 给我们开一个机器来执行一些我们编写的操作（类似与 CI ），所以我们可以通过 Github Action 就实现自动部署 Hexo. 

所以我们需要做的事情其实和我们为一台新电脑配置 git 的权限，然后用这台新电脑 `git clone` `npm instal` `hexo g` `hexo d`一样，只不过我们需要将这些操作写成一个yml 文件，让 Github Action 在我们每次 push 的时候自动帮我们部署到 Github Page 上而已。所以我们需要
1. 配置 git: 生成 ssh 密钥，私钥交给 Github Action 的机器，公钥放在仓库的`deploy keys`中。
2. 按照 Github Action 文档来编写 workflow 文件
   1. 拉取代码
   2. 设置 ssh 私钥
   3. 安装node依赖
   4. npm install
   5. hexo g
   6. hexo d

这里有一个问题，有的人的博客原文和 hexo 仓库是不同的仓库，有的人是同一个仓库不同分支，这个其实问题不大。因为不管我们最终要将静态页面部署到哪里是 hexo 的 _config.yml 来负责的，我们只需要对存放博客原文的仓库进行 Github Action 的设置就可以了。

## ssh密钥生成与配置
1. 运行`ssh-keygen -f github-action-hexo -C hexo`
> 在`ssh-keygen`的说明书里可以看到：`ssh-keygen -c [-a rounds] [-C comment] [-f keyfile] [-P passphrase]`. 即-C参数只是用来作为一个注释，写什么都可以。而-f是输出的文件名，ssh-keygen会在当前目录下生成2个文件，一个是私钥一个是公钥(.pub). 
2. 我们需要将生成的私钥和公钥放到对应的Github仓库中，其中公钥需要放到仓库的`Deploy keys`中，而私钥是用来设置在Github Action给我们的机器中的。具体操作：
    1. 添加公钥：首先打开我们的hexo仓库，也就是\<username\>.github.io，然后打开仓库的Settings，点击左侧的`Deploy keys`，点击`Add deploy key`，然后将我们在第一步中生成的`github-action-hexo`文件中的内容全部复制进来，这里给这个key取什么名字都没什么所谓。**注意，一定要勾选`Allow write access`**
    2. 添加私钥：打开hexo仓库的设置，在左侧点击Secrets，再点击Actions。点击`New repository secret`，将`github-action-hexo.pub`中的内容全部复制进来，同时给这个secret取名为`HEXO_DEPLOY_PRI`. 
    > 添加私钥这一步其实相当于是添加了一个私密的全局变量，别人是看不到这个变量的值的，也就保证了仓库的安全。这个变量会在后面初始化git ssh key的时候用到，所以如果你取了别的名字的话就需要在后续的步骤中也换成同样的名字。

## 编写workflow
如果懒得查文档的话，按照下面的yml修改一下即可：
```yaml
name: Hexo deployment # 这个是在Github Action上显示的名字

on:
  push:
    branches:
    - source # 当哪个分支发生 push 动作时执行jobs, 在本文中也就是博客原文所在的分支

jobs:
  build:
    runs-on: ubuntu-latest # 选择机器的系统和版本
    strategy:
      matrix:
        node-version: [14.x] # 选择node版本

    steps:
      - uses: actions/checkout@v2 # 相当于拉取源代码

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@main
        with:
          node-version: ${{ matrix.node-version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}} # 这里的secrets.HEXO_DEPLOY_PRI如果在上文中有修改的话也要记得修改
        run: |
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name "<username>" # 你的用户名
          git config --global user.email "<email>" # 你的邮箱
      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i
      - name: Deploy hexo
        run: |
          hexo clean && hexo generate && hexo deploy
```

## 参考
[hexo配合github action 自动构建（多种形式） - 掘金](https://segmentfault.com/a/1190000040767893)
[Github Action 文档](https://docs.github.com/cn/actions)