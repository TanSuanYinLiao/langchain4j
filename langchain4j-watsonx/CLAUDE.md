# langchain4j-watsonx 模块

## 模块职责
**IBM watsonx.ai 集成** — 对接 IBM watsonx.ai 平台，功能覆盖聊天、嵌入、评分、审核和 token 估算。

## 核心类

| 类 | 职责 |
|----|------|
| `WatsonxChatModel` | 同步聊天 |
| `WatsonxStreamingChatModel` | 流式聊天 |
| `WatsonxEmbeddingModel` | 文本嵌入 |
| `WatsonxScoringModel` | 文档评分 |
| `WatsonxModerationModel` | 内容审核 |
| `WatsonxTokenCountEstimator` | Token 数估算 |
| `WatsonxModelCatalog` | 模型目录 |

## 核心执行流程

```
WatsonxChatModel.doChat(ChatRequest)
  │
  ├── 使用 IBM watsonx SDK (com.ibm.watsonx.ai)
  ├── POST https://{region}.ml.cloud.ibm.com/ml/v1/text/chat
  │   Headers: Authorization: Bearer {iamToken}
  ├── 支持 region-based 配置
  └── ContentFilteredException: 内容被过滤时抛出
```

## 特有功能

| 功能 | 说明 |
|------|------|
| **Region 配置** | 支持多个 IBM Cloud 区域 |
| **内容过滤** | ContentFilteredException 处理 |
| **Token 估算** | WatsonxTokenCountEstimator 本地估算 token 数 |
| **全面覆盖** | 聊天 + 嵌入 + 评分 + 审核 + Token |

## Provider 标识

`ModelProvider.WATSONX`
