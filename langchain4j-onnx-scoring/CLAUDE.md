# langchain4j-onnx-scoring 模块

## 模块职责
**本地 ONNX 评分模型** — 使用 ONNX Runtime 运行 BERT 交叉编码器模型进行本地文档评分/重排序，无需外部 API。

## 核心类

| 类 | 职责 |
|----|------|
| `OnnxScoringModel` | 实现 ScoringModel 接口 |
| `OnnxScoringBertCrossEncoder` | BERT 交叉编码器 ONNX 推理 |
| `AbstractInProcessScoringModel` | 抽象基类 |

## 核心执行流程

```
OnnxScoringModel.scoreAll(queries, segments)
  │
  ├── 加载 ONNX 模型文件 (从 classpath 或文件系统)
  ├── 对每个 query-document 对:
  │   ├── Tokenizer: [CLS] query [SEP] document [SEP]
  │   ├── ONNX Runtime 推理 → relevance score
  │   └── 可选: normalization
  └── 返回 List<Score>
```

## 配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `maxLength` | 510 | 最大 token 长度 |
| `normalize` | false | 是否归一化分数 |

## 特点

- 完全本地运行，无网络请求
- 适合隐私敏感场景
- 内存中运行 ONNX 模型
