## 第 1 章 认识 REA

> 本章依据：本地 raw 文档 R1–R12 与 wiki 页面 W1–W9（文末列精确路径/行号）。教程解读只负责组织与举例，不引入未标注来源的事实。

### 1 一句话定义

REA（Reverse Engineer Anything）是一个本地运行的逆向工程工具：它把多种分析引擎统一成一套 CLI + MCP 工具目录，让分析者或 AI agent 按“问题 → 工具 → Evidence → 下一步”逐步调查二进制、应用、网站和运行时行为。

> 依据：R1 `README.md:63-65,165-178`（本地分析、agent/MCP、native provider、调查流程）；W1 `topic-rea-overview.md`。

### 2 不是什么

- 不是 Ghidra/IDA 的替代品；它是这些引擎之上的统一工作流。
- 不是“自动生成完整源码”的工具。
- 不是沙箱；运行时操作可能改变目标状态。
- 不是权限系统；它不会替你判断是否有权分析某个目标。

### 3 架构

```text
        ┌──────────────┐        ┌──────────────────────┐
        │  CLI (rea)   │        │  MCP server (rea mcp) │
        └──────┬───────┘        └──────────┬───────────┘
               │                           │
               └───────────┬───────────────┘
                           ▼
                  ┌──────────────────┐
                  │ shared workflows │
                  │ tool catalog     │
                  └────────┬─────────┘
                           ▼
   ┌──────────────┬──────────────┬──────────────┬─────────────┐
   │ native       │ artifact     │ runtime      │ debug/tool  │
   │ Hopper/Ghidra│ ELF/PE/Mach-O│ Frida/browser│ objdump/GDB │
   │ IDA/Kuna/JEB │ .NET/JS/APK  │ ADB/TShark   │ Rizin/Cutter│
   └──────┬───────┴──────┬───────┴──────┬───────┴──────┬──────┘
          └──────────────┴──────────────┴──────────────┘
                           ▼
                  target artifact / app / site / process
                           ▼
                  Evidence + observations + unknowns
```

> 依据：R1 `README.md:165-178`（How REA works）；R2 `docs/index.md:48-63`（CLI/MCP 指南地图）；W1 `topic-rea-overview.md`。

**名称解析**

| 名称 | 含义 |
|---|---|
| ELF | Linux/Unix 可执行与可链接格式。 |
| PE | Windows Portable Executable（.exe/.dll）。 |
| Mach-O | macOS/iOS 可执行与库格式。 |

### 4 五个核心词

| 词 | 含义 | 新手要点 |
|---|---|---|
| target | 被分析的文件、应用、网站或进程 | 先确认格式和授权 |
| provider | 实际执行分析的引擎或工具 | 能力、版本、可用性都可能不同 |
| tool | REA 暴露的原子能力 | 工具名通常 provider-neutral |
| prompt | 引导 agent 调查的工作流 | prompt 不会自行调用工具或授权 |
| Evidence | 观察、来源、覆盖、限制、未知 | 结论必须能回到 Evidence |

**名称解析**

| 名称 | 含义 |
|---|---|
| MCP | Model Context Protocol，agent 调用外部工具的协议；REA 通过 stdio/HTTP 暴露工具。 |
| Evidence | 观察 + 来源 + 覆盖 + 限制 + unknown 的结果封装。 |
| provider | 实际执行分析的引擎/适配器；REA 负责编排和 Evidence。 |
| tool / prompt | tool 是原子能力；prompt 是引导 agent 调查的工作流。 |
| Electron | 用 Chromium + Node.js 构建的桌面应用运行时。 |

### 5 逆向工具链：Ghidra、IDA、Hopper 到底是什么

REA 本身不实现完整的反汇编器和反编译器。它把外部逆向工具作为 **provider** 接入，统一成 CLI/MCP 工具。理解这一点，才能理解为什么同一操作在不同机器上结果不同，以及为什么“没装 Ghidra”时某些命令会直接报 `provider_unavailable`。

#### 5.1 Ghidra

- Ghidra 是开源逆向分析套件（NSA 维护、Apache 2.0）。REA 的 provider 要求是版本、JDK、启动超时和行为约束（R3/R5）。
- 能力：反汇编、反编译、函数/类型/引用分析、脚本化、headless 分析。
- 支持多种架构和 ELF/PE/Mach-O 等格式。
- 需要 Java/JDK；启动和导入较重，适合自动化、批量和无商业许可的场景。
- REA 的 Ghidra provider 要求 12.1.x、声明的 64 位 JDK 和宿主原生 decompiler；用 `GHIDRA_INSTALL_DIR`、`JAVA_HOME` 指定；启动超时默认 330000 ms。
- 使用临时数据库；取消运行中的操作会结束会话并丢注释，下次需要重新导入。
- 它是**静态分析**工具：读取字节、建立数据库、生成伪代码；不执行目标。

> 依据：R3 `docs/installation.md:965-1000`（12.1.x、JDK、26 个只读操作、临时数据库）；R5 `docs/ghidra-provider.md:1-40`（ephemeral database、只读边界）；R1 `README.md:65`。

#### 5.2 IDA

- IDA 是 Hex-Rays 的反汇编/反编译器；IDA Pro/IDA Home 是商业产品，IDA Free 有免费版本但功能/许可受限。
- 官方 IDA MCP（`HexRaysSA/ida-mcp`，发布说明中也称 Code Mode）是 MIT 开源的桥接层，但要求已安装 IDA 9.4+ with idalib 和 Python 3.11+；MCP 本身不附带 IDA 许可证。
- REA 的 IDA provider 做只读分析；不保存/关闭已打开的 GUI 数据库；可以连接已打开的 GUI 目标或 headless 目标，还提供可选的 native IDA MCP bridge。
- REA 适配已有的 IDA MCP 注册，不安装/激活/授权 IDA。REA 文档当前验证的上游是 `mrexodia/ida-pro-mcp`（IDA Pro）和可选的 `ida-free-mcp`（IDA Free）；官方 Code Mode 是另一个 MCP 实现，能否直接接入取决于工具 profile 是否匹配，当前文档未列为已验证。
- 适合：已有 IDA 安装/许可、需要交互式深度分析、复杂加载器/架构的团队。

**IDA Free 的局限（不建议作为 REA 主路线）**

- IDA Free 免费，但架构、加载器、插件和自动化能力比 IDA Pro/Home 少。
- 反编译依赖云反编译：需要网络，可能要求同意条款；输出和遍历范围也有上游限制。
- 适合学习、轻量分析；不适合把 headless/idalib 流水线作为核心依赖。
- 在 REA 中，IDA Free 只能走 `ida-free-mcp` 的 GUI-attached 路线：没有 native headless，direct callers 不可用；上游插件默认绑定 `0.0.0.0`，必须用防火墙或环回限制。
- **结论**：要做可重复、自动化的 REA 分析，优先 IDA Pro/Home 或 Ghidra；IDA Free 只用于学习、临时查看或验证 GUI 行为。

> 依据：R4 `docs/ida-provider.md:1-80`（IDA Free 云反编译、GUI-attached、native headless 限制、`0.0.0.0` 绑定）；X1 `HexRaysSA/ida-mcp`（MIT；README 要求 IDA 9.4+ with idalib、Python 3.11+）；X2 `https://hex-rays.com/ida-free`（IDA Free 与 Pro/Home 的功能与许可差异以官方页面为准）。

#### 5.3 Hopper

- Hopper 是独立商业软件（R3 明确说明）；主要面向 macOS，也支持 Linux x86_64。
- REA 的 Hopper provider 启动超时默认 120000 ms；调用关系完整性可能 unknown，且不会对整个二进制做全量反扫。
- 适合：macOS 桌面应用、快速交互分析、已有 Hopper 许可的用户。

> 依据：R3 `docs/installation.md:637-650`（商业软件、macOS 12+/Linux x86_64、启动超时）；R1 `README.md:65`。

#### 5.4 对比

| 工具 | 许可 | 优势 | 代价 | 在 REA 中的角色 |
|---|---|---|---|---|
| Ghidra | Apache 2.0，免费 | 多架构、可脚本化、headless、生态完整 | JDK/启动较重、交互体验一般 | native provider，做静态反编译/分析 |
| IDA | IDA Pro/Home 商业；IDA Free 免费受限；IDA MCP 开源（MIT） | 交互式体验、Hex-Rays 反编译、加载器丰富 | 引擎许可成本、自动化需额外配置 | native provider，只读分析 |
| Hopper | 商业 | macOS 体验好、上手快 | 平台/许可限制 | native provider |
| JEB | 商业 | Android/字节码/DEX 强 | 许可成本 | Android/字节码 provider |
| Kuna | 第三方 | 反编译 provider | 许可/版本以项目为准 | native provider |

#### 5.5 其他工具与 REA 的关系

REA 不是只接 Ghidra/IDA。不同工具的接入方式不同：

| 工具 | REA 入口 | 适合回答 | 关键限制 |
|---|---|---|---|
| GNU objdump | `inspect-with-objdump` | 节区/符号/重定位/DWARF/快速反汇编 | 16 MiB 输出；默认 120s；输出不归一化 |
| GDB | `debug-with-gdb` / MCP `start_gdb_session` | Linux 断点/寄存器/内存/调用栈 | 仅 Linux；需要 inferior TTY；10s 启动/120s 命令；16 MiB |
| Rizin（静态） | `execute-rizin-command` | 一次性反汇编/函数/字符串/xref | 16 MiB；默认 120s；抑制用户 settings/scripts，保留插件 |
| Rizin（调试） | `debug-with-rizin` / MCP session | Linux 调试、断点、寄存器/内存 | 仅 Linux；需要 `inferior_tty`；`dbg.exitkills=false`；30s/120s；16 MiB/4096 帧 |
| Cutter | `cutter-command` | 在 Cutter GUI 里执行 Rizin 命令 | 需要运行中的 Cutter + 桥接；25s/16 MiB；256 描述符 |
| Frida | 运行时插桩工作流 | 运行时 hook/参数/返回值 | 需要注入；可能影响目标；按 Frida 指南 |
| Chrome/CDP/Playwright | `capture-browser-scenario` 等 | 页面结构/脚本/网络/截图 | 可能访问网络/改变页面 |
| adb/JADX/apktool | `list_adb_*`、`search_android_classes`、`inspect_android_class`、`decode_android_resources` | Android 设备/APK/资源 | 需要 adb/JADX/apktool |
| TShark/capinfos | `inspect_packet_capture` | pcap/协议 | 需要 tshark/capinfos |
| sigrok-cli | `decode_signal_recording` | 信号解码 | 需要 sigrok-cli |
| WABT | `inspect_wasm_artifact` | WASM 校验/文本 | 需要 WABT |
| mitmproxy | `inspect_web_network_capture` | 原生 mitmproxy 抓包 | 需要 mitmdump |

最短的两个例子：

```bash
rea inspect-with-objdump ./program disassemble --symbol my_function
rea debug-with-gdb --inferior-tty /dev/pts/7 'file ./program'
```

##### 5.5.1 Rizin 在 REA 中的三种用法

Rizin 不是 Ghidra/IDA 的替代品；它更像“可脚本化的逆向框架 + 调试器”。在 REA 里有三种入口：

1. **一次性静态命令**
   ```bash
   rea execute-rizin-command ./program iI
   rea execute-rizin-command ./program 'aaa; afl'
   rea execute-rizin-command ./program 'pdf @ main'
   ```
   - 使用 `-N -q -e scr.color=0 -c` 一次性执行，抑制用户 settings/scripts，但保留插件。
   - 输出原始 stdout/stderr，16 MiB 上限；默认 120s。
   - 适合快速看文件信息、字符串、函数列表、单函数反汇编、xref。
   - 命令/插件以 REA 进程用户权限运行，可能访问文件/网络；不是沙箱。

2. **Linux 调试会话**
   ```bash
   rea debug-with-rizin --inferior-tty /dev/pts/7 ./program dr
   ```
   - MCP 对应 `start_rizin_debug_session`、`rizin_debug_command`、`rizin_debug_session_status`、`close_rizin_debug_session`。
   - 仅 Linux；必须给 `inferior_tty`；`dbg.exitkills=false`，目标可能在 REA 关闭后继续运行并写 TTY。
   - 关闭时 REA 只 detach 唯一的 `ptrace://` 描述符，并校验 `/proc/<pid>/fd/1`、`/proc/<pid>/fd/2` 仍指向该 TTY。
   - 启动 30000 ms、每命令 120000 ms；输出 16 MiB、4096 帧；没有运行时覆盖。
   - 适合断点、寄存器、内存、运行时反汇编；不要把它当静态反编译器。

3. **Cutter GUI 桥接**
   - `cutter-command` 通过本地 Python bridge 和 authenticated loopback 与运行中的 Cutter 通信。
   - `REA_CUTTER_BRIDGE_DIR` 必须在 REA 和启动 Cutter 的环境一致；需要 v2 握手。
   - 每请求 25000 ms、16 MiB；发现上限 256 描述符。
   - 适合已经在 Cutter 里交互分析、想让 agent 复用当前会话的场景；不是 headless。

##### 5.5.2 工具选择顺序

1. `inspect-artifact` / 内置读取器：格式、摘要、Go buildinfo、ELF notes、.NET 元数据。
2. `inspect-with-objdump` / `execute-rizin-command`：快速反汇编、节区、符号、xref。
3. Ghidra/IDA/Hopper：需要伪代码/类型/CFG/调用关系。
4. GDB/Rizin debug：需要断点、寄存器、内存、运行时路径。
5. Frida/浏览器/ADB：需要运行时插桩、页面/设备观察。
6. 所有步骤都保留 Evidence、provider/version 和 unknown。

> 依据：R11 `docs/reverse-engineering-tools.md:3,15-92,94-148`（objdump/GDB/Rizin/Cutter 的命令、TTY、超时、输出上限、描述符校验）；R12 `docs/cli.md:56-133`（输出/退出码）；X1 `HexRaysSA/ida-mcp`（IDA MCP 与引擎许可区别）。

#### 5.6 哪些能力不需要这些工具

REA 内置了一些静态读取器，不需要 Ghidra/IDA/Hopper：

- `inspect-go-binary`：Go buildinfo、模块依赖、构建设置。
- `inspect-elf-notes`：ELF note。
- `inspect-managed-*`：.NET/CLI 元数据。
- `inspect-artifact`：格式、摘要、基础结构。
- `inspect-python-pyc`、`inspect-sqlite-database` 等专用读取器。

所以“REA 的依赖”不是一个 npm 包列表，而是“**必需运行时 + 按能力启用的外部工具**”。

#### 5.7 常见误解

- **“REA 依赖 Ghidra”**：只有需要原生反编译/函数分析时才需要。Go buildinfo、ELF notes、.NET 元数据、基础产物盘点都不需要 Ghidra。
- **“装了 Ghidra 就能分析所有目标”**：provider 支持的目标类型、架构、版本和平台都有限；不支持的格式仍然会报错。
- **“IDA/Hopper 是 REA 自带的”**：不是。它们是独立工具；IDA MCP 桥可以开源，但仍需已安装的 IDA；REA 只提供 adapter。
- **“provider 版本不影响结果”**：会影响。版本、能力、配置和限制都可能不同；未实测前应保持 unknown。

> 依据：R1 `README.md:65,186-222`；R3 `docs/installation.md:637-650,965-1000`；R4 `docs/ida-provider.md:1-80`；R5 `docs/ghidra-provider.md:1-40`；W2 `concept-rea-provider-architecture.md`。

**名称解析**

| 名称 | 含义 |
|---|---|
| Ghidra | 开源逆向分析套件；REA 通过 provider 调用。 |
| IDA | IDA Pro/Home 商业、IDA Free 免费受限；IDA MCP 开源桥接；REA provider 只读。 |
| Hopper | 商业反汇编/反编译器，主要 macOS/Linux x86_64。 |
| JEB / Kuna | 其他第三方/商业反编译 provider。 |
| objdump | GNU binutils 的静态反汇编/节区/符号/重定位工具。 |
| GDB | Linux 调试器；REA 用 persistent MCP session 或 one-shot CLI。 |
| Rizin | 逆向工程框架/调试器；REA 用 execute-rizin-command 做静态命令，用 debug-with-rizin 做 Linux 调试。 |
| Cutter | Rizin 的 GUI；REA 通过 cutter-command 桥接。 |
| Frida | 运行时插桩工具。 |
| CDP | Chrome DevTools Protocol，浏览器调试/观察协议。 |
| ADB / JADX / apktool | Android 调试桥 / DEX 反编译 / APK 资源反编译工具。 |
| TShark | Wireshark 命令行协议分析工具。 |
| sigrok | 逻辑信号采集/解码工具链。 |
| WABT | WebAssembly Binary Toolkit。 |
| mitmproxy | 可编程 HTTP(S) 代理/抓包工具。 |
| Playwright | 浏览器自动化库，用于执行有界浏览器场景。 |
| Go buildinfo | Go 二进制中嵌入的编译版本、模块、依赖和构建设置。 |

### 6 REA 的基础依赖

#### 6.1 必需依赖

| 依赖 | 用途 | 缺失时 |
|---|---|---|
| Node.js 22.23.4+ / 24.22.1+ / 26.12.1+ | CLI、MCP server、REA 运行时 | 无法启动或 setup 失败 |
| npm / npx | 安装/运行 `rea-agents` | 无法安装/启动 |
| Linux 或 macOS + shell + 文件系统 | 本地分析、Evidence、会话 | 无法运行 |
| `ps`（Linux 的 procps） | 进程所有权/清理检查 | 进程相关能力不可用或报错 |
| 原生 PTY | `capture-process` 等运行时捕获 | 新进程捕获不可用（Windows 尤其） |

> 依据：R3 `docs/installation.md:1526`（Node 版本）；R9 `docs/process-capture.md:9-31`（当前用户权限、PTY、Windows 限制）；R3 `docs/installation.md:377-379`（doctor）。

#### 6.2 可选依赖（按目标/能力启用）

| 依赖 | 用途 | 典型命令 | 缺失时 |
|---|---|---|---|
| JDK + Ghidra 12.1.x | 原生反汇编/反编译 | `analyze`、`function`、`decompile` | `provider_unavailable` |
| IDA + Hex-Rays | 交互式/深度原生分析 | `analyze --provider ida` | `provider_unavailable` |
| Hopper | macOS/Linux 原生分析 | `analyze --provider hopper` | `provider_unavailable` |
| JEB | Android/DEX/字节码 | `decompile-jeb-item` 等 | `provider_unavailable` |
| GNU objdump | 快速反汇编/节区/符号 | `inspect-with-objdump` | 对应工具不可用 |
| GDB | Linux 调试 | `debug-with-gdb` | 对应工具不可用 |
| Rizin / Cutter | 调试/交互分析 | `debug-with-rizin`、`cutter-command` | 对应工具不可用 |
| Python + pwntools 4.15.0 + pyelftools 0.33 + Unicorn 2.1.2 | `inspect-binary-layout` 等 | `inspect-binary-layout` | `provider_unavailable` |
| adb / JADX / apktool | Android 设备/APK/资源 | Android 工作流 | 对应工具不可用 |
| TShark / capinfos | pcap/协议分析 | `inspect-pcap-*` | 对应工具不可用 |
| sigrok-cli | 信号解码 | `decode-signal-recording` | 对应工具不可用 |
| Chrome-family / Playwright | 浏览器观察/场景 | `capture-browser-scenario` | 浏览器能力不可用 |
| WABT 1.0.42 | WASM 校验/文本 | `inspect-wasm-artifact` | 对应工具不可用 |
| mitmproxy | 原生抓包 | 网络捕获工作流 | 对应工具不可用 |

> 依据：R3 `docs/installation.md:637-650,965-1000`（Hopper/Ghidra 前置）；R4 `docs/ida-provider.md:1-80`；R10 `docs/binary-diagnostics.md:15-31`（pwntools/Python）；R11 `docs/reverse-engineering-tools.md:3,37-80`（objdump/GDB/Rizin/Cutter）。

#### 6.3 关于 Python/pwntools

`inspect-binary-layout` 不内置 ELF 布局解析，而是要求调用方提供一个绝对路径的 Python，并安装：

- `pwntools` 4.15.0
- `pyelftools` 0.33
- `Unicorn` 2.1.2

通过环境变量 `REA_PWNTOOLS_PYTHON` 指向该 Python。REA **不会**替你安装 Python 或这些包。缺失时的错误是 `provider_unavailable`，不是“布局为空”。

它回答的是**文件级布局**问题：节区/段、符号表、重定位、静态链接名、缓解标志（NX/PIE/RELRO 等），以及文件偏移与链接地址的对应；不反汇编、不执行，也不提供运行时加载基址。与 `readelf` 的交叉验证见第 3 章。它在教程里是**可选项**：`readelf`/`objdump` 已覆盖基本布局，只有需要结构化 Evidence 时才启用。

> 依据：R10 `docs/binary-diagnostics.md:15-31`；R13 `docs/testing.md:2014`（pwntools 4.15.0 / pyelftools 0.33 / Unicorn 2.1.2）；E1 本机 2026-10-26 实测错误信息。

#### 6.4 关于许可证

- Ghidra：Apache 2.0，免费。
- IDA Pro/Home、Hopper、JEB：商业工具，需要你自行合法获得许可证；官方 IDA MCP（`HexRaysSA/ida-mcp`）和部分桥接层是 MIT/开源，但不包含引擎许可证；REA 不提供许可证/激活。
- objdump、GDB、Rizin、Cutter、Frida、TShark、adb、JADX、apktool、WABT 等：许可证与版本以各自项目为准。
- 第三方工具的行为、版本和能力在实测前可能 unknown；`rea capabilities --json` 只反映当前安装。

#### 6.5 安装责任

`npx rea-agents setup` 只负责 REA 自身的 agent 注册/技能配置，**不会**安装或升级 Node、npm、Homebrew、Java、Ghidra、IDA、Hopper 等无关工具。provider 需要你按官方文档自行安装，再用 `rea doctor` 检查。

#### 6.6 缺失依赖的典型报错

| 现象 | 可能原因 |
|---|---|
| `provider_unavailable` | Ghidra/IDA/Hopper/Python 等 provider 未安装、路径错误或版本不匹配 |
| `Set REA_PWNTOOLS_PYTHON ...` | 缺少 pwntools/pyelftools/Unicorn，或没设绝对路径 |
| `couldn't bind HTTP server socket` | llama-server/其他服务端口冲突（不是 REA 错误） |
| 进程捕获失败 | 缺少原生 PTY / `ps` / 平台不支持 |
| 浏览器捕获失败 | 没有 Chrome-family 浏览器或 Playwright 依赖 |
| pcap 检查失败 | 缺少 tshark/capinfos |

**名称解析**

| 名称 | 含义 |
|---|---|
| PTY | Pseudo-Terminal，伪终端；让被捕获进程以为自己有终端。 |
| procps | Linux 进程工具集；REA 用其中的 `ps` 做所有权检查。 |
| pwntools / pyelftools / Unicorn | Python 的二进制/ELF/CPU 模拟库；`inspect-binary-layout` 需要。 |
| REA_PWNTOOLS_PYTHON | 指向安装了上述包的 Python 解释器的绝对路径。 |

### 7 REA 的工作原理

REA 的核心不是“自己反编译”，而是**编排 + 证据化**。可以把它分成五层：

```text
1. 适配层      CLI (rea) / MCP server (rea mcp)
                 ↓
2. 工作流层    tool catalog / prompts / completion / 参数校验
                 ↓
3. Provider 层 provider registry / readiness / adapter
                 ↓
4. 目标层      target session / snapshot / provider process
                 ↓
5. 证据层      raw_result + normalized_result + limitations + unknowns
```

#### 7.1 REA 自己负责什么

- 输入校验、目标身份（路径、SHA-256、格式、架构）。
- provider 选择、就绪检查、会话生命周期。
- 工具契约、参数校验、资源上限（文件大小、响应大小、超时）。
- 结果规范化、coverage/limitations/unknowns。
- Evidence 创建、保留、导出、比较、快照。
- CLI/MCP 输出、错误分类、退出码。
- agent prompts、completion、工具目录。

#### 7.2 Provider 负责什么

- 实际的 disassembly/decompilation/debugging。
- 自己的版本、能力、平台限制、许可证。
- 自己的项目/会话/临时数据库。
- 自己声明的副作用（例如启动进程、写临时文件、访问网络）。

#### 7.3 Target 负责什么

- 提供真实字节或行为。
- 不保证你能分析它；授权和范围由你确认。

#### 7.4 一次调用的内部步骤

1. **解析输入**：CLI 参数或 MCP 请求；MCP 要求绝对路径。
2. **识别目标**：读取文件头/大小/SHA-256，确定格式、架构、slice。
3. **选择 provider**：
   - 显式 `--provider`；
   - 环境变量 `REA_ANALYSIS_PROVIDER`；
   - MCP 的 `provider_id`；
   - 否则按工具/格式自动选择或使用内置读取器。
4. **检查就绪**：provider 是否存在、版本是否匹配、依赖是否齐全。
5. **建立会话**：Ghidra 启动 headless/JVM 并导入；Hopper 启动；IDA 连接/打开；内置读取器在进程内执行。
6. **执行操作**：反编译、列出函数、搜索字节、捕获进程等。
7. **应用资源上限**：文件大小、响应大小、超时、批处理上限；超限返回 `resource_constraint`。
8. **规范化结果**：统一字段、位置、coverage、limitations、unknowns。
9. **创建 Evidence**：保存 raw_result、normalized_result、analysis_profile；分配 `evidence_id`，放入会话 ledger。
10. **返回**：CLI 输出 TOON/JSON/JSONL；MCP 返回 `structuredContent`；失败返回 `isError` + `{error}`。
11. **生命周期**：复用快照、比较、导出；关闭目标并清理自有资源。

> 依据：R6 `docs/mcp-contracts.md:17-21,185-205,743-762`（结果结构、10 MiB、HTTP 传输）；R7 `docs/tool-design.md:51-77,133-205`（契约、生命周期）；R12 `docs/cli.md:56-133,921-973`（JSONL/退出码）；W3 `concept-rea-evidence-model.md`；W7 `entity-rea-cli.md`；W8 `entity-rea-mcp-server.md`。

#### 7.5 CLI 与 MCP 的差异

| 维度 | CLI | MCP |
|---|---|---|
| 使用方式 | 人/脚本执行单次命令 | agent 长连接调用工具 |
| 生命周期 | 以单次调用为主 | 会话可保留目标/Evidence |
| 输出 | TOON/JSON/JSONL + 退出码 | `structuredContent` + `evidence_id` |
| 状态 | 快照/文件复用 | 会话 ledger/retained evidence |
| 传输 | 本地进程 | stdio（默认）或 HTTP |
| 适合 | 学习、调试、CI | agent 调查、多步工作流 |

CLI 更适合新手先理解每个工具；MCP 更适合让 agent 连续调查。两者共享同一套工具契约和 Evidence 语义。

#### 7.6 为什么工具名是 provider-neutral

同一个 `analyze_function` 可以由 Ghidra、IDA 或 Hopper 执行。工具名不绑定某个引擎，agent 才能在不同机器/许可条件下复用同一套工作流；provider 差异通过 `analysis_profile` 和限制体现，而不是写死在工具名里。

#### 7.7 静态与运行时的分界

- **静态**：读取字节，不执行目标。Ghidra/IDA/Hopper/objdump/内置读取器。
- **运行时**：启动/附加/交互目标。GDB/Rizin/Frida/浏览器/ADB/进程捕获。
- 静态结果不证明运行时行为；运行时结果也不覆盖所有执行路径。

#### 7.8 失败时发生什么

REA 采用 fail-closed 思路：

- 输入不合法 → `invalid_request`，不猜。
- provider 不可用 → `provider_unavailable`，不静默换 provider。
- 资源超限 → `resource_constraint`，改用其他输入或工具。
- 超时/取消 → 保留可恢复状态；清理失败保留 `cleanup_pending`。
- 失败通常没有 `structuredContent`；你要读错误类别和恢复建议。

**名称解析**

| 名称 | 含义 |
|---|---|
| TOON | REA CLI 默认终端输出格式。 |
| raw_result / normalized_result | provider 原始输出 / REA 统一后的结果。 |
| analysis_profile | 本次分析的 provider/profile/设置身份。 |
| evidence_id | 保留 Evidence 的引用；不是有效性证明。 |
| structuredContent | MCP 结果中的结构化字段。 |
| completion | prompt 参数的候选补全；不是工具调用。 |
| coverage / limitation | 覆盖范围 / 工具声明的限制。 |
| snapshot | 可复用的分析结果。 |

### 8 一次 REA 调用到底发生了什么

以 WeKnora 为例，把四个典型命令放在一起看：

| 命令 | 使用 provider？ | 是否执行目标 | 主要限制 | 产出 |
|---|---|---|---|---|
| `inspect-artifact WeKnora-Lite` | 内置产物图 | 否 | 不同工具预算不同；本次 279 MB 可处理 | 格式、摘要、基础结构 |
| `go version -m WeKnora-Lite` | Go 工具链 | 否 | 无 256 MiB 限制（只读 buildinfo） | Go 版本、依赖与构建设置摘要 |
| `analyze WeKnora-Lite --provider ghidra` | Ghidra | 否（静态） | 启动/导入重，330 s 超时 | 函数/类型/引用概览 |
| `capture-process scenario.json` | 内置 PTY/运行时 | **是** | 有界场景、超时、PTY | 终端/生命周期/文件系统观察 |

> `inspect-go-binary` 仅适用于 **≤256 MiB** 的 Go 二进制；超限改用 `go version -m`（见第 3 章）。

> 依据：R8 `docs/go-binaries.md:80-100,186-193`（Go 字段、256 MiB）；R9 `docs/process-capture.md:9-31`（运行时边界）；W5 本地实验记录（Session 4–5，Lite 桌面端）；E1 本机 2026-10-27 命令实测。

#### 8.1 例 1：`inspect-artifact`

1. 读取 ELF 头、节区/段、大小。
2. 计算 SHA-256，确定架构/格式。
3. 不启动 provider、不执行目标。
4. 返回 Evidence：format、digest、architecture、provider、limitations。

它回答“这是什么文件”，不回答“函数做什么”。

#### 8.2 例 2：`inspect-go-binary`

1. 检查文件大小是否 ≤ 256 MiB。
2. 扫描 Go buildinfo（inline 或历史编码）。
3. 解析 Go 版本、主模块、依赖、构建设置。
4. 生成 Evidence；`complete` 表示模块文本解析完整。
5. 不启动 Go/Ghidra/IDA，不执行目标。

它回答“怎么构建的、依赖了什么”，不回答“运行时怎么走”。只适用于 Go 二进制；非 Go 没有 buildinfo，记为 unknown。超过 256 MiB 时不要用本工具，改用 `go version -m`（第 3 章）。

#### 8.3 例 3：`analyze --provider ghidra`

1. 选择 Ghidra provider；检查 JDK/路径/版本。
2. 启动 Ghidra headless，导入目标，建立临时数据库。
3. 运行分析/列函数/生成伪代码。
4. 规范化函数、地址、调用、类型等结果。
5. 保留 Evidence；关闭会话时清理临时库。
6. 全程静态，不执行目标。

它回答“代码结构大概是什么”，但伪代码可能不完整/不准确，且不证明运行时行为。

#### 8.4 例 4：`capture-process`

1. 解析场景 JSON（可执行文件、参数、超时、交互）。
2. 在 PTY 中启动命令，记录终端输出和生命周期。
3. 按配置观察文件系统快照。
4. 返回 capture Evidence；可比较两次运行。
5. 它不是沙箱；命令以当前用户权限运行，可能产生真实副作用。

它回答“这个命令实际做了什么”，但只覆盖这次场景。

#### 8.5 一个完整的调查闭环

```text
问题
  → inspect-artifact（格式/摘要）
  → inspect-go-binary（构建/依赖）
  → 源码/API 盘点
  → 需要时 analyze/function/decompile（provider）
  → 需要时 capture-process/browser（运行时）
  → Evidence 报告（observed / inferred / unknown）
  → 下一步或停止
```

### 9 目标矩阵（先知道边界）

| 类别 | 典型目标 | 主要能力 |
|---|---|---|
| 原生二进制 | ELF/PE/Mach-O | 伪代码、汇编、字符串、符号、调用、xref |
| Go/Rust | 带构建元数据的二进制 | 编译器版本、模块依赖、构建设置 |
| 模型/GPU | GGUF、safetensors、CUDA/HIP | 元数据、张量声明、内核元数据 |
| 托管代码 | .NET、IL2CPP、Godot PCK、Flutter | 元数据、成员、资源 |
| JS/Electron | 应用目录、ASAR、网站 | 静态图、模块、IPC、运行时观察 |
| 移动端 | APK、ADB、JEB、apktool | 清单、类/方法、签名、设备状态 |
| Apple | .app、IPS、plist、资产目录 | 包结构、崩溃、资源 |
| 其他 | 固件、WASM、EVM、SQLite、pyc | 结构、接口、schema、字节码 |
| 网络/信号 | HAR、pcap、qlog、sigrok | 请求、协议、TLS、信号 |
| 进程行为 | 本地命令 | 终端、交互、生命周期、文件系统观察 |

> 依据：R1 `README.md:186-222`（目标表）；R2 `docs/index.md:48-63`（指南地图）；W4 `concept-rea-target-matrix.md`。

**名称解析**

| 名称 | 含义 |
|---|---|
| APK | Android Package，Android 应用安装包。 |
| IL2CPP | Unity 把 C# 转成 C++ 后的中间层/元数据体系。 |
| Godot PCK | Godot 引擎资源包。 |
| WASM | WebAssembly 可移植字节码。 |
| EVM | Ethereum Virtual Machine 字节码/执行环境。 |
| pyc | CPython 编译后的字节码文件。 |
| ASAR | Electron 的应用归档格式。 |
| xref | cross-reference，交叉引用。 |
| HAR | HTTP Archive，浏览器网络请求/响应记录。 |
| pcap | Packet Capture，网络抓包文件。 |
| qlog | QUIC/HTTP3 事件日志。 |
| IPS | Apple 崩溃报告格式。 |

### 10 调查循环

1. **定义问题**：我要回答什么？版本？依赖？函数逻辑？API 行为？
2. **选目标与 provider**：格式、平台、可用工具。
3. **调用工具**：CLI 或 MCP。
4. **读 Evidence**：观察到了什么？覆盖了什么？哪些是 unknown？
5. **决定下一步**：换 provider、补输入、做运行时观察、停止并报告。

### 11 本章练习

1. 用一句话分别解释 Ghidra、IDA、Hopper 的定位和许可差异。
2. 判断下面任务需要哪些依赖：
   - 读取 Go buildinfo
   - 生成原生函数伪代码
   - 捕获一个 CLI 的终端输出
   - 检查 pcap
   - 观察浏览器页面
3. 对一个小 Go 二进制（≤256 MiB）跑 `rea inspect-artifact` 和 `rea inspect-go-binary`，判断哪一步用了 provider、哪一步没有。
4. 跑 `rea capabilities --json`，找出一个 `provider_unavailable` 的操作，并说明缺什么。
5. 把下面问题与工具配对：
   - “这个二进制用什么 Go 版本构建？” → ？
   - “这个函数有没有调用网络 API？” → ？
   - “这个 CLI 会写哪些文件？” → ？
   - “这个网站加载了哪些脚本？” → ？

参考答案：`inspect-go-binary`；`function`/`xrefs`（需 provider）；`capture-process`；JS/网站静态分析或浏览器场景。

### 12 自检

- REA 需要把 Ghidra/IDA 作为 npm 依赖打包进来吗？为什么？
- `provider_unavailable` 和“目标里没有这个函数”有什么区别？
- 为什么 `inspect-go-binary` 有 256 MiB 上限，而 `inspect-artifact` 这次能处理 279 MB？
- Evidence 里的 unknown 为什么不能当成“没有”？
- 为什么不能只看工具退出码？

### 13 本章来源与依据

| ID | 本地资料 | 本教程使用的内容 |
|---|---|---|
| R1 | `raw/articles/rea/README.md:63-65,165-178,186-222` | REA 定位、工作方式、目标矩阵、native provider |
| R2 | `raw/articles/rea/docs/index.md:28-63` | 文档任务地图、CLI/JS/MCP 入口 |
| R3 | `raw/articles/rea/docs/installation.md:41-143,377-379,417-421,637-650,965-1000,1526` | setup 参数、doctor、update/uninstall、Hopper/Ghidra、Node 版本 |
| R4 | `raw/articles/rea/docs/ida-provider.md:1-80` | IDA provider、attached/headless、只读、生命周期 |
| R5 | `raw/articles/rea/docs/ghidra-provider.md:1-40` | Ghidra 临时数据库、只读边界 |
| R6 | `raw/articles/rea/docs/mcp-contracts.md:17-21,185-205,743-762` | MCP 结果结构、10 MiB、HTTP 传输 |
| R7 | `raw/articles/rea/docs/tool-design.md:51-77,133-205` | 工具契约、observed/derived/inferred、生命周期 |
| R8 | `raw/articles/rea/docs/go-binaries.md:80-100,186-193` | Go buildinfo 字段、256 MiB |
| R9 | `raw/articles/rea/docs/process-capture.md:9-31` | 进程捕获权限、PTY、Windows |
| R10 | `raw/articles/rea/docs/binary-diagnostics.md:15-31` | pwntools/Python 依赖 |
| R11 | `raw/articles/rea/docs/reverse-engineering-tools.md:3,15-92,94-148` | objdump/GDB/Rizin/Cutter 命令、TTY、超时、输出上限 |
| R12 | `raw/articles/rea/docs/cli.md:56-133,921-973` | JSONL/退出码/输出 |
| R13 | `raw/articles/rea/docs/testing.md:2014` | pwntools/pyelftools/Unicorn 版本 |
| X1 | `https://github.com/HexRaysSA/ida-mcp` | 官方 IDA MCP（MIT）；README 要求 IDA 9.4+ with idalib、Python 3.11+；MCP 不含引擎许可 |
| X2 | `https://hex-rays.com/ida-free` | IDA Free 官方页面；免费版与 Pro/Home 的功能、许可差异以官方说明为准 |
| W1 | `wiki/topics/topic-rea-overview.md` | REA 全景与架构 |
| W2 | `wiki/concepts/21-rea/concept-rea-provider-architecture.md` | provider 架构 |
| W3 | `wiki/concepts/21-rea/concept-rea-evidence-model.md` | Evidence 模型 |
| W4 | `wiki/concepts/21-rea/concept-rea-target-matrix.md` | 目标矩阵 |
| W5 | 本地实验记录（Session 4–5） | WeKnora Lite 桌面端实例：构建、交互与行为观察 |
| W6 | `wiki/entities/21-rea/entity-rea-installation.md` | 安装/依赖摘要 |
| W7 | `wiki/entities/21-rea/entity-rea-cli.md` | CLI 摘要 |
| W8 | `wiki/entities/21-rea/entity-rea-mcp-server.md` | MCP 摘要 |
| W9 | `wiki/queries/query-how-to-analyze-go-binary-with-rea.md` | Go 二进制操作问答 |
| E1 | 本机 2026-10-26 实测 | `inspect-artifact`、`inspect-go-binary`、`rea capabilities`、256 MiB 错误 |

> 说明：所有 REA 行为、版本、限制与命令均来自上表 raw/wiki/E1；X1/X2 是官方外部资料，用于澄清 IDA MCP、IDA Free 与 IDA 引擎许可的区别。教程中的类比和章节组织属于解释性内容，不新增事实。

---

