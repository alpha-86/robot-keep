# 003_语音交互技术调研

## 1. 概述

robot-keep项目需要具备双向通话能力（麦克风+扬声器），以及语音交互能力。本文档调研现有开源方案。

## 2. 核心需求

- **双向通话**：实时语音传输
- **语音识别 (ASR)**：将语音转为文字
- **语音合成 (TTS)**：将文字转为语音
- **回声消除 (AEC)**：防止扬声器声音被麦克风采集
- **噪声抑制 (NS)**：降低环境噪声

## 3. 开源方案对比

### 3.1 麦克风阵列方案

#### ReSpeaker Mic Array
- Seeed Studio产品
- 2Mic/4Mic/6Mic版本
- 支持树莓派
- 开源驱动：https://github.com/respeaker/mic_hat

**特点：**
- 波束成形（Beamforming）
- 声源定位（DOA）
- 噪声抑制

#### ManyEars
- 加拿大舍布鲁克大学开源
- 实时麦克风阵列处理
- 声音定位、跟踪、分离

**GitHub：** https://github.com/AaronAncuta/ManyEars

#### 讯飞麦克风阵列
- 6+1环形/4线性阵列
- AEC/ANS/AGC
- 商业方案

---

### 3.2 语音识别 (ASR) 开源方案

#### Whisper (OpenAI)
- 多语言语音识别
- 开源模型（多规格）
- 支持实时流式识别

**模型规模：**
- tiny (~39M params)
- base (~74M)
- small (~244M)
- medium (~769M)
- large (~1550M)

**优点：**
- 识别准确率高
- 支持中文
- 离线部署

**GitHub：** https://github.com/openai/whisper

#### Vosk
- 轻量级语音识别
- 离线运行
- 支持嵌入式设备

**优点：**
- 模型小（~50MB）
- 适合实时
- 支持多种语言

**GitHub：** https://github.com/alphacep/vosk-api

---

### 3.3 语音合成 (TTS) 开源方案

#### Coqui TTS
- 高质量语音合成
- 支持多语言
- 神经网络模型

**GitHub：** https://github.com/coqui-ai/TTS

#### VITS (Vocal Impostor Text-to-Speech)
- 高质量端到端TTS
- 开源效果较好

#### Piper
- ONNX优化TTS
- 适合本地运行
- 速度快

**GitHub：** https://github.com/rhasspy/piper

---

### 3.4 音频处理

#### WebRTC
- 实时通信音频处理
- AEC/ANS/AGC内置
- 广泛使用

**模块：**
- AEC (Echo Cancellation)
- ANS (Active Noise Suppression)
- VAD (Voice Activity Detection)

#### SpeexDSP
- 回声消除
- 噪声抑制
- 轻量级

---

## 4. 推荐方案

### 一期方案（验证阶段）

**硬件选型：**
- ReSpeaker 4-Mic Array（性价比高）
- 或使用USB麦克风+扬声器

**软件栈：**

| 功能 | 方案 | 理由 |
|------|------|------|
| ASR | Whisper base | 准确率高，支持中文 |
| TTS | Piper/MiniMax API | 效果好/成本可控 |
| AEC | WebRTC | 成熟稳定 |

### 二期考虑

1. 本地ASR部署（降低API成本）
2. 语音活动检测（VAD）优化
3. 打断唤醒能力

## 5. 双向通话关键技术点

### 5.1 回声消除 (AEC)

**问题：** 扬声器播放的声音被麦克风采集，导致回声

**解决方案：**
1. 硬件：指向性麦克风
2. 软件：WebRTC AEC算法
3. 参考信号：播放前采集参考信号

### 5.2 语音活动检测 (VAD)

**作用：** 检测是否有人说话，控制传输和唤醒

**方案：**
- WebRTC VAD
- Silero VAD（效果好）

### 5.3 噪声抑制 (NS)

**方案：**
- WebRTC ANS
- RNNoise（基于RNN）

## 6. 成本估算

| 组件 | 方案 | 成本 |
|------|------|------|
| 麦克风阵列 | ReSpeaker 4-Mic | ~$30 |
| ASR | Whisper (本地) | 免费 |
| TTS | Minimax API | 按量计费 |
| 音频处理 | WebRTC | 免费开源 |
