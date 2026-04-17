# 03 — 前端 Next.js 与 pnpm

本章目标：在 **`frontend/`** 创建与 DeerFlow 同栈的前端：**Next.js（App Router）+ TypeScript + pnpm**，并为后续对接 LangGraph / 网关预留脚本名。

## 1. 版本与包管理器

对照 `../deer-flow/frontend/package.json`：

- **pnpm**（上游在 `packageManager` 字段里锁了版本，你可照抄思路用 `corepack enable` + `packageManager` 锁 pnpm）。
- **Next.js、React 主版本**以当前上游为准；你复刻时**差一个小版本通常无碍**，但尽量别跨越 major。

## 2. 创建项目（推荐手敲 `create-next-app`）

在仓库根目录：

```powershell
cd D:\flows\sakura-flow
mkdir frontend
cd frontend
pnpm create next-app@latest .
```

在交互式提示中，按 DeerFlow 风格选择（以当前 create-next-app 选项为准）：

- TypeScript：**是**
- ESLint：**是**
- Tailwind CSS：**是**（上游用 Tailwind v4 + PostCSS）
- `src/` 目录：按你习惯；上游未强制 `src/`，你可对齐 `../deer-flow/frontend` 结构再迁移。
- App Router：**是**
- Turbopack：`pnpm dev` 时上游用 `--turbo`，可在脚本里对齐。

非交互方式因 CLI 版本而异，若你更希望一条命令搞定，可先读 `pnpm create next-app@latest --help` 再决定。

## 3. 对齐 `package.json` 脚本

上游核心脚本大致为：

- **`dev`**：`next dev --turbo`
- **`build`** / **`start`**：生产构建与启动
- **`check`**：`eslint` + `tsc --noEmit`

你可以在最小模板生成后，打开 `package.json`，把这些 **scripts 名称**先对齐，依赖包（Radix、langgraph-sdk、ai 等）**按需逐步添加**，避免第一次 `pnpm install` 就装上百个包却用不到。

## 4. 环境变量与网关地址

DeerFlow 前端会通过环境变量指向网关 / LangGraph HTTP API。你复刻时建议：

1. 在 `frontend` 放 `.env.example`（仅键名，不给秘密）。
2. 在 README 写清楚：开发时网关监听端口（上游常见为 **8001** 一类，以 `../deer-flow` 实际为准）。

具体变量名请对照 `../deer-flow/frontend` 里对 `@t3-oss/env-nextjs` 或 `process.env` 的用法。

## 5. 与后端联调的顺序建议

1. 页面能打开、无编译错误。
2. 配置代理或 CORS，使浏览器能访问本机网关（视你 `serve` 脚本是 **同一源经 Nginx** 还是**直连端口**）。
3. 再接入 **thread / run / SSE**（对齐 `@langchain/langgraph-sdk` 用法）。

## 6. 本章验收

- [ ] `pnpm install` 成功。
- [ ] `pnpm run dev` 能启动开发服务器。
- [ ] 你已打开上游 `frontend` 的 `package.json` 与 `next.config.*`，知道 **下一批要补的依赖类别**（而不是一次性全装）。

下一篇：**[04-配置Makefile与运行链路.md](./04-配置Makefile与运行链路.md)**。
