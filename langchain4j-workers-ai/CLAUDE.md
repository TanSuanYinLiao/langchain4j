# langchain4j-workers-ai 模块

## 模块职责
**Cloudflare Workers AI 集成** — 对接 Cloudflare Workers AI 平台，支持聊天、文本补全、嵌入和图片生成。

## 核心类

| 类 | 职责 |
|----|------|
| `WorkersAiChatModel` | 聊天模型 |
| `WorkersAiLanguageModel` | 文本补全 |
| `WorkersAiEmbeddingModel` | 文本嵌入 |
| `WorkersAiImageModel` | 图片生成 |
| `WorkersAiClient` / `WorkersAiApi` | HTTP 客户端 |

## 核心执行流程

```
WorkersAiChatModel.doChat(ChatRequest)
  │
  ├── POST https://api.cloudflare.com/client/v4/accounts/{accountId}/ai/run/{model}
  │   Headers: Authorization: Bearer {apiToken}
  └── 响应 → ChatResponse
```

## 认证

- `accountId`: Cloudflare 账户 ID
- `apiToken`: Cloudflare API Token

## Provider 标识

`ModelProvider.CLOUDFLARE_WORKERS_AI`
