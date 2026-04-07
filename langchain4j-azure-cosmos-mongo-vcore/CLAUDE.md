# langchain4j-azure-cosmos-mongo-vcore 模块

## 模块职责
**Azure Cosmos DB MongoDB vCore 向量搜索集成** — 对接 Azure Cosmos DB 的 MongoDB vCore API，支持向量索引和搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `AzureCosmosDbMongoVCoreEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `MappingUtils` | 数据映射工具 |
| `VectorIndexType` | 枚举: VECTOR_IVF, VECTOR_HNSW |

## 核心执行流程

### 搜索嵌入

```
AzureCosmosDbMongoVCoreEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── $search 聚合管道:
  │   { $search: { cosmosSearch: { vector: [...], path: "embedding", k: 10 } } }
  ├── 返回匹配文档
  └── score 计算: 余弦相似度
```

### 索引类型

| 类型 | 参数 | 说明 |
|------|------|------|
| `VECTOR_HNSW` | m, efConstruction, efSearch | 分层可导航小世界图 |
| `VECTOR_IVF` | numLists | 倒排文件索引 |

## 注意事项

- 仅支持余弦相似度 (SimilarityMetric.COS)
- 不支持 `removeAll()` / `removeAll(filter)` / `removeAll(ids)` 操作

## Provider 标识

`ModelProvider.AZURE_COSMOS_MONGO_VCORE`
