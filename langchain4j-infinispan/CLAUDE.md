# langchain4j-infinispan 模块

## 模块职责
**Infinispan 向量搜索集成** — 对接 Infinispan 分布式缓存，使用 IQL (Infinispan Query Language) 进行向量搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `InfinispanEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `InfinispanMetadataFilterMapper` | Filter → IQL 查询条件转换 |
| `LangChainInfinispanItem` / `LangChainMetadata` | 数据模型 |
| `LangChainItemMarshaller` / `LangChainMetadataMarshaller` | ProtoStream 序列化 |
| `LangchainSchemaCreator` | ProtoStream Schema 初始化 |

## 核心执行流程

### 搜索嵌入

```
InfinispanEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── IQL 向量搜索查询:
  │   SELECT * FROM LangChainInfinispanItem
  │   WHERE embedding <-> ? < ?
  │   ORDER BY embedding <-> ?
  │   LIMIT ?
  ├── remoteCache.query(iqlQuery)
  └── 返回结果
```

### 删除操作

```
removeAll()       → remoteCache.clear()
removeAll(filter) → InfinispanMetadataFilterMapper + IQL DELETE
removeAll(ids)    → remoteCache.remove(id) 逐个删除
```

## Provider 标识

`ModelProvider.INFINISPAN`
