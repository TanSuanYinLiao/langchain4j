# langchain4j-nomic 模块

## 模块职责
**Nomic AI 集成** — 仅支持 Nomic 的文本嵌入服务。

## 核心类

| 类 | 职责 |
|----|------|
| `NomicEmbeddingModel` | 文本嵌入 |
| `NomicClient` / `NomicApi` | HTTP 客户端 |

## 核心执行流程

```
NomicEmbeddingModel.embedAll(List<TextSegment> segments)
  │
  ├── POST https://api-atlas.nomic.ai/v1/embedding/text
  │   body: { texts: [...], model: "nomic-embed-text-v1.5" }
  └── 返回: { embeddings: [[...], ...] }
```

## Provider 标识

`ModelProvider.NOMIC`
