# code-execution-engines 模块组

## 模块职责
**代码执行引擎** — 提供多种代码执行环境，允许 LLM Agent 执行代码。

## 统一接口

```java
public interface CodeExecutionEngine {
    String execute(String code);
}
```

## 子模块列表

| 子模块 | 执行环境 | 语言 | 关键类 |
|--------|---------|------|--------|
| `langchain4j-code-execution-engine-graalvm-polyglot` | GraalVM 进程内 | JavaScript, Python | `GraalVmJavaScriptExecutionEngine`, `GraalVmPythonExecutionEngine` |
| `langchain4j-code-execution-engine-judge0` | Judge0 API (远程) | JavaScript | `Judge0JavaScriptEngine` |
| `langchain4j-code-execution-engine-azure-acads` | Azure Container Apps | 多语言 | `SessionsREPLTool` |

## 核心执行流程

### GraalVM 进程内执行

```
GraalVmJavaScriptExecutionEngine.execute(String code)
  │
  ├── Context.newBuilder("js")
  │   .allowHostAccess(HostAccess.ALL)
  │   .sandbox(SandboxPolicy.CONSTRAINED)  ← 安全沙箱
  │   .build()
  ├── context.eval("js", code)
  └── 返回执行结果.toString()
```

### Judge0 远程执行

```
Judge0JavaScriptEngine.execute(String code)
  │
  ├── POST https://judge0-ce.p.rapidapi.com/submissions
  │   body: { source_code: code, language_id: 63 (JavaScript) }
  ├── 轮询: GET /submissions/{token}
  └── 返回 stdout
```

### Azure ACADS 执行

```
SessionsREPLTool.execute(String code)
  │
  ├── POST https://{endpoint}/python/execute
  │   body: { code: code, identifier: sessionId }
  └── 返回执行结果
```

## 安全性

| 引擎 | 安全措施 |
|------|---------|
| GraalVM | `SandboxPolicy.CONSTRAINED` 沙箱 |
| Judge0 | 远程隔离执行环境 |
| Azure ACADS | Azure 容器隔离 |

## 使用场景

- Agent 代码执行工具
- 数据分析和计算
- 动态代码生成和验证
