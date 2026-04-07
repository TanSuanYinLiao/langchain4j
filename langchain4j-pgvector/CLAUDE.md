# langchain4j-pgvector 模块

## 模块职责
**PgVector 向量数据库集成** — 基于 PostgreSQL pgvector 扩展的向量存储，支持向量搜索和混合搜索。

## 核心类

| 类 | 职责 |
|----|------|
| `PgVectorEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `MetadataHandler` / `JSONBMetadataHandler` / `ColumnsMetadataHandler` | 元数据存储策略 |
| `PgVectorFilterMapper` / `JSONFilterMapper` / `ColumnFilterMapper` | Filter → SQL WHERE 子句转换 |
| `MetadataStorageConfig` | 元数据存储配置 |

## 核心执行流程

### 添加嵌入

```
PgVectorEmbeddingStore.addAll(List<String> ids, List<Embedding> embeddings, List<TextSegment> segments)
  │
  ├── INSERT INTO {table} (embedding_id, embedding, text, metadata)
  │   VALUES (?, ?::vector, ?, ?::jsonb)
  │   ON CONFLICT (embedding_id) DO UPDATE SET ...
  └── 使用 JDBC PreparedStatement 批量插入
```

### 向量搜索

```
PgVectorEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── SearchMode.VECTOR (默认):
  │   ├── SELECT *, (2 - (embedding <=> ?::vector)) / 2 AS score
  │   ├── FROM {table}
  │   ├── WHERE metadata @> ?::jsonb  ← Filter 条件 (如有)
  │   ├── ORDER BY embedding <=> ?::vector  ← 余弦距离排序
  │   └── LIMIT ?
  │
  └── SearchMode.HYBRID:
      ├── 向量搜索 + 全文搜索
      ├── 使用 Reciprocal Rank Fusion (RRF) 合并结果
      ├── rrfK 参数控制融合权重 (默认 60)
      └── 两轮查询后合并排序
```

### 元数据存储模式

| 模式 | 说明 | Filter 方式 |
|------|------|-------------|
| `COMBINED_JSONB` | 所有元数据存入单个 JSONB 列 | `metadata @> '{"key": "value"}'` |
| `COLUMN_PER_KEY` | 每个元数据键一个独立列 | `key = 'value'` |

### 删除操作

```
removeAll()        → TRUNCATE TABLE
removeAll(filter)  → DELETE WHERE {filter clause}
removeAll(ids)     → DELETE WHERE embedding_id = ANY(?)
```

## 构建器模式

两种 Builder:
- `PgVectorEmbeddingStoreBuilder`: 通过 host/port/database 直连
- `DatasourceBuilder`: 通过已有 `DataSource` 连接

## Provider 标识

`ModelProvider.PGVECTOR`
