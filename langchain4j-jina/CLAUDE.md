# langchain4j-jina 模块

## 模块职责
**Jina AI 集成** — 对接 Jina AI API，支持文本嵌入和文档重排序。

## 核心类

| 类 | 职责 |
|----|------|
| `JinaEmbeddingModel` | 文本嵌入 |
| `JinaScoringModel` | 文档重排序 |
| `JinaClient` / `JinaApi` | HTTP 客户端 |

## 核心执行流程

### 嵌入

```
JinaEmbeddingModel.embedAll(List<TextSegment> segments)
  │
  ├── POST https://api.jina.ai/v1/embeddings
  │   body: { input: [...], model: "jina-embeddings-v3" }
  └── 返回: { data: [{ embedding: [...] }] }
```

### 重排序

```
JinaScoringModel.scoreAll(queries, segments)
  │
  ├── POST https://api.jina.ai/v1/rerank
  │   body: { query: "...", documents: [...], model: "jina-reranker-v2" }
  └── 返回: { results: [{ relevance_score }] }
```

## Provider 标识

`ModelProvider.JINA`
