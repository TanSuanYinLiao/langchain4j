# langchain4j-vespa 模块

## 模块职责
**Vespa 向量搜索引擎集成** — 对接 Vespa 搜索引擎，支持向量搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `VespaEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `VespaClient` / `VespaApi` | HTTP 客户端 (OkHttp + Retrofit2) |

## 核心执行流程

### 搜索嵌入

```
VespaEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── YQL 查询:
  │   SELECT * FROM {documentType}
  │   WHERE [{targetHits}]nearestNeighbor(embedding, query_embedding)
  │   AND userInput(@query)
  │   ranking={rankProfile}
  ├── vespaClient.query(yql, ranking, query_embedding)
  └── 返回: QueryResponse → EmbeddingSearchResult
```

### 删除操作

```
removeAll() → Vespa batch delete API (deleteAll namespace documentType clusterName)
removeAll(filter) → 不支持
removeAll(ids) → 不支持
```

## 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `namespace` | "namespace" | 文档命名空间 |
| `rankProfile` | "langchain4j_relevance_score" | 排序配置文件 |
| `targetHits` | 10 | 近似搜索目标命中数 |
| `avoidDups` | false | 使用 text hash 作为 ID 去重 |

## 注意事项

- 不支持元数据存储
- 支持 SSL 证书认证 (Vespa Cloud)
- 使用 Vespa Feed Client 进行文档操作

## Provider 标识

`ModelProvider.VESPA`
