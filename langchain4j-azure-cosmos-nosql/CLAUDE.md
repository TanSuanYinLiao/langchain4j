# langchain4j-azure-cosmos-nosql 模块

## 模块职责
**Azure Cosmos DB NoSQL 向量搜索集成** — 对接 Azure Cosmos DB NoSQL API，支持向量搜索、全文搜索和混合搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `AbstractAzureCosmosDBNoSqlEmbeddingStore` | 向量存储实现 |
| `AzureCosmosDBNoSqlContentRetriever` | RAG 内容检索器 |
| `AzureCosmosDBNoSqlMemoryStore` | 聊天记忆存储 |
| `AzureCosmosDBNoSqlFilterMapper` | Filter → Cosmos DB SQL 转换 |

## 查询类型

| 类型 | 说明 |
|------|------|
| `VECTOR` | 纯向量搜索: `VectorDistance(c.embedding, @embedding)` |
| `FULL_TEXT_SEARCH` | 全文搜索: `FullTextContains(c.text, @query)` |
| `FULL_TEXT_RANKING` | 全文排序: `FullTextScore(c.text, @query)` |
| `HYBRID` | 混合搜索: `RANK RRF(FullTextScore(...), VectorDistance(...))` |

## 核心执行流程

### 搜索嵌入

```
AbstractAzureCosmosDBNoSqlEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建 Cosmos SQL 查询:
  │   SELECT TOP 10 c.id, c.text, c.metadata,
  │          VectorDistance(c.embedding, @embedding) AS score
  │   FROM c WHERE {filter}
  │   ORDER BY VectorDistance(c.embedding, @embedding)
  ├── cosmosAsyncClient.queryItems(querySpec, requestOptions)
  └── 返回结果
```

### 特殊功能

- **分区键**: 支持分层分区键 (MULTI_HASH)
- **全文搜索**: FullTextContains, FullTextContainsAny, FullTextContainsAll
- **容器管理**: `deleteContainer()` 方法

## Provider 标识

`ModelProvider.AZURE_COSMOS_NOSQL`
