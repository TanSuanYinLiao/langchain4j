# langchain4j-open-ai 模块

## 模块职责
**OpenAI 集成模块** — LangChain4j 的**参考实现**，其他 provider 模块以此为模板。实现 ChatModel、StreamingChatModel、EmbeddingModel、ImageModel、ModerationModel 等核心接口，对接 OpenAI Chat Completions / Embeddings / Images / Moderations API。

## 架构分层

```
OpenAiChatModel / OpenAiStreamingChatModel  (实现核心接口)
  │
  ├── OpenAiUtils (请求/响应转换)
  │
  ├── OpenAiClient (abstract, SPI 工厂)
  │   └── DefaultOpenAiClient (HTTP 客户端)
  │       │
  │       └── RequestExecutor
  │           ├── SyncRequestExecutor (同步 HTTP)
  │           └── StreamingRequestExecutor (SSE 流式)
  │
  └── HttpClient (langchain4j-http-client 抽象层)
```

## 核心执行流程

### 同步聊天

```
用户调用 OpenAiChatModel.chat(ChatRequest)
  │
  ├── ChatModel.chat() [core 接口，触发 listener]
  │
  ├── OpenAiChatModel.doChat(ChatRequest)
  │   ├── validate(parameters)
  │   ├── OpenAiUtils.toOpenAiChatRequest()
  │   │   ├── SystemMessage → system message
  │   │   ├── UserMessage → user message (含图片处理)
  │   │   ├── AiMessage → assistant message (含 tool_calls)
  │   │   ├── ToolExecutionResultMessage → tool message
  │   │   ├── ToolSpecification → OpenAI function/tool 格式
  │   │   └── 设置 response_format, temperature, topP, stop 等
  │   │
  │   ├── client.chatCompletion(openAiRequest)  → RequestExecutor
  │   │   ├── 构建 HttpRequest (POST baseUrl/chat/completions)
  │   │   ├── 设置 headers: Authorization: Bearer {apiKey}
  │   │   └── 返回 RequestExecutor<ChatCompletionResponse>
  │   │
  │   ├── RequestExecutor.executeRaw()
  │   │   └── httpClient.execute(httpRequest) → JSON 反序列化
  │   │
  │   └── ChatCompletionResponse → ChatResponse
  │       ├── aiMessageFrom(): 提取 text, thinking, tool_calls
  │       └── 构建 TokenUsage, FinishReason, metadata
  │
  └── 触发 listener onResponse/onError
  v
返回 ChatResponse
```

### 流式聊天

```
OpenAiStreamingChatModel.doChat(ChatRequest, StreamingChatResponseHandler)
  ├── 设置 stream(true), streamOptions
  ├── client.chatCompletion(request).onRawPartialResponse(handler)
  │   ├── StreamingRequestExecutor 创建 SSE 连接
  │   ├── ServerSentEventListener 接收 SSE 事件
  │   ├── 每个 chunk: JSON → ChatCompletionResponse → handler.onPartialResponse()
  │   └── [DONE] 触发 onClose → streamingCompletionCallback
  └── OpenAiStreamingResponseBuilder 累积内容/工具调用/thinking
```

## 核心类

| 类 | 职责 |
|----|------|
| `OpenAiChatModel` | 同步聊天模型，编排请求构建→执行→响应转换 |
| `OpenAiStreamingChatModel` | 流式聊天模型 |
| `OpenAiClient` (abstract) | 客户端接口 + SPI 工厂 |
| `DefaultOpenAiClient` | 具体 HTTP 客户端，为每个端点构建 HttpRequest |
| `RequestExecutor` | 同步/流式执行决策 |
| `SyncRequestExecutor` | 同步 HTTP 执行 + JSON 反序列化 |
| `StreamingRequestExecutor` | SSE 流式通过 ServerSentEventListener |
| `OpenAiUtils` | 核心转换逻辑：LangChain4j ↔ OpenAI 格式 |
| `OpenAiEmbeddingModel` | 向量化操作，支持批处理 |
| `OpenAiImageModel` | 图片生成 |
| `OpenAiModerationModel` | 内容审核 |
| `OpenAiAudioModel` | 音频转录 |

## 重试机制

所有同步调用通过 `withRetryMappingExceptions()` 包装，自动处理网络异常重试。

## Provider 标识

`ModelProvider.OPEN_AI`
