---
title: Elasticsearch 学习笔记
tags: note elasticsearch
---

笔者最近在项目中需要使用到Elasticsearch来进行搜索, 在此记下学习过程.

## Elasticsearch
官网上是这么描述Elasticsearch的: 
> Elasticsearch is the distributed search and analytics engine at the heart of the Elastic Stack.

Elastic Stack指的是ELK, 而ELK其实就是三种工具的首字母集合: ELK = Elasticsearch + Logstash&Beats + Kibana. 
1. Elasticsearch: 负责搜索
2. Logstash&Beats: 负责读取, 处理, 并将数据存储在Elasticsearch中
3. Kibana: 用于数据分析, 可视化等

官方说明:
> Logstash and Beats facilitate collecting, aggregating, and enriching your data and storing it in Elasticsearch. Kibana enables you to interactively explore, visualize, and share insights into your data and manage and monitor the stack. Elasticsearch is where the indexing, search, and analysis magic happens.

不过本文主要是专注在Elasticsearch, 所以不必过于在意ELK的另外两个成员的细节.

## 快速搜索原理
Elasticsearch使用倒排索引来快速索引文档, 倒排索引是将文档中的内容都拆分出来做索引, 所以搜索文本的时候可以根据索引快速定位到文档. 举一个官方文档的例子: [倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html#inverted-index)

很容易看出来的一点是, Elasticsearch创建的索引数量很多. 如果要将句子中所有单词都建立倒排索引的话, 会导致原本占用的磁盘空间多出至少一倍, 这是一种典型的空间换时间策略, 而当前大部分场景下, 用户的时间相比于存储占用会更加有价值. 

## 使用
### 简单安装
最简单的安装我觉得还是直接上`docker`, 运行: `docker run -dp 9200:9200 -p 9300:9300 elasticsearch:7.xx.x`等待安装完毕即可. (7.xx.x要改成自己想要的版本).
然后装完elasticsearch后我们最好再装一个Kibana, 用来跑Elasticsearch的查询语句非常方便: `docker run -dp 5601:5601 kibana:7.xx.x`. (同样, 要自己手动换版本, **版本要和Elasticsearch一致!!!**)

> 注: 给不熟悉docker的同学: 如果安装过程出错了, 可以输入`docker ps` 来查看目前还在运行的容器, 如果没有发现elasticsearch和kibana的话说明容器可能异常退出了. 那么这个时候可以输入`docker logs 容器id`来查看容器中的日志来进行排错( `docker ps -a`可以查看所有容器, 容器id前n位不重复的话只需要输入前n位就行了).

**关于版本的一些闲聊**
Elasticsearch 的版本迭代非常快, 如果没记错的话, 笔者在一个月前看到最新的还是7.14.x, 然后现在已经有7.16.2和8.0了(然而官方的中文文档还停留在2.X). 当然了, 版本迭代快也有可能会导致旧资料很快失效等问题, 例如_doc的type就在Elasticsearch7被移除, 所以在使用Elasticsearch的时候还是需要注意一下版本的问题.  

### 准备
打开浏览器, 输入`localhost:9200`, 如果看到如下json说明elasticsearch成功启动了.
```json
{
    "name": "97060d5fde2a",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "i_EW5L-NS4upXHSuLr14qA",
    "version": {
        "number": "7.10.1",
        "build_flavor": "default",
        "build_type": "docker",
        "build_hash": "1c34507e66d7dxxxxxxxx513706fdf548736aa",
        "build_date": "2020-12-05T01:00:33.671820Z",
        "build_snapshot": false,
        "lucene_version": "8.7.0",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```
我们可以用RESTFUL的方式来对上述地址进行请求来完成某些操作, 例如我们可以用curl来发送一个`PUT`请求来创建一个索引: `curl -XPUT "http://localhost:9200/nothing" -H 'Content-Type: application/json' -d'{  }'`. 这样我们就创建了一个名为`nothing`的索引.

> 这里名为`nothing`的索引其实更像是一个数据库, 如果不熟悉的话就直接将其当作数据库即可. 不要和倒排索引所指的索引混淆, 两者是不同的概念.

实际上我们有更好的方法来对Elasticsearch进行CURD, 那就是: Kibana. 如果你已经在本文的安装步骤中安装好了Kibana, 那么直接在浏览器访问`localhost:5601`即可. 成功进入Kibana的界面后, 点击一个叫做`Dev tools`的扳手图案, 或者直接访问`localhost:5601/app/dev_tools#/console`就可以进入Kibana的控制台, 在这个控制台我们就可以写查询语句来对Elasticsearch进行CRUD了. 

### 使用Kibana
走到这一步的时候已经可以不用管Elasticsearch了, 我们所有操作基本上都可以在Kibana上直接完成.


## 纯文本搜索

