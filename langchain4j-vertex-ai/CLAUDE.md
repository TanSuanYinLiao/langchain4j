# langchain4j-vertex-ai 模块

## 模块职责
**Google Vertex AI PaLM 集成** — 对接 Google Vertex AI PaLM API (非 Gemini)，支持聊天、嵌入、图片生成和评分。

## 核心类

| 类 | 职责 |
|----|------|
| `VertexAiChatModel` | 聊天模型 |
| `VertexAiLanguageModel` | 文本补全 |
| `VertexAiEmbeddingModel` | 文本嵌入 |
| `VertexAiImageModel` | 图片生成 |
| `VertexAiScoringModel` | 文档评分/重排序 |

## 核心执行流程

```
VertexAiChatModel.doChat(ChatRequest)
  │
  ├── 使用 com.google.cloud.aiplatform.v1.PredictionServiceClient
  ├── endpoint: projects/{project}/locations/{location}/publishers/google/models/{model}
  ├── client.predict(endpoint, instances, parameters)
  └── 响应 → ChatResponse 映射
```

## 认证

使用 Google Cloud 凭证:
- Application Default Credentials (ADC)
- Service Account JSON Key

## 注意

此模块对接的是 PaLM 时代的 API。新的 Gemini 模型请使用 `langchain4j-vertex-ai-gemini`。

## Provider 标识

`ModelProvider.GOOGLE_VERTEX_AI`
