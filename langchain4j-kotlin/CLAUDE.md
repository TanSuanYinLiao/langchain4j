# langchain4j-kotlin 模块

## 模块职责
**Kotlin 扩展** — 提供 Kotlin 协程支持和 Flow 适配器。

## 核心类 (Kotlin)

| 类 | 职责 |
|----|------|
| `ChatModelExtensions.kt` | `suspend fun ChatModel.chatAsync()` 协程包装 |
| `StreamingChatModelExtensions.kt` | 流式聊天协程扩展 |
| `TokenStreamToReplyFlowAdapter.kt` | TokenStream → Kotlin Flow 转换 |
| `TokenStreamToStringFlowAdapter.kt` | TokenStream → String Flow 转换 |
| `DocumentLoaderExtensions.kt` | 文档加载器协程扩展 |
| `DocumentParserExtensions.kt` | 文档解析器协程扩展 |
| `FileSystemDocumentLoaderExtensions.kt` | 文件系统文档加载器扩展 |

## 核心用法

### 协程聊天

```kotlin
val model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4o")
    .build()

// 协程风格
val response = model.chatAsync("Hello")
```

### Flow 流式输出

```kotlin
val streamingModel = OpenAiStreamingChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4o")
    .build()

// Flow 风格
streamingModel.chat("Tell me a story")
    .toReplyFlow()
    .collect { token -> print(token) }
```

## 特点

- 支持 `Dispatchers` 和虚拟线程
- 无缝集成 Kotlin 协程生态
- 从 Java 流式 API 到 Kotlin Flow 的自然适配
