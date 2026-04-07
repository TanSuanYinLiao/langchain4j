# langchain4j-mcp-docker 模块

## 模块职责
**MCP Docker 传输层** — 在 Docker 容器中运行 MCP 服务器，通过容器 stdin/stdout 通信。

## 核心类

| 类 | 职责 |
|----|------|
| `DockerMcpTransport` | 实现 McpTransport 接口，管理 Docker 容器 |

## 核心执行流程

```
DockerMcpTransport.start(handler)
  │
  ├── dockerClient.pullImageCmd(imageName)  ← 拉取 Docker 镜像
  ├── dockerClient.createContainerCmd(imageName)
  │   ├── withCmd(...)
  │   ├── withEnv(...)
  │   ├── withBinds(...)  ← 绑定挂载
  │   └── exec() → 创建容器
  ├── dockerClient.startContainerCmd(containerId)  ← 启动容器
  ├── dockerClient.attachCmd(containerId)
  │   ├── withStdOut(true).withStdErr(true).withStdIn(pipedInputStream)
  │   └── exec(callback)  ← DockerResultCallback 读取输出
  └── 消息通过 PipedOutputStream → 容器 stdin
```

### 消息收发

```
// 发送消息
transport.executeOperationWithResponse(request)
  │
  ├── JSON 序列化请求
  ├── pipedOutputStream.write(json.getBytes())
  └── 等待响应 (通过 callback)

// 接收消息
DockerResultCallback.onNext(frame)
  │
  ├── 解析 stdout 帧
  └── handler.onMessage(json)
```

## 配置参数

| 参数 | 说明 |
|------|------|
| `imageName` | Docker 镜像名 |
| `dockerHost` | Docker 守护进程地址 |
| `registryAuth` | 镜像仓库认证信息 |
| `binds` | 绑定挂载 (host:container) |
| `env` | 环境变量 |
| `attachTimeout` | 连接超时时间 |
