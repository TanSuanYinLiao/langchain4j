# embeddings 模块组

## 模块职责
**本地 ONNX 嵌入模型** — 提供基于 ONNX Runtime 的进程内嵌入模型，无需外部 API。

## 统一架构

所有嵌入子模块依赖基础模块 `langchain4j-embeddings`:

```
langchain4j-embeddings (基础)
  ├── OnnxEmbeddingModel — 通用 ONNX 嵌入模型
  ├── AbstractInProcessEmbeddingModel — 抽象基类
  ├── OnnxBertBiEncoder — BERT 双编码器
  ├── HuggingFaceTokenCountEstimator — Token 计数器
  └── PoolingMode — 池化模式 (CLS, MEAN)

langchain4j-embeddings-all-MiniLM-L6-v2
  └── AllMiniLmL6V2EmbeddingModel (标准)
  └── AllMiniLmL6V2QuantizedEmbeddingModel (量化)

langchain4j-embeddings-bge-small-en
  └── BgeSmallEnEmbeddingModel

langchain4j-embeddings-bge-small-en-v15
  └── BgeSmallEnV15EmbeddingModel (标准)
  └── BgeSmallEnV15QuantizedEmbeddingModel (量化)

langchain4j-embeddings-bge-small-zh-v15
  └── BgeSmallZhV15EmbeddingModel (标准)
  └── BgeSmallZhV15QuantizedEmbeddingModel (量化)

langchain4j-embeddings-e5-small-v2
  └── E5SmallV2EmbeddingModel (标准)
  └── E5SmallV2QuantizedEmbeddingModel (量化)
```

## 核心执行流程

```
XxxEmbeddingModel.embedAll(List<TextSegment> segments)
  │
  ├── 加载 ONNX 模型 (从 classpath: /model.onnx)
  ├── 对每个 segment:
  │   ├── Tokenizer: 文本 → token IDs
  │   ├── ONNX Runtime 推理 → 隐藏层输出
  │   ├── Pooling: CLS/MEAN 池化 → 固定维度向量
  │   └── 可选: L2 归一化
  └── 返回 List<Embedding>
```

## 模型变体

| 模型 | 维度 | 语言 | 量化版 |
|------|------|------|--------|
| all-MiniLM-L6-v2 | 384 | 英文 | 有 |
| BGE-small-en | 384 | 英文 | 无 |
| BGE-small-en-v15 | 384 | 英文 | 有 |
| BGE-small-zh-v15 | 384 | 中文 | 有 |
| E5-small-v2 | 384 | 英文 | 有 |

## 特点

- 每个子模块包含一个 `*Factory` 类用于懒加载和缓存模型实例
- 量化版 (`-q`) 使用 int8 权重，内存更小、推理更快
- 完全进程内运行，无需网络请求
