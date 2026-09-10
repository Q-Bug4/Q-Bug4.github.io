---
title: 如何自动化评估你自己的 Agent
date: 2026-09-09 19:30:00
tags: [AI生成, Agent, 评估, 教程]
permalink: /articles/2026/09/09/1757416200000.html
published: false
---

> 📝 本文由 AI agent 起草,尚未经过人工审核。发布前请由 Kael 本人审阅修改。
> 撰写 agent:Hermes · 起草日期:2026-09-09

# 如何自动化评估你自己的 Agent

面向场景:你有一个能调工具、多轮执行的 agent(编码、运维、检索问答都行),
想知道它到底行不行、改动有没有让它变差——并且这个过程要自动化,不能靠人肉点测。

## 0. 一个反直觉的前提:先建评测集,再选框架

大多数人上来纠结"用哪个框架",顺序错了。评估的资产是**评测集**,框架只是跑
评测集的引擎,随时可换。评测集从哪来:

1. 翻你过去一个月真实用 agent 干的活,挑 20-30 个典型任务
2. 每个任务记三样东西:输入(用户话术)、环境(涉及哪些文件/服务)、**判定标准**
3. 判定标准优先级:**可执行断言 > 关键事实包含 > LLM-judge rubric > 人工**
   (能写测试就不写 rubric,能写 rubric 就不靠人眼)
4. 放进 git 仓库,像对待代码一样对待它——评测集本身也要 review

## 1. 按 agent 的形态选评估策略

你的 agent 属于哪类,决定测什么:

| Agent 形态 | 正确性的单位 | 评估策略 |
|---|---|---|
| 单轮问答/摘要 | 单个输出 | 指标打分:相关性、幻觉率、格式合规 |
| RAG(检索+生成) | 一次检索+生成 | RAGAS 三指标:faithfulness、context precision/recall |
| 工具调用 agent | 一次调用的参数对不对 | 工具名+参数断言(参数级判分) |
| 多步任务 agent(编码/运维) | 任务终态(测试过没过、服务起没起) | 沙箱跑真任务,结果断言 |

大多数实用 agent 是最后一种,下面的主线按它展开;前三类是它的子问题,
工具和框架全部复用。

## 2. 主线方案:沙箱 + 结果断言 + 轨迹留痕

这是 2026 年 agent 评估的事实标准形态(terminal-bench、Inspect AI、SWE-bench
全是这个形状),自己搭只需三个组件:

```
┌─────────────┐    ┌──────────────────┐    ┌─────────────┐
│  评测集       │ →  │  Runner:起沙箱、  │ →  │  Scorer:    │
│ (tasks/*.yaml)│    │  跑agent、录轨迹   │    │  结果断言判分 │
└─────────────┘    └──────────────────┘    └─────────────┘
```

### 组件一:任务格式(评测集的落地形式)

每个任务一个目录,借鉴 terminal-bench 的约定:

```
tasks/
  fix-cache-penetration/
    task.yaml        # 任务描述、初始环境、超时
    environment/     # Dockerfile + 带bug的初始代码
    solution.sh      # 参考答案(可选,用来验证任务本身可解)
    tests/           # 判分测试,agent 不知道内容
```

`task.yaml` 核心:任务 prompt + 判定命令(`pytest tests/ -q` 之类)。
**agent 永远看不到 tests 目录**——判分依据对它是黑盒,防它"应试"。

### 组件二:Runner(两个现成选择)

**选择 A:Inspect AI**(推荐,通用性最好)

```python
from inspect_ai import task, Task, eval
from inspect_ai.dataset import Sample
from inspect_ai.solver import basic_agent
from inspect_ai.tools import bash, edit

@task
def fix_cache_bug():
    return Task(
        dataset=[Sample(
            input="修复 src/cache.py 的缓存穿透问题,让 pytest 全绿",
            files={"src/cache.py": "environment/src/cache.py"},  # 初始环境
        )],
        solver=[use_tools(bash(timeout=120), edit()), basic_agent(max_steps=30)],
        scorer=includes_checker(),   # 见组件三
        sandbox="docker",            # 每个任务一个干净容器
    )
```

```bash
pip install inspect-ai
inspect eval eval_fixbug.py \
  --model-host vllm/http://your-endpoint:8000/v1 \
  --trace
```

**选择 B:terminal-bench**(如果你的 agent 就是编码/终端型)

```bash
tb run --dataset terminal-bench-2.0 --agent terminus \
  --model openai/<你的endpoint>       # 公开任务,对表社区
tb run --dataset ./my-tasks --agent <你的agent适配器>   # 自建任务
```

判分脚本自带,还挂了 SWE-bench Verified 等数据集的 registry,一套 agent 适配器
两边跑。**建议公开任务用 tb 对表、自建任务用 Inspect**,agent 适配器写一份两边用。

### 组件三:Scorer(判分哲学:结果断言,不用 LLM 打分)

```python
# 好:可执行断言——确定、可复现、零 judge 成本
scorer=includes_checker()                     # 关键文件包含关键改动
# 或自定义:跑容器里的测试套件
def pytest_green():
    async def score(state):
        r = await statesandbox_exec("pytest tests/ -q")
        return Score(value=1 if "passed" in r.stdout else 0)
    return score

# 差:LLM 给文本打分——慢、贵、有偏,只留给无法断言的开放任务
```

优先级铁律:**测试通过 > 文件状态 > 接口返回 > LLM-judge > 人工**。
LLM-judge 只处理最后 10% 无法断言的开放任务,且 judge 模型固定、定期人工抽检 10%。

### 组件四:轨迹留痕(比分数更重要)

`--trace` 把 agent 每一步(思考、工具调用、参数、输出)落成 JSON。分数告诉你
"行不行",轨迹告诉你"为什么不行"。每个失败 case 的轨迹归档进评测仓库,是
改进 prompt/工具描述的第一手材料,也是评测集持续扩容的来源。

## 3. 质量层:对输出文本的指标(单轮/RAG 场景)

多步任务的判分靠结果断言,但输出文本质量还要不要测?按需:

- **RAG 管线** → RAGAS:faithfulness(答案是否忠于检索内容)、context
  precision/recall(检索好坏),指标有论文定义,`evaluate(dataset, metrics=[...])` 一行
- **通用输出**(相关性/幻觉/ Toxicity)→ DeepEval:`assert_test()` 嵌 pytest,
  阈值不过即 CI 失败;G-Eval 可以用中文写你们自己的业务 rubric
- **多模型/多 prompt 对比** → Promptfoo:YAML 声明矩阵,`promptfoo eval` 出
  side-by-side 表;红队扫描(`redteam run`)上线前跑一遍注入/越狱

注意共性坑:凡 LLM-judge 参与的指标都有偏差(偏爱长回答、偏爱特定风格),
换 judge 模型后评测集要整包重跑校准。

## 4. 工程化:让它真正"自动"

评估不自动化 = 不会有人跑。三件事:

1. **CI 挂钩**:评测集测试写进 pytest(DeepEval 天然支持),PR 级改动跑快速子集
   (5 条,分钟级),主分支合并跑全量。评估和单测一样成为门禁
2. **防回归**:**任何 agent prompt/工具描述/模型版本的改动,必须附带评测集全量
   结果**——这是 eval-driven development 的核心纪律。分数跌了但改动必须上?
   在 PR 里写明取舍,留痕
3. **统计严谨**:agent 是非确定性的,同一任务跑 3 次取 pass@1(pass@1 = 三次里
   至少一次过的比例按任务平均,或直接报告中位数),单次结果不做任何结论;
   评测集 <30 条时,±1 题的波动没有意义,别过度解读

## 5. 冷启动路线图(两周内可完成)

```
第 1-3 天   从历史真实使用攒 20 个任务,写成 tasks/ 目录格式
第 4-5 天   Inspect AI 跑通 1 个任务端到端(沙箱+agent+scorer+trace)
第 6-8 天   补齐 20 个任务的判分(优先可执行断言),建立基线分数
第 9-10 天  CI 挂钩(快速子集),把基线分数写进 README
之后        每次翻车 case 入库;每两周看一次分数趋势
```

不需要一次上全部框架。Inspect(跑 agent)+ DeepEval(可选,文本指标)两个就够,
RAGAS 等真做 RAG 再加,Promptfoo 等真要对比多模型再加。

## 参考链接

- Inspect AI: inspect.aisi.org.uk(英国 AISI 出品,任务=solver+scorer+sandbox)
- terminal-bench: tbench.ai(任务格式约定 + registry)
- DeepEval: github.com/confident-ai/deepeval | RAGAS: docs.ragas.io | Promptfoo: promptfoo.dev
- 框架对比长文: agentbrisk.com/blog/agent-evaluation-frameworks-comparison
