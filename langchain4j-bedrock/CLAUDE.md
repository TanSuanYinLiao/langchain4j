# langchain4j-bedrock 模块

## 模块职责
**AWS Bedrock 集成模块** — 使用 AWS SDK (`BedrockRuntimeClient`) 对接 Amazon Bedrock Converse API，支持 Bedrock 特有功能如护栏、缓存点和跨区域推理。

## 核心执行流程

```
用户调用 BedrockChatModel.chat(ChatRequest)
  │
  ├── BedrockChatModel.doChat(ChatRequest)
  │   ├── validate(parameters)
  │   ├── buildConverseRequest(chatRequest)  ← AbstractBedrockChatModel
  │   │   ├── 消息 → Bedrock Converse 格式 (ContentBlock: text/image/document/video)
  │   │   ├── SystemMessages + 可选缓存点 (最多 4 个/请求)
  │   │   ├── 工具 → Bedrock ToolSpec (ToolConfiguration + toolChoice)
  │   │   ├── inferenceConfig (maxTokens, temperature, topP, stopSequences)
  │   │   ├── guardrailConfig (如配置)
  │   │   ├── reasoningConfig (thinking/reasoning 支持)
  │   │   └── additionalModelRequestFields (扩展 thinking 等)
  │   │
  │   ├── client.converse(converseRequest)  ← AWS SDK
  │   │   ├── SDK 内部处理 HTTP (POST Bedrock Converse API)
  │   │   ├── 认证: AWS IAM (role, access key 等)
  │   │   └── 返回 ConverseResponse
  │   │
  │   └── aiMessageFrom(converseResponse)
  │       ├── text → content
  │       ├── reasoning (ReasoningContentBlock: text + signature)
  │       ├── tool use (ToolUseBlock → ToolRequest)
  │       └── stopReason (end_turn, tool_use, max_tokens, content_filtered)
  │
  v
返回 ChatResponse
```

### 流式流程

```
BedrockStreamingChatModel.doChat(...)
  ├── client.converseStream(request) ← AWS SDK 流式 API
  ├── 处理 ConverseStreamResponse 事件
  ├── contentBlockDelta, contentBlockStart, contentBlockStop, messageStop
  └── 累积 reasoning content + signatures
```

## 核心类

| 类 | 职责 |
|----|------|
| `BedrockChatModel` | 同步聊天，继承 AbstractBedrockChatModel |
| `BedrockStreamingChatModel` | 流式聊天 |
| `AbstractBedrockChatModel` | 基类：缓存点管理(最多4)、guardrail 配置、reasoning/thinking 支持、结构化输出 |
| AWS SDK `BedrockRuntimeClient` | 官方 SDK 处理 HTTP、认证、序列化 |
| `BedrockExceptionMapper` | AWS SDK 异常 → LangChain4j 异常映射 |

## Bedrock 特有功能

| 功能 | 说明 |
|------|------|
| **Cache Points** | EPHEMERAL 缓存类型，最多 4 个/请求 |
| **Guardrails** | Bedrock 原生护栏配置 |
| **Reasoning** | ReasoningContentBlock + signatures (多轮 thinking) |
| **Converse API** | 使用新版 Converse API (非 InvokeModel) |
| **跨区域推理** | additionalModelRequestFields 支持 |

## Provider 标识

`ModelProvider.AMAZON_BEDROCK`

## HTTP 客户端模式

使用 AWS SDK（与 Azure OpenAI 类似），非自定义 HTTP 客户端。
