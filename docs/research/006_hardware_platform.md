# 006_硬件平台选型调研

## 1. 概述

robot-keep项目需要选择合适的硬件平台，兼顾性能和成本。本文档调研适合的硬件方案。

## 2. 核心需求

- 支持摄像头（视觉输入）
- 支持麦克风阵列
- 支持扬声器输出
- 能运行边缘AI推理
- 成本可控
- 功耗合理

## 3. 平台对比

### 3.1 单板计算机

#### Raspberry Pi 5
- 4核Cortex-A76
- 8GB RAM可选
- 树莓派生态成熟
- **优点：** 便宜、配件丰富、功耗低
- **缺点：** AI推理能力有限

**价格：** ~$60（4GB）/ $80（8GB）

#### NVIDIA Jetson Nano / Orin Nano
- NVIDIA GPU
- CUDA支持
- 边缘AI优化

| 型号 | GPU | 价格 |
|------|-----|------|
| Jetson Nano | 128-core Maxwell | ~$150 |
| Jetson Orin Nano 4GB | 512-core Ampere | ~$200 |
| Jetson Orin Nano 8GB | 1024-core Ampere | ~$350 |

**优点：** GPU加速、CUDA生态
**缺点：** 价格较高、功耗较高

#### Orange Pi 5 / Rockchip RK3588
- 8核ARM
- NPU可选（6TOPS）
- 价格便宜

**价格：** ~$80-120

---

### 3.2 边缘AI设备

#### Google Edge TPU
- 专用AI加速器
- 配合树莓派使用
- TFLite加速

**Coral Dev Board：** ~$150

#### Intel Neural Compute Stick
- Movidius VPU
- OpenVINO支持

---

### 3.3 工业级方案

#### 瑞芯微RK3588开发板
- 8K视频处理
- NPU 6TOPS
- 适合AI视觉

**价格：** ~$100-200

#### 算能科技(算能)AI模组
- 国产方案
- 性价比高

---

### 3.4 全志系列
- 全志H616/H618
- 入门级
- 性价比极高

## 4. 推荐方案

### 一期方案（验证阶段）

**推荐：Raspberry Pi 5 (8GB)**

**理由：**
1. 性价比高，配件丰富
2. 适合算法验证
3. 社区支持好
4. 可以运行：
   - MediaPipe（实时姿态估计）
   - Whisper（语音识别）
   - 人脸识别（InsightFace）

**不足：**
- AI推理速度较慢
- 适合验证，不适合长期产品

### 二期方案（产品化）

**推荐：Jetson Orin Nano 8GB 或 RK3588 + NPU**

**理由：**
1. GPU/NPU加速推理
2. 实时性更好
3. 支持CUDA生态（如需）

### 硬件成本估算（一期）

| 组件 | 方案 | 价格 |
|------|------|------|
| 主控 | Raspberry Pi 5 8GB | ~$80 |
| 摄像头 | Logitech C920 / IMX219 | $50-80 |
| 麦克风阵列 | ReSpeaker 4-Mic | ~$30 |
| 存储 | 64GB SD卡 | ~$10 |
| **总计** | | **~$170-200** |

## 5. 软件栈对应硬件需求

| 功能 | 算法 | Raspberry Pi 5 | Jetson Orin Nano |
|------|------|----------------|------------------|
| 人脸检测 | MediaPipe/YOLOv8 | 可运行，慢 | 流畅 |
| 人脸识别 | InsightFace | 可运行，慢 | 流畅 |
| 姿态估计 | MediaPipe Pose | 流畅 | 流畅 |
| 坐姿判断 | 自定义逻辑 | 流畅 | 流畅 |
| ASR | Whisper base | 可运行，需优化 | 可运行 |

## 6. 一期硬件采购建议

```
验证阶段（桌面测试）：
- 普通开发电脑（已有）
- USB摄像头
- USB麦克风
- 扬声器（普通音箱）

嵌入式阶段（小车）：
- Raspberry Pi 5
- 摄像头模块
- 麦克风阵列
- 扬声器模块
```
