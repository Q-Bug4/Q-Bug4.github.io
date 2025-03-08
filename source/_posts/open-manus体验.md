---
title: open_manus体验
date: 2025-03-08 14:51:22
tags: ai, manus
---

# Overview
这篇文章主要用于记录体验open_manus（称Manus）的情况和遇到的问题。

# 安装
安装非常简单，根据Readme一步一步来即可。

## LLM配置
按照Readme配置即可。但是模型选择官方是建议用openai或者Anthropic的Claude，但是这俩在国内都不提供支持，我们这里使用下面的LLM配置来体验。一开始我使用的llm是`qwen-turbo`，但是在Manus会无法确定选择什么工具，导致无法调用工具。

```toml
# Global LLM configuration
[llm]
model = "qwen-plus-latest"
base_url = "https://dashscope.aliyuncs.com/compatible-mode/v1"
api_key = "sk-xxxx"
max_tokens = 4096
temperature = 0.0

# Optional configuration for specific LLM models
[llm.vision]
model = "qwen2.5-vl-72b-instruct"
base_url = "https://dashscope.aliyuncs.com/compatible-mode/v1"
api_key = "sk-xxxx"
```

## 网络环境
经过体验，manus在执行过程中会调用Google进行搜索，或者访问一些境内无法访问的网站。如果是在当前Terminal中调用还比较好处理，只需要提前在Terminal中进行proxy的环境设置即可：
```bash
# Windows
set HTTP_PROXY=your_proxy_path
set HTTPS_PROXY=your_proxy_path

# Linux，这里偷懒直接用ALL了
export ALL_PROXY=your_proxy_path
```

但是由于manus还会调用浏览器来进行网页访问，网页内容抓取，所以还需要针对其使用的浏览器进行代理配置。下面有几种办法，我使用的是第二种
1. 当Manus调用浏览器后，给浏览器设置上代理：打开设置-搜索代理-配置代理
2. 如果你用了某只科学上网的猫，可以打开TUN模式，这样你电脑的全部流量都会走猫设置的代理
3. 利用路由器，软路由对接入的网络进行代理，如此你的电脑不用修改任何东西。

一切准备就绪，让我们开始体验Manus吧！

# 开始体验
首先运行main.py，可以看到Terminal中让你输入prompt，这个时候输入你要让Manus干的事情即可，例如：把bitcoin最近2周的波动情况输出一个文件给我。

然后你可能会发现怎么没动静，这个时候检查一下Terminal里是否有以下的提示，如果有，按提示运行`playwright install`，等待浏览器安装完毕，然后重新来一次即可。

## 例子：Manus分析Rust在LLM上的发展
整个过程可以查看[执行过程](RustLLM/terminal.txt)

这次我们让Manus帮我们分析Rust近期在大模型上的发展：

**一年来Rust在大模型上的生态和发展如何，给我一个.** 

这里文本描述因为输入法的问题被截断了，但是不影响Manus理解。

Manus第一次给出的结果[rust_large_model_ecosystem_report](RustLLM/rust_large_model_ecosystem_report.txt)是几段简单的英文描述，并且附带上一些文章的引用。这篇文档说了但是又好像没说，比较没有内容。于是，我们给Manus第二次指令：

**输出一篇5000字的报告，着重体现Rust现在在大模型上的生态，应用和发展**

得到的结果[rust_large_model_ecosystem_detailed_report](RustLLM/rust_large_model_ecosystem_detailed_report.txt)大概有4800字符，简单看了一下，这次的结果比上次好多了，将相关的生态也列了出来，同时也做了一些总结。但是他是英文的，还是不如中文的阅读速度来得快和舒服，于是我们下达了第三次指令

**改为中文，并且使用markdown语法，扩充内容**

得到的结果[rust_large_model_ecosystem_detailed_report_cn](rust_large_model_ecosystem_detailed_report_cn.txt)实际上和上一版是一样的，更像是翻译成中文了。
这个时候我想再了解一下关于模型推理加速的部分，我们给出最后一条指令：

**补充现在大模型推理加速如何使用Rust**

这个时候Manus理解错我的意思了，它重新生成了一个结果[rust_large_model_inference_acceleration](RustLLM/rust_large_model_inference_acceleration.txt)来单独说明Rust如何在模型推理加速中应用。但是其实我的本意是：在原来的文件上处理。这里可能在prompt上还是说清楚如何输出结果。

## 失败案例1：比特币最近波动情况（agent工具调用设计问题）
[执行过程](API调用超时崩溃.txt)

一开始对比特币价格进行搜索还是比较顺利的，后续在页面上无法获取到比特币的价格，于是Manus改为调用免费的API来查询。这个时候，第一次调用API的时候成功了，但是随后的API调用全部超时，每3分钟重试一次，重试N次后失败了。至此整个流程中断。

## 失败案例2：小红书最火话题（幻觉问题）
博主让Manus找到小红书最近最火的话题，一开始Manus直接搜索，从别人编写的文章中获取了答案。这个时候还属于正常范围，接下来博主想让Manus直接去小红书官方获取，毕竟一手资料真实性高一些。结果Manus装模做样在浏览器看了一眼小红书官网，直接给我胡诌出3篇笔记，甚至演都不演了，笔记id分别是:`123`, `456`, `789`

[执行过程](胡诌小红书最火笔记.txt)

输出结果：[xiaohongshu_top_notes.txt](xiaohongshu_top_notes.txt)
