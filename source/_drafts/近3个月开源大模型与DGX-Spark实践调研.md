---
title: 近 3 个月开源大模型发布与 DGX Spark 实践调研
date: 2026-09-09 20:00:00
tags: [AI生成, DGX-Spark, 调研, 开源模型]
permalink: /articles/2026/09/09/1757416800000.html
published: false
---

> 📝 本文由 AI agent 起草,尚未经过人工审核。发布前请由 Kael 本人审阅修改。
> 撰写 agent:Hermes · 起草日期:2026-09-09

# 近 3 个月开源大模型发布与 DGX Spark 实践调研

覆盖范围:2026 年 6 月-9 月上旬发布的开源/开放权重模型,逐个核对 DGX Spark(GB10)
社区实践情况。所有地址均实测可访问(HF 模型页 + Spark 实践帖)。

## 一、总览表

| 模型 | 发布 | 规格 | 许可 | Spark 实践 | 单机可行性 |
|---|---|---|---|---|---|
| DeepSeek-V4-Flash-0731 | 7/31 | 284B/13B | MIT | ✅ 大量(最热) | ✅ 单机/双机 |
| GLM-5.3-Flash | 8/26 | 320B/18B | MIT | ✅ 双机方案 | ⚠️ 双机起 |
| GLM-5.2(含 Turbo) | 6/13, 8/17 | 753B/40B | MIT | ✅ 3-4 机 | ❌ 集群 |
| Qwen3.8-27B | 8/14 | 27.8B dense | Apache 2.0 | ✅ 大量 | ✅ 单机 |
| Qwen3.8-Flash-Next | 8/26 | 176B/6B | 开源 | ✅ 大量 | ✅ 单机 |
| MiniMax-M3 | 6/1 | 428B/~23B | Community | ✅ 2-4 机 | ⚠️ 双机起 |
| MiniMax H3 | 8/3 | 465B/30B 视频生成 | Community(区域限制) | ⚠️ 未见 Spark 帖 | ❌ |
| Kimi K3 | 7/27 权重 | 2.8T/104B | Kimi K3 License | ⚠️ 仅"塞进去"实验 | ❌ 16 台起 |
| Inkling / Inkling-Small | 7/15, 7/30 | 975B/41B, 276B/12B | Apache 2.0 | ✅ 双机 | Small ✅ 双机 |
| Nemotron 3.5 Lightning | 8/11 | 30B/3B | OpenMDW-1.1 | ✅ 官方 recipe | ✅ 单机 |
| Muse Glimmer 30B | 8/9 | 30B | Apache 2.0 | ⚠️ 未见 Spark 帖 | ✅ 理论可行 |
| Tencent Hy4 preview | 8/28 | 770B/49B | Apache 2.0 | ⚠️ 未见 Spark 帖 | ❌ 4+ 台 |
| Ling 3.0 系列 | 7-8 月 | 124B/5.1B 等 | 开源 | ⚠️ 未见 Spark 帖 | ✅ 部分单机 |
| KAT-Coder-V2.5-Dev | 7/23 | 35B/3B | Apache 2.0 | ⚠️ 未见 Spark 帖 | ✅ 单机 |

## 二、有 Spark 实践的模型(重点)

### 1. DeepSeek-V4-Flash-0731 —— Spark 社区最热的模型

- 权重:https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731
- 实践帖(按热度):
  - 单机 52 tok/s(prefill 1058 tok/s),全 256 专家,75 回复:
    https://forums.developer.nvidia.com/t/c1-1058pp-s-52-tg-s-on-1x-dgx-spark-on-deepseek-v4-flash-0731-full-256-experts/381377
  - 双机官方 FP8 TP=2 + MTP + 200K ctx(261 回复,2.9 万浏览):
    https://forums.developer.nvidia.com/t/deepseek-v4-flash-official-fp8-running-across-2x-dgx-spark-tp-2-mtp-200k-ctx-recipe-numbers/370309
  - Aiden recipe 双机 1M ctx(740 回复,4.2 万浏览,论坛最热):
    https://forums.developer.nvidia.com/t/deekseek-v4-flash-aiden-recipe-from-reddit-1m-token-session-operational-cuda-12-1-tailored-for-dgx-spark-gb10/372268
    (镜像 aidendle94/sparkrun-vllm-ds4-gb10:production-ready,C1 42-45 tok/s)
  - 复现指南(GitHub):https://github.com/GanyX19/deepseek-v4-1m-on-dgx-spark
    (双机 1M ctx ~37 tok/s;256K ~40 tok/s/150 聚合)
  - 一键安装(单机 DSpark 调优,35 tok/s):
    https://github.com/entrpi/ds4-on-spark
  - 4 节点集群(8/23):decode ~56 tok/s,prefill 100K @ 2.7K tok/s
- 结论:1-4 机全有方案,单机 52 tok/s 过 40 门槛,双机 82 tok/s(noze.it 8 月实测汇总)

### 2. Qwen3.8-27B(8/14, Apache 2.0,dense 27.8B)

- 权重:https://huggingface.co/Qwen/Qwen3.8-27B
- 实践帖:
  - 最快配置汇总(GitHub,SGLang+NVFP4+DFlash2,单机 50 tok/s 贪心中位数,8 流 148):
    https://github.com/hasso5703/dgx-spark-qwen38
  - 论坛"SGLang on GB10 完整教程"帖(4x:125 / 2x:85 / 1x:75 tok/s 代码持续):
    https://forums.developer.nvidia.com/t/velogb10-v0-4-0-release-qwen-3-8-27b-code-sustained-tok-s-4x-125-tok-s-2x-85-tok-s-1x-75tok-s/381027
- 结论:单机 50-75 tok/s,dense 里唯一过 40 的(靠投机解码);原生多模态+262K ctx

### 3. Qwen3.8-Flash-Next(8/26,Qwen4 架构预览)

- 权重:https://huggingface.co/Qwen/Qwen3.8-Flash-Next
- 实践帖:
  - 单机编码 ~43 tok/s:https://forums.developer.nvidia.com/t/single-dgx-spark-qwen-3-8-flash-next-at-43tok-sec-in-coding/381189
  - HashK-PLE NVFP4 单机:https://forums.developer.nvidia.com/t/qwen3-8-flash-next-180b-single-solo-dgx-spark-with-hashk-ple-nvfp4/381436
  - NVIDIA 官方 NVFP4 多机(64 tok/s 峰值):https://forums.developer.nvidia.com/t/qwen3-8-flash-next-on-1-2-and-4-dgx-sparks-with-nvidias-official-nvfp4-quant-64-tok-s-peak-single-stream/381470
  - NVFP4 vs GLM-5.3-Flash 对比帖:https://forums.developer.nvidia.com/t/qwen3-8-flash-next-nvfp4-vs-glm5-3-flash-nvfp4/381497
- 结论:单机 34-47 tok/s(结构化输出更高),当前论坛讨论量最大的模型

### 4. GLM-5.3-Flash(8/26, MIT, 320B/18B)

- 权重:https://huggingface.co/zai-org/GLM-5.3-Flash
- 实践帖:
  - 双机 EXL3 一键安装(33-74 tok/s, 1.3M ctx, 带视觉):
    https://github.com/Entrpi/glm-5.3-flash-exl3-2x-spark
  - 双机 NVFP4 TP=2 + MTP + 262K ctx(7 个 day-0 bug 修复记录):
    https://forums.developer.nvidia.com/t/glm-5-3-flash-nvfp4-on-2x-nvidia-dgx-spark-vllm-tp2-262k-context-mtp/(见 NVFP4 对比帖内链接)
  - 主讨论帖(335+ 回复,含 4 机 TP=4 NVFP4 实测):https://forums.developer.nvidia.com/t/glm-5-3-flash-320b-total-parameters-18b-active/381350
  - 另一双机 NVFP4 仓库(0rand):https://github.com/0rand/glm-5.3-flash-nvfp4-2x-dgx-sparks
- 结论:单机只能塞 Q2 量化(速度不达标),双机 33-74 tok/s 是正确打开方式

### 5. MiniMax-M3(6/1, 428B)

- 权重:https://huggingface.co/MiniMaxAI/MiniMax-M3
- 实践帖:
  - 双机 36.6 tok/s 全参数(复现仓库):
    https://github.com/tonyd2wild/MiniMax-M3-2x-DGX-Spark-36-tok-s(原帖 forums.developer.nvidia.com/t/375595)
  - sparkrun 官方 recipe(4x/2x REAP50):
    https://forums.developer.nvidia.com/t/minimax-m3-nvfp4-and-nvfp4-reap-50-for-4x-2x-dgx-sparks/373177
- 结论:双机 36 tok/s(略低于 40 门槛),社区许可非商用限制注意

### 6. Inkling-Small(7/30, Apache 2.0, 276B/12B, Thinking Machines)

- 权重:https://huggingface.co/thinking-machines-lab/Inkling-Small(实验室官方 org)
- 实践帖:
  - 双机 NVFP4 集群:https://forums.developer.nvidia.com/t/inkling-small-nvfp4-on-a-dual-dgx-spark-cluster/378645
  - 主讨论帖:https://forums.developer.nvidia.com/t/inkling-small/378630
- 结论:双机 ~33 tok/s;美国开源阵营的旗舰,文本+图像+音频输入

### 7. GLM-5.2(6/13, 753B/40B)—— 集群参考

- 权重:https://huggingface.co/zai-org/GLM-5.2
- 实践帖:
  - 4 机 22 tok/s decode + 256K ctx(AWQ-INT4 + 15% 专家剪枝):
    http://forums.developer.nvidia.com/t/glm-5-2-on-a-4x-gb10-cluster-22-tok-s-decode-256k-ctx-recipe/374125
  - 混合量化 NVFP4+AQLM 2bit,3 机 348K ctx 带视觉(noze.it 汇总,8/5)
- 结论:3-4 机项目,速度不达标但质量最高档;6 台如组大模型集群可参考

### 8. Nemotron 3.5 Lightning(8/11, 30B/3B, NVIDIA)

- 权重:https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Lightning-30B-A3B-NVFP4
- 实践:eugr/spark-vllm-docker 内置 recipe(含 DSpark 投机 drafter):
  https://github.com/eugr/spark-vllm-docker
- 结论:官方系,单机 56K 下载/3 天,高速本地推理定位

## 三、没有 Spark 实践的模型(避免踩坑)

| 模型 | 为什么没有 | 备注 |
|---|---|---|
| **Kimi K3**(2.8T) | 权重 1.56TB,13 台是数学下限、16 台才够规划;kingy.ai 有完整容量分析 https://kingy.ai/ai/ai-guides/deepseek-v4-flash-kimi-k3-dgx-spark/ | 有人用 Pulsar SSD 专家流化"跑起来"了:0.70 tok/s,https://dev.classmethod.jp/en/articles/dgx-spark-pulsar-ssd-expert-streaming/ ——是实验不是部署。HF 官方讨论区明确回复 Spark 不适合:https://huggingface.co/moonshotai/Kimi-K3/discussions/13 |
| **MiniMax H3**(视频生成) | 465B + 视频管线,Spark 无实践帖;且许可限制美欧英韩主体 | 权重:https://huggingface.co/MiniMaxAI/MiniMax-H3 |
| **Tencent Hy4 preview**(770B/49B) | 8/28 刚发,量化/适配未成熟,再等 2-4 周 | 权重:https://huggingface.co/tencent/Hy4-preview |
| **Muse Glimmer 30B**(Meta) | Apache 2.0 单机可行,但社区当前注意力在 Qwen3.8/GLM 上,无 Spark 帖 | 权重:https://huggingface.co/facebook/Muse-Glimmer-30B |
| **Ling 3.0 / KAT-Coder / Nanbeige4.2** | 规模小或太新,无 Spark 帖 | 有兴趣可做首批实践(反而是机会) |

## 四、给团队的结论

1. **单机 >40 tok/s 的本月任选**:DeepSeek-V4-Flash-0731(52)、Qwen3.8-27B(50-75)、
   Qwen3.8-Flash-Next(34-47,结构化输出更高)——三个都是单机方案
2. **双机档**:GLM-5.3-Flash(33-74,带视觉,MIT 最干净)、MiniMax-M3(36,许可注意)、
   Inkling-Small(33,Apache 2.0)
3. **明确别碰**:Kimi K3(容量物理不可行)、H3(许可+无实践)
4. **抢首发机会**:Hy4 preview、Muse Glimmer、Ling 3.0 还没人发 Spark 实践,
   有余力可做首批(Reference: 首批实践帖通常 1-2 周内是论坛热点)

## 数据来源

- NVIDIA 论坛 GB10 板块(各帖链接见上文)
- noze.it 双月实测汇总(2026-08-05):单机/双机/三机配置与 tok/s 对照表
- kingy.ai Kimi K3 容量分析;dev.classmethod.jp Pulsar 实验
- local-ai-zone / futureagi / presenc.ai 2026 年 7-9 月发布追踪
