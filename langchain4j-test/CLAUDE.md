# langchain4j-test 模块

## 模块职责
**测试工具** — 提供 AssertJ 风格的护栏结果断言工具。

## 核心类

| 类 | 职责 |
|----|------|
| `GuardrailAssertions` | 断言入口 |
| `GuardrailResultAssert` | GuardrailResult 断言 |
| `InputGuardrailResultAssert` | InputGuardrailResult 断言 |
| `OutputGuardrailResultAssert` | OutputGuardrailResult 断言 |

## 使用示例

```java
// 验证输入护栏成功
assertThat(inputGuardrailResult).succeeded();

// 验证输出护栏失败
assertThat(outputGuardrailResult).failed();

// 验证带消息的失败
assertThat(outputGuardrailResult).failedWithMessage("内容不当");
```
