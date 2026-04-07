# langchain4j-qdrant 模块

## 模块职责
**Qdrant 向量数据库集成** — 对接 Qdrant 向量搜索引擎，通过 gRPC 协议通信。

## 核心类

| 类 | 职责 |
|----|------|
| `QdrantEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `QdrantFilterConverter` | Filter → Qdrant Filter 条件转换 |
| `ValueMapFactory` / `ObjectFactory` | 数据类型转换 |

## 核心执行流程

### 添加嵌入

```
QdrantEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── 构建 PointStruct 列表: { id (UUID), vectors, payload }
  ├── qdrantClient.upsertAsync(collectionName, points)
  │   └── gRPC: QdrantGrpcClient.upsert()
  └── payload: { text: "...", metadata: { key: value } }
```

### 搜索嵌入

```
QdrantEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建 SearchPoints:
  │   ├── collectionName, vector (float[]), limit
  │   ├── withPayload(true), withVectors(false)
  │   └── filter → QdrantFilterConverter.convertExpression(filter)
  ├── qdrantClient.searchAsync(searchPoints)
  │   └── gRPC: QdrantGrpcClient.search()
  ├── 返回: List<ScoredPoint> (id, score, payload)
  └── score = RelevanceScore.fromCosineSimilarity(cosineSimilarity)
```

### 删除操作

```
removeAll()    → qdrantClient.deleteAsync(collectionName, empty filter)
removeAll(ids) → qdrantClient.deleteAsync(collectionName, PointsIdsList)
removeAll(filter) → qdrantClient.deleteAsync(collectionName, converted filter)
```

## Provider 标识

`ModelProvider.QDRANT`
