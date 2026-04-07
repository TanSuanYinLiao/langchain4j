# langchain4j 模块 (高层 API)

## 模块职责
面向用户的**高层 API 模块**。提供 `AiServices` 动态代理机制，通过 JDK Dynamic Proxy 将用户定义的 Java 接口自动映射为 LLM 调用。依赖 `langchain4j-core`（接口层），是整个框架最核心的用户交互层。

## 核心类一览

| 类 | 职责 |
|----|------|
| `AiServices` | 抽象工厂 + Builder，`create(Class<T>, ChatModel)` 创建代理 |
| `DefaultAiServices` | 核心实现，`build()` 创建 JDK Proxy，`invoke()` 是 InvocationHandler |
| `AiServiceContext` | 状态容器：chatModel, streamingChatModel, chatMemory, tools, guardrails 等 |
| `ServiceOutputParser` | 解析 LLM 文本为 Java 类型（String, enum, POJO, List, Set），生成 JSON Schema |
| `ToolService` | 管理 @Tool 注册、ToolSpecification 创建、工具执行循环 |
| `AiServiceTokenStream` | 流式响应 TokenStream 实现 |
| `ChatMemoryService` | ChatMemory 实例管理（单实例或按 memoryId 分实例） |
| `GuardrailService` | 执行输入/输出护栏 |

## 完整执行流程

### 阶段 1: 创建代理

```
AiServices.create(MyService.class, chatModel)
  → AiServices.builder(MyService.class).chatModel(chatModel).build()
  → DefaultAiServices.build()
      ├── 校验：至少有 ChatModel 或 StreamingChatModel
      ├── 校验：如有 tools，模型必须支持
      ├── 初始化 ToolService (@Tool 对象 + ToolProvider)
      ├── 初始化 GuardrailService (builder 配置 + 注解)
      ├── 初始化 ChatMemoryService (单实例或 ChatMemoryProvider)
      ├── 注册 AiServiceEventListeners
      └── Proxy.newProxyInstance(classLoader, [MyService.class], this::invoke)
```

### 阶段 2: 调用（用户调用 service.chat("Hello") 时）

```
invoke(proxy, method, args)
  │
  ├── 1. 从 @MemoryId 注解解析 memoryId（默认 "default"）
  ├── 2. 获取/创建 ChatMemory
  ├── 3. 构建 SystemMessage（@SystemMessage 注解 / systemMessageProvider）
  ├── 4. 构建 UserMessage（@UserMessage 注解 + 模板变量替换）
  ├── 5. 触发 AiServiceStartedEvent
  ├── 6. RAG 增强（如配置 retrievalAugmentor）
  │     retrievalAugmentor.augment(request) → 替换 UserMessage
  ├── 7. 添加多模态内容（图片/音频/视频/PDF）
  ├── 8. 执行 InputGuardrails
  │     失败 → 抛出异常
  ├── 9. 判断执行路径：同步 or 流式
  │
  ├── [同步路径]
  │   ├── 10. 生成输出格式指令（JSON Schema 或 文本提示）
  │   ├── 11. 构建消息列表 [SystemMessage, ChatMemory.messages(), UserMessage]
  │   ├── 12. 触发 moderation（@Moderate 注解）
  │   ├── 13. 构建 ChatRequest
  │   ├── 14. chatModel.chat(chatRequest) → ChatResponse
  │   ├── 15. ToolService 工具执行循环:
  │   │     while (aiMessage.hasToolExecutionRequests()) {
  │   │       执行工具 → ToolExecutionResultMessage
  │   │       更新 ChatMemory
  │   │       chatModel.chat(newChatRequest) → 新 AiMessage
  │   │     }
  │   ├── 16. 执行 OutputGuardrails（支持 RETRY/REPROMPT）
  │   └── 17. ServiceOutputParser.parse(response, returnType)
  │
  ├── [流式路径]
  │   └── AiServiceTokenStream.start()
  │       ├── streamingChatModel.chat(chatRequest, handler)
  │       ├── onPartialResponse → 用户回调
  │       ├── onCompleteResponse → 执行工具 + 解析响应
  │       └── onError → 用户回调
  │
  ├── 18. 包装 Result<T>（如返回类型为 Result）
  ├── 19. 触发 AiServiceCompletedEvent
  v
返回解析后的 Java 对象
```

### 工具执行循环详解

```
ToolService.executeInferenceAndToolsLoop(...)
  while (aiMessage.hasToolExecutionRequests()) {
    ├── 获取 toolExecutionRequests
    ├── 对每个请求（可并行）:
    │   ├── 触发 beforeToolExecution 回调
    │   ├── toolExecutor.execute(request, memoryId)
    │   │   → 反射调用 @Tool 注解方法
    │   │   → ToolExecutionResultMessage
    │   └── 触发 onToolExecuted 回调
    ├── 添加 ToolExecutionResultMessages 到 ChatMemory
    ├── 检查 IMMEDIATE_RETURN 工具 → 跳出循环
    ├── 刷新动态 ToolProvider
    ├── 构建新 ChatRequest
    ├── chatModel.chat(newChatRequest) → 新 AiMessage
    └── 添加新 AiMessage 到 ChatMemory
  }
```

## 注解体系

| 注解 | 目标 | 作用 |
|------|------|------|
| `@SystemMessage` | 方法/类型 | 定义系统提示词（支持资源文件） |
| `@UserMessage` | 方法/参数 | 定义用户消息模板（支持 `{{it}}`, `{{paramName}}`） |
| `@V` | 参数 | 模板变量名 |
| `@Tool` | 方法 | 标记工具方法（name, description, metadata） |
| `@MemoryId` | 参数 | 标识对话记忆 ID（多用户/多会话） |
| `@Moderate` | 方法 | 启用内容审核 |
| `@InputGuardrails` | 方法/类型 | 声明输入护栏 |
| `@OutputGuardrails` | 方法/类型 | 声明输出护栏 |

## 注解体系流程图

```
@SystemMessage          @UserMessage            @Tool
    │                       │                      │
    v                       v                      v
SystemMessage         UserMessage           ToolSpecification
    │                       │                      │
    └───────┬───────────────┘                      │
            v                                      v
      ChatRequest.messages   ←── toolSpecifications
            │
            v
      ChatModel.chat(request)
            │
            v
      AiMessage (可能包含 toolExecutionRequests)
            │
            v
      @Tool 方法反射执行 → ToolExecutionResultMessage
            │
            v
      再次 ChatModel.chat() → 最终 AiMessage
            │
            v
      ServiceOutputParser.parse() → Java 返回值
```

## Builder 配置项

```
AiServices.builder(MyService.class)
  .chatModel(model)                    // 同步聊天模型
  .streamingChatModel(streamModel)     // 流式聊天模型
  .chatMemory(memory)                  // 单个对话记忆
  .chatMemoryProvider(id -> memory)    // 按 ID 创建记忆
  .contentRetriever(retriever)         // 内容检索器
  .retrievalAugmentor(augmentor)       // RAG 增强器
  .tools(toolObj1, toolObj2)           // @Tool 注解对象
  .toolProvider(provider)              // 动态工具发现
  .guardrails(inputG, outputG)         // 输入/输出护栏
  .moderationModel(modModel)           // 内容审核模型
  .listeners(listener1, listener2)     // 可观测性监听器
  .systemMessageProvider(func)         // 动态系统消息
  .userMessageProvider(func)           // 动态用户消息
  .build()
```

## SPI 依赖

通过 `ServiceHelper.loadFactory()` 加载：
- `AiServicesFactory` — 可替换 AiServices 实现
- `ServiceOutputParser` — 可替换响应解析逻辑
- `TokenStreamAdapter` — 可替换流式适配（如 Reactor）

## 入口/出口

**入口：**
- `AiServices.create(Class<T>, ChatModel)` — 一行代码创建
- `AiServices.builder(Class<T>)` — Builder 高级配置
- 生成的代理方法调用（如 `service.chat("Hello")`）

**出口：**
- 解析后的 Java 对象（String, enum, POJO, List, Set）
- `Result<T>`（包含 content + tokenUsage + finishReason + sources）
- `TokenStream`（流式响应）
