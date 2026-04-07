# langchain4j-agentic 模块

## 模块职责
**Agent 框架核心** — 提供声明式 Agent 编排能力，支持顺序、并行、循环、条件、监督者和规划器等组合模式。

## 核心类

| 类/接口 | 职责 |
|---------|------|
| `AgenticServices` | 中央工厂：创建所有工作流类型的 Builder |
| `@Agent` | 标注方法为 Agent 的注解 |
| `UntypedAgent` | 动态 Agent 接口 (接受 Map<String, Object>) |
| `AgentExecutor` | 组合 AgentInvoker + InternalAgent |
| `AgenticScope` | 跨 Agent 共享状态容器 |
| `Planner` | 规划策略接口 |

## 声明式注解

| 注解 | 说明 |
|------|------|
| `@SequenceAgent` | 顺序执行子 Agent |
| `@ParallelAgent` | 并行执行子 Agent |
| `@LoopAgent` | 循环执行子 Agent |
| `@ConditionalAgent` | 条件执行子 Agent |
| `@ParallelMapperAgent` | 集合元素并行映射 |
| `@SupervisorAgent` | LLM 驱动的监督者 |
| `@PlannerAgent` | 规划器驱动的 Agent |
| `@A2AClientAgent` | 远程 A2A Agent 客户端 |
| `@McpClientAgent` | MCP 工具 Agent |
| `@ActivationCondition` | 条件激活谓词 |
| `@HumanInTheLoop` | 人工干预点 |

## 核心执行流程

```
用户定义 Agent 接口:
  interface ResearchAgent {
      @Agent("Research assistant")
      String research(@UserMessage String topic);
  }

AgenticServices.createAgenticSystem(ResearchAgent.class)
  │
  ├── 扫描 @SequenceAgent, @ParallelAgent 等声明式注解
  ├── 构建组合 Agent:
  │   ├── Sequence: 按顺序执行子 Agent，通过 AgenticScope 传递输出
  │   ├── Parallel: 并发执行所有子 Agent
  │   ├── Loop: 重复执行直到 maxIterations 或退出条件
  │   ├── Conditional: 根据 @ActivationCondition 激活子 Agent
  │   └── Supervisor: LLM 动态决定调用哪个子 Agent
  │
  ├── 返回 Agent 实例 (JDK 动态代理 AgentInvocationHandler)
  │
  └── 执行:
      agent.research("Java performance")
      ├── 创建 AgenticScope
      ├── 按工作流执行子 Agent
      │   ├── 每个子 Agent: agentInvoker.invoke(scope)
      │   ├── 从 scope 读取输入参数
      │   └── 将输出写入 scope (outputKey)
      └── 返回最终结果
```

## 工作流类型详解

### Sequence (顺序)
```
@SequenceAgent
interface Pipeline {
    @Agent Step1 step1;
    @Agent Step2 step2;
    @Agent Step3 step3;
}
执行: step1 → step2 → step3 (输出通过 scope 传递)
```

### Parallel (并行)
```
@ParallelAgent
interface ParallelResearch {
    @Agent WebSearch webSearch;
    @Agent DatabaseQuery dbQuery;
}
执行: webSearch || dbQuery (并发) → 结果合并
```

### Loop (循环)
```
@LoopAgent(maxIterations = 5)
interface RefinementLoop {
    @Agent DraftAgent draft;
    @Agent ReviewAgent review;
}
执行: draft → review → draft → review → ... (最多 5 轮)
```

### Supervisor (监督者)
```
@SupervisorAgent(supervisor = SupervisorLLM.class)
interface CustomerSupport {
    @Agent BillingAgent billing;
    @Agent TechAgent tech;
    @Agent GeneralAgent general;
}
执行: Supervisor LLM 根据输入动态选择调用 billing/tech/general
```

## 可观测性

| 类 | 职责 |
|----|------|
| `AgentListener` | before/after Agent 调用和 Tool 执行的回调 |
| `AgentMonitor` | 收集 Agent 调用数据 |
| `HtmlReportGenerator` | 生成 HTML 执行追踪报告 |
