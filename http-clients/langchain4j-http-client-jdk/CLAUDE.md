# langchain4j-http-client-jdk 模块

## 模块职责
**JDK HttpClient 实现** — 基于 Java 11+ `java.net.http.HttpClient` 的 HTTP 客户端实现，作为 LangChain4j 默认的 HTTP 客户端。

## 核心类

| 类 | 职责 |
|----|------|
| `JdkHttpClient` | 实现 `HttpClient` 接口，封装 JDK HttpClient |
| `JdkHttpClientBuilder` | 实现 `HttpClientBuilder` 接口，配置连接超时、读取超时、代理、SSL 等 |
| `JdkHttpClientBuilderFactory` | SPI 工厂，注册到 `META-INF/services/` |
| `MultipartBodyPublisher` | 构建 multipart/form-data 请求体（文件上传） |

## 核心执行流程

### 同步请求

```
JdkHttpClient.execute(HttpRequest request)
  │
  ├── 构建 java.net.http.HttpRequest
  │   ├── URI.create(request.url())
  │   ├── 设置 method + body (HttpRequest.BodyPublishers)
  │   ├── 设置 headers
  │   ├── 设置 timeout (connectTimeout, readTimeout)
  │   └── 可选: proxy, SSLContext
  │
  ├── java.net.http.HttpClient.send(request, BodyHandlers.ofString())
  │   ├── JDK 内部处理 HTTP 连接
  │   └── 返回 java.net.http.HttpResponse
  │
  ├── 检查状态码
  │   ├── 2XX → 返回 SuccessfulHttpResponse
  │   └── 4XX/5XX → 抛出 HttpException
  │
  └── 返回 SuccessfulHttpResponse(statusCode, headers, body)
```

### 流式 SSE 请求

```
JdkHttpClient.execute(HttpRequest request, ServerSentEventParser parser, ServerSentEventListener listener)
  │
  ├── 构建 java.net.http.HttpRequest
  │
  ├── java.net.http.HttpClient.sendAsync(request, BodyHandlers.ofInputStream())
  │   ├── 异步执行，返回 CompletableFuture
  │   └── .thenAccept() 处理响应
  │
  ├── 检查状态码
  │   ├── 2XX → parser.parse(response.body(), listener)
  │   └── 非 2XX → listener.onError(new HttpException(...))
  │
  ├── parser.parse(inputStream, listener)
  │   └── DefaultServerSentEventParser 逐行解析 SSE 数据
  │
  └── .exceptionally() → listener.onError(throwable)
```

## SPI 注册

```
META-INF/services/dev.langchain4j.http.client.HttpClientBuilderFactory
  → dev.langchain4j.http.client.jdk.JdkHttpClientBuilderFactory
```

当 classpath 中只有此模块时，`HttpClientBuilderLoader.loadHttpClientBuilder()` 自动加载 JDK HttpClient。
