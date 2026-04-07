# langchain4j-opensearch 模块

## 模块职责
**OpenSearch 向量搜索集成** — 对接 OpenSearch，使用 k-NN 搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `OpenSearchEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `OpenSearchMetadataFilterMapper` | Filter → OpenSearch query DSL 转换 |

## 核心执行流程

### 搜索嵌入

```
OpenSearchEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── k-NN 查询:
  │   { "query": { "script_score": { "query": { "match_all": {} },
  │     "script": { "source": "cosineSimilarity(params.query_vector, 'embedding') + 1.0" } } } }
  ├── + filter 条件 (如有) → OpenSearchMetadataFilterMapper
  ├── score: 0.5 * (cosineSimilarity + 1.0) → 归一化到 [0, 1]
  └── 返回结果
```

### 删除操作

```
removeAll()       → drop index
removeAll(filter) → deleteByQuery with filter
removeAll(ids)    → bulk delete
```

### 连接方式

| 模式 | 说明 |
|------|------|
| 本地 | serverUrl + 用户名/密码 |
| AWS | serverUrl + AwsSdk2TransportOptions (IAM 认证) |
| 自定义 | 直接传入 OpenSearchClient |

## Provider 标识

`ModelProvider.OPENSEARCH`
