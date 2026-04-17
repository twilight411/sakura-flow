# Sakura Flow 项目学习笔记

本目录把搭建与阅读 DeerFlow 系项目时容易卡住的**概念**和**命令**整理成笔记，和 **[教程流程](../教程流程/README.md)** 互补：教程偏「步骤」，这里偏「为什么」。

## 推荐阅读顺序

| 顺序 | 文档 | 适合场景 |
|------|------|----------|
| 1 | [01-命令行与工具链.md](./01-命令行与工具链.md) | Windows / PowerShell、`uv` 装不上、路径问题 |
| 2 | [02-依赖锁定与项目文件.md](./02-依赖锁定与项目文件.md) | `toml`、`lock`、`sync`、`.venv`、为什么一行命令出一堆文件 |
| 3 | [03-Gateway与LangGraph概念.md](./03-Gateway与LangGraph概念.md) | 网关、SSE、LangGraph、`workspace`、`checkpoint`、`langgraph.json` |
| 4 | [04-文件格式与学习边界.md](./04-文件格式与学习边界.md) | 文本与二进制、PDF、和《龙书》等教材的分工 |
| 5 | [05-根后端与harness-pyproject详解.md](./05-根后端与harness-pyproject详解.md) | 根 `pyproject` vs `packages/harness`、TOML 块、依赖分工表 |

## 笔记维护约定（轻量）

- **一条笔记一个主题**：新增内容优先归入上面对应文件，必要时再拆新文件并在本 README 登记。
- **区分「事实」与「类比」**：类比（如餐厅前台）会标注，避免当严谨定义背。
- **和仓库状态对齐**：涉及版本、命令时注明「以本机 `uv --version` / 文档日期为准」。

## 速查：术语表

详见各篇文末或文中加粗术语；最常用几个：

- **元数据**：描述项目/包本身的信息（名称、版本），不是业务数据。
- **sync（uv）**：把虚拟环境与 `pyproject` / `lock` **对齐**，与 TCP 的 SYN 无关。
- **checkpoint（LangGraph）**：运行时状态的持久化与恢复，不是审批「检查点」。
- **根后端 / harness**：根 `pyproject` 偏 Gateway 与部署依赖；`packages/harness` 偏 Agent 与 LangGraph 重依赖，见第 5 篇。
