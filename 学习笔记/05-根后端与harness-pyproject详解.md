# 05 — 「根后端」与 `packages/harness` 有什么区别？

以 **DeerFlow**（`deer-flow/backend`）为例：仓库里**不止一个** `pyproject.toml`，职责不同。

## 1. 一句话区别

| | **根后端** `backend/pyproject.toml` | **Harness 子包** `backend/packages/harness/pyproject.toml` |
|---|-------------------------------------|------------------------------------------------------------|
| **`[project].name`** | `deer-flow` | `deerflow-harness` |
| **角色** | 对外 **Gateway（FastAPI）**、进程入口、与 IM 等渠道相关的**薄粘合层**依赖 | **Agent 框架本体**：LangGraph 图、工具链、沙箱、各类模型适配、checkpoint 等 **重依赖** |
| **Python 包名（import）** | 根项目通常对应 **`app`** 等（见仓库 `backend/app`） | wheel 里声明的是 **`deerflow`** 包（见 harness 里 `[tool.hatch.build.targets.wheel] packages = ["deerflow"]`） |
| **谁依赖谁** | **根项目依赖** `deerflow-harness`（通过 `[tool.uv.sources]` workspace 链接） | **不**依赖根 `deer-flow` 包（避免循环）；被根项目当作库使用 |

**类比（非严谨）**：根后端像 **门店前台 + 接线**；harness 像 **后厨配方与生产线**——客人点单走前台，真正「怎么做菜」在 harness。

---

## 2. 为什么拆成两个 PyPI 风格项目（workspace）？

1. **依赖分层**：Gateway 需要 **FastAPI、uvicorn、SSE、各 IM SDK**；Agent 核心需要 **langchain、langgraph、sandbox、搜索/爬虫** 等。拆开以后 **清晰**，也避免「一个 pyproject 里几百条依赖」难以维护。  
2. **可复用**：`deerflow-harness` 理论上可被**别的服务**当作库引用（若发布）；根项目只负责部署形态。  
3. **构建方式**：harness 用 **hatchling** 明确打出 `deerflow` 包；根项目用 **uv workspace** 把本地 harness **editable** 链进来。

---

## 3. 根后端 `backend/pyproject.toml` —— TOML 结构与依赖表

### 3.1 语法块说明

- **`[project]`**：PEP 621 风格的项目元数据 + **运行依赖** `dependencies`。  
- **`[dependency-groups]`**：开发用依赖分组（如 `dev`），**不**等同于「生产必装」，具体安装方式见 uv 文档。  
- **`[tool.uv.workspace]`**：声明 workspace **成员目录**（此处 `packages/harness`）。  
- **`[tool.uv.sources]`**：告诉 uv：`deerflow-harness` 这个名字从 **workspace 本地成员**解析，而不是默认只去 PyPI 找。

### 3.2 `dependencies` 各包在 DeerFlow 里**大概**干什么

（精确到函数级以源码为准；此处是**角色**说明。）

| 依赖 | 角色 |
|------|------|
| `deerflow-harness` | 本仓 harness 包；**Agent / 图 / 工具 / checkpoint** 的主体实现。 |
| `fastapi` | **HTTP API 框架**（Gateway）。 |
| `httpx` | **HTTP 客户端**（后端主动请求外部服务）。 |
| `python-multipart` | **表单与文件上传**解析（与 FastAPI 上传相关）。 |
| `sse-starlette` | **SSE 流式响应**（推给前端的 token/事件流）。 |
| `uvicorn[standard]` | **ASGI 服务器**，运行 FastAPI。 |
| `lark-oapi` | **飞书**开放平台接口（若启用飞书渠道）。 |
| `slack-sdk` | **Slack** 集成。 |
| `python-telegram-bot` | **Telegram Bot** 集成。 |
| `langgraph-sdk` | 与 **LangGraph 服务**交互的客户端能力（与前端/网关侧衔接）。 |
| `markdown-to-mrkdwn` | **Markdown → Slack mrkdwn** 等消息格式转换。 |

---

## 4. Harness `packages/harness/pyproject.toml` —— 结构与依赖分组理解

### 4.1 特有块

- **`[build-system]`** + **`hatchling`**：定义如何 **构建 wheel**（打包 `deerflow` 包）。  
- **`[tool.hatch.build.targets.wheel] packages = ["deerflow"]`**：打 wheel 时包含 **`deerflow` 这个 Python 包目录**（物理路径在 `packages/harness/deerflow/` 下，以仓库为准）。

### 4.2 `dependencies` 按类别理解（不必死记每一条）

| 类别 | 代表依赖 | 用途直觉 |
|------|----------|----------|
| **Agent / 协议** | `agent-client-protocol` | 与 agent 协议相关集成。 |
| **沙箱与编排** | `agent-sandbox`, `kubernetes` | 隔离执行环境；K8s 场景。 |
| **LangChain / LangGraph 栈** | `langchain`, `langchain-*`, `langgraph`, `langgraph-api`, `langgraph-cli`, `langgraph-runtime-inmem`, `langgraph-checkpoint-sqlite`, `langgraph-sdk` | **模型接入、图运行时、CLI、内存运行时、SQLite checkpoint** 等。 |
| **工具与搜索** | `tavily-python`, `firecrawl-py`, `ddgs`, `readabilipy` 等 | 联网搜索、爬取、可读性提取。 |
| **文档与数据** | `markitdown`, `markdownify`, `duckdb` 等 | 文档转换、结构化/分析。 |
| **基础** | `pydantic`, `pyyaml`, `httpx`, `dotenv`, `tiktoken` | 数据模型、配置、HTTP、环境变量、分词计数。 |

版本里的 **上界**（如 `langgraph>=...,<...`）是上游为 **兼容性** 刻意收窄，升级时需跑测试。

---

## 5. 和 `langgraph.json` 的关系（补一句）

`langgraph.json` 通常在 **`backend/`** 目录，指向 **图入口**（如 `deerflow.agents:...`）和 **checkpointer**——这些 **实现代码在 harness 的 `deerflow` 包内**；根后端负责 **起网关进程、加载配置、对外暴露 HTTP**。二者分工：**配置在 backend 根，`deerflow` 包内是图与运行时逻辑**。

---

## 6. 复刻 sakura-flow 时怎么对应？

- 根 `pyproject.toml`：你的项目名、`app`、FastAPI、uvicorn、（按需）SSE、渠道 SDK。  
- `packages/harness`（或你命名的子包）：**sakura 包名**、`[tool.uv.sources]` 指向 workspace、`langgraph.json` 里 **graphs** 指向你的模块路径。  

**命名不要和上游 `deerflow` 混用**，避免 import 与发布混淆。

---

上一篇：**[04-文件格式与学习边界.md](./04-文件格式与学习边界.md)** · 回到索引：**[README.md](./README.md)**
