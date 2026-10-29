# REA 反编译工程与教程

公开站点：<https://sing173.github.io/rea-tutorial/>

本仓库包含两部分内容：

- **工程阶段汇报**（站点首页）：REA 反编译工程的阶段总结、能力边界与后续方向。
- **REA 新手教程**：以 REA 6.8.0 为基准的 12 章教程与 4 个附录，覆盖安装、文件画像、函数级分析、运行时观察、MCP 集成和 WeKnora Lite 综合实战。

教程强调 Evidence 模型：区分 observed、inferred 和 unknown；静态检查不证明运行行为。

## 本地预览

```bash
python -m http.server 3000
```

然后访问 <http://localhost:3000/>。站点使用 docsify，首次加载需要允许浏览器执行 JavaScript。

## 目录

```text
.
├── index.html          # docsify 入口
├── _sidebar.md         # 侧边栏
├── _navbar.md          # 顶部导航
├── _404.md             # 404 页面
├── assets/docsify/     # 本地化的 docsify 资源
└── docs/
    ├── rea-learning/   # 工程阶段汇报（首页）
    └── tutorials/      # 教程主页与 REA 教程
```

## 参考与致谢

- [REA](https://github.com/morluto/rea)：逆向工程 CLI/MCP 工具，本教程的分析对象。
- [WeKnora](https://github.com/Tencent/WeKnora)：MIT 许可的本地知识库/RAG 平台，教程综合实战的练习靶场。
