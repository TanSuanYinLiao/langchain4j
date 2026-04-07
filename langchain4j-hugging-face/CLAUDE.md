# langchain4j-hugging-face 模块

## 模块职责
**Hugging Face Inference API 集成** (已弃用) — 对接 Hugging Face 推理 API。

> **警告**: 此模块自 1.7.0-beta13 起已弃用。请使用 `OpenAiChatModel` 配合 `baseUrl("https://router.huggingface.co/v1")` 替代。

## 核心类

| 类 | 职责 |
|----|------|
| `HuggingFaceChatModel` | 聊天模型 |
| `HuggingFaceLanguageModel` | 文本补全 |
| `HuggingFaceEmbeddingModel` | 文本嵌入 |
| `HuggingFaceClient` / `HuggingFaceApi` | HTTP 客户端 |

## Provider 标识

`ModelProvider.HUGGING_FACE` (已弃用)
