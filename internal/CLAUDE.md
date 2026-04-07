# internal 模块组

## 模块职责
**内部工具** — 项目内部使用的工具和辅助模块。

## 子模块列表

### langchain4j-docu-chatbot-updater

| 类 | 职责 |
|----|------|
| `Updater` | 更新项目的文档聊天机器人内容 |

```
执行流程:
  1. 收集项目最新文档和源码
  2. 处理和分块文档
  3. 更新向量存储中的嵌入
  4. 保持文档聊天机器人与项目同步
```

### langchain4j-internal-test-retry

| 类 | 职责 |
|----|------|
| `GlobalTestRetryExtension` | JUnit 扩展，自动重试不稳定的测试 |

```
执行流程:
  1. JUnit 检测到测试失败
  2. GlobalTestRetryExtension 拦截
  3. 自动重试 (默认最多 3 次)
  4. 记录重试历史
```

## 注意

这些模块仅供项目内部使用，不作为公开 API 发布。
