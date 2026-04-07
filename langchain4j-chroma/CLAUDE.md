# langchain4j-chroma 模块

## 模块职责
**Chroma 向量数据库集成** — 对接 ChromaDB 向量数据库，支持文档嵌入的存储、检索和删除操作。

## 核心类

| 类 | 职责 |
|----|------|
| `ChromaEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `ChromaClient` / `ChromaClientV1` / `ChromaClientV2` | HTTP 客户端封装 |
| `ChromaApiImpl` / `ChromaApiV2Impl` | Chroma REST API 调用 |
| `ChromaMetadataFilterMapper` | 将 LangChain4j `Filter` 转换为 Chroma 元数据过滤条件 |

## 核心执行流程

### 添加嵌入

```
ChromaEmbeddingStore.addAll(List<Embedding> embeddings, List<TextSegment> segments)
  │
  ├── 生成 UUID 作为每个嵌入的 ID
  ├── 序列化: ids, embeddings (float[]), documents (text), metadatas (metadata map)
  ├── ChromaClient.add(collectionId, addRequest)
  │   └── POST /api/v1/collections/{collectionId}/add (V1)
  │       或 POST /api/v2/tenants/{tenant}/databases/{db}/collections/{id}/add (V2)
  └── JSON body: { ids: [...], embeddings: [[...]], documents: [...], metadatas: [...] }
```

### 搜索嵌入

```
ChromaEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建查询: queryEmbedding (float[]), maxResults, minScore
  ├── 如果有 filter → ChromaMetadataFilterMapper.toChromaFilter(filter)
  │   └── 转换为 Chroma where 条件 (如 {"category": {"$eq": "tech"}})
  ├── ChromaClient.query(collectionId, queryRequest)
  │   └── POST /api/v1/collections/{collectionId}/query
  │       body: { query_embeddings: [[...]], n_results: 10, where: {...} }
  ├── Chroma 返回: { ids: [[...]], distances: [[...]], documents: [[...]], metadatas: [[...]] }
  └── 距离转分数: score = 1 - (distance / 2)  ← 余弦距离转换
      构建 EmbeddingMatch 列表，按分数排序
```

### 删除操作

```
removeAll() → DELETE 并重建 collection (Chroma 无批量删除 API)
removeAll(filter) → POST /delete with where 条件
removeAll(ids) → POST /delete with ids 列表
```

## API 版本差异

| 特性 | V1 | V2 |
|------|----|----|
| Tenant/Database | 无 | 支持多租户 |
| URL 路径 | /api/v1/... | /api/v2/tenants/.../databases/.../collections/... |
| 认证 | API Key header | API Key header |

## Provider 标识

`ModelProvider.CHROMA`（通过 EmbeddingStore 使用）
