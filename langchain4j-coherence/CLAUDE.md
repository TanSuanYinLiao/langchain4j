# langchain4j-coherence 模块

## 模块职责
**Oracle Coherence 向量搜索集成** — 对接 Oracle Coherence 分布式缓存，支持向量相似度搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `CoherenceEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `CoherenceMetadataFilterMapper` | Filter → Coherence 查询条件转换 |
| `CoherenceChatMemoryStore` | 基于 Coherence 的聊天记忆存储 |

## 核心执行流程

### 搜索嵌入

```
CoherenceEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── SimilaritySearch aggregator:
  │   documentChunks.aggregate(SimilaritySearch.of(queryVector, topK))
  ├── 如果有 filter → CoherenceMetadataFilterMapper + CacheProcessors
  └── normalizeEmbeddings: 可选归一化处理
```

### 删除操作

```
removeAll()       → documentChunks.truncate()
removeAll(filter) → CacheProcessors.removeBlind() with filter
removeAll(ids)    → 逐个 remove
```

## Provider 标识

`ModelProvider.COHERENCE`
