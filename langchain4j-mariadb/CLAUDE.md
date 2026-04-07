# langchain4j-mariadb 模块

## 模块职责
**MariaDB 向量搜索集成** — 基于 MariaDB 的向量存储，使用 JDBC 直接操作。

## 核心类

| 类 | 职责 |
|----|------|
| `MariaDbEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `MetadataHandler` / `JSONFilterMapper` / `ColumnFilterMapper` | 元数据存储和过滤 |
| `MariaDbFilterMapper` | Filter → SQL WHERE 子句转换 |
| `MariaDbValidator` | 标识符验证 |
| `MariaDBDistanceType` | 距离函数类型 |

## 核心执行流程

### 搜索嵌入

```
MariaDbEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── SELECT *, (2 - vec_distance_cosine(embedding, ?)) / 2 AS score
  ├── FROM {table}
  ├── WHERE {filter clause}  ← metadataHandler.whereClause(filter)
  ├── ORDER BY vec_distance_cosine(embedding, ?)
  └── LIMIT ?
```

### 删除操作

```
removeAll()       → TRUNCATE TABLE
removeAll(filter) → DELETE WHERE {filter clause}
removeAll(ids)    → DELETE WHERE id IN (?, ?, ...)
```

### 元数据存储

与 PGVector 类似，支持两种模式:
- `COMBINED_JSON`: 所有元数据存入单个 JSON 列
- `COLUMN_PER_KEY`: 每个元数据键一个独立列

## Provider 标识

`ModelProvider.MARIADB`
