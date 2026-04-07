# langchain4j-github-models 模块

## 模块职责
**GitHub Models 集成** — 对接 GitHub Models 市场，使用 Azure AI Inference SDK。

## 核心类

| 类 | 职责 |
|----|------|
| `GitHubModelsChatModel` | 同步聊天 |
| `GitHubModelsStreamingChatModel` | 流式聊天 |
| `GitHubModelsEmbeddingModel` | 文本嵌入 |
| `InternalGitHubModelHelper` | 请求/响应转换 |

## 核心执行流程

```
GitHubModelsChatModel.doChat(ChatRequest)
  │
  ├── 使用 com.azure.ai.inference.ChatCompletionsClient
  ├── client.complete(request)
  │   └── Azure AI Inference SDK 内部
  └── 响应 → ChatResponse 映射
```

## 认证

使用 GitHub Personal Access Token 作为 API Key。

## Provider 标识

`ModelProvider.GITHUB_MODELS`
