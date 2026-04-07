# langchain4j-weaviate 模块

## 模块职责
**Weaviate 向量数据库集成** — 对接 Weaviate 向量搜索引擎，支持 gRPC 高性能插入。

## 核心类

| 类 | 职责 |
|----|------|
| `WeaviateEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |

## 核心执行流程

### 添加嵌入

```
WeaviateEmbeddingStore.addAll(ids, embeddings, segments)
  │
  ├── 构建 WeaviateObject 列表
  │   ├── 如果 avoidDups=true → 对 text 生成 hash 作为 ID (去重)
  │   ├── properties: { text, metadata (selected keys) }
  │   └── vector: float[]
  ├── weaviateClient.batch().objectsBatcher().withObjects(objects).run()
  │   └── 批量插入，可选 gRPC (useGrpcForInserts)
  └── consistencyLevel: ONE / QUORUM (default) / ALL
```

### 搜索嵌入

```
WeaviateEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── GraphQL nearVector 查询:
  │   { Get { ClassName(nearVector: { vector: [...], distance: 0.5 }) { text metadata _additional { distance } } } }
  ├── 执行查询 → 返回结果
  └── distance → RelevanceScore 转换
```

## 配置参数

| 参数 | 说明 |
|------|------|
| `avoidDups` | 使用 text hash 作为 ID，避免重复插入 |
| `consistencyLevel` | 写入一致性级别 |
| `useGrpcForInserts` | 使用 gRPC 协议提升插入性能 |
| `metadataKeys` | 选择性持久化的元数据键 |
| `textFieldName` / `metadataFieldName` | 自定义字段名 |

## Provider 标识

`ModelProvider.WEAVIATE`
