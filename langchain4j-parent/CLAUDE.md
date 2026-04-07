# langchain4j-parent 模块

## 模块职责
**Maven POM-only 项目**，作为所有 LangChain4j 模块的父 POM。不包含任何代码，只管理依赖版本和构建插件配置。

## 核心配置

- **坐标**: `dev.langchain4j:langchain4j-parent:1.13.0-beta23-SNAPSHOT`
- **Java 版本**: 17 (source + target)
- **编码**: UTF-8

## 第三方依赖版本管理

| 依赖 | 版本 |
|------|------|
| Jackson (BOM) | 2.21.1 |
| SLF4J | 2.0.17 |
| Logback | 1.5.18 |
| OkHttp | 4.12.0 |
| Retrofit | 2.9.0 |
| JUnit (BOM) | 6.0.1 |
| Mockito | 5.19.0 |
| AWS SDK (BOM) | 2.41.34 |
| Azure OpenAI SDK | 1.3.5 |
| Google Cloud VertexAI | 3.61.0 |
| Testcontainers (BOM) | 1.21.4 |
| Kotlin | 1.9.25 |
| Hibernate Validator | 7.3.0.Final |
| Pgvector | 0.1.14 |

## 构建插件

| 插件 | 版本 | 用途 |
|------|------|------|
| spotless | 2.44.4 | 代码格式化 (Palantir Java Format) |
| jacoco | 0.8.14 | 测试覆盖率 |
| maven-enforcer | - | 依赖版本一致性 |
| flatten-maven-plugin | - | CI-friendly 版本占位符 |
| kotlin-maven-plugin | - | Kotlin 编译 |
| central-publishing-maven-plugin | - | Maven Central 发布 |
| android-maven-plugin | 4.6.0 | Android 兼容性 Lint 检查 |

## 依赖继承关系

```
langchain4j-parent/pom.xml
  │
  ├── <dependencyManagement> 控制 Jackson, OkHttp, JUnit 等版本
  ├── <pluginManagement> 配置 spotless, jacoco, enforcer 等
  ├── <properties> 定义 java.version=17, 编码等
  │
  v
所有子模块继承:
  langchain4j-core/pom.xml     → <parent>langchain4j-parent</parent>
  langchain4j/pom.xml          → <parent>langchain4j-parent</parent>
  langchain4j-open-ai/pom.xml  → <parent>langchain4j-parent</parent>
  ... (80+ 模块)
```

## 构建命令

```bash
# 编译 + 单元测试 (8线程/核)
mvn -B -U -DembeddingsSkipCache -T8C test javadoc:aggregate

# 代码格式检查
mvn spotless:check

# 代码格式化
mvn spotless:apply
```
