# langchain4j-couchbase 模块

## 模块职责
**Couchbase 向量搜索集成** — 对接 Couchbase FTS (全文搜索) 索引，支持向量搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `CouchbaseEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |

## 核心执行流程

### 搜索嵌入

```
CouchbaseEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建 FTS 查询: knn 操作符 + 向量
  ├── cluster.searchQuery(indexName, searchQuery)
  └── 返回: SearchRows → 转换为 EmbeddingMatch
```

### 删除操作

```
removeAll()    → N1QL: DELETE FROM bucket.scope.collection WHERE true
removeAll(ids) → collection.remove(id) 逐个删除
removeAll(filter) → 不支持
```

## Provider 标识

`ModelProvider.COUCHBASE`
