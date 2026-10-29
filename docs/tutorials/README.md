# REA 新手教程

REA（Reverse Engineer Anything）把逆向工程能力封装成一套 CLI 与 MCP 工具，让分析者在本地检查二进制、应用、网站和运行时行为，并返回带 Evidence、限制和未知项的结果。

本教程以 REA 6.8.0 为基准，从安装与就绪检查开始，依次覆盖文件画像、函数级分析、运行时观察、MCP 集成和 WeKnora Lite 综合实战。教程强调 **Evidence**：区分 observed、inferred 和 unknown；静态检查不证明运行行为，反编译成功也不等于结论完整。

> 只分析自有或获授权的目标。公开可得、拥有文件或能访问调试端点，都不自动等于有权进行所有操作。

## 开始阅读

- [教程目录与阅读顺序](rea/README.md)
- [第 0 章 开始之前与反编译工程基础](rea/00-开始之前.md)

## 章节

| 章节 | 主题 |
|---|---|
| [第 0 章](rea/00-开始之前.md) | 开始之前与反编译工程基础（前置必读） |
| [第 1 章](rea/01-认识REA.md) | 认识 REA：定位、能力边界与工作方式 |
| [第 2 章](rea/02-安装注册与就绪检查.md) | 安装、注册与就绪检查 |
| [第 3 章](rea/03-文件画像.md) | 文件画像：结构、布局与构建来源 |
| [第 4 章](rea/04-函数级分析.md) | 函数级分析：主流流程、全量与证据边界 |
| [第 5 章](rea/05-运行时观察-进程捕获.md) | 运行时观察：进程捕获 |
| [第 6 章](rea/06-JS与Electron与网站目标.md) | JS/Electron 与网站目标 |
| [第 7 章](rea/07-MCP与Agent集成.md) | MCP 与 Agent 集成 |
| [第 8 章](rea/08-Evidence比较与报告.md) | Evidence、比较与报告 |
| [第 9 章](rea/09-AgentEnvironments.md) | Agent Environments |
| [第 10 章](rea/10-综合实战-WeKnoraLite桌面端.md) | 综合实战：WeKnora Lite 桌面端 |
| [第 11 章](rea/11-常见坑与检查表.md) | 常见坑与检查表 |

## 附录

- [附录 A 命令速查](rea/附录A-命令速查.md)
- [附录 B 术语表](rea/附录B-术语表.md)
- [附录 C 配套 wiki 与原始文档](rea/附录C-配套wiki与原始文档.md)
- [附录 D 模板](rea/附录D-模板.md)

## 参考

- [REA 上游仓库](https://github.com/morluto/rea)
- [REA 文档目录](https://github.com/morluto/rea/tree/main/docs)
- [WeKnora](https://github.com/Tencent/WeKnora)（MIT）
