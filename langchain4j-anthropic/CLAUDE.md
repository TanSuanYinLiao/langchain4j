# langchain4j-anthropic 模块

## 模块职责
**Anthropic/Claude 集成模块** — 对接 Anthropic Messages API，支持扩展 thinking、服务端工具（web_search, code_execution）、提示缓存、工具选择控制等 Anthropic 特有功能。

## 核心执行流程

```
用户调用 AnthropicChatModel.chat(ChatRequest)
  │
  ├── ChatModel.chat() [core 接口，触发 listener]
  │
  ├── AnthropicChatModel.doChat(ChatRequest)
  │   ├── InternalAnthropicHelper.createAnthropicRequest(chatRequest)
  │   │   ├── 消息映射到 Anthropic 格式
  │   │   ├── SystemMessage + cache_system_messages (EPHEMERAL)
  │   │   ├── cache_tools (缓存工具定义)
  │   │   ├── thinking config (type: "enabled", budget_tokens)
  │   │   ├── ToolSpecification → Anthropic tool 格式
  │   │   ├── tool_choice (auto/any/specific name + disable_parallel_tool_use)
  │   │   └── 构建 AnthropicCreateMessageRequest
  │   │
  │   ├── client.createMessageWithRawResponse(request)
  │   │   ├── 构建 HttpRequest (POST baseUrl/messages)
  │   │   ├── headers: x-api-key, anthropic-version, anthropic-beta
  │   │   └── httpClient.execute(httpRequest) → JSON 反序列化
  │   │
  │   └── AnthropicCreateMessageResponse → ChatResponse
  │       ├── 提取 text blocks, thinking blocks, tool_use blocks
  │       ├── 处理 thinking signatures (多轮 thinking 场景)
  │       ├── tool_use → AiMessage with tool requests
  │       ├── 服务端工具结果 (web_search, code_execution)
  │       └── 提取 usage (input_tokens, output_tokens, cache_creation, cache_read)
  │
  └── 触发 listener
  v
返回 ChatResponse
```

### 流式流程

```
AnthropicStreamingChatModel.doChat(...)
  ├── SSE 连接到 /messages
  ├── 命名 SSE 事件: message_start, content_block_start, content_block_delta,
  │   content_block_stop, message_delta, message_stop, error
  ├── CONTENT_BLOCK_THINKING / CONTENT_BLOCK_REDACTED_THINKING delta 特殊处理
  ├── thinking 内容和 signatures 在 attributes map 中累积
  └── tool_calls 跨 content_block_start → delta → stop 累积
```

## 核心类

| 类 | 职责 |
|----|------|
| `AnthropicChatModel` | 同步聊天模型，特有：cacheSystemMessages, cacheTools, thinkingType, thinkingBudgetTokens, serverTools, toolChoiceName, disableParallelToolUse |
| `AnthropicStreamingChatModel` | 流式聊天模型 |
| `AnthropicClient` (abstract) | 客户端接口 + SPI 工厂 |
| `DefaultAnthropicClient` | HTTP 客户端，构建 /messages 请求 |
| `InternalAnthropicHelper` | 请求构建和转换逻辑 |

## Anthropic 特有功能

| 功能 | 说明 |
|------|------|
| **Prompt Caching** | SystemMessage 和 tools 可标记为 EPHEMERAL 缓存类型 |
| **Extended Thinking** | thinking config (type: "enabled", budget_tokens) |
| **Server Tools** | 内置 web_search, code_execution 工具 |
| **Tool Choice** | 支持 auto/any/specific name + disable_parallel_tool_use |
| **Thinking Signatures** | 多轮对话中 thinking 内容的签名验证 |

## Provider 标识

`ModelProvider.ANTHROPIC`

## HTTP 客户端模式

使用直接 HTTP 客户端模式（与 OpenAI 相同），非官方 SDK。
