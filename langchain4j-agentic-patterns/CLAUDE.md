# langchain4j-agentic-patterns 模块

## 模块职责
**Agent 规划模式** — 实现 GOAP (目标导向行动规划) 和 P2P (点对点) 两种高级 Agent 规划模式。

## 核心接口

```java
public interface Planner {
    void init(InitPlanningContext context);
    Action firstAction(PlanningContext context);
    Action nextAction(PlanningContext context);
    Map<String, Object> executionState();
    void restoreExecutionState(Map<String, Object> state);
}
```

## GOAP (Goal-Oriented Action Planning)

### 核心类

| 类 | 职责 |
|----|------|
| `GoalOrientedPlanner` | 实现 Planner 接口 |
| `GoalOrientedSearchGraph` | 构建 Agent 依赖图 |
| `DependencyGraphSearch` | 在图中搜索到达目标的最短路径 |

### 执行流程

```
GoalOrientedPlanner.firstAction(context)
  │
  ├── 建模子 Agent 为有向图节点
  │   ├── 每个 Agent 声明: inputs (需要的参数), outputKey (产出的值)
  │   └── 如: SearchAgent { inputs: ["query"], outputKey: "searchResults" }
  │
  ├── 分析当前 AgenticScope 状态 (已有哪些 key)
  ├── 使用图搜索算法找从当前状态到目标的路径
  │   └── 类似 STRIPS 规划: 每步选择能推进到目标的 Agent
  │
  └── 返回 AgentCall(第一个需要执行的 Agent)
      ├── 执行后 → nextAction()
      ├── 重新计算路径 (无状态，每次重算)
      └── 直到返回 Done
```

### 图搜索示例

```
目标: "finalReport"
当前 scope: { "topic": "AI" }

图:
  SearchAgent: inputs=["topic"] → outputKey="searchResults"
  AnalyzeAgent: inputs=["searchResults"] → outputKey="analysis"
  ReportAgent: inputs=["analysis"] → outputKey="finalReport"

规划路径: SearchAgent → AnalyzeAgent → ReportAgent
```

## P2P (Peer-to-Peer)

### 核心类

| 类 | 职责 |
|----|------|
| `P2PPlanner` | 实现 Planner 接口 |
| `P2PAgent` | P2P Agent 接口 |
| `VariablesExtractorAgent` | 从用户输入提取变量 |

### 执行流程

```
P2PPlanner.firstAction(context)
  │
  ├── VariablesExtractorAgent: 从用户 prompt 提取变量到 scope
  │   └── 如: "研究 Java 性能并写报告" → { "topic": "Java 性能", "format": "报告" }
  │
  ├── 迭代激活:
  │   ├── 找到输入参数全部被 scope 满足的 Agent
  │   ├── 执行该 Agent → 输出写入 scope
  │   ├── 重新评估哪些 Agent 可被激活
  │   ├── Agent 在输入变化时重新执行
  │   └── 支持 maxAgentsInvocations 限制
  │
  └── 退出条件: 无更多可激活的 Agent 或达到限制
```

## 使用场景

| 模式 | 适用场景 |
|------|---------|
| GOAP | 明确目标、需要最优执行路径的场景 |
| P2P | 需求驱动、Agent 间有依赖关系的自适应执行 |
