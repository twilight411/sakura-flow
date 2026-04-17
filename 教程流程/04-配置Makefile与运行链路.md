# 04 — 配置、Makefile 与运行链路

本章解释 DeerFlow **从配置到三条进程**的关系，便于你在 `sakura-flow` 里复刻同等体验：`make config` → `make install` → `make dev`。

## 1. 配置文件职责

| 文件 | 作用 |
|------|------|
| `config.example.yaml` | 模板；不含敏感信息或只有占位符 |
| `config.yaml` | 本地实际配置；**不要提交密钥**（应已被 `.gitignore` 忽略） |
| `.env` | 环境变量；网关、第三方、LangSmith 等 |

上游用 `make config` 调 `scripts/configure.py` 从示例生成本地文件。你复刻时可以：

- **第一期**：手写 `cp config.example.yaml config.yaml`。
- **第二期**：把 `configure.py` 的逻辑简化后搬过来。

## 2. `make install` 在做什么

对照 `../deer-flow/Makefile`：

1. **`cd backend && uv sync`**：安装 Python 及 workspace。
2. **`cd frontend && pnpm install`**：安装前端依赖。

你在自己的 `Makefile` 里保持相同顺序即可；包管理器已定时不必再加一层。

## 3. `make dev` / `serve.sh` 背后的服务

阅读 `../deer-flow/scripts/serve.sh`（不必一次读懂），其核心是同时拉起类似下面几类进程（具体以脚本为准）：

1. **LangGraph**：`langgraph dev` 或等价，读取 `backend/langgraph.json`。
2. **Gateway**：`uvicorn` 指向 FastAPI 应用（如 `app.gateway.app:app`）。
3. **前端**：`pnpm run dev`（Next dev）。
4. **Nginx**（可选）：用 `docker/nginx/nginx.local.conf` 做反代，把浏览器 **统一入口** 指到一个端口。

你在复刻时可以先 **少一点**：不用 Nginx，直接前端一个端口、网关一个端口、LangGraph 一个端口，确认链路通了再上 Nginx。

## 4. Windows 注意点

上游 `serve.sh` 是 bash；Windows 下通过 `scripts/run-with-git-bash.cmd` 调用。你若纯 PowerShell 开发，可以：

- 写 **`serve.ps1`** 做等价事；或
- 用 **WSL2** / **Git Bash** 跑同名 sh。

关键是：**进程启动顺序与 kill 逻辑**要清晰，避免端口占用。

## 5. 环境变量：网关如何找到 `config.yaml`

`serve.sh` 里会检查 `DEER_FLOW_CONFIG_PATH` 或 `backend/config.yaml` 或 **仓库根** `config.yaml`。你复刻时：

- 要么沿用同一策略（改名 `SAKURA_FLOW_CONFIG_PATH` 也可以，但要全仓库统一）。
- 要么固定只读 `./config.yaml`。

## 6. 本章验收

- [ ] 你能用自己的方式同时起 **至少两个**：Gateway + 前端，或 LangGraph + Gateway。
- [ ] 你清楚 **`config.yaml` 中 `models` 至少一条**时主流程才能走模型调用。

下一篇：**[05-技能脚本与Docker.md](./05-技能脚本与Docker.md)**。
