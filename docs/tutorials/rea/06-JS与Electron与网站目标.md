## 第 6 章 JS/Electron 与网站目标

> 状态：初稿，已按章节拆分。本轮重点校对第 1–2 章；本章后续按 raw/wiki 逐条补证。

### 1 静态分析

适用于应用目录、ASAR、页面脚本。常见入口：

```bash
rea analyze-javascript-application /absolute/path/to/app --json
```

关注：

- 模块、导入、source map
- 路由、IPC、原生插件关系
- 静态图与文件清单

静态分析不执行应用。

**名称解析**

| 名称 | 含义 |
|---|---|
| ASAR | Electron 的应用归档格式。 |
| Electron | Chromium + Node.js 桌面运行时；见第 0 章。 |
| source map | 把生成代码位置映射回原始源码的文件。 |

### 2 浏览器/网站观察

如果需要页面结构、脚本、网络观察：

- `capture-browser-scenario`：有界浏览器场景
- `inspect-web-page` / `inspect-web-network-capture`：页面/抓包检查
- `capture-web-screenshot`：视口截图

这些操作可能访问网络或改变页面状态；只对隔离测试实例和测试数据使用。

**名称解析**

| 名称 | 含义 |
|---|---|
| Playwright | 浏览器自动化库，用于执行有界浏览器场景。 |
| CDP / HAR | 见第 1 章名称解析。 |

### 3 选择原则

- 能用 API/静态文件回答，就不要上浏览器。
- 需要运行时行为时，先设计有界场景和停止条件。
- 不要把真实用户数据、凭据或生产流量带进 capture。

### 4 本章练习

1. 选一个本地 JS/Electron 应用，跑 `analyze-javascript-application`。
2. 记录 5 个模块/文件、2 个 unknown。
3. 说明为什么静态分析不能证明运行时行为。

---

