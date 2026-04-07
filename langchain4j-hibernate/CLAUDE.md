# langchain4j-hibernate 模块

## 模块职责
**Hibernate ORM 向量搜索集成** — 使用 JPA/Hibernate ORM 存储和搜索向量嵌入，支持多种数据库的距离函数。

## 核心类

| 类 | 职责 |
|----|------|
| `HibernateEmbeddingStore<E>` | 泛型实现 `EmbeddingStore<TextSegment>` |
| `EmbeddingEntity` / `Embedding` | 实体类和嵌入值对象 |
| `@Embedding` / `@EmbeddedText` / `@MetadataAttribute` / `@UnmappedMetadata` | 实体映射注解 |
| `DistanceFunction` | 枚举: 8 种距离函数 |
| `DatabaseKind` | 数据库方言检测 |

## 核心执行流程

### 搜索嵌入

```
HibernateEmbeddingStore.search(EmbeddingSearchRequest request)
  │
  ├── Hibernate Criteria API 构建查询:
  │   SELECT * FROM entity
  │   WHERE {filter → Criteria Predicate}
  │   ORDER BY {distance_function}(embedding, ?) ASC
  │   LIMIT ?
  ├── 支持 8 种距离函数:
  │   COSINE, EUCLIDEAN, EUCLIDEAN_SQUARED, MANHATTAN,
  │   INNER_PRODUCT, NEGATIVE_INNER_PRODUCT, HAMMING, JACCARD
  └── score = RelevanceScore 转换
```

### 构建器模式

| Builder | 场景 |
|---------|------|
| `Builder<E>` | 使用已有 SessionFactory |
| `DynamicBuilder` | 动态创建 SessionFactory |
| `DynamicDatasourceBuilder` | 从 DataSource 动态创建 |

### 删除操作

```
removeAll()       → TRUNCATE 或 DELETE (取决于数据库)
removeAll(filter) → Criteria Delete + filter → Predicate
removeAll(ids)    → Criteria Delete WHERE id IN (?)
```

## 特殊功能

- `query(Restriction<E>)` — 使用 Hibernate 限制条件查询实体
- `addAllEntities(List<E>)` — 直接插入实体对象
- `generateIds()` — 自动生成 ID
- 支持 Hibernate ORM 7.1+

## Provider 标识

`ModelProvider.HIBERNATE`
