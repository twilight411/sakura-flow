# Sakura Flow — 从 0 复刻 DeerFlow 全流程教程

本目录是你在 **`sakura-flow`** 里**手敲代码、逐步对齐 [DeerFlow 2.0](https://github.com/bytedance/deer-flow)** 的学习路线文档。目标不是「克隆一份跑起来就算了」，而是理解架构、自己搭脚手架、再按模块填满实现；细节可以略有不同，但整体方向一致。

概念、术语与常见误区（`uv`、`lock`、Gateway、checkpoint 等）见仓库根目录 **[学习笔记](../学习笔记/README.md)**。

## 你会得到什么

- 与上游对照的**阶段划分**（先搭什么、后补什么）。
- 每阶段建议**亲手输入的命令**（避免复制粘贴成习惯，但文档里仍会给出完整命令供核对）。
- 本地 **`../deer-flow`**（若你放在同级的上游仓库）作为**对照阅读**，而不是直接改上游跑通。

## 文档索引（建议按顺序读）

| 序号 | 文档 | 内容提要 |
|------|------|----------|
| 0 | [00-复刻目标与对照表.md](./00-复刻目标与对照表.md) | DeerFlow 2.0 是什么、仓库地图、你复刻时的命名与边界 |
| 1 | [01-脚手架与仓库骨架.md](./01-脚手架与仓库骨架.md) | **第一步**：空仓库、目录、`git`、根级约定（Makefile、config 占位等） |
| 2 | [02-后端Python与UV工作区.md](./02-后端Python与UV工作区.md) | `backend/`、`uv`、workspace、`harness` 包、`langgraph.json`、Gateway 思路 |
| 3 | [03-前端Next与pnpm.md](./03-前端Next与pnpm.md) | `frontend/`、pnpm、Next.js、与 LangGraph SDK 对接 |
| 4 | [04-配置Makefile与运行链路.md](./04-配置Makefile与运行链路.md) | `make config` / `make install` / `make dev` 背后在起什么服务 |
| 5 | [05-技能脚本与Docker.md](./05-技能脚本与Docker.md) | `skills/`、`docker/`、Sandbox、可选渠道 |

## 推荐目录关系

你在磁盘上建议这样放（便于一边看上游一边写自己的）：

```text
D:\flows\
  deer-flow\          # 上游参考（只读对照，不必提交到你项目里）
  sakura-flow\        # 你的复刻工程
    教程流程\         # 本教程（当前文件夹）
    backend\
    frontend\
    ...
```

## 环境与版本（与上游 README 对齐）

- **Python**：3.12+，包管理优先 **[uv](https://docs.astral.sh/uv/)**
- **Node.js**：22+，包管理 **[pnpm](https://pnpm.io/)**

版本会随上游迭代变化；以你本机 `../deer-flow` 里 `backend/pyproject.toml`、`frontend/package.json` 为准。

## 下一步

打开 **[01-脚手架与仓库骨架.md](./01-脚手架与仓库骨架.md)**，从空文件夹开始搭第一层脚手架。
