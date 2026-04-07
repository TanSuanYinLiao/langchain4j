# experimental 模块组

## 模块职责
**实验性功能** — 包含尚未稳定的实验性功能模块。

## 子模块列表

### langchain4j-experimental-hibernate

| 类 | 职责 |
|----|------|
| `HibernateContentRetriever` | 实现 ContentRetriever，通过 Hibernate 查询数据 |

```
执行流程:
  1. 使用 LLM 将自然语言转换为 HQL 查询
  2. 序列化 Hibernate 元模型作为上下文
  3. 执行 HQL 查询 → 返回相关数据
  4. 包装为 Content 返回
```

### langchain4j-experimental-sql

| 类 | 职责 |
|----|------|
| `SqlDatabaseContentRetriever` | 实现 ContentRetriever，通过 JDBC 查询数据 |

```
执行流程:
  1. 内省数据库元数据 (表名、列名、类型)
  2. 使用 LLM 将自然语言转换为 SQL
  3. JSQLParser 验证 SQL 合法性
  4. 执行 SQL → 返回相关数据
  5. 包装为 Content 返回
```

### langchain4j-experimental-skills-shell

| 类 | 职责 |
|----|------|
| `ShellSkills` | Shell 命令执行技能定义 |
| `RunShellCommandToolExecutor` | 工具执行器 |
| `ShellCommandRunner` | 命令运行 |

```
执行流程:
  1. Agent 调用 run_shell_command 工具
  2. ShellCommandRunner 执行系统命令
  3. 返回 stdout/stderr
```

## 注意

这些模块处于实验阶段，API 可能会在未来版本中变更。
