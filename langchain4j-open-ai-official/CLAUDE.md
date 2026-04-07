# langchain4j-open-ai-official 模块

## 模块职责
**OpenAI 官方 SDK 集成** — 使用 OpenAI 官方 Java SDK (`com.openai.client.OpenAIClient`)，不同于社区 `langchain4j-open-ai` 模块。

## 核心类

| 类 | 职责 |
|----|------|
| `OpenAiOfficialChatModel` | 同步聊天 |
| `OpenAiOfficialStreamingChatModel` | 流式聊天 |
| `OpenAiOfficialResponsesStreamingChatModel` | Responses API 流式聊天 |
| `OpenAiOfficialEmbeddingModel` | 文本嵌入 |
| `OpenAiOfficialImageModel` | 图片生成 |
| `OpenAiOfficialBaseChatModel` | 共享基类 |

## 核心执行流程

```
OpenAiOfficialChatModel.doChat(ChatRequest)
  │
  ├── 使用 com.openai.client.OpenAIClient 构建请求
  ├── client.chat().completions().create(request)
  │   └── SDK 内部: HTTP POST /v1/chat/completions
  └── SDK 响应 → ChatResponse 映射
```

## 多后端支持

| 后端 | 说明 |
|------|------|
| OpenAI | 标准 OpenAI API |
| Azure OpenAI | 通过 `AzureInternalOpenAiOfficialHelper` |
| Microsoft Foundry | Azure AI Studio |
| GitHub Models | GitHub 模型市场 |

## 与 langchain4j-open-ai 的区别

| 特性 | langchain4j-open-ai | langchain4j-open-ai-official |
|------|---------------------|------------------------------|
| HTTP 客户端 | 自定义 HttpClient | OpenAI 官方 SDK |
| 序列化 | 手动 JSON | SDK 内部处理 |
| Azure 支持 | 需要单独的 Azure 模块 | 内置 Azure 支持 |

## Provider 标识

`ModelProvider.OPEN_AI_OFFICIAL`
