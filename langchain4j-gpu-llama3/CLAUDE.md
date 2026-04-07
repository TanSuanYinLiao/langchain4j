# langchain4j-gpu-llama3 模块

## 模块职责
**GPU Llama3 本地推理** — 使用 GPULlama3.java 库进行 GPU/CPU 本地 Llama3 推理。

## 核心类

| 类 | 职责 |
|----|------|
| `GPULlama3ChatModel` | 同步聊天 |
| `GPULlama3StreamingChatModel` | 流式聊天 |
| `GPULlama3BaseModel` | 基类 |
| `GPULlama3ResponseParser` | 响应解析 (分离 thinking 内容) |

## 核心执行流程

```
GPULlama3ChatModel.doChat(ChatRequest)
  │
  ├── 加载 GGUF 模型文件
  ├── 使用 GPULlama3.java 进行推理
  │   ├── onGPU=true → GPU 推理
  │   ├── onGPU=false → CPU 推理
  │   └── 逐 token 生成
  ├── GPULlama3ResponseParser:
  │   ├── 自动分离 <thinking> 内容 → thinking content block
  │   └── 提取主要回复内容
  └── 返回 ChatResponse
```

## Provider 标识

`ModelProvider.GPU_LLAMA3`
