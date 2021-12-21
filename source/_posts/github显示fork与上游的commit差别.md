---
title: github显示fork与上游的commit差别
date: 2021-12-21 10:52:35
tags: 杂症
---
今天在用[Elasticsearch-head拓展](https://github.com/TravisTX/elasticsearch-head-chrome)的时候发现在点击文档的时候会出现多层重叠的问题, 所以想看一下有没有修复这个问题, 然而仓库上一次commit的时间是2年前, 所以想看看有没有其他人fork后在继续维护, 但是点开fork列表后发现Github并没有显示哪些fork是领先上游, 哪些是落后的.

搜索了一下发现以及有人提出过这个问题: [How to determine which forks on GitHub are ahead?  StackOverflow](https://stackoverflow.com/questions/54868988/how-to-determine-which-forks-on-github-are-ahead)

其中有一个老哥写了一段js的代码, 非常好用.
```javascript
(async () => {
  /* while on the forks page, collect all the hrefs and pop off the first one (original repo) */
  const aTags = [...document.querySelectorAll('div.repo a:last-of-type')].slice(1);

  for (const aTag of aTags) {
    /* fetch the forked repo as html, search for the "This branch is [n commits ahead,] [m commits behind]", print it directly onto the web page */
    await fetch(aTag.href)
      .then(x => x.text())
      .then(html => aTag.outerHTML += `${html.match(/This branch is.*/).pop().replace('This branch is', '').replace(/([0-9]+ commits? ahead)/, '<font color="#0c0">$1</font>').replace(/([0-9]+ commits? behind)/, '<font color="red">$1</font>')}`)
      .catch(console.error);
  }
})();
```

甚至还有个老哥用go写了个项目, 似乎是在本地CLI上直接跑的(没用上就没细看): [forkizard
](https://github.com/hbbio/forkizard), 作者在Readme中展现的效果如下:
```bash
$ ./forkizard k0kubun/pp
2019/06/20 16:27:35 44 forks
 44 / 44 [===============================================================================================================================] 100.00% 32s
done
/wtertius/pp       +6 -3
/hbbio/pp          +5 -3
/quanticko/pp      +2 -12
/federico-bollo/pp +1 -12
/juntaki/pp        +1 -12
/sniperkit/pp      +1 -12
/yudai/pp          +4 -39
/wangkechun/pp     +2 -57
```