## 附录 B 术语表

> 只收录逆向/REA 领域术语。Java/JVM、Linux 常用命令、HTTP/JSON、Docker、Redis、Postgres、Git、SHA-256 等高级开发常识不在此表。

### 1 REA 核心

| 术语 | 含义 |
|---|---|
| REA | Reverse Engineer Anything，本地运行的逆向工程 CLI + MCP 工具。 |
| MCP | Model Context Protocol，agent 调用外部工具的协议；REA 通过 stdio 或 HTTP 暴露工具。 |
| provider | 实际执行分析的引擎/适配器；REA 负责编排、工具契约和 Evidence。 |
| tool | REA 暴露的原子能力，例如 `inspect-go-binary`。 |
| prompt | 引导 agent 调查的工作流；不会自行调用工具或授权。 |
| completion | prompt 参数的候选补全；不是工具调用。 |
| tool catalog | REA 暴露的工具清单；不是安全策略或完整能力证明。 |
| session | MCP 或分析会话；`initialize` / `DELETE` 管理 MCP 会话。 |
| Evidence | 观察 + 来源 + 覆盖 + 限制 + unknown 的结果封装。 |
| raw_result | provider 原始输出。 |
| normalized_result | REA 统一后的结果。 |
| analysis_profile | 本次分析的 provider/profile/设置身份。 |
| evidence_id | 保留 Evidence 的引用；不是有效性证明。 |
| coverage | 本次分析覆盖的范围；不等于完整性。 |
| limitation | 工具明确声明的限制。 |
| unknown | 没有覆盖/无法确认的事实；不等于不存在。 |
| observed / derived / inferred | 观察 / 按规则推导 / 基于证据推断。 |
| snapshot | 可复用的分析结果，要求输入/参数/provider 一致。 |
| retained evidence | 会话中保留的完整 Evidence。 |
| analysis view | 对保留 Evidence 的投影视图（summary/分页/选择字段）。 |
| structuredContent | MCP 工具结果中的结构化字段。 |

### 2 产物与分析格式

| 术语 | 含义 |
|---|---|
| ELF | Linux/Unix 可执行与可链接格式。 |
| PE | Windows Portable Executable（.exe/.dll）。 |
| Mach-O | macOS/iOS 可执行与库格式。 |
| container | 文件/打包容器（ELF、PE、Mach-O、APK、ASAR、PCK 等）；不指 Docker 容器。 |
| segment | ELF 的装载段（program header 描述），与 section 不同。 |
| note | ELF 的元数据表（PT_NOTE/SHT_NOTE），常见 build-id、ABI tag、Go note。 |
| ASAR | Electron 的应用归档格式。 |
| APK | Android Package，Android 应用安装包。 |
| IL2CPP | Unity 把 C# 转成 C++ 后的中间层/元数据体系。 |
| Godot PCK | Godot 引擎资源包。 |
| WASM | WebAssembly 可移植字节码。 |
| EVM | Ethereum Virtual Machine 字节码/执行环境。 |
| pyc | CPython 编译后的字节码文件。 |
| HAR | HTTP Archive，浏览器网络请求/响应记录。 |
| pcap | Packet Capture，网络抓包文件。 |
| qlog | QUIC/HTTP3 事件日志。 |
| IPS | Apple 崩溃报告格式。 |
| stripped | 二进制符号表被移除，函数名/调试信息减少。 |
| symbol | 符号表中的函数/变量名及其地址。 |
| section | ELF 中的代码/数据分区（.text/.rodata 等）。 |
| xref | cross-reference，交叉引用。 |
| CFG | Control-Flow Graph，控制流图。 |
| decompiler | 把汇编/IR 还原成伪代码的工具。 |
| disassembler | 把机器码转成汇编的工具。 |
| headless | 不启动 GUI，适合自动化和 CI。 |
| ephemeral database | Ghidra 的临时分析数据库；取消/关闭后清理。 |
| dossier | REA 对某个函数/目标的证据集合。 |
| p-code / high-pcode | Ghidra 的中间表示；high-pcode 更接近高级语义。 |
| Windows P0 | REA 的 Windows Ghidra 实验性只读边界。 |
| mutation authority | 修改目标/数据库的权限；P0 没有。 |

### 3 Go 与编译

| 术语 | 含义 |
|---|---|
| buildinfo | Go 二进制中嵌入的构建元数据。 |
| module / replacement | Go 模块路径/版本；replacement 是 `replace` 指令替换的源。 |
| ldflags | 链接器参数，常用 `-X` 注入版本/commit。 |
| CGO | Go 调用 C 的机制；`CGO_ENABLED=1` 表示启用。 |
| GOOS / GOARCH | 目标 OS/架构，如 linux/amd64。 |
| pclntab | Go 的程序计数器行表，保存函数/行号映射。 |
| resource_constraint | REA 的资源约束错误类别（大小/超时/预算）。 |

### 4 运行时与传输

| 术语 | 含义 |
|---|---|
| PTY | Pseudo-Terminal，伪终端；让被捕获进程以为自己有终端。 |
| procps | Linux 进程工具集；REA 用其中的 `ps` 做所有权检查。 |
| Streamable HTTP | MCP 的 HTTP 传输；REA 在 `/mcp` 暴露，默认 loopback、无 TLS/认证。 |
| JSON-RPC | MCP 使用的 RPC 消息格式。 |
| TOON | REA CLI 默认终端输出格式。 |
| source map | 把生成代码位置映射回原始源码的文件。 |
| embedding | 把文本转成向量表示；RAG 检索的基础。 |

### 5 工具

| 术语 | 含义 |
|---|---|
| Ghidra | 开源逆向分析套件；REA 通过 provider 调用。 |
| IDA | Hex-Rays 商业反汇编/反编译器；REA provider 只读。 |
| Hopper | 商业反汇编/反编译器，主要 macOS/Linux x86_64。 |
| JEB | 商业逆向/反编译工具，常用于 Android/DEX。 |
| Kuna | 第三方反编译 provider；许可/版本以项目为准。 |
| objdump | GNU binutils 的反汇编/节区/符号工具。 |
| GDB | GNU 调试器；REA 的持久会话仅在 Linux 可用。 |
| Rizin | 逆向工程框架/调试器。 |
| Cutter | Rizin 的 GUI。 |
| Frida | 运行时插桩工具。 |
| CDP | Chrome DevTools Protocol，浏览器调试/观察协议。 |
| ADB | Android Debug Bridge，Android 设备/应用调试桥。 |
| JADX | Android DEX 反编译工具。 |
| apktool | Android APK 资源反编译/打包工具。 |
| TShark | Wireshark 命令行协议分析工具。 |
| sigrok | 逻辑信号采集/解码工具链。 |
| WABT | WebAssembly Binary Toolkit。 |
| mitmproxy | 可编程 HTTP(S) 代理/抓包工具。 |
| Playwright | 浏览器自动化库，用于执行有界浏览器场景。 |
| pwntools / pyelftools / Unicorn | Python 的二进制/ELF/CPU 模拟库；`inspect-binary-layout` 需要。 |
| REA_PWNTOOLS_PYTHON | 指向安装了上述包的 Python 解释器的绝对路径。 |

### 6 AgentEnv

| 术语 | 含义 |
|---|---|
| AgentEnv | 从录制数据构建的可运行 agent 环境框架。 |
| manifest | 描述环境工具、数据、读写的清单。 |
| state | 从录制推导出的本地状态，用于回答读请求。 |
| replay | 重放录制响应，不是真正执行原应用。 |
| probe | 对原应用的独立调用序列，用于和导出的环境比较。 |
| bundle | 导出的环境目录（manifest/evidence/Dockerfile/测试）。 |
| smoke task | 检查环境能启动并响应基本工具调用的最小测试。 |

### 7 项目与依赖

| 术语 | 含义 |
|---|---|
| WeKnora | 腾讯开源的本地知识库/RAG 平台；本教程的练习靶场。 |
| IMA | 商用知识库/Agent 产品；本教程只讨论授权、范围与方法，具体能力以官方文档为准。 |
| desktop / sqlite_fts5 | WeKnora Lite 的构建 tag；桌面壳与 SQLite 全文检索。 |
| asynq | 基于 Redis 的 Go 后台任务队列。 |
| pgx | Go 的 PostgreSQL 驱动。 |
| Milvus / Qdrant / Weaviate | 向量数据库，用于相似度检索。 |
| mcp-go | Go 的 MCP 实现库。 |
| chromedp | Go 的 Chrome DevTools Protocol 客户端。 |
| MinIO | S3 兼容对象存储。 |
| Neo4j | 图数据库，用于知识图谱。 |
| Ollama | 本地 LLM 运行时。 |
