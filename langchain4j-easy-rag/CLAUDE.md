# langchain4j-easy-rag 模块

## 模块职责
**简易 RAG 配置** — 提供开箱即用的 RAG (检索增强生成) 配置，简化文档分块设置。

## 核心类

| 类 | 职责 |
|----|------|
| `RecursiveDocumentSplitterFactory` | SPI 工厂：创建递归文本分块器 |

## 核心执行流程

```
EmbeddingStoreIngestor.ingest(document, embeddingStore)
  │
  ├── SPI 加载 DocumentSplitterFactory
  │   └── 找到 RecursiveDocumentSplitterFactory
  ├── 创建 RecursiveDocumentSplitter:
  │   ├── chunkSize: 300 tokens
  │   ├── overlap: 30 tokens
  │   └── tokenizer: HuggingFace ONNX tokenizer
  ├── splitter.split(document) → List<TextSegment>
  ├── embeddingModel.embedAll(segments) → List<Embedding>
  └── embeddingStore.addAll(embeddings, segments)
```

## 完整使用流程

```java
// 1. 加载文档
Document document = FileSystemDocumentLoader.loadDocument("docs/guide.pdf");

// 2. 摄入到向量存储 (自动使用 SPI 加载的分块器)
EmbeddingStoreIngestor.ingest(document, embeddingStore);

// 3. 创建 AI 服务
Assistant assistant = AiServices.builder(Assistant.class)
    .chatModel(model)
    .contentRetriever(EmbeddingStoreContentRetriever.from(embeddingStore))
    .build();

// 4. 提问
String answer = assistant.chat("How does RAG work?");
```

## 默认配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| chunkSize | 300 | 每块的 token 数 |
| overlap | 30 | 相邻块的重叠 token 数 |
| tokenizer | HuggingFace ONNX | 用于计算 token 数 |

## SPI 注册

```
META-INF/services/dev.langchain4j.data.document.DocumentSplitterFactory
  → dev.langchain4j.data.document.splitter.recursive.RecursiveDocumentSplitterFactory
```
