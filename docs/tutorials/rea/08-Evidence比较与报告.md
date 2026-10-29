## 第 8 章 Evidence、比较与报告

> 状态：初稿，已按章节拆分。本轮重点校对第 1–2 章；本章后续按 raw/wiki 逐条补证。

### 1 Evidence 分层

| 层 | 作用 |
|---|---|
| target snapshot | 路径、SHA-256、字节长度 |
| `raw_result` | provider 原始输出 |
| `normalized_result` | 统一后的观察 |
| `analysis_profile` | provider/profile/设置 |
| limitations | 工具声明的限制 |
| unknowns | 未解决或无法确认的事实 |

### 2 四个事实等级

- **observed**：实际读到的字节/输出。
- **derived**：按明确规则推导。
- **inferred**：基于证据的推断。
- **unknown**：没有覆盖或无法确认；不等于不存在。

#### 2.1 什么时候需要结构化 Evidence

`readelf`/`objdump` 的输出适合人工阅读，但不是 Evidence 模型。以下场景应使用返回 `evidence_id` 的工具（`inspect-artifact`、`inspect-binary-layout`、Ghidra 等）：

| 场景 | 需要的原因 |
|---|---|
| 报告/审计要引用结论 | 每条结论能指向 `evidence_id`、provider 和 limitations |
| 版本或构建对比 | 同一工具、同一参数下的字段才可比较 |
| CI/自动化断言 | 需要 JSON 字段、退出码和明确的资源上限 |
| 交接/复现 | Evidence 可导出/导入，跨会话、跨机器复用 |
| 坐标敏感的分析 | 需要文件偏移↔链接地址、符号/重定位表索引 |
| 保留不确定性 | coverage/limitations/unknowns 必须和结论一起记录 |

不需要的场景：一次性人工查看、快速排错、只在 Ghidra 内部做函数分析（Ghidra 的 `binary_overview`/`analyze_function` 本身已返回 Evidence）。

### 3 快照、保留视图与比较

- 完整 Evidence 可保留在会话 ledger，用 `inspect-analysis-view` 分页查看。
- MCP 的 stdio 响应预算默认 **10 MiB**。`inspect_binary_layout` 在 279 MB 文件上会超出预算，但分析已完成并保留为 Evidence；用 `inspect_analysis_view` 取 summary，或导出 Evidence/使用 CLI 完整 JSON。
- 快照复用要求字节、操作、参数、provider、设置完全一致。
- `compare` 只陈述比较覆盖的部分；相同 CFG 不等于语义相同。

**名称解析**

| 名称 | 含义 |
|---|---|
| coverage | 本次分析覆盖的范围；不等于完整性。 |
| limitation | 工具明确声明的限制。 |
| unknown | 没有覆盖/无法确认的事实；不等于不存在。 |
| observed / derived / inferred | 观察 / 按规则推导 / 基于证据推断。 |
| retained evidence | 会话中保留的完整 Evidence。 |
| analysis view | 对保留 Evidence 的投影视图。 |
| snapshot | 可复用的分析结果。 |
| CFG | Control-Flow Graph，控制流图；相同 CFG 不等于语义相同。 |

### 4 报告模板

一份最小报告：

```text
# 调查目标
- 目标：
- 版本/摘要：
- 授权范围：
- 分析日期：

## 方法
- provider/工具：
- 操作：
- 输入：

## 观察（observed）
1.
2.

## 推断（inferred）
1.

## 未知（unknown）
1.

## 覆盖与限制
- 覆盖：
- 未覆盖：
- 下一步：
```

### 5 本章练习

为 WeKnora 写一份 1 页报告：3 observed、2 inferred、2 unknown。

---

