# langchain4j-vertex-ai-gemini 模块

## 模块职责
**Google Vertex AI Gemini 集成** — 对接 Google Vertex AI 的 Gemini 模型，支持聊天和流式聊天。

## 核心类

| 类 | 职责 |
|----|------|
| `VertexAiGeminiChatModel` | 同步聊天 |
| `VertexAiGeminiStreamingChatModel` | 流式聊天 |
| `FunctionCallHelper` | 工具调用处理 |
| `SchemaHelper` | 结构化输出 Schema |
| `SafetySettingsMapper` | 安全设置映射 |
| `ResponseGrounding` | Google Search grounding 结果 |

## 核心执行流程

```
VertexAiGeminiChatModel.doChat(ChatRequest)
  │
  ├── 使用 com.google.cloud.vertexai.generativeai.GenerativeModel
  ├── model.generateContent(request)
  │   ├── 消息 → Gemini Content 格式
  │   ├── GenerationConfig (temperature, topP, maxOutputTokens)
  │   ├── SafetySettings
  │   └── Tools (FunctionDeclaration 格式)
  ├── 响应处理:
  │   ├── text parts → content
  │   ├── function_call parts → ToolRequest
  │   ├── grounding_metadata → attributes
  │   └── finishReason mapping
  └── 返回 ChatResponse
```

## 特有功能

| 功能 | 说明 |
|------|------|
| **Safety Settings** | 按危害类别 (HARASSMENT, HATE_SPEECH 等) 配置阈值 |
| **Google Search Grounding** | ResponseGrounding 提供搜索结果元数据 |
| **工具调用模式** | 配置 ANY / AUTO / NONE 工具调用策略 |
| **结构化输出** | SchemaHelper 生成 JSON Schema |

## Provider 标识

`ModelProvider.GOOGLE_VERTEX_AI_GEMINI`
