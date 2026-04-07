# langchain4j-mistral-ai 模块

## 模块职责
**Mistral AI 平台集成** — 功能最全面的提供商模块之一，支持聊天、嵌入、FIM 代码补全、内容审核和模型列表。

## 核心类

| 类 | 职责 |
|----|------|
| `MistralAiChatModel` | 同步聊天 |
| `MistralAiStreamingChatModel` | 流式聊天 |
| `MistralAiEmbeddingModel` | 文本嵌入 |
| `MistralAiFimModel` / `MistralAiStreamingFimModel` | FIM (Fill-in-the-Middle) 代码补全 |
| `MistralAiModerationModel` | 内容审核 |
| `MistralAiModels` | 模型列表查询 |
| `MistralAiModelCatalog` | Mistral 模型目录 |

## 核心执行流程

```
MistralAiChatModel.doChat(ChatRequest)
  │
  ├── 构建 Mistral API 请求
  │   ├── 消息映射到 Mistral 格式
  │   ├── model, temperature, topP, maxTokens
  │   ├── tools → Mistral function 格式
  │   └── responseFormat (支持 JSON schema)
  │
  ├── POST https://api.mistral.ai/v1/chat/completions
  │   └── Header: Authorization: Bearer {apiKey}
  │
  └── 响应处理: 提取 content, tool_calls, thinking
```

## 特有功能

| 功能 | 说明 |
|------|------|
| **FIM 补全** | Fill-in-the-Middle: prompt + suffix → 补全中间代码 |
| **内容审核** | MistralAiModerationModel 检测有害内容 |
| **模型列表** | 查询可用模型列表 |
| **Thinking** | MistralAiThinkingContent 支持思考链 |
| **JSON Schema** | 结构化输出支持 |

## Provider 标识

`ModelProvider.MISTRAL_AI`
