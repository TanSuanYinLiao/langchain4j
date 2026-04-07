# langchain4j-oracle 模块

## 模块职责
**Oracle 数据库向量搜索集成** — 基于 Oracle 数据库原生向量类型 (VECTOR_FLOAT32) 的向量存储。

## 核心类

| 类 | 职责 |
|----|------|
| `OracleEmbeddingStore` | 实现 `EmbeddingStore<TextSegment>` 接口 |
| `EmbeddingTable` | 向量表管理 (建表、索引) |
| `IVFIndexBuilder` / `JSONIndexBuilder` | 向量索引构建器 |
| `SQLFilter` / `SQLFilters` | Filter → SQL WHERE 子句转换 |
| `CreateOption` | 枚举: CREATE_IF_NOT_EXISTS, CREATE_OR_REPLACE 等 |

## 核心执行流程

### 搜索嵌入

```
OracleEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── SELECT id, text, metadata,
  │        1 - (VECTOR_DISTANCE(embedding, ?, COSINE) / 2) AS score
  ├── FROM {table}
  ├── WHERE {filter}  ← SQLFilters.create(filter)
  ├── ORDER BY VECTOR_DISTANCE(embedding, ?, COSINE)
  ├── FETCH {FIRST|APPROXIMATE FIRST} ? ROWS ONLY
  └── exactSearch: true → FETCH FIRST (精确搜索)
      exactSearch: false → FETCH APPROXIMATE FIRST (近似搜索, 需要索引)
```

### 添加嵌入

```
OracleEmbeddingStore.add(id, embedding, segment)
  │
  └── MERGE INTO {table} USING dual ON (id = ?)
      WHEN MATCHED THEN UPDATE SET ...
      WHEN NOT MATCHED THEN INSERT ...
```

### 索引管理

| 索引类型 | 说明 |
|---------|------|
| IVF | 倒排文件索引，适合大数据集 |
| JSON | JSON 字段索引，加速元数据过滤 |

### 删除操作

```
removeAll()       → TRUNCATE TABLE
removeAll(filter) → DELETE WHERE {filter clause}
removeAll(ids)    → DELETE WHERE id = ? (batch)
```

## 额外功能

Oracle 模块还包含:
- `OracleDocumentLoader` — 从 Oracle 数据库加载文档
- `OracleDocumentSplitter` — 文档分块
- `OracleEmbeddingModel` — 基于 Oracle 的嵌入模型
- `OracleSummaryLanguageModel` — 基于 Oracle 的摘要语言模型

## Provider 标识

`ModelProvider.ORACLE`
