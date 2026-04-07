# langchain4j-observation 模块

## 模块职责
**Micrometer Observation 集成** — 使用 Micrometer Observation API 实现 ChatModel 遥测，可与 Spring Boot 集成。

> **注意**: 此模块标记为 `@Experimental`。

## 核心类

| 类 | 职责 |
|----|------|
| `ObservationChatModelListener` | 实现 ChatModelListener 接口 |
| `ChatModelObservationContext` | 观测上下文 |
| `DefaultChatModelConvention` | 默认约定 (属性名称等) |
| `ChatModelConvention` | 约定接口 |

## 核心执行流程

```
ObservationChatModelListener.onRequest(ChatModelRequestContext)
  │
  ├── 创建 Observation: observationRegistry.observation("chat-model")
  │   .lowCardinalityKeyValue("model", modelName)
  │   .lowCardinalityKeyValue("provider", provider)
  │   .start()
  └── 存储到 context attributes

ObservationChatModelListener.onResponse(ChatModelResponseContext)
  │
  ├── 获取 Observation
  ├── 添加 highCardinalityKeyValue: token usage
  │   .highCardinalityKeyValue("input_tokens", inputTokens)
  │   .highCardinalityKeyValue("output_tokens", outputTokens)
  └── observation.stop()

ObservationChatModelListener.onError(ChatModelErrorContext)
  │
  ├── 获取 Observation
  ├── observation.error(exception)
  └── observation.stop()
```

## 追踪的指标

| 属性 | 基数 | 说明 |
|------|------|------|
| `model` | 低 | 模型名称 |
| `provider` | 低 | 提供商名称 |
| `operation` | 低 | 操作类型 (chat) |
| `input_tokens` | 高 | 输入 token 数 |
| `output_tokens` | 高 | 输出 token 数 |

## 使用场景

- Spring Boot Actuator + Micrometer 可观测性
- 分布式追踪 (OpenTelemetry)
- 自定义监控面板
