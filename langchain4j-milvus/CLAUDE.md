# langchain4j-milvus 模块

## 模块职责
**Milvus 向量数据库集成** — 对接 Milvus 向量数据库，通过 gRPC 协议进行高性能向量操作。

## 核心类

| 类 | 职责 |
|----|------|
| `MilvusEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `CollectionOperationsExecutor` | 封装 Milvus collection 操作 |
| `CollectionRequestBuilder` | 构建 Milvus 请求 |
| `Mapper` | 数据映射 |
| `MilvusMetadataFilterMapper` | Filter → Milvus 表达式转换 |

## 核心执行流程

### 添加嵌入

```
MilvusEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── 构建 InsertParam: collectionName, fields (id, vector, text, metadata)
  ├── collectionOperationsExecutor.insert(insertParam)
  │   └── gRPC: MilvusServiceClient.insert()
  └── 可选: autoFlushOnInsert → 自动刷新到磁盘
```

### 搜索嵌入

```
MilvusEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建 SearchParam:
  │   ├── collectionName, vectorFieldName
  │   ├── topK, metricType (默认 COSINE)
  │   ├── vectors: [queryEmbedding]
  │   └── expr: metadata filter (如有) → MilvusMetadataFilterMapper
  ├── collectionOperationsExecutor.search(searchParam)
  │   └── gRPC: MilvusServiceClient.search()
  ├── 返回: SearchResults (ids, distances, fields)
  ├── 可选: retrieveEmbeddingsOnSearch → 额外查询获取嵌入向量
  └── 距离转分数: RelevanceScore.fromCosineSimilarity()
```

## 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `metricType` | COSINE | 距离度量类型 |
| `indexType` | FLAT | 索引类型 (FLAT, IVF_FLAT, HNSW 等) |
| `consistencyLevel` | EVENTUALLY | 一致性级别 |
| `retrieveEmbeddingsOnSearch` | false | 搜索时是否返回嵌入向量 |
| `autoFlushOnInsert` | false | 插入后自动刷新 |

## Provider 标识

`ModelProvider.MILVUS`
