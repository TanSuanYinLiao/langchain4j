# langchain4j-azure-open-ai 模块

## 模块职责
**Azure OpenAI 集成模块** — 使用 Azure 官方 SDK (`com.azure.ai.openai.OpenAIClient`) 对接 Azure 托管的 OpenAI 模型，支持 Azure 特有功能如数据源、内容过滤和 Entra ID 认证。

## 核心执行流程

```
用户调用 AzureOpenAiChatModel.chat(ChatRequest)
  │
  ├── ChatModel.chat() [core 接口，触发 listener]
  │
  ├── AzureOpenAiChatModel.doChat(ChatRequest)
  │   ├── 构建 ChatCompletionsOptions
  │   │   ├── 消息 → Azure SDK ChatRequestMessage 格式
  │   │   ├── 设置 model deployment name, temperature, topP, maxTokens, stop, logprobs
  │   │   ├── responseFormat (Text, JsonObject, JsonSchema)
  │   │   ├── Azure 特有: dataSources, enhancements, seed
  │   │   └── 工具 → Azure SDK FunctionDefinition 格式
  │   │
  │   ├── client.getChatCompletions(modelName, options)  ← Azure SDK
  │   │   ├── SDK 内部处理 HTTP (POST Azure OpenAI endpoint)
  │   │   ├── 认证: API key / KeyCredential / TokenCredential (Entra ID)
  │   │   └── 返回 Response<ChatCompletions>
  │   │
  │   └── ChatCompletions → ChatResponse
  │       ├── 提取 message content, tool calls
  │       ├── finishReason (STOP, TOOL_CALLS, CONTENT_FILTERED)
  │       └── TokenUsage
  │
  └── 触发 listener
  v
返回 ChatResponse
```

### 流式流程

```
AzureOpenAiStreamingChatModel.doChat(...)
  ├── client.getChatCompletionsStream(modelName, options) ← Azure SDK 流式 API
  ├── SDK 返回 Stream<ChatCompletions>
  └── 累积部分内容和工具调用，处理 CONTENT_FILTERED
```

## 核心类

| 类 | 职责 |
|----|------|
| `AzureOpenAiChatModel` | 同步聊天，包装 Azure SDK OpenAIClient |
| `AzureOpenAiStreamingChatModel` | 流式聊天 |
| `AzureOpenAiEmbeddingModel` | 向量化 |
| `AzureOpenAiImageModel` | 图片生成 |
| Azure SDK `OpenAIClient` | 官方 SDK 处理 HTTP、序列化、认证 |

## 与其他模块的关键区别

| 特性 | OpenAI/Anthropic/Ollama | Azure OpenAI |
|------|------------------------|--------------|
| HTTP 客户端 | 自定义 HttpClient | Azure SDK |
| 认证 | API Key header | API Key / KeyCredential / TokenCredential (Entra ID) |
| 请求构建 | 手动 JSON | SDK 类型 |
| 流式 | SSE + ServerSentEventListener | SDK Stream API |

## Azure 特有功能

| 功能 | 说明 |
|------|------|
| **Data Sources** | Azure 内置 RAG 数据源集成 |
| **Content Filtering** | 内容过滤，CONTENT_FILTERED finishReason |
| **Entra ID** | Azure Active Directory 认证 |
| **3 种认证方式** | API Key (Azure OpenAI) / KeyCredential (非 Azure) / TokenCredential (AAD) |

## Provider 标识

`ModelProvider.AZURE_OPEN_AI`
