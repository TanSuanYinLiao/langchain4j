# langchain4j-elasticsearch 模块

## 模块职责
**Elasticsearch 向量数据库集成** — 对接 Elasticsearch，支持多种搜索策略（KNN、暴力搜索、全文搜索、混合搜索）。

## 核心类

| 类 | 职责 |
|----|------|
| `AbstractElasticsearchEmbeddingStore` | 抽象基类，实现 `EmbeddingStore<TextSegment>` |
| `ElasticsearchEmbeddingStore` | 具体实现 |
| `ElasticsearchConfiguration` 及其子类 | 不同搜索策略的配置 |
| `ElasticsearchMetadataFilterMapper` | Filter → ES query DSL 转换 |
| `ElasticsearchContentRetriever` | 扩展抽象基类，支持 RAG |

## 搜索策略

| 策略 | 类 | 说明 |
|------|----|------|
| **KNN** | `ElasticsearchConfigurationKnn` | 近似 KNN 搜索，使用 `knn` 查询 |
| **Script** | `ElasticsearchConfigurationScript` | 暴力搜索，使用 cosineSimilarity 脚本 |
| **Full Text** | `ElasticsearchConfigurationFullText` | 基于 BM25 的全文搜索 |
| **Hybrid** | `ElasticsearchConfigurationHybrid` | KNN + 全文搜索结合 |

## 核心执行流程

### 搜索嵌入

```
ElasticsearchEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 策略选择 (基于 ElasticsearchConfiguration)
  │
  ├── KNN 策略:
  │   ├── { "knn": { "field": "embedding", "query_vector": [...], "k": 10, "num_candidates": 100 } }
  │   └── + filter 条件 (如有)
  │
  ├── Script 策略:
  │   ├── { "script_score": { "query": { "match_all": {} }, "script": { "source": "cosineSimilarity(...) + 1.0" } } }
  │   └── 暴力扫描所有文档
  │
  ├── Hybrid 策略:
  │   ├── KNN 查询 + match 查询
  │   └── 使用 rank fusion 或自定义合并
  │
  └── 返回: EmbeddingSearchResult
```

### 删除操作

```
removeAll()       → drop index
removeAll(filter) → deleteByQuery with ElasticsearchMetadataFilterMapper
removeAll(ids)    → bulk delete
```

## Provider 标识

`ModelProvider.ELASTICSEARCH`
