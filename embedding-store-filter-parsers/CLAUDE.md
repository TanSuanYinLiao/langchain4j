# embedding-store-filter-parsers 模块

## 模块职责
**向量存储过滤器解析器** — 将 SQL 风格的过滤条件解析为 LangChain4j Filter 对象，或使用 LLM 从自然语言生成 SQL 过滤条件。

## 核心类

| 类 | 职责 |
|----|------|
| `SqlFilterParser` | 将 SQL WHERE 子句解析为 LangChain4j Filter 对象 |
| `LanguageModelSqlFilterBuilder` | 使用 LLM 从自然语言生成 SQL 过滤表达式 |
| `TableDefinition` / `ColumnDefinition` | 表结构定义，用于 schema 感知生成 |

## 核心执行流程

### SQL → Filter 解析

```
SqlFilterParser.parse("category = 'tech' AND price > 100")
  │
  ├── 解析 SQL WHERE 子句
  ├── 构建 Filter 对象:
  │   Filter.builder()
  │     .and(Equal("category", "tech"), GreaterThan("price", 100))
  │     .build()
  └── 返回 Filter
```

### 自然语言 → SQL Filter

```
LanguageModelSqlFilterBuilder.build("tech articles under $100")
  │
  ├── System prompt: "Generate a SQL WHERE clause for the following query"
  ├── 提供 TableDefinition 上下文 (表名、列名、类型)
  ├── LLM 生成: "category = 'tech' AND price < 100"
  ├── SqlFilterParser.parse(生成的 SQL)
  └── 返回 Filter 对象
```

## 使用场景

- 用于需要自然语言过滤的 RAG 应用
- 与支持 Filter 的向量存储 (PGVector, Elasticsearch 等) 配合使用
- 支持表/列元数据定义，生成 schema 感知的 SQL
