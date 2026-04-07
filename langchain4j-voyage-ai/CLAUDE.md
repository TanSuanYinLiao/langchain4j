# langchain4j-voyage-ai 模块

## 模块职责
**Voyage AI 集成** — 对接 Voyage AI，支持文本嵌入和文档重排序。

## 核心类

| 类 | 职责 |
|----|------|
| `VoyageAiEmbeddingModel` | 文本嵌入 |
| `VoyageAiScoringModel` | 文档重排序 |
| `VoyageAiClient` | HTTP 客户端 |
| `VoyageAiEmbeddingModelName` / `VoyageAiScoringModelName` | 模型目录 |

## 核心执行流程

### 嵌入

```
VoyageAiEmbeddingModel.embedAll(List<TextSegment> segments)
  │
  ├── POST https://api.voyageai.com/v1/embeddings
  │   body: { input: [...], model: "voyage-3" }
  └── 返回: { data: [{ embedding: [...] }] }
```

### 重排序

```
VoyageAiScoringModel.scoreAll(queries, segments)
  │
  ├── POST https://api.voyageai.com/v1/rerank
  │   body: { query: "...", documents: [...], model: "rerank-2" }
  └── 返回: { results: [{ relevance_score }] }
```

## Provider 标识

`ModelProvider.VOYAGE_AI`
