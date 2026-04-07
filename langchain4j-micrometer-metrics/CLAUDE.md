# langchain4j-micrometer-metrics 模块

## 模块职责
**Micrometer 指标收集** — 直接使用 Micrometer 收集聊天模型交互指标，遵循 OpenTelemetry GenAI 语义约定。

> **注意**: 此模块标记为 `@Experimental`。

## 核心类

| 类 | 职责 |
|----|------|
| `MicrometerMetricsChatModelListener` | 实现 ChatModelListener，记录指标 |
| `OTelGenAiAttributes` | OpenTelemetry GenAI 属性定义 |
| `OTelGenAiMetricName` | 指标名称 (gen_ai.client.token.usage) |
| `OTelGenAiOperationName` | 操作名称 (chat) |
| `OTelGenAiProviderName` | 提供商名称 |
| `OTelGenAiTokenType` | Token 类型 (input, output) |

## 核心执行流程

```
MicrometerMetricsChatModelListener.onResponse(context)
  │
  ├── DistributionSummary.builder("gen_ai.client.token.usage")
  │   .tag("gen_ai.operation.name", "chat")
  │   .tag("gen_ai.system", provider)
  │   .tag("gen_ai.request.model", modelName)
  │   .tag("gen_ai.token.type", "input")
  │   .register(meterRegistry)
  │   .record(inputTokens)
  │
  └── DistributionSummary.builder("gen_ai.client.token.usage")
      .tag("gen_ai.token.type", "output")
      .register(meterRegistry)
      .record(outputTokens)
```

## 与 langchain4j-observation 的区别

| 特性 | langchain4j-observation | langchain4j-micrometer-metrics |
|------|------------------------|-------------------------------|
| API | Observation API | 直接 Micrometer API |
| 依赖 | 较重 | 轻量 |
| 追踪 | 支持分布式追踪 | 仅指标 |
| Spring 集成 | 深度集成 | 基础集成 |

## 使用场景

- 简单的 Prometheus/Grafana 监控
- Token 使用量统计
- 成本追踪
