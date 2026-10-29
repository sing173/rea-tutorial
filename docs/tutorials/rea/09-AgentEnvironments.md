## 第 9 章 Agent Environments

> 状态：初稿，已按章节拆分。本轮重点校对第 1–2 章；本章后续按 raw/wiki 逐条补证。

> 进阶内容。不是每个 REA 用户都需要；先掌握前 9 章。

### 1 它解决什么问题

把一次录制的 Web 应用会话（HAR 或 Evidence）转成可运行的 AgentEnv 环境，让 agent 在没有原应用的情况下练习任务。

### 2 构建与运行

```bash
rea environment-build ./session.har ./my-env --name my-env --json
agent-env run ./my-env
```

### 3 state 与 replay

| 模式 | 含义 |
|---|---|
| state | 从录制推导本地状态回答读请求 |
| replay | 重放录制响应 |
| unavailable | 没有可用状态或录制 |

写操作只在录制确认“操作后读回效果”时提供；否则 withheld 并给补录提示。

**名称解析**

| 名称 | 含义 |
|---|---|
| AgentEnv | 从录制数据构建的可运行 agent 环境框架。 |
| manifest | 描述环境工具、数据、读写的清单。 |
| state | 从录制推导出的本地状态，用于回答读请求。 |
| replay | 重放录制响应，不是真正执行原应用。 |
| probe | 对原应用的独立调用序列，用于和导出的环境比较。 |
| bundle | 导出的环境目录（manifest/evidence/Dockerfile/测试）。 |
| smoke task | 检查环境能启动并响应基本工具调用的最小测试。 |

### 4 服务与比较

- `environment-serve` 暴露 `/mcp`、`/.well-known/agent-env.json`、`/agentenv`。
- `environment-compare` 离线比较 probe；只覆盖 probe 调用。

### 5 风险

bundle 可能包含完整响应体、绝对路径、查询串 token 和自定义头。标准认证头会被排除，但 `X-Api-Key` 等仍可能保留；分享前人工审查。

### 6 本章练习

1. 用一个本地测试应用或 HAR 构建环境。
2. 检查 `coverage.reads`/`coverage.writes`。
3. 列出 3 个分享前必须检查的内容。

---

