# 007_MiniMax大模型API集成调研

## 1. 概述

本文档介绍MiniMax平台大模型API的调用方式，以及如何集成到robot-keep项目。

## 2. MiniMax平台模型概览

### 2.1 语言/文本模型

| 模型 | 说明 | 适用场景 |
|------|------|---------|
| MiniMax-M2 | 高性能大语言模型 | 通用对话、推理 |
| MiniMax-M2.7 | 旗舰模型，更强推理 | 复杂任务 |
| M2-her | 角色扮演、多轮对话 | 情感交互 |
| M2.5-highspeed | 高速版本 | 实时响应 |

### 2.2 视觉模型

| 模型 | 说明 |
|------|------|
| image-01 | 文生图/图生图 |
| image-01-live | 手绘/卡通风格增强 |

### 2.3 语音模型

| 模型 | 说明 |
|------|------|
| Speech-2.8-HD | 高清语音合成 |
| Speech-02-HD | 标准高清语音 |

### 2.4 视频模型

| 模型 | 说明 |
|------|------|
| MiniMax Hailuo 2.3 | 视频生成 |

## 3. API调用方式

### 3.1 Anthropic兼容API

MiniMax支持Anthropic API兼容调用方式，可通过标准Anthropic SDK调用。

**API地址：**
```
https://api.minimaxi.com/anthropic
```

**认证：** 通过 `ANTHROPIC_API_KEY` 环境变量

**关键端点：**
- `/v1/messages` - 发送消息对话

**请求示例：**
```python
from anthropic import Anthropic

client = Anthropic(
    api_key="your-api-key",
    base_url="https://api.minimaxi.chat/v1"
)

message = client.messages.create(
    model="MiniMax-M2.7",  # 或 "MiniMax-M2"
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "分析这张图片中人物的坐姿是否正确"}
    ]
)
```

### 3.2 视觉理解调用

```python
from anthropic import Anthropic

client = Anthropic(
    api_key="your-api-key",
    base_url="https://api.minimaxi.chat/v1"
)

# 发送图片进行视觉理解
with open("posture_image.jpg", "rb") as f:
    image_data = f.read()

message = client.messages.create(
    model="MiniMax-M2.7",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "data": image_data
                },
                {
                    "type": "text",
                    "text": "分析这张图片中人物的坐姿，给出评估和建议"
                }
            ]
        }
    ]
)
```

## 4. 在robot-keep中的应用场景

### 4.1 坐姿分析（视觉理解）

**流程：**
1. 摄像头捕获儿童坐姿图像
2. 本地MediaPipe提取关键点
3. 将图像+关键点数据发送给MiniMax M2
4. MiniMax分析坐姿并返回建议
5. 本地TTS语音播报

**提示词设计：**
```
你是一个儿童坐姿分析专家。请分析这张图片中儿童的坐姿是否正确。
关键骨骼点坐标：{keypoints}
请判断：
1. 头部是否前倾
2. 背部是否挺直
3. 肩膀是否水平
4. 给出0-100的坐姿评分
5. 如有问题，给出具体纠正建议
```

### 4.2 人员识别增强

**流程：**
1. 本地人脸检测+特征提取
2. 首次识别时发送人脸给MiniMax进行身份确认
3. 后续可本地缓存识别结果

### 4.3 语音对话

**流程：**
1. Whisper ASR转写语音为文字
2. 发送给MiniMax对话
3. MiniMax返回回复文本
4. Piper/MiniMax TTS播放

## 5. LLM Adapter实现

```python
# llm_adapter.py
from anthropic import Anthropic
from typing import List, Dict, Union

class MiniMaxAdapter:
    def __init__(self, api_key: str):
        self.client = Anthropic(
            api_key=api_key,
            base_url="https://api.minimaxi.chat/v1"
        )
        self.model = "MiniMax-M2.7"

    async def chat(self, messages: List[Dict[str, str]]) -> str:
        """文本对话"""
        response = self.client.messages.create(
            model=self.model,
            max_tokens=1024,
            messages=messages
        )
        return response.content[0].text

    async def vision_chat(self, image_data: bytes, prompt: str) -> str:
        """视觉理解对话"""
        response = self.client.messages.create(
            model=self.model,
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": [
                    {"type": "image", "data": image_data},
                    {"type": "text", "text": prompt}
                ]
            }]
        )
        return response.content[0].text
```

## 6. 成本考虑

| 模型 | 定价 | 适用场景 |
|------|------|---------|
| MiniMax-M2.7 | 按token计费 | 复杂分析 |
| MiniMax-M2 | 按token计费 | 普通对话 |
| M2-her | 按token计费 | 情感对话 |

**成本优化建议：**
1. 本地能做的小任务（人脸比对、姿态判断）优先本地
2. 仅在需要复杂推理时调用云端API
3. 考虑使用缓存减少重复调用

## 7. 获取API Key

访问 https://platform.minimaxi.com 获取API Key

## 8. 相关文档

- [MiniMax API文档](https://platform.minimaxi.com/docs/api-reference/text-anthropic-api)
- [Anthropic SDK](https://github.com/anthropics/anthropic-sdk-python)
