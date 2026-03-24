# 005_大模型集成架构调研

## 1. 概述

robot-keep项目需要调用大语言模型能力（Minimax优先，也需支持未来替换其他模型）。本文档调研LLM集成架构。

## 2. 核心需求

- 调用Minimax API
- 支持未来方便替换其他LLM
- 模块化设计
- 成本可控

## 3. 开源方案对比

### 3.1 LLM Router / Gateway

#### LiteLLM
- 统一接口调用100+LLM
- 支持OpenAI兼容格式
- 多模型路由

**优点：**
- 简单易用
- 支持MiniMax（OpenAI兼容）
- 成本追踪

**GitHub：** https://github.com/BerriAI/litellm

#### OpenRouter
- 多模型聚合平台
- 统一API调用
- 30+免费模型

**特点：**
- 无需自己部署
- 支持模型对比
- 备用模型

#### LLMRouter (UIUC)
- 16+路由策略
- 自动模型选择
- 适合研究

**GitHub：** https://github.com/ulab-uiuc/LLMRouter

---

### 3.2 应用框架

#### LangChain / LangGraph
- LLM应用开发框架
- 工具调用、Agent
- 记忆管理

#### LlamaIndex
- RAG框架
- 知识库问答
- 数据连接

#### Dify
- 开源LLM应用平台
- 可视化编排
- 支持自定义模型

---

### 3.3 模型协议

#### MCP (Model Context Protocol)
- AI与工具/数据连接标准
- Go-MCP / Python-MCP
- 工具调用标准化

**GitHub：** https://github.com/modelcontextprotocol

---

## 4. 推荐方案

### 架构设计：LLM Adapter Pattern

```
┌─────────────────────────────────────┐
│           Application              │
│  (人体识别 / 坐姿检测 / 语音交互)   │
└────────────────┬──────────────────┘
                 │
┌────────────────▼──────────────────┐
│        LLM Service Layer            │
│  ┌────────────────────────────────┐│
│  │      LLM Router/Gateway       ││
│  │   (支持多模型动态切换)          ││
│  └────────────────────────────────┘│
└────────────────┬──────────────────┘
                 │
┌────────────────▼──────────────────┐
│        LLM Adapter                 │
│  ┌──────────┐  ┌──────────────┐   │
│  │ MiniMax  │  │  OpenAI     │   │
│  │ Adapter  │  │  Adapter     │   │
│  └──────────┘  └──────────────┘   │
└────────────────┬──────────────────┘
                 │
┌────────────────▼──────────────────┐
│        External APIs               │
│  MiniMax / OpenAI / Claude / ...  │
└─────────────────────────────────────┘
```

### 一期方案

**技术选型：**

| 组件 | 方案 | 说明 |
|------|------|------|
| LLM调用 | LiteLLM | 统一接口 |
| 本地模型 | Ollama | 本地模型管理 |
| Agent框架 | LangChain LCEL | 简单够用 |

**理由：**
1. LiteLLM支持OpenAI兼容接口，MiniMax API可兼容
2. 设计LLM Adapter接口，方便后续扩展
3. 支持模型热切换

### 二期考虑

1. 引入LLMRouter实现智能路由
2. 成本追踪与优化
3. 缓存层设计

## 5. 核心接口设计

```python
class LLMAdapter(ABC):
    @abstractmethod
    async def chat(self, messages: List[Message]) -> str:
        pass

    @abstractmethod
    async def vision(self, image: Image, prompt: str) -> str:
        """用于图像理解"""
        pass

class MiniMaxAdapter(LLMAdapter):
    # 实现MiniMax API调用

class OpenAIAdapter(LLMAdapter):
    # 实现OpenAI兼容接口

class LLMRouter:
    def __init__(self, adapters: Dict[str, LLMAdapter]):
        self.adapters = adapters

    async def chat(self, messages: List[Message], model: str = "default") -> str:
        # 路由到对应模型
```

## 6. 成本估算

| 组件 | 方案 | 成本 |
|------|------|------|
| LiteLLM | 开源自部署 | 免费 |
| Ollama | 本地模型 | GPU成本 |
| MiniMax API | 按量计费 | 按需 |
