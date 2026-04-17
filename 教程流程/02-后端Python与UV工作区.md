# 02 — 后端 Python 与 uv 工作区

本章目标：在 **`backend/`** 内建成与 DeerFlow 同构的形态：

- 根后端包：FastAPI Gateway、应用入口。
- **Workspace 成员**：例如 `packages/harness`，承载 LangGraph agent、工具与 checkpoint 逻辑。
- **`langgraph.json`**：告诉 LangGraph CLI 图入口、依赖、checkpointer 等。

以下命令建议在 **`D:\flows\sakura-flow\backend`** 下执行；**请尽量手敲**，复制时也要逐行核对。

## 1. Python 版本锁定

DeerFlow 使用 **Python ≥3.12**。建议在 `backend` 放 `.python-version`：

```text
3.12
```

（若你用 `pyenv` / `uv python pin` 等，按你工具链来。）

## 2. 用 uv 初始化后端项目

进入 `backend`：

```powershell
cd D:\flows\sakura-flow\backend
```

### 2.1 创建 `pyproject.toml`（方式 A：uv 引导）

若已安装 uv，可参考：

```powershell
uv init --name sakura-flow --package
```

然后根据需要把 **workspace** 配进去（见下节）。若 `uv init` 选项与你本机 uv 版本不一致，以 `uv init --help` 为准，或采用 **方式 B：手写 `pyproject.toml`**。

### 2.2 方式 B：对齐 DeerFlow 的 workspace 形态

打开对照文件：

- `../deer-flow/backend/pyproject.toml`
- `../deer-flow/backend/packages/harness/pyproject.toml`

你要在自己的 `pyproject.toml` 里体现：

1. `[project]`：`name`、`version`、`requires-python = ">=3.12"`。
2. **依赖**：上游列了 `deerflow-harness`、`fastapi`、`uvicorn`、`langgraph-sdk` 等；你复刻时可先 **少装几个能 import**，再按模块实现逐步增加，避免一上来 `uv sync` 过慢或失败。
3. `[tool.uv.workspace]`：`members = ["packages/harness"]`（路径与你目录一致）。
4. `[tool.uv.sources]`：把工作区里的 harness 包映射成一个可安装的本地名，例如 `sakura-harness = { workspace = true }`，并在 `[project].dependencies` 里引用 `sakura-harness`。

> 命名建议：不要用 `deerflow-harness`，改用你自己的包名，避免与上游 PyPI / 本地包混淆。

## 3. 目录骨架

```text
backend/
  app/
    gateway/           # FastAPI 路由、依赖注入（后期逐步写）
    __init__.py
  packages/
    harness/
      pyproject.toml   # 子项目：你的 agent 框架包
      sakura/          # 或 deerflow 同级命名空间，但不要与上游混名
        agents/
        ...
  pyproject.toml
  langgraph.json
  .python-version
```

`packages/harness` 下 **至少**要有可被 `hatchling` 或 `uv` 识别的包结构（`pyproject.toml` 里 `[build-system]`、`[tool.hatch.build.targets.wheel]` 等照上游改包名）。

## 4. `langgraph.json`

对照 `../deer-flow/backend/langgraph.json`，你自己要有：

- **`python_version`**：如 `"3.12"`。
- **`dependencies`**：通常 `["."]` 指向当前 backend 可编辑安装。
- **`env`**：如 `".env"`。
- **`graphs`**：入口指向 **你的**工厂函数，例如 `"lead_agent": "sakura.agents:make_lead_agent"`（名称自定，但要和 Python 模块一致）。
- **`checkpointer`**：指到你在 harness 里实现的 `make_checkpointer`（可先 stub）。

## 5. Gateway（FastAPI）最小思路

上游入口在 `app/gateway` 一组 router + `uvicorn app.gateway.app:app`。你复刻时建议顺序：

1. 先写一个 **健康检查** 路由 `/health`。
2. 再对接 LangGraph SDK / threads / runs（对照上游 `routers/` 文件名分文件实现）。
3. SSE 与 run 管理可放中后期。

## 6. 安装与检查

在 `backend`：

```powershell
uv sync
```

（若 workspace 尚未配好会报错，按报错补齐 `packages/harness`。）

可选：

```powershell
uv run python -c "import fastapi; print('ok')"
```

## 7. 本章验收

- [ ] `uv sync` 成功，虚拟环境可激活。
- [ ] `langgraph.json` 存在且 JSON 有效。
- [ ] `packages/harness` 可被 workspace 安装为一个包。

下一篇：**[03-前端Next与pnpm.md](./03-前端Next与pnpm.md)**。
