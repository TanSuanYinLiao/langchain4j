# langchain4j-mcp 模块

## 模块职责
**MCP (Model Context Protocol) 客户端** — 实现 MCP 协议客户端，连接 MCP 服务器获取工具、资源和提示模板。

## 核心类

| 类 | 职责 |
|----|------|
| `McpClient` | 核心接口：工具/资源/提示管理 |
| `DefaultMcpClient` | 默认实现，管理连接、缓存、健康检查 |
| `McpToolProvider` | 实现 ToolProvider，聚合多个 MCP 客户端的工具 |
| `McpToolExecutor` | 实现 ToolExecutor，委托 MCP 客户端执行工具 |
| `McpTransport` | 传输层抽象 |

## 传输层实现

| 实现 | 协议 | 说明 |
|------|------|------|
| `StdioMcpTransport` | 子进程 stdin/stdout | 启动 MCP 服务器子进程 |
| `HttpMcpTransport` | HTTP SSE | 连接 SSE 端点 |
| `StreamableHttpMcpTransport` | HTTP 流式 | 可流式传输的 HTTP |
| `WebSocketMcpTransport` | WebSocket | WebSocket 连接 |
| `DockerMcpTransport` | Docker stdin/stdout | Docker 容器内运行 (见 langchain4j-mcp-docker) |

## 核心执行流程

### 初始化

```
DefaultMcpClient (builder 构建)
  │
  ├── transport.start(handler)  ← 启动传输层
  ├── transport.initialize(request)  ← MCP 初始化握手
  │   └── 双方交换: protocolVersion, capabilities, clientInfo
  ├── 启动健康检查 (ScheduledExecutorService)
  └── 缓存工具/资源/提示列表 (AtomicReference)
```

### 工具发现与执行

```
// 工具发现
McpToolProvider.apply(toolServiceRequest)
  │
  ├── 遍历所有 McpClient
  ├── mcClient.listTools() → List<ToolSpecification>
  │   └── JSON-RPC: { "method": "tools/list" }
  ├── 合并工具列表 (可选: 过滤、重命名)
  └── 返回 ToolProviderResult

// 工具执行
McpToolExecutor.execute(request, memoryId, toolServiceRequest)
  │
  ├── 找到对应的 McpClient
  ├── mcpClient.executeTool(request)
  │   └── JSON-RPC: { "method": "tools/call", "params": { "name": "...", "arguments": {...} } }
  └── 返回工具执行结果
```

### 资源管理

```
mcpClient.listResources()    → JSON-RPC: resources/list
mcpClient.readResource(uri)  → JSON-RPC: resources/read
mcpClient.subscribeToResource(uri)  → JSON-RPC: resources/subscribe
```

### 提示模板

```
mcpClient.listPrompts()  → JSON-RPC: prompts/list
mcpClient.getPrompt(name, arguments)  → JSON-RPC: prompts/get
```

## 协议消息 (JSON-RPC 2.0)

| 消息类型 | 方法 |
|---------|------|
| 初始化 | `initialize` |
| 工具 | `tools/list`, `tools/call` |
| 资源 | `resources/list`, `resources/read`, `resources/subscribe` |
| 提示 | `prompts/list`, `prompts/get` |
| 健康 | `ping` |

## 使用示例

```java
McpClient mcpClient = DefaultMcpClient.builder()
    .transport(new HttpMcpTransport("http://localhost:3000/sse"))
    .build();

McpToolProvider toolProvider = McpToolProvider.builder()
    .mcpClients(mcpClient)
    .build();

Assistant assistant = AiServices.builder(Assistant.class)
    .chatModel(model)
    .toolProvider(toolProvider)
    .build();
```
