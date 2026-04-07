# langchain4j-google-ai-gemini 模块

## 模块职责
**Google Gemini 集成模块** — 对接 Google AI API，支持 Gemini 模型的聊天、流式、向量化、代码执行、Google Search 地面化、URL 上下文和扩展思考。

## 核心执行流程

```
用户调用 GoogleAiGeminiChatModel.chat(ChatRequest)
  │
  ├── BaseGeminiChatModel.doChat(ChatRequest)
  │   ├── createGenerateContentRequest(chatRequest)
  │   │   ├── 消息 → Gemini 格式 (parts: text, inline_data, function_call, function_response)
  │   │   ├── GenerationConfig (temperature, topP, topK, stopSequences, maxOutputTokens, responseMimeType)
  │   │   ├── SafetySettings
  │   │   ├── 工具 → Gemini FunctionDeclaration/Tool 格式
  │   │   ├── allowCodeExecution, allowGoogleSearch, allowGoogleMaps, allowUrlContext
  │   │   └── ThinkingConfig (thinkingBudget, includeThoughts)
  │   │
  │   ├── geminiService.generateContent(modelName, request)
  │   │   ├── POST {baseUrl}/models/{model}:generateContent
  │   │   ├── header: x-goog-api-key
  │   │   └── JSON 反序列化
  │   │
  │   └── processResponse(geminiResponse)
  │       ├── 迭代 candidates → 提取 parts
  │       ├── text parts → content
  │       ├── thought parts → thinking content
  │       ├── function_call parts → ToolRequest
  │       ├── executable_code, code_execution_result
  │       ├── grounding_metadata → attributes
  │       └── finishReason (STOP, MAX_TOKENS, SAFETY, RECITATION, TOOL_USE)
  │
  v
返回 ChatResponse
```

### 流式流程

```
GoogleAiGeminiStreamingChatModel.doChat(...)
  └── POST /models/{model}:streamGenerateContent?alt=sse
      └── SSE 解析 + 累积部分响应
```

## 核心类

| 类 | 职责 |
|----|------|
| `GoogleAiGeminiChatModel` | 同步聊天模型 |
| `GoogleAiGeminiStreamingChatModel` | 流式聊天模型 |
| `BaseGeminiChatModel` | 共享逻辑：请求构建、响应处理、工具/安全/thinking 配置 |
| `GeminiService` | HTTP 客户端：generateContent, streamGenerateContent, embeddings, token counting, batch |

## Gemini 特有功能

| 功能 | 说明 |
|------|------|
| **Google Search 地面化** | `allowGoogleSearch` 内置工具 |
| **Google Maps** | `allowGoogleMaps` 内置工具 |
| **URL Context** | `allowUrlContext` 内置工具 |
| **Code Execution** | `allowCodeExecution` 允许模型执行代码 |
| **Thinking** | ThinkingConfig (thinkingBudget, includeThoughts) |
| **Grounding Metadata** | 搜索结果元数据返回在 attributes 中 |

## Provider 标识

`ModelProvider.GOOGLE_AI_GEMINI`
