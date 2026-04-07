# langchain4j-agentic-mcp 模块

## 模块职责
**MCP Agent 集成** — 将 MCP 服务器的工具包装为可组合的 Agent，在 agentic 工作流中使用。

## 核心类

| 类 | 职责 |
|----|------|
| `McpAgent` | 工厂：创建包装 MCP 工具的 UntypedAgent |
| `McpClientAgentInvoker` | 实现 AgentInvoker，执行 MCP 工具 |
| `McpClientInstance` | 管理 MCP 客户端连接 |
| `DefaultMcpClientBuilder` | MCP 客户端构建器 |

## 核心执行流程

```
McpAgent.builder(mcpClient)
    .toolName("search_web")
    .build()
  │
  ├── 创建 UntypedAgent: 包装单个 MCP 工具
  │
  ├── 在 Agentic 工作流中使用:
  │   @SequenceAgent
  │   interface Pipeline {
  │       @Agent AiAgent aiAgent;      // AI Agent 生成搜索查询
  │       @Agent McpAgent searchTool;  // MCP 工具执行搜索
  │       @Agent AiAgent summarize;    // AI Agent 总结结果
  │   }
  │
  └── 调用时:
      McpClientAgentInvoker.invoke(scope)
      ├── 从 scope 读取输入参数
      ├── mcpClient.executeTool(toolRequest)
      └── 将结果写入 scope
```

## 使用场景

- 将 MCP 工具 (如数据库查询、API 调用) 作为工作流中的非 AI 步骤
- 在 Sequence/Loop/Supervisor 工作流中组合 MCP 工具
- 实现 AI Agent 和传统工具的混合编排
