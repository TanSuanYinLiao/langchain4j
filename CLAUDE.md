# LangChain4j 项目全面分析

> 基于项目源码 (v1.13.0-beta23-SNAPSHOT) 的深度技术分析，共 12 个维度。

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
│              用户应用层 (AiServices)              │
│    @SystemMessage / @UserMessage / @Tool 注解     │
├─────────────────────────────────────────────────┤
│            langchain4j (高层 API)                │
│    DefaultAiServices, ServiceOutputParser        │
├─────────────────────────────────────────────────┤
│           langchain4j-core (核心接口)             │
│  ChatModel, EmbeddingStore, ChatMemory, Tool,    │
│  RetrievalAugmentor, Guardrail                   │
├─────────────────────────────────────────────────┤
│        Provider 模块 (具体实现)                   │
│  OpenAI, Anthropic, Ollama, Azure, Gemini, ...   │
├─────────────────────────────────────────────────┤
│        基础设施层 (HTTP Client, SPI)              │
│  langchain4j-http-client, ServiceHelper           │
└─────────────────────────────────────────────────┘
```

- **langchain4j-core**: 纯接口层，定义 `ChatModel`、`EmbeddingModel`、`EmbeddingStore`、`ChatMemory`、`RetrievalAugmentor` 等核心契约，零实现依赖
- **langchain4j**: 高层 API 层，核心是 `AiServices` 动态代理机制和 `DefaultAiServices` 实现
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

### 2.3 模块组织

聚合 POM (`/pom.xml`) 将 80+ 模块按功能分类：
- **核心**: `langchain4j-core`, `langchain4j`, `langchain4j-test`
- **HTTP 客户端**: `langchain4j-http-client`, `langchain4j-http-client-jdk`, `langchain4j-http-client-apache`
- **模型提供商**: 20+ 模块 (OpenAI, Anthropic, Azure, Bedrock, Gemini, Ollama 等)
- **嵌入存储**: 15+ 模块 (Chroma, Pinecone, Milvus, PgVector, Elasticsearch 等)
- **文档处理**: Loaders (S3, Azure, GitHub, Selenium), Parsers (PDF, POI, Tika, Markdown), Transformers
- **实验性**: `langchain4j-agentic`, `langchain4j-mcp`, `langchain4j-skills`, SQL, Hibernate
- **可观测性**: `langchain4j-micrometer-metrics`, `langchain4j-observation`

---

## 3. 核心源码解读

### 3.1 ChatModel 接口 (`langchain4j-core/.../model/chat/ChatModel.java`)

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

### 3.2 AiServices — 动态代理核心

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

### 3.3 DefaultAiServices 实现

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

### 3.4 SPI 机制 (`langchain4j-core/.../spi/ServiceHelper.java`)

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

### 3.5 Tool/Function 调用

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

### 3.6 RAG 管道

- **入口**: `RetrievalAugmentor.augment(AugmentationRequest)` → `AugmentationResult`
- **检索**: `ContentRetriever` 接口从 `EmbeddingStore` 检索相关内容
- **嵌入**: `EmbeddingModel` 接口 (`embed()`, `embedAll()`, `dimension()`)
- **存储**: `EmbeddingStore<Embedded>` 接口提供 `add()`, `search()`, `removeAll()` 操作
- **文档处理**: `DocumentParser` → `DocumentSplitter` → `EmbeddingModel` → `EmbeddingStore`

### 3.7 ChatMemory

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

### 3.8 Guardrails

护栏框架位于 `langchain4j-core/.../guardrail/` 包：
- `InputGuardrail` — 输入验证（在 LLM 调用前执行）
- `OutputGuardrail` — 输出验证（在 LLM 响应后执行）
- `GuardrailResult` — 验证结果（成功/失败/重试）
- `ChatExecutor` — 护栏内部调用 LLM 的抽象

---

## 4. 性能与高可用

### 4.1 流式处理
- `StreamingChatModel` 接口支持逐 token 流式输出
- `StreamingChatResponseHandler` 回调处理部分响应、思考过程、工具调用
- 通过 `TokenStreamAdapter` (SPI) 实现 Reactor 等响应式集成

### 4.2 连接管理
- 各提供商模块独立管理 HTTP 连接池
- `langchain4j-http-client` 提供统一 HTTP 客户端抽象
- 子模块 `langchain4j-http-client-jdk`（JDK HttpClient）和 `langchain4j-http-client-apache`（Apache HttpClient）
- 各提供商可选择使用官方 SDK 或自定义 HTTP 客户端

### 4.3 并发构建
- Maven 构建使用 `-T8C`（每个 CPU 核心一个线程）加速多模块编译
- CI 矩阵测试 JDK 17、21、25 三个版本

### 4.4 Embedding Store 高可用
- 支持的后端均为主流生产级数据库：Elasticsearch、PgVector、Milvus、Pinecone、Qdrant、Weaviate 等
- 这些数据库本身提供集群、分片、副本等高可用机制

---

## 5. 安全设计

### 5.1 Guardrails 框架
- `InputGuardrail` 在 LLM 调用前拦截，可防止 prompt injection、敏感信息泄露
- `OutputGuardrail` 在 LLM 响应后验证，可过滤不当内容
- 支持 `GuardrailResult` 状态机：SUCCESS / FAILURE / RECOVER
- 内置 `ChatExecutor` 抽象允许护栏内部二次调用 LLM 进行验证

### 5.2 Moderation Model
- `AiServices.builder().moderationModel()` 支持绑定内容审核模型
- OpenAI 等提供商提供专门的 Moderation API 集成

### 5.3 依赖安全
- 项目避免引入不必要的依赖（CONTRIBUTING.md 明确要求）
- 使用 `mvn dependency:analyze` 检查多余依赖
- 无硬编码凭证，所有 API Key 通过构造参数传入

### 5.4 局限性
- Guardrails 是可选的，不是强制的 — 用户需自行配置
- 无内置的输入长度限制或速率限制
- API Key 管理依赖用户应用层

---

## 6. 测试与质量

### 6.1 测试规模
| 指标 | 数量 |
|------|------|
| 单元测试文件 | ~506 个 |
| 集成测试文件 | ~484 个 |
| 总测试文件 | ~990 个 |
| 测试目录 | `src/test/java` |

### 6.2 测试基础设施
- **Testcontainers**: 用于集成测试的数据库容器化（Chroma, PgVector, Elasticsearch 等）
- **JUnit 5**: 测试框架（JUnit 6.0.1 版本管理）
- **Mockito**: Mock 框架（5.19.0）
- **抽象测试基类**: `AbstractChatModelIT`, `AbstractStreamingChatModelIT`, `AbstractAiServiceWithToolsIT`, `EmbeddingStoreWithFilteringIT` 等 — 确保各提供商实现一致性

### 6.3 CI/CD 流水线 (`.github/workflows/main.yaml`)

```
compile_and_unit_test (JDK 17, 21, 25)  →  integration_test (JDK 21)
```

- 单元测试: `mvn -B -U -DembeddingsSkipCache -T8C test javadoc:aggregate`
- 集成测试: `continue-on-error: true`，需要 API Key 环境变量
- JUnit 报告: `mikepenz/action-junit-report` 自动注释测试失败

### 6.4 代码质量工具
- **Spotless** (2.44.4): 代码格式化，使用 Palantir Java Format，`ratchetFrom: origin/main`
- **JaCoCo**: 代码覆盖率检查
- **Detekt**: Kotlin 静态分析
- **Maven Enforcer**: 依赖版本一致性
- **Android Lint**: `android-maven-plugin` 进行 Android 兼容性检查

### 6.5 贡献规范
- Java 17 兼容性要求
- 遵循 Google's Best Practices for Java Libraries
- "no tests, no review" — 必须编写测试
- 使用官方 SDK 集成第三方服务
- 避免破坏性变更，标记 `@Deprecated` 代替删除

---

## 7. 生态与社区

### 7.1 核心生态
- **20+ LLM 提供商**: OpenAI, Anthropic, Azure OpenAI, Google AI Gemini, Vertex AI, AWS Bedrock, Ollama, Mistral, Cohere, HuggingFace, WatsonX, LocalAI, OVH AI, Workers AI 等
- **30+ 嵌入存储**: Chroma, Pinecone, Milvus, Qdrant, Weaviate, PgVector, Elasticsearch, MongoDB Atlas, Azure AI Search, Cassandra, Couchbase, Coherence, Oracle, Infinispan, MariaDB, OpenSearch, Tablestore, Vespa 等
- **文档处理**: S3, Azure Blob, GitHub, Selenium, Playwright, Tencent COS, Google Cloud Storage 加载器；PDF, POI, Tika, Markdown, YAML 解析器
- **框架集成**: Spring Boot (`langchain4j-spring`), Quarkus (`quarkus-langchain4j`), Micronaut, Helidon
- **实验性**: Agentic 模式 (Agent, Supervisor, Workflow, Planner), MCP 协议, Skills 框架

### 7.2 社区渠道
- **GitHub Discussions**: 问题讨论和功能请求
- **Discord**: 实时社区交流
- **BlueSky / X (Twitter)**: `@langchain4j` 官方账号
- **文档站**: docs.langchain4j.dev
- **文档聊天机器人**: chat.langchain4j.dev (实验性)

### 7.3 版本管理
- **BOM (Bill of Materials)**: `langchain4j-bom` 统一管理版本
- 双版本体系: `stable.version` (1.13.0-SNAPSHOT) 和 `beta.version` (1.13.0-beta23-SNAPSHOT)
- 活跃发布，SNAPSHOT 版本号表明持续迭代

### 7.4 新集成规范
- 新模型/嵌入存储集成建议先提交到 `langchain4j-community` 仓库
- 必须扩展指定的抽象测试基类（如 `AbstractChatModelIT`）
- 需要同步更新 BOM、README、文档站、示例仓库

---

## 8. 优缺点总结

### 优点
1. **统一抽象**: 20+ LLM 和 30+ 向量数据库的统一 API，切换提供商零代码改动
2. **AiServices 注解驱动**: `@SystemMessage`, `@UserMessage`, `@Tool` 注解极大简化 AI 应用开发
3. **模块化设计**: 80+ Maven 模块按需引入，避免依赖膨胀
4. **全面的 RAG 支持**: 从文档加载、解析、分块、嵌入到检索的完整管道
5. **成熟的可观测性**: Listener 体系覆盖 ChatModel、EmbeddingStore、AiService 全链路
6. **生产级安全**: Guardrails 框架提供输入/输出验证机制
7. **强大的扩展性**: SPI 机制 + 工厂模式，易于添加新提供商
8. **完善的测试**: 抽象测试基类确保各提供商实现一致性
9. **活跃的社区**: 频繁更新，多框架集成（Spring, Quarkus, Micronaut）

### 缺点
1. **Java 生态限制**: 仅支持 Java/Kotlin，Python 生态（LangChain）更成熟
2. **文档覆盖不均**: 部分实验性模块（Agentic, MCP, Skills）文档较少
3. **模块成熟度差异**: 核心模块稳定，但实验性模块仍在快速迭代
4. **无内置认证管理**: API Key 管理完全依赖用户实现
5. **Guardrails 非强制**: 安全机制是可选的，容易被忽略
6. **学习曲线**: AiServices + RAG + Tool + Memory 的组合配置较复杂
7. **响应式支持有限**: 主要面向同步模型，Reactor/RxJava 集成不够深入

---

## 9. 适用场景与选型建议

### 9.1 最佳适用场景
- **Java 企业应用 AI 集成**: 需要在现有 Spring Boot / Quarkus / Micronaut 应用中集成 LLM
- **RAG 应用**: 需要从企业文档/知识库中检索信息并增强 LLM 回答
- **多提供商切换**: 需要在 OpenAI、Anthropic、Gemini 等提供商之间灵活切换
- **工具调用/Agent**: 需要 LLM 调用外部 API/数据库/工具
- **内容审核应用**: 需要对 LLM 输入输出进行安全验证

### 9.2 不适用场景
- Python 为主的技术栈（应选 LangChain / LlamaIndex）
- 需要极致性能的实时推理场景（应直接使用提供商 SDK）
- 纯前端 AI 应用（应选 LangChain.js）

### 9.3 与竞品对比

| 维度 | LangChain4j | Spring AI | Semantic Kernel (Java) |
|------|-------------|-----------|----------------------|
| **定位** | 通用 Java AI 框架 | Spring 生态 AI | Microsoft 生态 AI |
| **依赖** | JDK 17+ | Spring Boot 3+ | JDK 17+ |
| **提供商数量** | 20+ | 10+ | 主要 Azure |
| **RAG 支持** | 完整管道 | 完整管道 | 基础支持 |
| **Agent 支持** | 实验性 | 基础 | 成熟 |
| **社区活跃度** | 高 (GitHub 16k+ stars) | 高 (Spring 背书) | 中 |
| **框架耦合** | 低 (纯 Java) | 中 (依赖 Spring) | 中 (依赖 Azure) |

**选型建议：**
- 纯 Java / Kotlin 项目、需要多提供商支持 → **LangChain4j**
- 已有 Spring Boot 技术栈 → **Spring AI** 或 LangChain4j
- Microsoft/Azure 生态 → **Semantic Kernel**
- 需要最广泛的提供商和向量数据库选择 → **LangChain4j**

---

## 10. 集成与使用指南

### 10.1 Maven 依赖

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

### 10.2 快速开始示例

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

### 10.3 带 Tool 和 Memory 的示例

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

---

## 11. 二次开发指南

### 11.1 添加新的 LLM 提供商

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

### 11.2 SPI 扩展点

- `AiServicesFactory` — 自定义 AiServices 创建逻辑
- `ServiceOutputParser` — 自定义响应解析
- `TokenStreamAdapter` — 自定义流式适配
- `ModelProvider` — 模型提供商发现

### 11.3 贡献流程

1. Fork → 创建功能分支
2. 遵循 Java 17 兼容性、Google 代码规范
3. 编写测试（单元 + 集成）
4. 运行 `make lint` 和 `make format`
5. 提交 Draft PR → 审核通过后标记 Ready
6. 更新文档和示例

---

## 12. 综合评分

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

## 13. StreamingChatResponseHandler 流式事件详解

> 源文件：`langchain4j-core/.../model/chat/response/StreamingChatResponseHandler.java`

### 事件总览

`StreamingChatResponseHandler` 是流式聊天模型的回调接口，定义了 **6 个事件**（3 个维度 × 每个维度 2 个重载方法）+ **2 个生命周期事件**。共 8 个方法。

### 事件时序图

```
StreamingChatModel.chat(request, handler)
  │
  ├─ 重复触发（每个 token）:
  │   ├── onPartialThinking()      ← 1. 推理/思考过程的 token
  │   ├── onPartialResponse()      ← 2. 正式回复的 token
  │   └── onPartialToolCall()      ← 3. 工具调用参数的 token
  │
  ├─ 单次触发（工具调用完成）:
  │   └── onCompleteToolCall()     ← 4. 单个工具调用组装完成
  │
  └─ 单次触发（流结束）:
      ├── onCompleteResponse()     ← 5. 整个响应完成（必须实现）
      └── onError()                ← 6. 发生错误（必须实现）
```

### 逐事件详解

#### 1. `onPartialResponse` — 正式回复 token

| 项 | 说明 |
|---|------|
| **触发时机** | 模型生成正式回复文本，通常每个 token 触发一次 |
| **承载数据** | `PartialResponse`，内部只有一个 `text` 字段 |
| **两种重载** | `onPartialResponse(String)` — 简单字符串版本 |
| | `onPartialResponse(PartialResponse, PartialResponseContext)` — 带上下文版本 |
| **Context 提供** | `StreamingHandle` — 可通过 `cancel()` 中止流式传输 |
| **注意** | 部分 provider（如 Bedrock、Google）不逐 token 流式，而是批量返回，此时一次回调可能包含多个 token |

```java
// 简单用法
handler.onPartialResponse(String partialResponse) { ... }

// 带取消能力
handler.onPartialResponse(PartialResponse resp, PartialResponseContext ctx) {
    String token = resp.text();
    if (shouldStop) ctx.streamingHandle().cancel();
}
```

#### 2. `onPartialThinking` — 推理/思考 token

| 项 | 说明 |
|---|------|
| **触发时机** | 模型生成思考/推理过程时（如 Claude 的 extended thinking、DeepSeek 的 reasoning） |
| **承载数据** | `PartialThinking`，内部只有一个 `text` 字段 |
| **两种重载** | `onPartialThinking(PartialThinking)` — 简单版本 |
| | `onPartialThinking(PartialThinking, PartialThinkingContext)` — 带 `StreamingHandle` 版本 |
| **注意** | 并非所有模型都支持 thinking，不支持的模型不会触发此事件 |
| **@since** | 1.2.0 |

```java
handler.onPartialThinking(PartialThinking thinking) {
    System.out.print("[思考] " + thinking.text());
}
```

#### 3. `onPartialToolCall` — 工具调用参数 token

| 项 | 说明 |
|---|------|
| **触发时机** | 模型以流式方式生成工具调用的参数 JSON 时 |
| **承载数据** | `PartialToolCall`，包含 4 个字段 |
| **核心字段** | `index` — 工具调用序号（从 0 递增，用于关联多次调用） |
| | `id` — provider 生成的唯一标识（部分 provider 可能省略） |
| | `name` — 被调用的工具名称 |
| | `partialArguments` — 参数 JSON 的片段 |
| **注意** | 部分 provider（Bedrock、Google、Mistral、Ollama）不逐 token 流式传输工具调用参数，而是直接返回完整工具调用，此时此事件不会触发，仅触发 `onCompleteToolCall` |

```java
// 流式工具调用示例：
// 1. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="{\"")
// 2. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="city")
// 3. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="\":\"")
// 4. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="Mun")
// 5. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="ich")
// 6. onPartialToolCall(index=0, id="call_abc", name="get_weather", partialArguments="\"}")
// 7. onCompleteToolCall(index=0, id="call_abc", name="get_weather", arguments="{\"city\":\"Munich\"}")
```

#### 4. `onCompleteToolCall` — 单个工具调用完成

| 项 | 说明 |
|---|------|
| **触发时机** | 模型完成了一个工具调用的流式生成（即所有 `partialArguments` 已拼装完整） |
| **承载数据** | `CompleteToolCall`，包含 `index` + `ToolExecutionRequest` |
| **ToolExecutionRequest** | 包含 `id`、`name`、`arguments`（完整 JSON 字符串），可直接用于执行 |
| **多次触发** | 如果模型决定调用多个工具，每个工具完成时都会触发一次 |
| **@since** | 1.2.0 |

#### 5. `onCompleteResponse` — 整个响应完成（必须实现）

| 项 | 说明 |
|---|------|
| **触发时机** | 模型完成了整个流式响应 |
| **承载数据** | `ChatResponse`，包含完整的 `AiMessage` + `ChatResponseMetadata` |
| **AiMessage** | 包含完整文本 + thinking + 所有 `ToolExecutionRequest` 列表 |
| **Metadata** | 包含 `id`、`modelName`、`tokenUsage`、`finishReason` |
| **必须实现** | 此方法没有 default 实现，是唯一必须实现的方法 |

#### 6. `onError` — 流式传输错误（必须实现）

| 项 | 说明 |
|---|------|
| **触发时机** | 流式传输过程中发生错误 |
| **承载数据** | `Throwable` 异常对象 |
| **必须实现** | 此方法没有 default 实现，是唯一必须实现的方法 |

### StreamingHandle — 流控制

```java
// 通过 Context 可获取 StreamingHandle
public interface StreamingHandle {
    void cancel();          // 中止流式传输
    boolean isCancelled();  // 查询是否已中止
}
```

可用于在 `onPartialResponse` 或 `onPartialThinking` 的带 Context 版本中提前终止流。

### 方法实现要求总结

| 方法 | 是否必须实现 | @since | @Experimental |
|------|:---:|------:|:---:|
| `onPartialResponse(String)` | 否 (default 空实现) | — | 否 |
| `onPartialResponse(PartialResponse, PartialResponseContext)` | 否 (default 委托到 String 版本) | 1.8.0 | 是 |
| `onPartialThinking(PartialThinking)` | 否 (default 空实现) | 1.2.0 | 是 |
| `onPartialThinking(PartialThinking, PartialThinkingContext)` | 否 (default 委托到无 Context 版本) | 1.8.0 | 是 |
| `onPartialToolCall(PartialToolCall)` | 否 (default 空实现) | 1.2.0 | 是 |
| `onPartialToolCall(PartialToolCall, PartialToolCallContext)` | 否 (default 委托到无 Context 版本) | 1.8.0 | 是 |
| `onCompleteToolCall(CompleteToolCall)` | 否 (default 空实现) | 1.2.0 | 是 |
| **`onCompleteResponse(ChatResponse)`** | **是** | — | 否 |
| **`onError(Throwable)`** | **是** | — | 否 |

---

*分析基于 LangChain4j v1.13.0-beta23-SNAPSHOT 源码，生成于 2026-04-07。*
