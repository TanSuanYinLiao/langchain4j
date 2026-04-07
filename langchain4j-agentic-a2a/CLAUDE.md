# langchain4j-agentic-a2a 模块

## 模块职责
**A2A (Agent-to-Agent) 协议集成** — 允许 Agent 通过 A2A 协议与远程 Agent 通信。

## 核心类

| 类 | 职责 |
|----|------|
| `DefaultA2AService` | A2A SPI 实现，通过 ServiceLoader 加载 |
| `A2AClientAgentInvoker` | 实现 AgentInvoker，调用远程 Agent |
| `A2AClientInstance` | 管理与远程 A2A 服务器的连接 |
| `DefaultA2AClientBuilder` | A2A 客户端构建器 |

## 核心执行流程

```
@A2AClientAgent(a2aServerUrl = "http://remote-agent:8080")
interface MyAgent {
    @Agent String process(String input);
}

执行流程:
  │
  ├── DefaultA2AService (SPI) 被加载
  ├── 创建 A2AClientInstance
  │   ├── GET {a2aServerUrl}/.well-known/agent.json → AgentCard
  │   └── 获取远程 Agent 的能力描述
  │
  ├── 调用时:
  │   ├── A2AClientAgentInvoker.invoke(scope)
  │   ├── AgenticScope → 输入参数映射
  │   ├── POST {a2aServerUrl}/tasks/send → 发送任务
  │   ├── 接收响应
  │   └── 响应 → AgenticScope 映射
  │
  └── 返回结果
```

## SPI 注册

```
META-INF/services/dev.langchain4j.agentic.a2a.A2AService
  → dev.langchain4j.agentic.a2a.DefaultA2AService
```
