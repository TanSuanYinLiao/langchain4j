# langchain4j-bom 模块

## 模块职责
**Bill of Materials (BOM)** — 统一管理所有 LangChain4j 模块版本。用户只需在 `<dependencyManagement>` 中 import 此 BOM，即可无需指定版本号地引用任何 LangChain4j 模块。

## 双版本体系

| 版本属性 | 值 | 用途 |
|----------|-----|------|
| `langchain4j.stable.version` | 1.13.0-SNAPSHOT | 生产稳定模块 |
| `langchain4j.beta.version` | 1.13.0-beta23-SNAPSHOT | Beta/实验性模块 |

## 稳定模块 (stable.version)

- `langchain4j` — 高层 API
- `langchain4j-core` — 核心接口
- `langchain4j-anthropic` — Anthropic 集成
- `langchain4j-azure-open-ai` — Azure OpenAI 集成
- `langchain4j-bedrock` — AWS Bedrock 集成
- `langchain4j-mistral-ai` — Mistral AI 集成
- `langchain4j-ollama` — Ollama 集成
- `langchain4j-open-ai` — OpenAI 集成
- `langchain4j-google-ai-gemini` — Google Gemini 集成

## Beta 模块 (beta.version, 70+ 模块)

所有嵌入存储集成、文档解析集成、其余模型提供商、实验性功能等。

## 使用方式

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>dev.langchain4j</groupId>
      <artifactId>langchain4j-bom</artifactId>
      <version>1.13.0-beta23-SNAPSHOT</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <!-- 无需版本号，由 BOM 管理 -->
  <dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
  </dependency>
</dependencies>
```

## 版本解析流程

```
用户 POM import langchain4j-bom
  → Maven 解析 BOM 的 <dependencyManagement>
  → langchain4j-open-ai  → 1.13.0-SNAPSHOT (stable)
  → langchain4j-pinecone → 1.13.0-beta23-SNAPSHOT (beta)
```
