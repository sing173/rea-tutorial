## 第 10 章 综合实战：WeKnora Lite 桌面端

> 状态：审阅稿（2026-10-27 按构建机上自建的 Lite 桌面端实测整理；来源见 9，待审阅）。

### 1 目标

围绕 **WeKnora Lite Linux 桌面端**（构建机上自建），在不修改共享数据的前提下回答：

1. 构建信息是什么（版本、commit、Go、tags）？
2. 依赖清单里，哪些与桌面壳、存储、检索相关？
3. 桌面端的行为如何组织（窗口、后端、SQLite/FTS5、auto-setup）？
4. 哪些结论是 observed，哪些是 inferred/unknown？

### 2 Phase 0：范围与快照

- 构建机：`<build-host>`；源码 commit `97d7f0a`，VERSION `0.8.3`；`wails build` 自建。
- 产物：`/root/weknora-lite-linux/WeKnora Lite`，`279,195,936 B`，SHA-256 `f748a5972aa145684ba0293854f7a1493f765ee7d0673105cd7ae175e267aa16`。
- 取到分析机：

```bash
mkdir -p /root/weknora-lite
scp root@<build-host>:"/root/weknora-lite-linux/WeKnora Lite" /root/weknora-lite/WeKnora-Lite
sha256sum /root/weknora-lite/WeKnora-Lite
```

数据边界：只使用本地测试 SQLite（`data/weknora.db`）；不接触共享 Postgres/Nextcloud/Milvus。

### 3 Phase 1：产物盘点

```bash
rea inspect-artifact /root/weknora-lite/WeKnora-Lite --json
file /root/weknora-lite/WeKnora-Lite
readelf -hW /root/weknora-lite/WeKnora-Lite | head -12
```

产出：ELF64 `EXEC`、42 个 section header（41 个有名节）、入口 `0xf0e8e0`、BuildID `e8137e29…`。
注意：`inspect-artifact` 的 `architecture` 是 `null`，架构信息来自 `file`/`readelf`（原因见第 3 章）。

### 4 Phase 2：Go 元数据

完整文件 279 MB，超过 `inspect-go-binary` 的 256 MiB 固定上限。在构建机（Go 1.27.3）上直接读 buildinfo：

```bash
cd /root/weknora-lite-linux
go version -m "WeKnora Lite" > /root/weknora-lite-go-version-m.txt
```

实测 **0.005 秒**；输出包含主模块、依赖与构建设置（本次 312 条依赖、13 项设置，完整清单见输出文件）。

产出：Go `1.27.3`、主包 `cmd/desktop`、`Edition=lite`、tags `desktop,sqlite_fts5,wv2runtime.download,production`、`CommitID=97d7f0a`（与源码 clone 一致）；依赖清单 312 条，覆盖 Wails、SQLite、gin、Milvus 等。方法细节见第 3 章。

### 5 Phase 3：行为与 API

1. 在 headless 环境中启动桌面端：`Xvfb :99` + `openbox` + `x11vnc`（远程查看窗口）。
2. 记录窗口尺寸、菜单、后端监听地址（默认 `127.0.0.1:32111`，端口来自 `desktop-prefs.json`）。
3. 用后端日志/接口验证：
   - `registered 528 routes`、`/health` → ok
   - SQLite `data/weknora.db` 出现 `lite_embeddings_fts` 及 FTS5 影子表
   - `POST /api/v1/auth/auto-setup` → 200，自动创建本地账号
4. 记录桌面特有行为：前端相对请求走 `wails://` 自定义协议，POST 会被降级为 GET；改用壳注入的 `__WEKNORA_API_BASE__` 走 `http://127.0.0.1:32111` 后才能登录（observed，详见本地实验记录 Session 5）。

产出：一条“窗口 → 后端路由 → SQLite”的行为链，而不是零散命令。

**名称解析**

| 名称 | 含义 |
|---|---|
| Wails | Go 的桌面应用框架；Linux 下用 WebKitGTK 显示前端。 |
| WebKitGTK | GTK 的 WebKit 绑定，负责渲染 SPA。 |
| SQLite FTS5 | SQLite 的全文检索扩展；`lite_embeddings_fts` 用它做本地检索。 |
| wails:// | Wails 的自定义协议；桌面壳用它提供前端资源。 |
| auto-setup | Lite 版的免登录初始化接口，需要桌面 token。 |

### 6 Phase 4：native 分析

```bash
rea analyze /root/weknora-lite/WeKnora-Lite --provider ghidra --json
```

优先从源码找高价值入口（`cmd/desktop/main.go`、鉴权 handler、SQLite 检索），再用 Ghidra 验证；不要从 279 MB 全量反编译开始。

产出：至少 1 条函数级调用链（方法见第 4 章），并标出哪些结论静态分析给不了。

### 7 Phase 5：报告

最小结论：

- WeKnora Lite `0.8.3`，Go `1.27.3`，linux/amd64，CGO，tags `desktop,sqlite_fts5,wv2runtime.download,production`。
- 依赖覆盖桌面壳 `wails/v2`、SQLite 存储、Web/API `gin` 等（完整清单 312 条）；其余后端组件仍在二进制中，但 Lite 运行时走 SQLite。
- 行为：窗口 1280×800、后端 `127.0.0.1:32111`、528 路由、FTS5 影子表、auto-setup 200。
- 已观察的桌面特有问题：`wails://` 协议下 POST 被降级为 GET（前端相对路径请求）；本地补丁后恢复正常。
- 未完成：函数级反编译、运行时调试、真实用户数据行为。

### 8 IMA 阶段

> **名称解析**：IMA 是商用知识库/Agent 产品；本教程只讨论授权、范围与方法，具体能力以官方文档为准。

IMA 是商用闭源产品，和 WeKnora 不同。开始前必须明确：

1. 授权与许可：是否获准分析客户端/服务/协议。
2. 目标形态：客户端二进制、Web 前端、移动端、公开 API、云服务。
3. 允许的方法：公开 API/互操作、静态二进制、动态观察、抓包？
4. 数据边界：不接触真实用户数据、凭据、生产环境。
5. 停止条件：遇到登录、DRM、许可校验、反爬、加密协议时停止并回到授权范围。

在授权范围内，合理起点是：公开文档/官方 API → 授权账号的行为观察 → 客户端静态产物盘点 → 再决定是否 native 分析。**不要**用 REA 绕过登录、许可、DRM、反爬或提取用户数据。

### 9 本章来源与依据

| ID | 本地资料 | 本教程使用的内容 |
|---|---|---|
| R1 | 本地实验记录（Session 4） | Lite 桌面端构建、产物 SHA-256、Xvfb 验证 |
| R2 | 本地实验记录（Session 5） | VNC 交互、auto-setup、`wails://` POST 降级与本地补丁 |
| R3 | `website-docs/05-clients/05-desktop.md:191`（WeKnora 仓库） | Windows/Linux 自行 `wails build` 的官方说明 |
| R4 | `raw/articles/rea/docs/go-binaries.md:64-67,78-115` | buildinfo 字段与 summary 交付要求 |
| R5 | `raw/articles/rea/docs/ghidra-provider.md:1-40` | Ghidra 临时数据库与只读边界 |
| T1 | 本机实测（2026-10-27，REA 6.8.0 容器） | 279 MB 产物、`go version -m` 依赖与构建设置摘要、后端路由与 FTS5 |

---
