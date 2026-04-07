# langchain4j-cassandra 模块

## 模块职责
**Cassandra 向量数据库集成** — 对接 Apache Cassandra 和 DataStax Astra DB，通过 CassIO 库进行向量操作。

## 核心类

| 类 | 职责 |
|----|------|
| `CassandraEmbeddingStore` | 通用 Cassandra 实现 |
| `AstraDbEmbeddingStore` | DataStax Astra DB 专用实现 |
| `CassandraChatMemoryStore` | 基于 Cassandra 的聊天记忆存储 |

## 核心执行流程

### 添加嵌入

```
CassandraEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── MetadataVectorTable (CassIO) 处理:
  │   ├── 创建表 (如不存在)
  │   ├── INSERT INTO table (row_id, vector, body_blob, metadata_s) VALUES (...)
  │   └── 使用 DataStax Java Driver 的 CqlSession 执行
  └── 批量插入
```

### 搜索嵌入

```
CassandraEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── MetadataVectorTable.similaritySearch(vector, topK, metric)
  │   └── ORDER BY vector ANN OF ? LIMIT ? (Cassandra Vector Search)
  └── score = RelevanceScore.fromCosineSimilarity()
```

## 注意事项

- `removeAll()` 不支持 (继承默认 UnsupportedOperationException)
- `removeAll(filter)` 不支持
- Filter 在 `search()` 中不支持 (抛出 UnsupportedOperationException)
- 需要 Cassandra 5.0+ 或 Astra DB 支持向量搜索

## Provider 标识

`ModelProvider.CASSANDRA`
