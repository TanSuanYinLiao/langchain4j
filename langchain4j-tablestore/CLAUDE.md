# langchain4j-tablestore 模块

## 模块职责
**阿里云 Tablestore 向量搜索集成** — 对接阿里云表格存储 (Tablestore)，支持向量搜索和全文搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `TablestoreEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `TablestoreMetadataFilterMapper` | Filter → Tablestore 查询条件转换 |
| `TablestoreChatMemoryStore` | 基于 Tablestore 的聊天记忆存储 |
| `IsTextMatch` / `IsTextMatchPhrase` | 自定义文本匹配 Filter |

## 核心执行流程

### 初始化

```
TablestoreEmbeddingStore.init()  ← 必须先调用
  ├── 创建表 (如不存在)
  └── 创建搜索索引 (含向量字段和元数据字段)
```

### 搜索嵌入

```
TablestoreEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── KnnVectorQuery: { fieldName: "embedding", topK: 10, float32QueryVector: [...] }
  ├── filter → TablestoreMetadataFilterMapper
  └── 返回: SearchResponse → EmbeddingSearchResult
```

### 删除操作

```
removeAll()       → GetRange 遍历所有数据，逐行删除
removeAll(filter) → 遍历所有数据，filter 匹配后删除
removeAll(ids)    → 逐个 ID 删除
```

## Provider 标识

`ModelProvider.TABLESTORE`
