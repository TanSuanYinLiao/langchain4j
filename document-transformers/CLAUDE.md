# document-transformers 模块组

## 模块职责
**文档转换器集合** — 将文档内容从一种格式转换为另一种格式 (如 HTML → 纯文本)。

## 子模块

| 子模块 | 功能 | 关键类 |
|--------|------|--------|
| `langchain4j-document-transformer-jsoup` | HTML → 纯文本 | `HtmlToTextDocumentTransformer` |

## 核心执行流程

### HTML 转文本

```
HtmlToTextDocumentTransformer.transform(Document document)
  │
  ├── Jsoup.parse(html).traverse(visitor)
  │   ├── custom NodeVisitor 遍历 DOM 树
  │   ├── 文本节点 → 追加到输出
  │   ├── 如果 includeLinks → 追加链接 URL
  │   └── 忽略 script/style 等标签
  │
  ├── 可选: cssSelector → 只提取匹配元素的文本
  │   element.select(cssSelector).eachText()
  │
  ├── 可选: metadataCssSelectors → 从 CSS 选择器提取元数据
  │   如: { "title": "h1", "date": ".publish-date" }
  │
  └── 返回 Document(纯文本, metadata)
```

## 配置参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `cssSelector` | String | CSS 选择器，只提取匹配元素 |
| `metadataCssSelectors` | Map<String, String> | 从 CSS 选择器提取元数据键值对 |
| `includeLinks` | boolean | 是否在文本中内联链接 URL |

## 使用场景

- 网页爬取后提取可读内容
- 从 HTML 邮件中提取纯文本
- RAG 管道中清洗 HTML 文档
