# langchain4j-pinecone 模块

## 模块职责
**Pinecone 向量数据库集成** — 对接 Pinecone 托管向量数据库服务。

## 核心类

| 类 | 职责 |
|----|------|
| `PineconeEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `PineconeHelper` | 辅助工具类 |
| `PineconeMetadataFilterMapper` | Filter → Pinecone metadata filter 转换 |
| `PineconeIndexConfig` / `PineconeServerlessIndexConfig` / `PineconePodIndexConfig` | 索引配置 |

## 核心执行流程

### 添加嵌入

```
PineconeEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── 构建 Vector 列表: { id, values (float[]), metadata }
  ├── index.upsert(vectors, nameSpace)
  │   └── Pinecone SDK 内部: POST /vectors/upsert
  └── nameSpace 默认为 "default"
```

### 搜索嵌入

```
PineconeEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建查询: vector (float[]), topK, filter (如有)
  ├── index.query(queryRequest)
  │   └── Pinecone SDK 内部: POST /query
  │       body: { vector: [...], topK: 10, filter: {...}, includeMetadata: true }
  ├── 返回: { matches: [{ id, score, metadata }] }
  └── score = RelevanceScore.fromCosineSimilarity(score)
```

### 删除操作

```
removeAll()    → index.deleteAll(nameSpace)
removeAll(ids) → index.deleteByIds(ids, nameSpace)
removeAll(filter) → 不支持 (继承默认 UnsupportedOperationException)
```

## 注意事项

- Pinecone 将所有数字存储为 float，会损坏 Integer/Long 类型的元数据值
- 支持 Serverless 和 Pod 两种索引类型
- nameSpace 提供数据隔离能力

## Provider 标识

`ModelProvider.PINECONE`
