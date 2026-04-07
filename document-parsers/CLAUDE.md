# document-parsers 模块组

## 模块职责
**文档解析器集合** — 将不同格式的文档 (PDF, Word, Markdown 等) 解析为 Document 对象。

## 统一接口

```java
public interface DocumentParser {
    Document parse(InputStream inputStream);
}
```

## 子模块列表

| 子模块 | 支持格式 | 底层库 | 关键类 |
|--------|---------|--------|--------|
| `langchain4j-document-parser-apache-pdfbox` | PDF | Apache PDFBox | `ApachePdfBoxDocumentParser` |
| `langchain4j-document-parser-apache-poi` | doc, docx, ppt, pptx, xls, xlsx | Apache POI | `ApachePoiDocumentParser` |
| `langchain4j-document-parser-apache-tika` | 1000+ 格式 | Apache Tika | `ApacheTikaDocumentParser` |
| `langchain4j-document-parser-markdown` | Markdown | CommonMark | `MarkdownDocumentParser` |
| `langchain4j-document-parser-yaml` | YAML | SnakeYAML | `YamlDocumentParser` |

## 核心执行流程

### Apache PDFBox 解析

```
ApachePdfBoxDocumentParser.parse(InputStream is)
  │
  ├── PDDocument.load(is)
  ├── PDFTextStripper.getText(document) → 提取文本
  ├── 可选: includeMetadata → 提取 PDF 元数据 (author, title, etc.)
  └── 返回 Document(text, metadata)
```

### Apache POI 解析

```
ApachePoiDocumentParser.parse(InputStream is)
  │
  ├── ExtractorFactory.createExtractor(is)  ← 自动检测格式
  ├── extractor.getText() → 提取文本
  └── 返回 Document(text)
```

### Apache Tika 解析

```
ApacheTikaDocumentParser.parse(InputStream is)
  │
  ├── AutoDetectParser (Supplier<Parser> 避免线程安全问题)
  ├── BodyContentHandler(handler)
  ├── parser.parse(is, handler, metadata, context)
  ├── 可选: includeMetadata → 将 Tika metadata 附加到 Document
  └── 返回 Document(text, metadata)
```

### Markdown 解析

```
MarkdownDocumentParser.parse(InputStream is)
  │
  ├── Parser.parse(markdown) → Node 树
  ├── TextContentRenderer.render(node) → 纯文本
  └── 返回 Document(text)
```

### YAML 解析

```
YamlDocumentParser.parse(InputStream is)
  │
  ├── new Yaml().load(is) → Map/List
  ├── Objects.toString(yamlObject) → 字符串表示
  └── 返回 Document(text)
```

## 使用场景

| 场景 | 推荐解析器 |
|------|-----------|
| 只需 PDF | `ApachePdfBoxDocumentParser` |
| 只需 Office 文档 | `ApachePoiDocumentParser` |
| 混合格式 (推荐) | `ApacheTikaDocumentParser` (最通用) |
| Markdown 文件 | `MarkdownDocumentParser` |
| 配置文件 | `YamlDocumentParser` |
