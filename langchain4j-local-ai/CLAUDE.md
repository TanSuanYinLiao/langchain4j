# langchain4j-local-ai 模块

## 模块职责
**LocalAI 集成** — 对接本地运行的 LocalAI 服务器，使用 OpenAI 兼容 API 格式。

## 核心类

| 类 | 职责 |
|----|------|
| `LocalAiChatModel` | 同步聊天 |
| `LocalAiStreamingChatModel` | 流式聊天 |
| `LocalAiLanguageModel` / `LocalAiStreamingLanguageModel` | 文本补全 |
| `LocalAiEmbeddingModel` | 文本嵌入 |

## 核心执行流程

与 OpenAI 模块相同的 API 格式，但默认 `baseUrl` 指向 `http://localhost:8080/v1`:

```
LocalAiChatModel.doChat(ChatRequest)
  │
  ├── 构建 OpenAI 兼容请求
  ├── POST http://localhost:8080/v1/chat/completions
  └── OpenAI 兼容响应处理
```

## 使用场景

- 本地部署的开源模型 (LLaMA, Mistral, etc.)
- 无需 API Key 的本地推理
- 隐私敏感场景

## Provider 标识

`ModelProvider.LOCAL_AI`
