# langchain4j-ollama 模块

## 模块职责
**Ollama 本地模型集成模块** — 对接本地运行的 Ollama 服务器，支持聊天、流式聊天、向量化和模型管理。

## 核心执行流程

```
用户调用 OllamaChatModel.chat(ChatRequest)
  │
  ├── OllamaChatModel.doChat(ChatRequest)
  │   ├── OllamaUtils.toOllamaChatRequest(chatRequest, parameters)
  │   │   ├── 消息映射到 Ollama 格式 (role: system/user/assistant/tool)
  │   │   ├── 图片 → base64 编码
  │   │   ├── 工具 → Ollama function 格式
  │   │   └── 设置 model, stream(false), options (temperature, topP 等)
  │   │       Ollama 特有: mirostat, mirostatEta, mirostatTau, numCtx,
  │   │       repeatLastN, repeatPenalty, seed, minP, think
  │   │
  │   ├── client.chat(ollamaChatRequest)
  │   │   ├── POST {baseUrl}/api/chat
  │   │   └── httpClient.execute(httpRequest) → JSON 反序列化
  │   │
  │   └── OllamaChatResponse → ChatResponse
  │       ├── aiMessageFrom(): 提取 content, tool_calls
  │       ├── 解析 thinking blocks (如 returnThinking 启用)
  │       ├── 映射 finish_reason (stop/tool_calls)
  │       └── TokenUsage (eval_count, prompt_eval_count)
  │
  v
返回 ChatResponse
```

### 流式流程

```
OllamaStreamingChatModel.doChat(...)
  ├── client.streamingChat(request)
  │   ├── POST /api/chat with stream(true)
  │   ├── OllamaServerSentEventParser 解析 SSE
  │   ├── ToolCallBuilder 累积部分工具调用
  │   └── 每个事件 → OllamaChatResponse → handler 回调
```

## 核心类

| 类 | 职责 |
|----|------|
| `OllamaChatModel` | 继承 OllamaBaseChatModel，同步聊天 |
| `OllamaStreamingChatModel` | 继承 OllamaBaseChatModel，流式聊天 |
| `OllamaBaseChatModel` | 共享基类：Ollama 特有参数 (mirostat, numCtx, think 等) |
| `OllamaClient` | HTTP 客户端：/api/chat, /api/generate, /api/embed, /api/tags, /api/show, /api/ps, /api/delete |
| `OllamaUtils` | 请求/响应转换 |

## Ollama 特有功能

| 功能 | 说明 |
|------|------|
| **think 参数** | Ollama 原生推理模式 |
| **mirostat** | 自适应采样算法 (mirostat, mirostatEta, mirostatTau) |
| **numCtx** | 上下文窗口大小 |
| **model 管理** | list/show/ps/delete 模型操作 |

## Provider 标识

`ModelProvider.OLLAMA`
