# langchain4j-core 模块

## 模块职责
整个项目的**接口层/契约层**。定义所有核心抽象接口、数据结构和工具类，零实现依赖。其他所有模块（langchain4j 高层 API、各 provider 模块）都依赖此模块。

## 核心接口一览

| 接口 | 包路径 | 职责 |
|------|--------|------|
| `ChatModel` | `model/chat/ChatModel.java` | 同步聊天模型统一抽象 |
| `StreamingChatModel` | `model/chat/StreamingChatModel.java` | 流式聊天模型统一抽象 |
| `EmbeddingModel` | `model/embedding/EmbeddingModel.java` | 文本向量化接口 |
| `EmbeddingStore<Embedded>` | `store/embedding/EmbeddingStore.java` | 向量数据库统一接口 |
| `ChatMemory` | `memory/ChatMemory.java` | 对话历史管理 |
| `RetrievalAugmentor` | `rag/RetrievalAugmentor.java` | RAG 增强入口 |
| `ContentRetriever` | `rag/content/retriever/ContentRetriever.java` | 内容检索接口 |
| `InputGuardrail` | `guardrail/InputGuardrail.java` | 输入护栏 |
| `OutputGuardrail` | `guardrail/OutputGuardrail.java` | 输出护栏 |
| `ChatMessage` | `model/chat/ChatMessage.java` | 消息基类 |
| `DocumentParser` | `data/document/DocumentParser.java` | 文档解析器接口 |
| `DocumentSplitter` | `data/document/splitter/DocumentSplitter.java` | 文档分块接口 |
| `WebSearchEngine` | `model/web/search/WebSearchEngine.java` | 网页搜索接口 |
| `ModerationModel` | `model/moderation/ModerationModel.java` | 内容审核接口 |
| `ImageModel` | `model/image/ImageModel.java` | 图片生成接口 |

## 消息类型体系

```
ChatMessage (interface)
├── SystemMessage          -- 系统提示词
├── UserMessage            -- 用户输入 (支持多模态 List<Content>)
│   ├── TextContent
│   ├── ImageContent
│   ├── AudioContent
│   ├── VideoContent
│   └── PdfFileContent
├── AiMessage              -- LLM 响应 (text + thinking + toolExecutionRequests)
└── ToolExecutionResultMessage -- 工具执行结果 (id + toolName + contents + isError)
```

## 核心数据流

### 1. ChatModel 调用流程

```
调用者
  |
  v
ChatModel.chat(ChatRequest)
  ├── 合并默认参数与请求参数
  ├── 触发 ChatModelListener.onRequest()
  ├── 调用 doChat(ChatRequest)  <-- 子类实现
  ├── 触发 ChatModelListener.onResponse() 或 onError()
  v
ChatResponse (AiMessage + TokenUsage + FinishReason)
```

### 2. RAG 管道流程

```
DefaultRetrievalAugmentor.augment(AugmentationRequest)
  │
  ├── 1. 从 UserMessage 提取查询文本
  │
  ├── 2. QueryTransformer.transform(originalQuery) → Collection<Query>
  │
  ├── 3. 对每个 Query (并行 CompletableFuture):
  │     QueryRouter.route(query) → Collection<ContentRetriever>
  │     对每个 ContentRetriever:
  │       retriever.retrieve(query) → List<Content>
  │
  ├── 4. ContentAggregator.aggregate() → List<Content>
  │
  ├── 5. ContentInjector.inject(contents, chatMessage) → 增强后的 ChatMessage
  │
  v
AugmentationResult (augmentedChatMessage, contents)
```

### 3. EmbeddingStore 操作流程

```
添加: TextSegment → EmbeddingModel.embed() → Embedding
      Embedding + TextSegment → EmbeddingStore.add()

搜索: Query → EmbeddingModel.embed() → QueryEmbedding
      QueryEmbedding → EmbeddingStore.search(EmbeddingSearchRequest) → EmbeddingSearchResult
      (包含 maxResults, minScore, Filter 等条件)
```

## SPI 扩展点

`ServiceHelper` 包装 Java ServiceLoader，提供 `loadFactory()` / `loadFactories()` 方法。

关键 SPI 接口（在 `spi/` 包下）：
- `AiServicesFactory` — 自定义 AiServices 创建
- `ServiceOutputParser` — 自定义响应解析
- `JsonCodecFactory` — 自定义 JSON 编解码
- `DocumentSplitterFactory` — 自定义文档分块
- `DocumentParserFactory` — 自定义文档解析
- `PromptTemplateFactory` — 自定义提示模板

注册方式：在 `META-INF/services/` 下创建文件，内容为实现类全限定名。

## Tool 机制

`@Tool` 注解标注在方法上，`ToolSpecifications.from(Method)` 将其转换为 `ToolSpecification`：
- `name` — 工具名称
- `description` — 工具描述
- `parameters` — JSON Schema 格式的参数定义

## 关键类说明

| 类 | 职责 |
|----|------|
| `ChatRequest` | 包含 `List<ChatMessage>` + `ChatRequestParameters` (temperature, tools, responseFormat 等) |
| `ChatResponse` | 包含 `AiMessage` + `ChatResponseMetadata` (id, modelName, tokenUsage, finishReason) |
| `EmbeddingSearchRequest` | 向量搜索请求：queryEmbedding + maxResults + minScore + filter |
| `EmbeddingSearchResult<Embedded>` | 搜索结果：List<EmbeddingMatch<Embedded>> |
| `Capability` | 枚举：`RESPONSE_FORMAT_JSON_SCHEMA` |
| `DefaultRetrievalAugmentor` | RAG 默认实现：QueryTransformer → QueryRouter → ContentAggregator → ContentInjector |
| `ListeningEmbeddingStore` | 装饰器：为任意 EmbeddingStore 添加 Listener 支持 |

## 入口/出口

**入口点：**
- `ChatModel.chat(ChatRequest)` — 同步 LLM 调用
- `StreamingChatModel.chat(ChatRequest, handler)` — 流式 LLM 调用
- `EmbeddingModel.embed(String)` / `embedAll(List)` — 向量化
- `EmbeddingStore.search(EmbeddingSearchRequest)` — 向量搜索
- `RetrievalAugmentor.augment(AugmentationRequest)` — RAG 增强
- `InputGuardrail.validate(request)` / `OutputGuardrail.validate(request)` — 护栏验证

**出口点：**
- `ChatResponse` — LLM 响应
- `Embedding` — 向量结果
- `EmbeddingSearchResult` — 搜索结果
- `AugmentationResult` — RAG 增强结果
- `InputGuardrailResult` / `OutputGuardrailResult` — 护栏验证结果
