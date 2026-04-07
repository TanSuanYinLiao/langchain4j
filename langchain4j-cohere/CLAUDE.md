# langchain4j-cohere 模块

## 模块职责
**Cohere AI 集成** — 对接 Cohere API，支持文本嵌入和文档重排序 (reranking)。

## 核心类

| 类 | 职责 |
|----|------|
| `CohereEmbeddingModel` | 文本嵌入 (继承 DimensionAwareEmbeddingModel) |
| `CohereScoringModel` | 文档重排序 (ScoringModel 接口) |
| `CohereClient` / `CohereApi` | HTTP 客户端 |

## 核心执行流程

### 嵌入

```
CohereEmbeddingModel.embedAll(List<TextSegment> segments)
  │
  ├── POST https://api.cohere.ai/v1/embed
  │   body: { texts: [...], model: "embed-english-v3.0", input_type: "search_document" }
  ├── 返回: { embeddings: [[...], ...] }
  └── 最多 96 segments/批次
```

### 重排序

```
CohereScoringModel.scoreAll(List<String> queries, List<TextSegment> segments)
  │
  ├── POST https://api.cohere.ai/v1/rerank
  │   body: { query: "...", documents: [...], model: "rerank-english-v3.0" }
  ├── 返回: { results: [{ index, relevance_score }] }
  └── relevance_score → Score
```

## Provider 标识

`ModelProvider.COHERE`
