# langchain4j-guardrails 模块

## 模块职责
**内置护栏实现** — 提供开箱即用的输入/输出护栏实现。

## 核心类

| 类 | 职责 |
|----|------|
| `JsonExtractorOutputGuardrail` | 验证 LLM 响应可反序列化为 JSON，失败时自动重试 |
| `MessageModeratorInputGuardrail` | 使用 ModerationModel 验证用户消息的安全性 |

## 核心执行流程

### JsonExtractorOutputGuardrail

```
JsonExtractorOutputGuardrail.validate(output)
  │
  ├── 尝试将 LLM 响应反序列化为指定类型
  ├── 成功 → SUCCESS
  ├── 失败:
  │   ├── 构建重试消息: "你的响应不是有效的 JSON，请重新输出"
  │   ├── 可选: 调用 LLM 修正 (通过 ChatExecutor)
  │   └── 返回 RECOVER (允许重试)
  └── 超过重试次数 → FAILURE
```

### MessageModeratorInputGuardrail

```
MessageModeratorInputGuardrail.validate(input)
  │
  ├── moderationModel.moderate(userMessage)
  │   └── 调用 Moderation API (如 OpenAI Moderation)
  ├── 如果 flagged → FAILURE (阻止请求)
  └── 如果 not flagged → SUCCESS
```

## 使用场景

```java
Assistant assistant = AiServices.builder(Assistant.class)
    .chatModel(model)
    .inputGuardrails(new MessageModeratorInputGuardrail(moderationModel))
    .outputGuardrails(new JsonExtractorOutputGuardrail<>(MyResponse.class))
    .build();
```
