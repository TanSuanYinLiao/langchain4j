# langchain4j-vertex-ai-anthropic 模块

## 模块职责
**Vertex AI Anthropic 集成** — 通过 Google Vertex AI Model Garden 访问 Anthropic Claude 模型。

## 核心类

| 类 | 职责 |
|----|------|
| `VertexAiAnthropicChatModel` | 同步聊天 |
| `VertexAiAnthropicStreamingChatModel` | 流式聊天 |
| `VertexAiAnthropicClient` | HTTP 客户端 |
| `AnthropicRequestMapper` | LangChain4j 请求 → Anthropic 格式 |
| `AnthropicResponseMapper` | Anthropic 响应 → LangChain4j 格式 |

## 核心执行流程

```
VertexAiAnthropicChatModel.doChat(ChatRequest)
  │
  ├── 使用 Google Cloud 凭证获取 access token
  ├── AnthropicRequestMapper → Anthropic API 格式
  ├── POST https://{region}-aiplatform.googleapis.com/v1/projects/{project}/locations/{region}/publishers/anthropic/models/{model}:rawPredict
  │   Headers: Authorization: Bearer {googleAccessToken}
  ├── AnthropicResponseMapper → ChatResponse
  └── 支持 cache control (AnthropicCacheControl)
```

## 与 langchain4j-anthropic 的区别

| 特性 | langchain4j-anthropic | langchain4j-vertex-ai-anthropic |
|------|----------------------|-------------------------------|
| 认证 | Anthropic API Key | Google Cloud 凭证 |
| 端点 | api.anthropic.com | vertex AI endpoint |
| 缓存 | EPHEMERAL | AnthropicCacheControl |

## Provider 标识

`ModelProvider.GOOGLE_VERTEX_AI_ANTHROPIC`
