# langchain4j-http-client-apache 模块

## 模块职责
**Apache HttpClient 实现** — 基于 Apache HttpClient 5.x 的 HTTP 客户端实现，提供更灵活的连接池和代理配置。

## 核心类

| 类 | 职责 |
|----|------|
| `ApacheHttpClient` | 实现 `HttpClient` 接口，封装 Apache CloseableHttpClient |
| `ApacheHttpClientBuilder` | 实现 `HttpClientBuilder` 接口，配置连接池、代理、SSL、自定义 HttpClient |
| `ApacheHttpClientBuilderFactory` | SPI 工厂，注册到 `META-INF/services/` |
| `MultipartBodyPublisher` | 构建 multipart/form-data 请求体 |

## 核心执行流程

### 同步请求

```
ApacheHttpClient.execute(HttpRequest request)
  │
  ├── 构建 Apache HttpUriRequest
  │   ├── method → HttpPost, HttpGet 等
  │   ├── URI.create(request.url())
  │   ├── 设置 headers
  │   ├── 设置 entity (ByteArrayEntity / StringEntity)
  │   └── 设置 requestConfig (connectTimeout, responseTimeout)
  │
  ├── closeableHttpClient.execute(request)
  │   ├── Apache 内部连接池管理
  │   └── 返回 CloseableHttpResponse
  │
  ├── 检查状态码
  │   ├── 2XX → EntityUtils.toString(entity) → SuccessfulHttpResponse
  │   └── 4XX/5XX → 抛出 HttpException
  │
  └── 返回 SuccessfulHttpResponse
```

### 流式 SSE 请求

```
ApacheHttpClient.execute(HttpRequest request, ServerSentEventParser parser, ServerSentEventListener listener)
  │
  ├── 构建 Apache HttpUriRequest
  │
  ├── 异步执行 (使用 ExecutorService)
  │   ├── closeableHttpClient.execute(request)
  │   ├── response.getEntity().getContent() → InputStream
  │   └── parser.parse(inputStream, listener)
  │
  ├── 检查状态码
  │   ├── 2XX → parser.parse(response.body(), listener)
  │   └── 非 2XX → listener.onError(new HttpException(...))
  │
  └── 异常 → listener.onError(throwable)
```

## 与 JDK HttpClient 的区别

| 特性 | JDK HttpClient | Apache HttpClient |
|------|---------------|-------------------|
| 连接池 | JDK 内置 | Apache 连接池管理器 (可配置 maxTotal, maxPerRoute) |
| 代理配置 | 基础代理 | 丰富的代理配置 (HTTP/SOCKS) |
| 依赖 | 零外部依赖 (JDK 11+) | 需要 Apache HttpClient 5.x |
| 自定义 | 有限 | 支持传入自定义 CloseableHttpClient |
| Cookie 管理 | 基础 | 完整 Cookie 策略支持 |

## SPI 注册

```
META-INF/services/dev.langchain4j.http.client.HttpClientBuilderFactory
  → dev.langchain4j.http.client.apache.ApacheHttpClientBuilderFactory
```

当 classpath 中同时存在 JDK 和 Apache 实现时，需要通过系统属性 `langchain4j.http.clientBuilderFactory` 显式指定。
