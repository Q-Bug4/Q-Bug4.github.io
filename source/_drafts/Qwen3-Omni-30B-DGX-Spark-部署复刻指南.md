---
title: Qwen3-Omni-30B DGX Spark 部署复刻指南
date: 2026-09-09 18:10:00
tags: [AI生成, DGX-Spark, Qwen3-Omni, 部署]
permalink: /articles/2026/09/09/1757411400000.html
published: false
---

> 📝 本文由 AI agent 起草,尚未经过人工审核。发布前请由 Kael 本人审阅修改。
> 撰写 agent:Hermes · 起草日期:2026-09-09

# Qwen3-Omni-30B DGX Spark 部署复刻指南

模型:`Qwen/Qwen3-Omni-30B-A3B-Instruct`(30B/3B 激活 MoE,文本/图像/音频/视频输入,文本+语音输出)
硬件:DGX Spark GB10(128GB 统一内存,sm_121,ARM64)

## 核心结论:两条部署路径,按需选择

| 路径 | 能力 | 依据 |
|---|---|---|
| **A. vllm-omni 项目** | 全模态输入 + **语音输出**(thinker-talker) | vLLM 官方 omni 分支项目,Qwen 官方 README 推荐 |
| **B. ohoachuck 的 Spark fork** | 文本/图像/音频输入 + 文本输出(无语音) | Spark 社区实测 40-50 tok/s,一键脚本 |

关键认知:**主线上游 vLLM 不支持 Qwen3-Omni 的音频输出**(Reddit/社区共识),只有 vllm-omni 实现了 thinker-talker 完整流水线。想要 TTS 必须走路径 A。

## 路径 A:vllm-omni(推荐,能语音输出)

### 安装

```bash
git clone https://github.com/vllm-project/vllm-omni.git
cd vllm-omni
pip install -e .           # 或 pip install vllm-omni
pip install 'vllm-omni[demo]'   # 可选,Gradio WebUI
```

⚠️ GB10 注意:vllm-omni 的 aarch64 wheel 支持情况需实测。若安装后 CUDA kernel 报
illegal instruction(SM121 的老问题),退路是从源码装 vllm-omni(它 fork 自 vLLM,
带着全部 Blackwell 修复),或参考路径 B 的环境变量(`TORCH_CUDA_ARCH_LIST=12.0f`)。

### 下载权重(~60GB)

```bash
hf download Qwen/Qwen3-Omni-30B-A3B-Instruct --local-dir ~/models/Qwen3-Omni-30B-A3B-Instruct
# 国内: modelscope download --model Qwen/Qwen3-Omni-30B-A3B-Instruct --local_dir ~/models/...
```

### 启动(单进程,最简单)

```bash
vllm serve Qwen/Qwen3-Omni-30B-A3B-Instruct --omni --port 8091
```

- 默认自动加载 `qwen3_omni_moe.yaml` 部署配置(Thinker + Talker + Code2Wav 三阶段)
- 默认 `VLLM_USE_FLASHINFER_MOE_FP16=0`(Triton 未量化 MoE 路径,避免 CUTLASS 回归)
- 异步分块流式输出默认开启

### 只文本输出(省显存省延迟,不加载 Talker)

```bash
vllm serve Qwen/Qwen3-Omni-30B-A3B-Instruct --omni --port 8091 \
    --deploy-config vllm_omni/deploy/qwen3_omni_moe_thinking.yaml
```

### 分阶段启动(Spark 单卡上意义不大,单进程即可)

vllm-omni 支持 stage-based CLI 把 Thinker/Talker/Code2Wav 分到不同 GPU;
Spark 只有一块 GB10,不适用。单进程内存:权重 ~60GB(BF16)+ Talker/KV,
`--stage-overrides '{"1": {"gpu_memory_utilization": 0.5}}'` 可压 Talker 占用。

### 请求示例

文本+音频输出(返回两个 choice):

```bash
curl http://localhost:8091/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "Qwen/Qwen3-Omni-30B-A3B-Instruct",
  "messages": [{"role": "user", "content": "用一句话介绍你自己"}],
  "modalities": ["text", "audio"]
}' | jq -r '.choices[0].message.content'          # 文本
# .choices[1].message.audio.data 是 base64 wav
```

- `modalities: ["text"]` 只文本(跳过语音生成,更快)
- `modalities: ["audio"]` 只语音
- 多模态输入:content 数组里 `{"type":"image_url"|"input_audio"|"video_url"}`
- 流式:`"stream": true`,文本和音频都支持流式

### WebUI(可选)

```bash
./run_gradio_demo.sh --model Qwen/Qwen3-Omni-30B-A3B-Instruct --server-port 8091 --gradio-port 7861
```

## 路径 B:ohoachuck/dgx-spark-vllm-qwen3-omni(Spark 原生 fork)

eelbaz/dgx-spark-vllm-setup 的 fork,专门为 Spark + Qwen3-Omni 适配,2026-01 实测通过。
**不支持语音输出**(只 thinker),但部署最省事、有实测数据(文本 40-50 tok/s,
音频 12 秒 M4A 处理 2-3 秒)。

```bash
git clone https://github.com/ohoachuck/dgx-spark-vllm-qwen3-omni.git
cd dgx-spark-vllm-qwen3-omni
./install.sh                                   # 20-30 分钟,编译 Blackwell 修复版 vLLM
cd vllm-install && source vllm_env.sh
pip install 'vllm[audio]' pydub
sudo apt-get install -y ffmpeg

./vllm-serve.sh text         # 纯文本模式(更快)
./vllm-serve.sh multimodal   # 多模态模式(音频+图像+文本)
./vllm-status.sh / ./vllm-stop.sh / ./vllm-restart.sh
```

它修好的坑(复刻时不用再踩):

1. 音频 token 格式:`<|audio_start|><|audio_pad|><|audio_end|>`(不是 Qwen2-Audio 的
   `<|audio_bos|><|AUDIO|><|audio_eos|>`)
2. 音频解码管线:soundfile → pydub → ffmpeg 三级 fallback(M4A/MP3/AAC/WAV/FLAC/OGG)
3. SM121 kernel:CMakeLists 加 `12.0f`、Triton main 分支、`TORCH_CUDA_ARCH_LIST=12.0f`
4. 进程管理:stop 时要连 EngineCore 子进程一起杀,否则僵尸进程占着统一内存

已知限制:视频不支持(Qwen3-Omni 本身无原生视频输入;要视频看 Nemotron-3-Nano-Omni);
多路音频线性增加延迟;多模态模式下文本流式但音频处理是阻塞的。

## 复刻建议(针对你们的 6× Spark)

1. **首台复刻走路径 B**(有 Spark 实测、脚本全、坑都填了),验证音频/图像输入质量
2. **要语音输出再上路径 A**:vllm-omni 在 Spark 上是待验证项(社区主要在 B200/H100 上跑),
   首启关注三点——aarch64 安装、SM121 kernel、三阶段内存合计是否爆 128GB
   (权重 60GB BF16 应该够,若紧张可试 FP8 量化版 thinker)
3. 端口规划:这台的 Omni 服务(8091)注册进 LiteLLM 网关,与文本机隔离,
   `modalities: ["text"]` 路由日常请求,语音交互才带 audio
4. 兄弟变体:-Thinking(CoT 版,无语音输出)、-Captioner(音频字幕专用,更小更专)

## 数据来源

- docs.vllm.ai vllm-omni Qwen3-Omni online serving 官方文档(命令均出自此处)
- github.com/ohoachuck/dgx-spark-vllm-qwen3-omni(Spark 实测 README)
- huggingface.co/Qwen/Qwen3-Omni-30B-A3B-Instruct(模型卡,vLLM/SGLang quickstart)
- r/LocalLLaMA(上游 vLLM 不支持 audio out 的社区确认)
