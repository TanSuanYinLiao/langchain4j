# LangChain4j 项目全面分析

> 基于项目源码 (v1.13.0-beta23-SNAPSHOT) 的深度技术分析。

---

## 1. 项目概述

**LangChain4j** 是一个 Java 开源框架，目标是简化 LLM（大语言模型）集成到 Java 应用中的过程。项目始于 2023 年初，灵感来源于 Python 生态的 LangChain、Haystack、LlamaIndex，并融合了社区的最佳实践。

| 维度 | 详情 |
|------|------|
| **仓库** | `langchain4j/langchain4j` (GitHub) |
| **许可证** | Apache 2.0 |
| **当前版本** | 1.13.0-beta23-SNAPSHOT |
| **最低 JDK** | Java 17 |
| **构建工具** | Maven (多模块聚合) |
| **模块数量** | 80+ 模块 |
| **源码规模** | ~1648 个 Java 源文件，~175K 行代码，9 个 Kotlin 文件 |
| **测试文件** | ~990 个 (含 484 个集成测试) |
| **语言** | Java (主)，Kotlin (辅) |

**三大核心支柱：**
1. **统一 API** — 对 20+ LLM 提供商和 30+ 向量数据库提供统一抽象接口
2. **综合工具箱** — 从底层提示模板、聊天记忆、函数调用到高层 Agent、RAG 模式
3. **丰富示例** — 配套 `langchain4j-examples` 仓库，覆盖多种框架集成

---

## 2. 架构设计分析

### 2.1 分层架构

```
┌─────────────────────────────────────────────────┐
│  最顶层: Agentic 编排层                           │
│  @SupervisorAgent / @PlannerAgent / @LoopAgent   │
│  多 Agent 工作流声明式编排                          │
├─────────────────────────────────────────────────┤
│  第2层: AiServices 高层 API                       │
│  @SystemMessage / @UserMessage / @Tool 注解       │
│  单个 Agent 的动态代理调用                          │
├─────────────────────────────────────────────────┤
│  第3层: langchain4j-core 核心接口                 │
│  ChatModel, EmbeddingStore, ChatMemory, Tool,     │
│  RetrievalAugmentor, Guardrail                    │
├─────────────────────────────────────────────────┤
│  第4层: Provider 实现层                           │
│  OpenAI, Anthropic, Ollama, Azure, Gemini, ...    │
├─────────────────────────────────────────────────┤
│  最底层: HTTP Client 网络通信层                    │
│  langchain4j-http-client 统一抽象                  │
│  ├── langchain4j-http-client-jdk (JDK HttpClient) │
│  └── langchain4j-http-client-apache               │
└─────────────────────────────────────────────────┘
```

- **langchain4j-core**: 纯接口层，定义 `ChatModel`、`EmbeddingModel`、`EmbeddingStore`、`ChatMemory`、`RetrievalAugmentor` 等核心契约，零实现依赖
- **langchain4j**: 高层 API 层，核心是 `AiServices` 动态代理机制和 `DefaultAiServices` 实现
- **langchain4j-agentic**: 最高层 Agent 编排层，支持顺序、并行、循环、条件、监督者、规划器等多 Agent 组合模式
- **Provider 模块**: 每个 LLM 提供商一个独立 Maven 模块（如 `langchain4j-open-ai`、`langchain4j-anthropic`），各自实现核心接口

### 2.2 核心设计模式

| 模式 | 应用位置 | 说明 |
|------|---------|------|
| **Builder** | 几乎所有类 | `ChatModel.builder()`, `AiServices.builder()` — 链式构建复杂对象 |
| **Dynamic Proxy** | `AiServices.create()` | JDK 动态代理将用户接口方法映射为 LLM 调用 |
| **Observer/Listener** | `ChatModelListener`, `EmbeddingStoreListener`, `AiServiceListener` | 请求/响应/错误事件的可观测性 |
| **Strategy** | `ToolSearchStrategy`, `DocumentSplitter` | 可替换的算法实现 |
| **Template Method** | `ChatModel.chat()` → `doChat()` | 模板方法定义骨架，子类实现具体逻辑 |
| **SPI Factory** | `ServiceHelper.loadFactory()` | Java ServiceLoader 机制实现插件化扩展 |
| **Factory Method** | `AiServicesFactory` | SPI 加载工厂创建 AiServices 实例 |

---

## 3. 完整模块清单

### 3.1 核心模块

| 模块 | 职责 |
|------|------|
| `langchain4j` | 高层 API：AiServices 动态代理、ServiceOutputParser、DefaultAiServices |
| `langchain4j-core` | 核心接口层：ChatModel, EmbeddingStore, ChatMemory, Tool, RetrievalAugmentor, Guardrail 等所有契约定义 |
| `langchain4j-test` | 测试工具：AssertJ 风格的 GuardrailResult 断言 |
| `langchain4j-kotlin` | Kotlin 扩展：协程支持 (`chatAsync()`)、Flow 适配器 (`TokenStream → Flow`) |
| `langchain4j-parent` | Maven Parent POM：统一依赖版本管理 |
| `langchain4j-bom** | Bill of Materials：统一管理所有模块版本号 |

### 3.2 HTTP 客户端

| 模块 | 职责 |
|------|------|
| `langchain4j-http-client` | HTTP 客户端统一抽象接口 |
| `http-clients/langchain4j-http-client-jdk` | JDK HttpClient 实现 |
| `http-clients/langchain4j-http-client-apache` | Apache HttpClient 实现 |

### 3.3 LLM 提供商 (22个)

| 模块 | 说明 |
|------|------|
| `langchain4j-open-ai` | OpenAI (旧 SDK) |
| `langchain4j-open-ai-official` | OpenAI (官方 SDK) |
| `langchain4j-anthropic` | Anthropic Claude |
| `langchain4j-azure-open-ai` | Azure OpenAI Service |
| `langchain4j-bedrock` | AWS Bedrock |
| `langchain4j-cohere` | Cohere |
| `langchain4j-google-ai-gemini` | Google AI Gemini |
| `langchain4j-vertex-ai` | Google Vertex AI |
| `langchain4j-vertex-ai-gemini` | Vertex AI Gemini |
| `langchain4j-vertex-ai-anthropic` | Vertex AI Anthropic |
| `langchain4j-hugging-face` | HuggingFace |
| `langchain4j-jina` | Jina |
| `langchain4j-local-ai` | LocalAI |
| `langchain4j-mistral-ai` | Mistral AI |
| `langchain4j-nomic` | Nomic |
| `langchain4j-ollama` | Ollama |
| `langchain4j-onnx-scoring` | ONNX 本地推理 |
| `langchain4j-ovh-ai` | OVH AI |
| `langchain4j-voyage-ai` | Voyage AI |
| `langchain4j-watsonx` | IBM WatsonX |
| `langchain4j-workers-ai` | Cloudflare Workers AI |
| `langchain4j-github-models` | GitHub Models |

**JDK 特定模块：**
| 模块 | 说明 |
|------|------|
| `langchain4j-jlama` | JLAMA 本地推理 (需 JDK 21+) |
| `langchain4j-gpu-llama3` | GPU Llama3 (需 JDK 25) |

### 3.4 Embedding 存储 (21个)

| 模块 | 说明 |
|------|------|
| `langchain4j-azure-ai-search` | Azure AI Search |
| `langchain4j-azure-cosmos-mongo-vcore` | Azure Cosmos DB (Mongo vCore) |
| `langchain4j-azure-cosmos-nosql` | Azure Cosmos DB (NoSQL) |
| `langchain4j-cassandra` | Apache Cassandra |
| `langchain4j-chroma` | Chroma |
| `langchain4j-coherence` | Oracle Coherence |
| `langchain4j-couchbase` | Couchbase |
| `langchain4j-elasticsearch` | Elasticsearch |
| `langchain4j-hibernate` | Hibernate (JPA) |
| `langchain4j-infinispan` | Infinispan |
| `langchain4j-mariadb` | MariaDB |
| `langchain4j-milvus` | Milvus |
| `langchain4j-mongodb-atlas` | MongoDB Atlas |
| `langchain4j-opensearch` | OpenSearch |
| `langchain4j-oracle` | Oracle |
| `langchain4j-pgvector` | PostgreSQL (pgvector) |
| `langchain4j-pinecone` | Pinecone |
| `langchain4j-qdrant` | Qdrant |
| `langchain4j-tablestore` | Tablestore (OTS) |
| `langchain4j-vespa` | Vespa |
| `langchain4j-weaviate` | Weaviate |

### 3.5 本地 Embedding 模型 (11个)

位于 `embeddings/` 目录，基于 ONNX Runtime 运行，无需外部 API：

| 模块 | 说明 |
|------|------|
| `langchain4j-embeddings` | 本地嵌入基类 |
| `langchain4j-embeddings-all-minilm-l6-v2` | all-MiniLM-L6-v2 |
| `langchain4j-embeddings-all-minilm-l6-v2-q` | all-MiniLM-L6-v2 (量化版) |
| `langchain4j-embeddings-bge-small-en` | BGE Small English |
| `langchain4j-embeddings-bge-small-en-q` | BGE Small English (量化版) |
| `langchain4j-embeddings-bge-small-en-v15` | BGE Small English v1.5 |
| `langchain4j-embeddings-bge-small-en-v15-q` | BGE Small English v1.5 (量化版) |
| `langchain4j-embeddings-bge-small-zh-v15` | BGE Small Chinese v1.5 |
| `langchain4j-embeddings-bge-small-zh-v15-q` | BGE Small Chinese v1.5 (量化版) |
| `langchain4j-embeddings-e5-small-v2` | E5 Small v2 |
| `langchain4j-embeddings-e5-small-v2-q` | E5 Small v2 (量化版) |

### 3.6 文档加载器 (7个)

位于 `document-loaders/` 目录：

| 模块 | 数据源 |
|------|--------|
| `langchain4j-document-loader-amazon-s3` | AWS S3 |
| `langchain4j-document-loader-azure-storage-blob` | Azure Blob Storage |
| `langchain4j-document-loader-github` | GitHub 仓库 |
| `langchain4j-document-loader-selenium` | 网页 (Selenium) |
| `langchain4j-document-loader-playwright` | 网页 (Playwright) |
| `langchain4j-document-loader-tencent-cos` | 腾讯云 COS |
| `langchain4j-document-loader-google-cloud-storage` | Google Cloud Storage |

### 3.7 文档解析器 (5个)

位于 `document-parsers/` 目录：

| 模块 | 解析格式 |
|------|---------|
| `langchain4j-document-parser-apache-pdfbox` | PDF |
| `langchain4j-document-parser-apache-poi` | Word/Excel/PPT (Office) |
| `langchain4j-document-parser-apache-tika` | 通用格式 (Tika) |
| `langchain4j-document-parser-markdown` | Markdown |
| `langchain4j-document-parser-yaml` | YAML |

### 3.8 文档转换器 (1个)

| 模块 | 职责 |
|------|------|
| `document-transformers/langchain4j-document-transformer-jsoup` | 基于 Jsoup 的 HTML 文档转换 |

### 3.9 代码执行引擎 (3个)

位于 `code-execution-engines/` 目录：

| 模块 | 引擎 |
|------|------|
| `langchain4j-code-execution-engine-graalvm-polyglot` | GraalVM Polyglot (多语言执行) |
| `langchain4j-code-execution-engine-judge0` | Judge0 (在线代码执行) |
| `langchain4j-code-execution-engine-azure-acads` | Azure Academic Service |

### 3.10 Web 搜索引擎 (3个)

位于 `web-search-engines/` 目录：

| 模块 | 引擎 |
|------|------|
| `langchain4j-web-search-engine-google-custom` | Google Custom Search |
| `langchain4j-web-search-engine-tavily` | Tavily |
| `langchain4j-web-search-engine-searchapi` | SearchAPI |

### 3.11 Embedding 过滤解析器 (1个)

| 模块 | 职责 |
|------|------|
| `embedding-store-filter-parsers/langchain4j-embedding-store-filter-parser-sql` | SQL 格式的过滤表达式解析 |

### 3.12 Agentic 模块 (4个)

| 模块 | 职责 |
|------|------|
| `langchain4j-agentic` | Agent 框架核心：声明式多 Agent 编排，支持 `@SequenceAgent`, `@ParallelAgent`, `@LoopAgent`, `@ConditionalAgent`, `@SupervisorAgent`, `@PlannerAgent` 等注解 |
| `langchain4j-agentic-a2a` | A2A (Agent-to-Agent) 协议：允许 Agent 通过 A2A 协议与远程 Agent 通信 |
| `langchain4j-agentic-mcp` | MCP Agent 集成：将 MCP 服务器的工具包装为可组合的 Agent，在 agentic 工作流中使用 |
| `langchain4j-agentic-patterns` | Agent 规划模式：实现 GOAP (目标导向行动规划) 和 P2P (点对点) 两种高级规划模式 |

**Agentic 工作流类型：**

```
@SequenceAgent    → 顺序执行子 Agent
@ParallelAgent    → 并发执行子 Agent
@LoopAgent        → 循环执行 (maxIterations)
@ConditionalAgent → 条件激活 (@ActivationCondition)
@SupervisorAgent  → LLM 动态选择调用哪个子 Agent
@PlannerAgent     → 规划器驱动 (GOAP / P2P)
@A2AClientAgent   → 调用远程 A2A Agent
@McpClientAgent   → 调用 MCP 工具 Agent
```

### 3.13 MCP 模块 (2个)

| 模块 | 职责 |
|------|------|
| `langchain4j-mcp` | MCP (Model Context Protocol) 客户端：连接 MCP 服务器获取工具、资源和提示模板。支持 Stdio / HTTP SSE / Streamable HTTP / WebSocket 传输层 |
| `langchain4j-mcp-docker` | MCP Docker 传输层：在 Docker 容器中运行 MCP 服务器 |

### 3.14 Skills 框架

| 模块 | 职责 |
|------|------|
| `langchain4j-skills` | Agent Skills 框架：实现 agentskills.io 规范，从 SKILL.md 文件加载可复用技能，暴露 `activate_skill` / `read_skill_resource` 工具给 LLM |

### 3.15 护栏模块

| 模块 | 职责 |
|------|------|
| `langchain4j-guardrails` | 内置护栏实现：`JsonExtractorOutputGuardrail` (验证 JSON 响应) 和 `MessageModeratorInputGuardrail` (内容审核) |

### 3.16 Easy RAG

| 模块 | 职责 |
|------|------|
| `langchain4j-easy-rag` | 简易 RAG 配置：开箱即用的文档分块 (300 tokens, 30 overlap)，SPI 自动加载 `RecursiveDocumentSplitterFactory` |

### 3.17 可观测性模块 (2个)

| 模块 | 职责 |
|------|------|
| `langchain4j-micrometer-metrics` | Micrometer 指标：直接收集 Token 使用量，遵循 OpenTelemetry GenAI 语义约定 (Experimental) |
| `langchain4j-observation` | Micrometer Observation 集成：支持分布式追踪 (OpenTelemetry)，深度 Spring Boot 集成 (Experimental) |

### 3.18 实验性模块 (4个)

| 模块 | 职责 |
|------|------|
| `experimental/langchain4j-experimental-sql` | SQL 查询：LLM 驱动的自然语言转 SQL |
| `experimental/langchain4j-experimental-hibernate` | Hibernate 集成：JPA 对象作为 Embedding 存储 |
| `experimental/langchain4j-experimental-skills-shell` | Skills Shell：命令行技能执行 |
| `langchain4j-skills` | Skills 框架（见 3.14） |

### 3.19 内部工具 (2个)

| 模块 | 职责 |
|------|------|
| `internal/langchain4j-internal-test-retry` | 测试重试机制：CI 环境下的 flaky test 处理 |
| `internal/langchain4j-docu-chatbot-updater` | 文档聊天机器人更新器：自动更新文档知识库 |

---

## 4. 核心源码解读

### 4.1 ChatModel 接口 (`langchain4j-core/.../model/chat/ChatModel.java`)

核心接口，所有聊天模型的统一抽象：

```java
public interface ChatModel {
    // 核心方法 — 模板方法模式
    default ChatResponse chat(ChatRequest request) {
        // 1. 触发 listener onRequest
        // 2. 调用 doChat() (由具体提供商实现)
        // 3. 触发 listener onResponse / onError
    }

    // 抽象方法 — 子类必须实现
    ChatResponse doChat(ChatRequest request);

    // 便捷方法
    default String chat(String userMessage) { ... }
    default AiMessage chat(List<ChatMessage> messages) { ... }

    // 能力声明
    default Set<Capability> supportedCapabilities() { return Set.of(); }
}
```

`StreamingChatModel` 是其流式对应接口，通过 `StreamingChatResponseHandler` 回调处理 `onPartialResponse`、`onPartialThinking`、`onPartialToolCall`、`onCompleteResponse`、`onError`。

### 4.2 AiServices — 动态代理核心

`AiServices` (`langchain4j/.../service/AiServices.java`) 是 LangChain4j 最具特色的 API，通过 JDK 动态代理将用户定义的接口自动映射为 LLM 调用：

```java
// 用户定义接口
interface Assistant {
    @SystemMessage("You are a helpful assistant")
    String chat(@UserMessage String message);
}

// 创建代理实例
Assistant assistant = AiServices.create(Assistant.class, chatModel);
String response = assistant.chat("Hello");
```

**Builder 模式提供丰富配置：**
- `chatModel` / `streamingChatModel` — 模型绑定
- `chatMemory` / `chatMemoryProvider` — 对话记忆
- `contentRetriever` / `retrievalAugmentor` — RAG 支持
- `tools` — `@Tool` 注解方法的函数调用
- `toolProvider` — 动态工具发现
- `guardrails` — 输入/输出护栏
- `listeners` — 可观测性监听器
- `moderationModel` — 内容审核

### 4.3 DefaultAiServices 实现

`DefaultAiServices` (`langchain4j/.../service/DefaultAiServices.java`) 继承 `AiServices`，在 `invoke()` 方法中编排完整的调用链：

1. 获取/创建 ChatMemory
2. 构建 SystemMessage（支持 `@SystemMessage` 注解）
3. 构建 UserMessage（支持 `@UserMessage` 注解、`@V` 变量替换）
4. 执行 RAG — `RetrievalAugmentor.augment()`
5. 执行输入 Guardrails
6. 调用 ChatModel / StreamingChatModel
7. 解析响应（通过 SPI 加载的 `ServiceOutputParser`）
8. 执行工具调用（循环直到模型不再调用工具）
9. 更新 ChatMemory
10. 执行输出 Guardrails

### 4.4 SPI 机制 (`langchain4j-core/.../spi/ServiceHelper.java`)

```java
public class ServiceHelper {
    public static <T> T loadFactory(Class<T> factoryClass) {
        // 1. 使用当前线程的 ClassLoader
        // 2. 回退到 ServiceHelper 自身的 ClassLoader
        // 3. 使用 ClassLoader.getSystemClassLoader()
    }
}
```

SPI 注册通过 `META-INF/services/` 文件实现，例如：
- `dev.langchain4j.spi.service.AiServicesFactory` → `dev.langchain4j.service.AiServicesFactory`
- `dev.langchain4j.spi.service.output.ServiceOutputParser` → 具体解析器
- `dev.langchain4j.spi.model.ModelProvider` → 模型提供商发现

### 4.5 Tool/Function 调用

`@Tool` 注解 (`langchain4j-core/.../agent/tool/Tool.java`) 标注在方法上：

```java
@Retention(RUNTIME) @Target(METHOD)
public @interface Tool {
    String name() default "";
    String value() default "";        // 描述
    ReturnBehavior returnBehavior() default LAST_VALUE;
    SearchBehavior searchBehavior() default AUTO;
    String[][] metadata() default {};
}
```

AiServices 扫描 `@Tool` 注解方法，构建 `ToolSpecification` 并传递给 ChatModel。当模型返回 `ToolExecutionRequest` 时，通过反射调用对应方法。

### 4.6 RAG 管道

- **入口**: `RetrievalAugmentor.augment(AugmentationRequest)` → `AugmentationResult`
- **检索**: `ContentRetriever` 接口从 `EmbeddingStore` 检索相关内容
- **嵌入**: `EmbeddingModel` 接口 (`embed()`, `embedAll()`, `dimension()`)
- **存储**: `EmbeddingStore<Embedded>` 接口提供 `add()`, `search()`, `removeAll()` 操作
- **文档处理**: `DocumentParser` → `DocumentSplitter` → `EmbeddingModel` → `EmbeddingStore`

### 4.7 ChatMemory

```java
public interface ChatMemory {
    Object id();
    void add(ChatMessage message);
    List<ChatMessage> messages();
    void clear();
    void set(List<ChatMessage> messages);  // 覆盖式更新
}
```

实现包括 `MessageWindowChatMemory`（固定窗口）和 `TokenWindowChatMemory`（基于 token 限制）。通过 `ChatMemoryProvider` 为每个用户/会话创建独立记忆实例。

### 4.8 Guardrails

护栏框架位于 `langchain4j-core/.../guardrail/` 包：
- `InputGuardrail` — 输入验证（在 LLM 调用前执行）
- `OutputGuardrail` — 输出验证（在 LLM 响应后执行）
- `GuardrailResult` — 验证结果（成功/失败/重试）
- `ChatExecutor` — 护栏内部调用 LLM 的抽象

---

## 5. 消息类型体系

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

---

## 6. 性能与高可用

### 6.1 流式处理
- `StreamingChatModel` 接口支持逐 token 流式输出
- `StreamingChatResponseHandler` 回调处理部分响应、思考过程、工具调用
- 通过 `TokenStreamAdapter` (SPI) 实现 Reactor 等响应式集成
- Kotlin 模块提供 Flow 适配器

### 6.2 连接管理
- 各提供商模块独立管理 HTTP 连接池
- `langchain4j-http-client` 提供统一 HTTP 客户端抽象
- 子模块 `langchain4j-http-client-jdk`（JDK HttpClient）和 `langchain4j-http-client-apache`（Apache HttpClient）
- 各提供商可选择使用官方 SDK 或自定义 HTTP 客户端

### 6.3 并发构建
- Maven 构建使用 `-T8C`（每个 CPU 核心一个线程）加速多模块编译
- CI 矩阵测试 JDK 17、21、25 三个版本

### 6.4 Embedding Store 高可用
- 支持的后端均为主流生产级数据库：Elasticsearch、PgVector、Milvus、Pinecone、Qdrant、Weaviate 等
- 这些数据库本身提供集群、分片、副本等高可用机制

---

## 7. 安全设计

### 7.1 Guardrails 框架
- `InputGuardrail` 在 LLM 调用前拦截，可防止 prompt injection、敏感信息泄露
- `OutputGuardrail` 在 LLM 响应后验证，可过滤不当内容
- 支持 `GuardrailResult` 状态机：SUCCESS / FAILURE / RECOVER
- 内置 `ChatExecutor` 抽象允许护栏内部二次调用 LLM 进行验证

### 7.2 内置护栏 (`langchain4j-guardrails`)
- `JsonExtractorOutputGuardrail` — 验证 LLM 响应可反序列化为 JSON，失败时自动重试
- `MessageModeratorInputGuardrail` — 使用 ModerationModel 验证用户消息的安全性

### 7.3 Moderation Model
- `AiServices.builder().moderationModel()` 支持绑定内容审核模型
- OpenAI 等提供商提供专门的 Moderation API 集成

### 7.4 依赖安全
- 项目避免引入不必要的依赖（CONTRIBUTING.md 明确要求）
- 使用 `mvn dependency:analyze` 检查多余依赖
- 无硬编码凭证，所有 API Key 通过构造参数传入

### 7.5 局限性
- Guardrails 是可选的，不是强制的 — 用户需自行配置
- 无内置的输入长度限制或速率限制
- API Key 管理依赖用户应用层

---

## 8. 测试与质量

### 8.1 测试规模
| 指标 | 数量 |
|------|------|
| 单元测试文件 | ~506 个 |
| 集成测试文件 | ~484 个 |
| 总测试文件 | ~990 个 |
| 测试目录 | `src/test/java` |

### 8.2 测试基础设施
- **Testcontainers**: 用于集成测试的数据库容器化（Chroma, PgVector, Elasticsearch 等）
- **JUnit 5**: 测试框架（JUnit 6.0.1 版本管理）
- **Mockito**: Mock 框架（5.19.0）
- **抽象测试基类**: `AbstractChatModelIT`, `AbstractStreamingChatModelIT`, `AbstractAiServiceWithToolsIT`, `EmbeddingStoreWithFilteringIT` 等 — 确保各提供商实现一致性
- **langchain4j-test**: GuardrailResult 断言工具

### 8.3 CI/CD 流水线 (`.github/workflows/main.yaml`)

```
compile_and_unit_test (JDK 17, 21, 25)  →  integration_test (JDK 21)
```

- 单元测试: `mvn -B -U -DembeddingsSkipCache -T8C test javadoc:aggregate`
- 集成测试: `continue-on-error: true`，需要 API Key 环境变量
- JUnit 报告: `mikepenz/action-junit-report` 自动注释测试失败

### 8.4 代码质量工具
- **Spotless** (2.44.4): 代码格式化，使用 Palantir Java Format，`ratchetFrom: origin/main`
- **JaCoCo**: 代码覆盖率检查
- **Detekt**: Kotlin 静态分析
- **Maven Enforcer**: 依赖版本一致性
- **Android Lint**: `android-maven-plugin` 进行 Android 兼容性检查

### 8.5 贡献规范
- Java 17 兼容性要求
- 遵循 Google's Best Practices for Java Libraries
- "no tests, no review" — 必须编写测试
- 使用官方 SDK 集成第三方服务
- 避免破坏性变更，标记 `@Deprecated` 代替删除

---

## 9. 生态与社区

### 9.1 核心生态
- **20+ LLM 提供商**: OpenAI, Anthropic, Azure OpenAI, Google AI Gemini, Vertex AI, AWS Bedrock, Ollama, Mistral, Cohere, HuggingFace, WatsonX, LocalAI, OVH AI, Workers AI 等
- **21 嵌入存储**: Chroma, Pinecone, Milvus, Qdrant, Weaviate, PgVector, Elasticsearch, MongoDB Atlas, Azure AI Search, Cassandra, Couchbase, Coherence, Oracle, Infinispan, MariaDB, OpenSearch, Tablestore, Vespa 等
- **11 本地 Embedding**: 基于 ONNX 的本地向量化模型 (含量化版)
- **文档处理**: 7 个加载器 (S3, Azure, GitHub, Selenium, Playwright, Tencent COS, GCS) + 5 个解析器 (PDF, POI, Tika, Markdown, YAML) + 1 个转换器 (Jsoup)
- **Web 搜索**: 3 个搜索引擎 (Google Custom, Tavily, SearchAPI)
- **代码执行**: 3 个引擎 (GraalVM, Judge0, Azure Acads)
- **MCP**: Model Context Protocol 客户端 (4 种传输层)
- **Agent**: Agentic 编排 (8 种工作流注解) + GOAP/P2P 规划模式
- **Skills**: Agent Skills 框架 (agentskills.io 规范)
- **可观测性**: Micrometer 指标 + Observation 追踪
- **框架集成**: Spring Boot (`langchain4j-spring`), Quarkus (`quarkus-langchain4j`), Micronaut, Helidon

### 9.2 社区渠道
- **GitHub Discussions**: 问题讨论和功能请求
- **Discord**: 实时社区交流
- **BlueSky / X (Twitter)**: `@langchain4j` 官方账号
- **文档站**: docs.langchain4j.dev
- **文档聊天机器人**: chat.langchain4j.dev (实验性)

### 9.3 版本管理
- **BOM (Bill of Materials)**: `langchain4j-bom` 统一管理版本
- 双版本体系: `stable.version` (1.13.0-SNAPSHOT) 和 `beta.version` (1.13.0-beta23-SNAPSHOT)
- 活跃发布，SNAPSHOT 版本号表明持续迭代

### 9.4 新集成规范
- 新模型/嵌入存储集成建议先提交到 `langchain4j-community` 仓库
- 必须扩展指定的抽象测试基类（如 `AbstractChatModelIT`）
- 需要同步更新 BOM、README、文档站、示例仓库

---

## 10. 优缺点总结

### 优点
1. **统一抽象**: 20+ LLM 和 30+ 向量数据库的统一 API，切换提供商零代码改动
2. **AiServices 注解驱动**: `@SystemMessage`, `@UserMessage`, `@Tool` 注解极大简化 AI 应用开发
3. **模块化设计**: 80+ Maven 模块按需引入，避免依赖膨胀
4. **全面的 RAG 支持**: 从文档加载、解析、分块、嵌入到检索的完整管道
5. **Agentic 编排**: 8 种工作流注解 + GOAP/P2P 规划模式，媲美 Spring AI Alibaba 的 Agent 能力
6. **MCP 协议支持**: 4 种传输层，可连接外部 MCP 服务器获取工具
7. **成熟的可观测性**: Listener 体系覆盖 ChatModel、EmbeddingStore、AiService 全链路
8. **生产级安全**: Guardrails 框架提供输入/输出验证机制
9. **强大的扩展性**: SPI 机制 + 工厂模式，易于添加新提供商
10. **完善的测试**: 抽象测试基类确保各提供商实现一致性
11. **活跃的社区**: 频繁更新，多框架集成（Spring, Quarkus, Micronaut）

### 缺点
1. **Java 生态限制**: 仅支持 Java/Kotlin，Python 生态（LangChain）更成熟
2. **文档覆盖不均**: 部分实验性模块（Agentic, MCP, Skills）文档较少
3. **模块成熟度差异**: 核心模块稳定，但实验性模块仍在快速迭代
4. **无内置认证管理**: API Key 管理完全依赖用户实现
5. **Guardrails 非强制**: 安全机制是可选的，容易被忽略
6. **学习曲线**: AiServices + RAG + Tool + Memory + Agentic 的组合配置较复杂
7. **响应式支持有限**: 主要面向同步模型，Reactor/RxJava 集成不够深入

---

## 11. 适用场景与选型建议

### 11.1 最佳适用场景
- **Java 企业应用 AI 集成**: 需要在现有 Spring Boot / Quarkus / Micronaut 应用中集成 LLM
- **RAG 应用**: 需要从企业文档/知识库中检索信息并增强 LLM 回答
- **多提供商切换**: 需要在 OpenAI、Anthropic、Gemini 等提供商之间灵活切换
- **工具调用/Agent**: 需要 LLM 调用外部 API/数据库/工具
- **多 Agent 编排**: 需要 Supervisor、Planner、循环等复杂 Agent 工作流
- **内容审核应用**: 需要对 LLM 输入输出进行安全验证

### 11.2 不适用场景
- Python 为主的技术栈（应选 LangChain / LlamaIndex）
- 需要极致性能的实时推理场景（应直接使用提供商 SDK）
- 纯前端 AI 应用（应选 LangChain.js）

### 11.3 与竞品对比

| 维度 | LangChain4j | Spring AI | Semantic Kernel (Java) |
|------|-------------|-----------|----------------------|
| **定位** | 通用 Java AI 框架 | Spring 生态 AI | Microsoft 生态 AI |
| **依赖** | JDK 17+ | Spring Boot 3+ | JDK 17+ |
| **提供商数量** | 20+ | 10+ | 主要 Azure |
| **RAG 支持** | 完整管道 | 完整管道 | 基础支持 |
| **Agent 支持** | 实验性 (8种工作流) | 基础 | 成熟 |
| **MCP 支持** | 有 (4种传输层) | 有 | 无 |
| **社区活跃度** | 高 (GitHub 16k+ stars) | 高 (Spring 背书) | 中 |
| **框架耦合** | 低 (纯 Java) | 中 (依赖 Spring) | 中 (依赖 Azure) |

**选型建议：**
- 纯 Java / Kotlin 项目、需要多提供商支持 → **LangChain4j**
- 已有 Spring Boot 技术栈 → **Spring AI** 或 LangChain4j
- Microsoft/Azure 生态 → **Semantic Kernel**
- 需要最广泛的提供商和向量数据库选择 → **LangChain4j**

---

## 12. 集成与使用指南

### 12.1 Maven 依赖

```xml
<!-- 引入 BOM 统一管理版本 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j-bom</artifactId>
            <version>1.13.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- 核心依赖 -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
</dependency>

<!-- LLM 提供商 (以 OpenAI 为例) -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
</dependency>
```

### 12.2 快速开始示例

```java
// 1. 创建 ChatModel
ChatModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4o")
    .build();

// 2. 定义 AI 服务接口
interface Assistant {
    @SystemMessage("You are a helpful assistant")
    String chat(@UserMessage String message);
}

// 3. 创建并使用
Assistant assistant = AiServices.create(Assistant.class, model);
System.out.println(assistant.chat("What is the capital of France?"));
```

### 12.3 带 Tool 和 Memory 的示例

```java
interface WeatherAssistant {
    @SystemMessage("You are a weather assistant")
    String chat(@UserMessage String message, @MemoryId String userId);
}

class WeatherTools {
    @Tool("Get current temperature for a city")
    double getTemperature(String city) { /* call weather API */ return 20.0; }
}

ChatMemoryProvider memoryProvider = userId ->
    MessageWindowChatMemory.withMaxMessages(10);

WeatherAssistant assistant = AiServices.builder(WeatherAssistant.class)
    .chatModel(model)
    .chatMemoryProvider(memoryProvider)
    .tools(new WeatherTools())
    .build();
```

### 12.4 Agentic 工作流示例

```java
// Supervisor 模式 — LLM 自动选择调用哪个子 Agent
@SupervisorAgent(supervisor = SupervisorLLM.class)
interface CustomerSupport {
    @Agent BillingAgent billing;
    @Agent TechAgent tech;
    @Agent GeneralAgent general;
}

// Sequence + Loop — 多步骤循环执行
@LoopAgent(maxIterations = 5)
interface RefinementLoop {
    @Agent DraftAgent draft;
    @Agent ReviewAgent review;
}
```

---

## 13. 二次开发指南

### 13.1 添加新的 LLM 提供商

1. **创建 Maven 模块**: 如 `langchain4j-my-provider`
2. **实现核心接口**:
   - `ChatModel` — 实现 `doChat(ChatRequest)` 方法
   - `StreamingChatModel` — 实现 `doChat(ChatRequest, StreamingChatHandler)` 方法
   - `EmbeddingModel` — 实现 `embedAll(List<TextSegment>)` 方法
3. **使用 Builder 模式**: 遵循项目现有的 `XxxChatModel.builder()` 风格
4. **编写测试**:
   - 单元测试覆盖基本逻辑
   - 集成测试继承 `AbstractChatModelIT` 等基类
5. **SPI 注册** (如需): 在 `META-INF/services/` 注册工厂
6. **更新 BOM**: 在 `langchain4j-bom/pom.xml` 添加新模块

### 13.2 SPI 扩展点

- `AiServicesFactory` — 自定义 AiServices 创建逻辑
- `ServiceOutputParser` — 自定义响应解析
- `TokenStreamAdapter` — 自定义流式适配
- `ModelProvider` — 模型提供商发现
- `DocumentSplitterFactory` — 自定义文档分块
- `DocumentParserFactory` — 自定义文档解析
- `PromptTemplateFactory` — 自定义提示模板
- `JsonCodecFactory` — 自定义 JSON 编解码

### 13.3 贡献流程

1. Fork → 创建功能分支
2. 遵循 Java 17 兼容性、Google 代码规范
3. 编写测试（单元 + 集成）
4. 运行 `make lint` 和 `make format`
5. 提交 Draft PR → 审核通过后标记 Ready
6. 更新文档和示例

---

## 14. 综合评分

| 维度 | 评分 (1-10) | 说明 |
|------|------------|------|
| **功能完整性** | 9.0 | 覆盖 LLM 集成全链路：模型、记忆、RAG、工具、Agent、护栏 |
| **架构设计** | 9.0 | 清晰分层，接口驱动，SPI 扩展，设计模式运用得当 |
| **代码质量** | 8.5 | Spotless 统一格式，遵循 Google 规范，但部分模块复杂度偏高 |
| **测试覆盖** | 8.5 | 990 个测试文件，抽象基类保证一致性，Testcontainers 集成测试 |
| **文档质量** | 7.0 | 核心文档完善，但实验性模块文档不足，部分 API 缺少 Javadoc |
| **社区活跃度** | 9.0 | GitHub 高 star，活跃 Discord，频繁发布，Spring/Quarkus 集成 |
| **性能表现** | 7.5 | 流式支持良好，但缺乏内置缓存和连接池优化，依赖各提供商实现 |
| **安全性** | 7.5 | Guardrails 框架设计合理但非强制，无内置认证管理 |
| **可扩展性** | 9.5 | SPI 机制 + Builder 模式，极易扩展新提供商和功能 |
| **生态丰富度** | 9.5 | 20+ LLM、30+ 向量库、完整文档处理管道、多框架集成 |
| **综合评分** | **8.5** | Java 生态最成熟的 LLM 集成框架，适合企业级 AI 应用开发 |

---

*分析基于 LangChain4j v1.13.0-beta23-SNAPSHOT 源码，更新于 2026-04-15。*
