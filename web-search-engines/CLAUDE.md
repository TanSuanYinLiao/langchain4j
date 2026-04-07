# web-search-engines 模块组

## 模块职责
**Web 搜索引擎集成** — 提供多种 Web 搜索引擎实现，用于 Agent 的搜索工具。

## 统一接口

```java
public interface WebSearchEngine {
    WebSearchResults search(WebSearchRequest request);
}
```

## 子模块列表

| 子模块 | 搜索后端 | 关键类 |
|--------|---------|--------|
| `langchain4j-web-search-engine-google-custom` | Google Custom Search JSON API | `GoogleCustomWebSearchEngine` |
| `langchain4j-web-search-engine-searchapi` | SearchAPI.io (Google/Bing 等) | `SearchApiWebSearchEngine` |
| `langchain4j-web-search-engine-tavily` | Tavily AI 搜索 | `TavilyWebSearchEngine` |

## 核心执行流程

```
WebSearchEngine.search(WebSearchRequest request)
  │
  ├── 构建搜索查询: query, maxResults, language
  ├── 调用搜索 API:
  │   ├── Google Custom: GET https://www.googleapis.com/customsearch/v1?key=...&cx=...&q=...
  │   ├── SearchAPI: GET https://www.searchapi.io/api/v1/search?engine=google&q=...
  │   └── Tavily: POST https://api.tavily.com/search { query, max_results, ... }
  ├── 解析响应 → WebSearchResults
  │   └── List<WebSearchResult>: title, url, snippet
  └── 返回结果
```

## Tavily 特有功能

| 功能 | 说明 |
|------|------|
| `includeRawContent` | 返回完整页面内容 |
| `includeAnswer` | 返回 AI 生成的答案 |
| `topic` | 过滤主题 ("general" 或 "news") |
| `days` | 按天数过滤结果 |

## 使用场景

- Agent 搜索工具
- 实时信息获取
- RAG 管道的外部知识源
