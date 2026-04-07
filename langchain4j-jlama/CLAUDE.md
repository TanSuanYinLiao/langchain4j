# langchain4j-jlama 模块

## 模块职责
**Jlama 本地推理** — 使用纯 Java LLM 运行时 Jlama 进行本地聊天、文本补全和嵌入。

## 核心类

| 类 | 职责 |
|----|------|
| `JlamaChatModel` | 同步聊天 |
| `JlamaStreamingChatModel` | 流式聊天 |
| `JlamaLanguageModel` / `JlamaStreamingLanguageModel` | 文本补全 |
| `JlamaEmbeddingModel` | 文本嵌入 |
| `JlamaModelRegistry` | 模型下载和管理 |

## 核心执行流程

```
JlamaChatModel.doChat(ChatRequest)
  │
  ├── JlamaModelRegistry.getModel(modelName)  ← 下载/加载模型
  ├── 使用 com.github.tjake.jlama 进行推理
  │   ├── SafeTensors 格式加载模型权重
  │   ├── 消息格式化为 prompt
  │   ├── 逐 token 生成
  │   └── 支持 tool calling (ToolCall/ToolResult prompt)
  └── 返回 ChatResponse
```

## 特点

- 纯 Java 实现，无需 Python/C++ 依赖
- SafeTensors 模型格式
- 支持 tool calling
- 模型注册表管理下载

## Provider 标识

`ModelProvider.JLAMA`
