# langchain4j-skills 模块

## 模块职责
**Agent Skills 框架** — 实现 Agent Skills 规范 (agentskills.io)，允许加载和管理可复用的 Agent 技能。

## 核心类

| 类 | 职责 |
|----|------|
| `Skill` | 核心接口：name, description, content, resources, toolProviders |
| `AbstractSkill` | 基类实现 |
| `DefaultSkill` / `FileSystemSkill` | 具体实现 |
| `Skills` | 技能聚合器：创建 ToolProvider 和系统消息片段 |
| `FileSystemSkillLoader` | 从文件系统加载技能 |
| `ClassPathSkillLoader` | 从 classpath 加载技能 |

## 技能格式 (SKILL.md)

```yaml
---
name: web_search
description: Search the web for information
---
# Web Search Skill

Instructions for using this skill...
```

## 核心执行流程

### 加载技能

```
FileSystemSkillLoader.load(basePath)
  │
  ├── 扫描目录下所有子目录
  ├── 每个子目录查找 SKILL.md:
  │   ├── YAML front matter → name, description
  │   ├── Markdown body → content (技能指令)
  │   └── 其他文件 → resources (模板、参考文档等)
  └── 返回 List<Skill>
```

### 编排技能

```
Skills.from(List<Skill> skills)
  │
  ├── skills.toolProvider() → ToolProvider
  │   ├── 暴露 activate_skill 工具: LLM 调用以读取完整指令
  │   └── 暴露 read_skill_resource 工具: LLM 调用以读取资源文件
  │
  ├── skills.formatAvailableSkills() → XML 系统消息片段:
  │   <available_skills>
  │     <skill name="web_search">Search the web...</skill>
  │     <skill name="calculator">Perform calculations...</skill>
  │   </available_skills>
  │
  └── 激活后:
      activate_skill("web_search")
      ├── 返回技能完整内容
      └── 该技能的 toolProviders 被加入后续工具调用
```

### 在 AiServices 中使用

```java
List<Skill> skills = FileSystemSkillLoader.load("skills/");
Skills skillSet = Skills.from(skills);

Assistant assistant = AiServices.builder(Assistant.class)
    .chatModel(model)
    .toolProvider(skillSet.toolProvider())
    .systemMessage("You are helpful. " + skillSet.formatAvailableSkills())
    .build();
```
