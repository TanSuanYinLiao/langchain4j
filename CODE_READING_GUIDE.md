# LangChain4j 源码阅读指南

> 按 **接口定义 → 高层实现 → 具体实现 → 扩展模块** 的顺序，从抽象到具体，从核心到外围。

## 阅读顺序总结

```
第一天（核心接口）：
  ChatModel.java → StreamingChatModel.java → ChatRequest.java → ChatResponse.java
  → UserMessage.java → AiMessage.java → Tool.java → ToolSpecification.java

第二天（RAG + 存储）：
  EmbeddingModel.java → EmbeddingStore.java → RetrievalAugmentor.java
  → ContentRetriever.java → Document.java → DocumentSplitter.java

第三天（高层 API）：
  DefaultAiServices.java → AiServiceContext.java → ServiceOutputParser.java
  → ToolService.java → ChatMemory.java

第四天（Provider 实现）：
  OpenAiChatModel.java → DefaultOpenAiClient.java
  → AnthropicChatModel.java（对比阅读）
```

**关键技巧**：
1. 阶段 1 理解了，就理解了 80% — 所有 Provider 模块都是这些接口的实现
2. `DefaultAiServices.invoke()` 是整个框架的"大脑"，画出它的调用流程图
3. 每读完一个接口，去看一个它的具体实现，加深理解
4. 测试用例是最好的文档 — 集成测试展示了实际使用方式

---

## 阶段 1：`langchain4j-core`（接口层）

### 1.1 Chat Model 核心接口（最先读）

```
langchain4j-core/src/main/java/dev/langchain4j/model/chat/
├── ChatModel.java                          ★ 核心接口，chat() → doChat()
├── StreamingChatModel.java                 ★ 流式模型接口
├── Capability.java                         模型能力枚举
├── ChatModelListenerUtils.java             Listener 工具类
├── DisabledChatModel.java                  禁用实现
├── DisabledStreamingChatModel.java         禁用流式实现
├── listener/
│   ├── ChatModelListener.java              ★ 观察者模式监听器
│   ├── ChatModelRequestContext.java         请求上下文
│   ├── ChatModelResponseContext.java        响应上下文
│   └── ChatModelErrorContext.java           错误上下文
├── request/
│   ├── ChatRequest.java                    ★ 请求结构（messages + parameters + tools）
│   ├── ChatRequestParameters.java          请求参数接口
│   ├── DefaultChatRequestParameters.java   默认参数实现
│   ├── ResponseFormat.java                 响应格式
│   ├── ResponseFormatType.java             响应格式类型
│   ├── ToolChoice.java                     工具选择策略
│   └── json/                               JSON Schema 体系（14个文件）
│       ├── JsonSchemaElement.java
│       ├── JsonObjectSchema.java
│       ├── JsonArraySchema.java
│       ├── JsonStringSchema.java
│       └── ...
└── response/
    ├── ChatResponse.java                   ★ 响应结构（AiMessage + metadata）
    ├── ChatResponseMetadata.java           响应元数据
    ├── CompleteToolCall.java               完整工具调用
    ├── StreamingChatResponseHandler.java   ★ 流式回调接口
    ├── StreamingHandle.java                流式处理句柄
    ├── PartialResponse.java                部分响应
    ├── PartialThinking.java                部分思考
    ├── PartialToolCall.java                部分工具调用
    └── *Context.java                       各 Context 类
```

### 1.2 Message 消息体系

```
langchain4j-core/src/main/java/dev/langchain4j/data/message/
├── ChatMessage.java                        ★ 消息基类接口
├── ChatMessageType.java                    消息类型枚举
├── SystemMessage.java                      系统提示
├── UserMessage.java                        ★ 用户消息（支持多模态）
├── AiMessage.java                          ★ AI 响应（text + toolCalls）
├── ToolExecutionResultMessage.java         工具执行结果
├── CustomMessage.java                      自定义消息
├── Content.java                            内容基类
├── ContentType.java                        内容类型
├── TextContent.java                        文本内容
├── ImageContent.java                       图片内容
├── AudioContent.java                       音频内容
├── VideoContent.java                       视频内容
├── PdfFileContent.java                     PDF 内容
├── ChatMessageJsonCodec.java              JSON 编解码
└── ChatMessageSerializer.java / Deserializer.java
```

### 1.3 Tool/Function 调用

```
langchain4j-core/src/main/java/dev/langchain4j/agent/tool/
├── Tool.java                               ★ @Tool 注解定义
├── ToolSpecification.java                  ★ 工具规范（name/desc/params）
├── ToolSpecifications.java                 工具规范构建工具
├── ToolExecutionRequest.java               模型的工具调用请求
├── ToolSpecificationJsonCodec.java         JSON 编解码
├── P.java                                  @P 参数注解
├── ReturnBehavior.java                     返回行为枚举
├── SearchBehavior.java                     搜索行为枚举
└── ToolMemoryId.java                       工具记忆标识注解
```

### 1.4 Embedding & 向量存储

```
langchain4j-core/src/main/java/dev/langchain4j/model/embedding/
├── EmbeddingModel.java                     ★ 嵌入模型接口
├── DimensionAwareEmbeddingModel.java       维度感知
├── ListeningEmbeddingModel.java            监听装饰器
└── listener/                               嵌入监听（4个文件）

langchain4j-core/src/main/java/dev/langchain4j/store/embedding/
├── EmbeddingStore.java                     ★ 向量存储接口
├── EmbeddingStoreIngestor.java             摄取器
├── EmbeddingMatch.java                     匹配结果
├── EmbeddingSearchRequest.java             搜索请求
├── EmbeddingSearchResult.java              搜索结果
├── RelevanceScore.java                     相关性评分
├── CosineSimilarity.java                   余弦相似度
├── ListeningEmbeddingStore.java            监听装饰器
├── filter/                                 过滤体系（14个文件）
│   ├── Filter.java
│   ├── FilterParser.java
│   ├── comparison/ (IsEqualTo, IsGreaterThan, ...)
│   └── logical/ (And, Or, Not)
└── listener/                               存储监听（4个文件）
```

### 1.5 RAG 管道

```
langchain4j-core/src/main/java/dev/langchain4j/rag/
├── RetrievalAugmentor.java                 ★ RAG 入口接口
├── DefaultRetrievalAugmentor.java          默认实现
├── AugmentationRequest.java                增强请求
├── AugmentationResult.java                 增强结果
├── content/
│   ├── Content.java                        内容封装
│   ├── DefaultContent.java
│   ├── ContentMetadata.java
│   ├── retriever/
│   │   ├── ContentRetriever.java           ★ 内容检索接口
│   │   ├── EmbeddingStoreContentRetriever.java  基于向量的检索
│   │   ├── WebSearchContentRetriever.java      Web 检索
│   │   ├── ListeningContentRetriever.java      监听装饰器
│   │   └── listener/ (4个文件)
│   ├── aggregator/
│   │   ├── ContentAggregator.java          内容聚合接口
│   │   ├── DefaultContentAggregator.java   默认聚合
│   │   ├── ReRankingContentAggregator.java 重排聚合
│   │   └── ReciprocalRankFuser.java        RRF 融合
│   └── injector/
│       ├── ContentInjector.java            内容注入接口
│       └── DefaultContentInjector.java     默认注入
└── query/
    ├── Query.java                          查询对象
    ├── Metadata.java
    ├── router/
    │   ├── QueryRouter.java                查询路由接口
    │   ├── DefaultQueryRouter.java
    │   └── LanguageModelQueryRouter.java   LLM 路由
    └── transformer/
        ├── QueryTransformer.java           查询转换接口
        ├── DefaultQueryTransformer.java
        ├── ExpandingQueryTransformer.java  扩展查询
        └── CompressingQueryTransformer.java 压缩查询
```

### 1.6 ChatMemory & Document

```
langchain4j-core/src/main/java/dev/langchain4j/memory/
└── ChatMemory.java                         ★ 聊天记忆接口

langchain4j-core/src/main/java/dev/langchain4j/data/document/
├── Document.java                           ★ 文档模型
├── DefaultDocument.java
├── DocumentParser.java                     文档解析接口
├── DocumentSplitter.java                   文档分块接口
├── DocumentTransformer.java                文档转换接口
├── DocumentLoader.java                     文档加载接口
├── DocumentSource.java                     文档来源
└── Metadata.java                           文档元数据
```

### 1.7 Guardrail 护栏

```
langchain4j-core/src/main/java/dev/langchain4j/guardrail/
├── Guardrail.java                          护栏基类接口
├── GuardrailResult.java                    ★ 结果状态机 (SUCCESS/FAILURE/RECOVER)
├── InputGuardrail.java                     输入护栏接口
├── OutputGuardrail.java                    输出护栏接口
├── ChatExecutor.java                       护栏内 LLM 调用抽象
├── SynchronousChatExecutor.java            同步执行器
├── GuardrailRequest.java
├── InputGuardrailResult.java / OutputGuardrailResult.java
├── config/
│   ├── GuardrailsConfig.java
│   ├── InputGuardrailsConfig.java
│   └── OutputGuardrailsConfig.java
└── ...
```

### 1.8 SPI 扩展机制

```
langchain4j-core/src/main/java/dev/langchain4j/spi/
├── ServiceHelper.java                      ★ SPI 加载核心
├── agent/tool/ToolSpecificationJsonCodecFactory.java
├── data/document/parser/DocumentParserFactory.java
├── data/document/splitter/DocumentSplitterFactory.java
├── data/message/ChatMessageJsonCodecFactory.java
├── json/JsonCodecFactory.java
├── model/embedding/EmbeddingModelFactory.java
├── prompt/PromptTemplateFactory.java
├── prompt/structured/StructuredPromptFactory.java
├── guardrail/ (5个 Factory 接口)
└── observability/AiServiceListenerRegistrarFactory.java
```

---

## 阶段 2：`langchain4j`（高层 API 层）

### 2.1 AiServices 核心

```
langchain4j/src/main/java/dev/langchain4j/service/
├── AiServices.java                         ★ Builder 入口，create() 静态方法
├── DefaultAiServices.java                  ★★★ 最关键文件：invoke() 完整调用链
├── AiServiceContext.java                   调用上下文
├── AiServiceStreamingResponseHandler.java  流式响应处理
├── AiServiceTokenStream.java               Token 流
├── AiServiceValidation.java                参数校验
├── SystemMessage.java                      @SystemMessage 注解
├── UserMessage.java                        @UserMessage 注解
├── V.java                                  @V 变量注入注解
├── MemoryId.java                           @MemoryId 注解
├── Moderate.java                           @Moderate 审核注解
├── TokenStream.java                        TokenStream 接口
├── Result.java                             结果封装
└── ...
```

### 2.2 Output 解析

```
langchain4j/src/main/java/dev/langchain4j/service/output/
├── ServiceOutputParser.java                ★ 输出解析入口
├── OutputParser.java                       解析器接口
├── OutputParserFactory.java                工厂接口
├── JsonSchemas.java                        JSON Schema 生成
├── PojoOutputParser.java                   POJO 解析
├── PojoListOutputParser.java               POJO 列表解析
├── EnumOutputParser.java                   枚举解析
├── StringListOutputParser.java             字符串列表解析
└── (数值类型: Integer/Long/Float/Double/Boolean 等)
```

### 2.3 Tool 执行

```
langchain4j/src/main/java/dev/langchain4j/service/tool/
├── ToolService.java                        ★ 工具服务编排
├── ToolExecutor.java                       工具执行器
├── DefaultToolExecutor.java                默认执行实现
├── ToolExecution.java                      工具执行过程
├── ToolProvider.java                       动态工具发现
├── ToolProviderRequest.java / Result.java
├── ToolServiceContext.java / Result.java
├── search/
│   ├── ToolSearchStrategy.java             工具搜索策略
│   ├── SimpleToolSearchStrategy.java       简单搜索
│   └── VectorToolSearchStrategy.java       向量搜索
└── ...
```

### 2.4 Guardrail 服务层

```
langchain4j/src/main/java/dev/langchain4j/service/guardrail/
├── GuardrailService.java                   护栏服务接口
├── DefaultGuardrailService.java            默认实现
├── InputGuardrails.java                    输入护栏编排
├── OutputGuardrails.java                   输出护栏编排
└── spi/GuardrailServiceBuilderFactory.java
```

### 2.5 ChatMemory 实现

```
langchain4j/src/main/java/dev/langchain4j/service/memory/
├── ChatMemoryService.java                  记忆服务接口
└── ChatMemoryAccess.java                   记忆访问

langchain4j-memory/
├── MessageWindowChatMemory.java            ★ 固定窗口记忆
└── TokenWindowChatMemory.java              ★ Token 窗口记忆
```

---

## 阶段 3：Provider 实现（以 OpenAI 为代表）

```
langchain4j-open-ai/src/main/java/dev/langchain4j/model/openai/
├── OpenAiChatModel.java                    ★ 实现 ChatModel.doChat()
├── OpenAiStreamingChatModel.java           ★ 实现 StreamingChatModel.doChat()
├── OpenAiEmbeddingModel.java               实现 EmbeddingModel
├── OpenAiTokenCountEstimator.java          Token 计数
├── OpenAiChatModelName.java                模型名枚举
├── internal/
│   ├── DefaultOpenAiClient.java            ★ HTTP 客户端核心
│   ├── OpenAiClient.java                   客户端接口
│   ├── RequestExecutor.java                请求执行
│   ├── SyncRequestExecutor.java            同步请求
│   ├── StreamingRequestExecutor.java       流式请求
│   ├── ErrorHandling.java                  错误处理
│   ├── chat/
│   │   ├── ChatCompletionRequest.java      请求 DTO
│   │   ├── ChatCompletionResponse.java     响应 DTO
│   │   ├── Message.java                    消息 DTO
│   │   ├── Tool.java / ToolCall.java       工具 DTO
│   │   └── ...（15+ 个内部 DTO）
│   ├── embedding/
│   │   ├── EmbeddingRequest.java
│   │   └── EmbeddingResponse.java
│   └── spi/OpenAiClientBuilderFactory.java
└── spi/
    ├── OpenAiChatModelBuilderFactory.java
    ├── OpenAiEmbeddingModelBuilderFactory.java
    └── ...
```

**对比 Anthropic**：
```
langchain4j-anthropic/src/main/java/dev/langchain4j/model/anthropic/
├── AnthropicChatModel.java                 对比 doChat() 实现差异
├── AnthropicStreamingChatModel.java
├── internal/api/                           Anthropic 专有 DTO（20+ 文件）
│   ├── AnthropicCreateMessageRequest.java
│   ├── AnthropicCreateMessageResponse.java
│   ├── AnthropicMessage.java
│   └── AnthropicTool.java
├── internal/client/
│   ├── AnthropicClient.java
│   └── DefaultAnthropicClient.java
└── internal/mapper/
    └── AnthropicMapper.java
```

---

## 阶段 4：扩展模块（按需阅读）

| 模块 | 用途 | 何时读 |
|------|------|--------|
| `langchain4j-easy-rag` | 简化 RAG 配置 | 需要 RAG 功能时 |
| `langchain4j-agentic/` | Agent 编排（实验性） | 需要 Agent 功能时 |
| `langchain4j-mcp/` | MCP 协议支持 | 需要工具集成时 |
| `langchain4j-guardrails/` | 内置护栏实现 | 需要安全验证时 |
| `langchain4j-micrometer-metrics/` | 指标监控 | 需要可观测性时 |
| `langchain4j-test/` | 测试基础设施 | 写测试时 |

---

## 核心设计模式速查

| 模式 | 应用位置 | 说明 |
|------|---------|------|
| **Builder** | 几乎所有类 | `ChatModel.builder()`, `AiServices.builder()` |
| **Dynamic Proxy** | `AiServices.create()` | JDK 动态代理将接口方法映射为 LLM 调用 |
| **Observer/Listener** | `ChatModelListener` | 请求/响应/错误事件可观测性 |
| **Strategy** | `ToolSearchStrategy`, `DocumentSplitter` | 可替换的算法实现 |
| **Template Method** | `ChatModel.chat()` → `doChat()` | 定义骨架，子类实现 |
| **SPI Factory** | `ServiceHelper.loadFactory()` | Java ServiceLoader 插件化扩展 |
| **Factory Method** | `AiServicesFactory` | SPI 加载工厂创建实例 |
