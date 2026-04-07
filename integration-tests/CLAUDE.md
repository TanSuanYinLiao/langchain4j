# integration-tests 模块组

## 模块职责
**集成测试** — 包含框架级的集成测试模块。

## 子模块列表

### integration-tests-class-instance-loader

测试 `ClassInstanceLoader` 和泛型类实例化:
- `ClassInstanceLoaderTests`: 验证各种场景下的类实例加载

### integration-tests-class-metadata-provider

测试 `ClassMetadataProvider` 和泛型类元数据解析:
- `ClassMetadataProviderTests`: 验证泛型参数解析和类型信息提取

### integration-tests-guardrails

护栏框架的全面集成测试 (~20+ 测试类):
- 输入护栏链式调用
- 输出护栏链式调用
- 重试机制
- 验证逻辑
- 类级别和方法级别护栏注解
- 流式响应中的护栏
- 提示模板与护栏的交互

## 执行方式

```bash
# 运行集成测试
mvn -pl integration-tests/integration-tests-guardrails verify
```
