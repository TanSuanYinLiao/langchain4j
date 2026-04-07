# langchain4j-azure-ai-search 模块

## 模块职责
**Azure AI Search 向量数据库集成** — 对接 Azure AI Search (原 Azure Cognitive Search)，支持向量搜索和语义搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `AbstractAzureAiSearchEmbeddingStore` | 抽象基类 |
| `AzureAiSearchEmbeddingStore` | 具体实现 |
| `AzureAiSearchContentRetriever` | RAG 内容检索器，支持多种查询类型 |
| `AzureAiSearchFilterMapper` / `DefaultAzureAiSearchFilterMapper` | Filter → OData 过滤表达式 |

## 核心执行流程

### 添加嵌入

```
AzureAiSearchEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── 构建 SearchDocument 列表: { id, embedding (float[]), text, metadata }
  ├── searchClient.uploadDocuments(documents)
  │   └── Azure SDK 内部: POST /indexes/{index}/docs/index
  └── 使用 AzureKeyCredential 或 TokenCredential 认证
```

### 搜索嵌入

```
AzureAiSearchEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建向量查询:
  │   ├── VectorizedQuery: { vector: [...], kNearestNeighbors: 10, fields: "embedding" }
  │   └── Filter: OData 表达式 (如有)
  ├── searchClient.search(null, searchOptions)
  │   └── Azure SDK 内部: POST /indexes/{index}/docs/search
  ├── 返回: SearchResults (包含向量距离)
  └── 分数 = 向量相似度
```

### AzureAiSearchContentRetriever 查询类型

| 类型 | 说明 |
|------|------|
| `VECTOR` | 纯向量搜索 |
| `FULL_TEXT` | 纯全文搜索 |
| `HYBRID` | 向量 + 全文混合搜索 |
| `SEMANTIC` | 语义搜索 (Azure 特有) |

## Provider 标识

`ModelProvider.AZURE_AI_SEARCH`
