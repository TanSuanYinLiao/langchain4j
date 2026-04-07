# langchain4j-http-client 模块

## 模块职责
**HTTP 客户端抽象层** — 定义统一的 HTTP 客户端接口和 SSE（Server-Sent Events）解析框架，为所有 LLM 提供商模块提供底层 HTTP 通信抽象。

## 核心接口

| 接口/类 | 职责 |
|---------|------|
| `HttpClient` | 核心接口：同步请求执行 + 异步 SSE 流式执行 |
| `HttpClientBuilder` | HTTP 客户端构建器接口 |
| `HttpClientBuilderFactory` | SPI 工厂接口，用于创建 HttpClientBuilder |
| `HttpClientBuilderLoader` | SPI 加载器：自动发现并加载 classpath 中的 HttpClientBuilderFactory |
| `HttpRequest` | HTTP 请求封装（method, url, headers, body, timeout） |
| `SuccessfulHttpResponse` | HTTP 响应封装（statusCode, headers, body） |
| `HttpMethod` | 枚举：GET, POST, PUT, DELETE, PATCH |

## 核心执行流程

### 同步请求流程

```
Provider 模块 (如 OpenAiChatModel)
  │
  ├── 构建 HttpRequest
  │   ├── method: POST
  │   ├── url: {baseUrl}/chat/completions
  │   ├── headers: Authorization, Content-Type
  │   └── body: JSON 序列化的请求体
  │
  ├── HttpClient.execute(request)
  │   ├── 具体实现: JdkHttpClient / ApacheHttpClient
  │   ├── 发送 HTTP 请求
  │   └── 返回 SuccessfulHttpResponse
  │
  └── 反序列化 response.body() → 响应对象
```

### 流式 SSE 请求流程

```
Provider 模块 (如 OpenAiStreamingChatModel)
  │
  ├── 构建 HttpRequest (stream: true)
  │
  ├── HttpClient.execute(request, listener)
  │   ├── 内部: execute(request, DefaultServerSentEventParser, listener)
  │   ├── 异步发起 HTTP 请求
  │   ├── DefaultServerSentEventParser.parse(inputStream, listener)
  │   │   ├── 逐行读取 input stream
  │   │   ├── 解析 "data: ..." 行 → ServerSentEvent
  │   │   ├── 解析 "event: ..." 行 → 事件类型
  │   │   ├── 解析 "id: ..." 行 → 事件 ID
  │   │   ├── 遇到空行 → listener.onEvent(event)
  │   │   └── 流结束或异常 → listener.onComplete() / listener.onError()
  │   │
  │   └── Provider 的 ServerSentEventListener 处理每个事件
  │       └── 累积部分响应，回调 StreamingChatResponseHandler
  │
  └── 返回 (异步，通过 listener 回调)
```

## SPI 自动发现机制

```
HttpClientBuilderLoader.loadHttpClientBuilder()
  │
  ├── System.getProperty("langchain4j.http.clientBuilderFactory")  ← 可选：手动指定
  │
  ├── ServiceHelper.loadFactories(HttpClientBuilderFactory.class)
  │   ├── 扫描 META-INF/services/dev.langchain4j.http.client.HttpClientBuilderFactory
  │   ├── 找到 JdkHttpClientBuilderFactory 或 ApacheHttpClientBuilderFactory
  │   └── 如果找到多个且未手动指定 → 抛出 Conflict 异常
  │
  └── factory.create() → HttpClientBuilder
```

## SSE 解析框架

```
ServerSentEvent 数据格式:
  id: 1
  event: message
  data: {"content": "Hello"}
                              ← 空行标识事件结束

DefaultServerSentEventParser.parse(inputStream, listener):
  ├── 逐行读取 inputStream
  ├── "id: ..." → eventBuilder.id(...)
  ├── "event: ..." → eventBuilder.type(...)
  ├── "data: ..." → eventBuilder.data(...)
  ├── "retry: ..." → eventBuilder.retryTime(...)
  ├── 空行 → listener.onEvent(eventBuilder.build())
  └── EOF → listener.onComplete()

ServerSentEvent:
  ├── id() — 事件 ID
  ├── type() — 事件类型 (如 "content_block_delta")
  ├── data() — 事件数据 (通常是 JSON 字符串)
  └── retryTime() — 重试间隔
```

## 日志装饰器

`LoggingHttpClient` 包装任意 `HttpClient`，在请求/响应时调用 `HttpRequestLogger` / `HttpResponseLogger` 记录日志，用于调试和监控。

## Provider 标识

无独立 Provider 标识，作为基础设施层被其他模块引用。
