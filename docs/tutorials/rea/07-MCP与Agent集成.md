## 第 7 章 MCP 与 Agent 集成

> 状态：初稿，已按章节拆分。本轮重点校对第 1–2 章；本章后续按 raw/wiki 逐条补证。

### 1 MCP 是什么

REA 的 MCP server 把工具目录、prompts 和 completion 暴露给 AI 客户端。stdin/stdout 默认；HTTP 模式在 `/mcp`。

```bash
rea mcp          # stdio
rea mcp --http   # HTTP，POST /mcp
```

**名称解析**

| 名称 | 含义 |
|---|---|
| Streamable HTTP | MCP 的 HTTP 传输；REA 在 `/mcp` 暴露，默认 loopback、无 TLS/认证。 |
| JSON-RPC | MCP 使用的 RPC 消息格式。 |
| tool catalog | REA 暴露的工具清单；不是安全策略。 |
| session | MCP 的有状态会话；`initialize` / `DELETE` 管理。 |

### 2 工具结果

MCP 返回：

- `structuredContent.normalized_result`
- `evidence_id`
- `raw_result`
- `analysis_profile`

失败时 `isError: true`，返回规范 `{error}`，通常没有 `structuredContent`。

### 3 Prompts

六个调查 prompts：

- `investigate_feature`
- `compare_application_versions`
- `verify_reconstruction`
- `trace_crash`
- `audit_residual_unknowns`
- `prepare_bounded_process_capture`

prompts 只给建议；不会自行调用工具或授予权限。

### 4 Completion

- 只用于 prompt 参数。
- 来自当前会话的 documents/procedures/providers/Evidence/unknowns。
- 无缓存；空候选不等于目标没有该对象。
- 选择候选不会执行工具。

**名称解析**

| 名称 | 含义 |
|---|---|
| prompt | 引导 agent 调查的工作流；不会自行调用工具。 |
| completion | prompt 参数的候选补全；不是工具调用。 |
| structuredContent | MCP 结果中的结构化字段。 |

### 5 传输预算与安全

- stdio 单次响应上限约 10 MiB。
- HTTP 请求体上限 4 MiB。
- HTTP 单客户端、明文、无 TLS/认证。
- loopback Host/Origin 违规返回 403。
- 不要把无认证 HTTP 暴露到不可信网络。

### 6 客户端接入

`npx rea-agents setup` 可注册到 Claude Code、Codex、Qoder、Kimi、OMP、Hermes 等。注册成功 ≠ 工具到达模型；用 `rea mcp doctor --json` 检查基础适配器，但 doctor 不证明客户端启动器或继承环境。

### 7 本章练习

1. 用 CLI 跑一个 `inspect-artifact`，再用 MCP/agent 跑同一个目标。
2. 对比两条路径的 Evidence 是否一致。
3. 记录一个“注册成功但工具不可见”的排查步骤。

---

