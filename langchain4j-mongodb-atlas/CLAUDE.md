# langchain4j-mongodb-atlas 模块

## 模块职责
**MongoDB Atlas 向量搜索集成** — 对接 MongoDB Atlas Vector Search，使用 `$vectorSearch` 聚合管道。

## 核心类

| 类 | 职责 |
|----|------|
| `MongoDbEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `MongoDbMetadataFilterMapper` | Filter → MongoDB 查询条件转换 |
| `IndexMapping` | 向量索引配置 |
| `MongoDbDocument` / `MongoDbMatchedDocument` | 数据模型 |

## 核心执行流程

### 搜索嵌入

```
MongoDbEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── 构建聚合管道:
  │   ├── $vectorSearch 阶段:
  │   │   { $vectorSearch: { index: "...", path: "embedding", queryVector: [...],
  │   │     numCandidates: 100, limit: 10 } }
  │   ├── $set 阶段: { $set: { score: { $meta: "vectorSearchScore" } } }
  │   ├── $match 阶段: filter 条件 (如有) → MongoDbMetadataFilterMapper
  │   └── $project 阶段: 选择返回字段
  ├── mongoCollection.aggregate(pipeline)
  └── 按 score 排序返回
```

### 删除操作

```
removeAll()       → 删除所有文档
removeAll(ids)    → deleteMany({ _id: { $in: ids } })
removeAll(filter) → 不支持 (继承默认)
```

## 配置参数

| 参数 | 说明 |
|------|------|
| `indexName` | Atlas Vector Search 索引名称 |
| `maxResultRatio` | 候选结果比例 |
| `globalPrefilter` | 全局预过滤条件 |
| `createIndex` | 是否自动创建搜索索引 |

## Provider 标识

`ModelProvider.MONGODB_ATLAS`
