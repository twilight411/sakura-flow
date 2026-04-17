# 03 — Gateway、LangGraph、Workspace、Checkpoint

本篇用**直觉类比**帮助建立心智模型；正式定义以 [LangGraph](https://langchain-ai.github.io/langgraph/) 与上游 DeerFlow 代码为准。

## 浏览器为什么需要「网关」（Gateway）？

- **浏览器**只会用 **HTTP(S)** 发请求、收响应，不会直接「执行 LangGraph 图」这种内部机制。
- 因此需要 **FastAPI 等写的 Web 服务** 作为**固定入口**：前端把请求发到这台服务，由它再去调内部运行时。人们常把这个对外层叫 **Gateway（网关）**。

**类比（非严谨定义）**：餐厅**前台**接单、传菜；**厨房**做菜。客人只接触前台，不直接进厨房。

## 「建线程、发消息、拉 SSE」是什么？

| 说法 | 直觉 |
|------|------|
| **建线程（thread）** | 后端登记一个**会话 id**，后续请求都带这个 id，表示同一段对话。 |
| **发消息** | 浏览器 POST JSON：用户输入的一句话。 |
| **拉 SSE 流** | AI 回复可能很长，服务端**分段推送**（常用 **SSE，Server-Sent Events**），前端像「直播」一样逐段显示。 |

## 「再调 LangGraph 运行时 / SDK」

- **LangGraph**：用**图**组织多步 Agent 逻辑、状态、分支的运行时。
- **SDK**：封装好的调用方式；网关在收到 HTTP 后，用 SDK 去**启动运行、传入 thread、读取流**。

**数据流（简化）**：

```text
浏览器 —HTTP/SSE—► Gateway（FastAPI）
                      │
                      ▼
              LangGraph 运行时（图 + checkpoint 等）
```

## uv 的 **workspace** 是什么？

- 这里 **不是** VS Code 的 workspace 文件。
- 指 **uv（或同类工具）的「单仓库多 Python 包」**：根目录一个 `pyproject.toml` 声明 **members**（如 `packages/harness`），子目录里还有各自的 `pyproject.toml`。
- **目的**：把 **Gateway 所在的后端** 和 **harness（agent / 工具 / checkpoint 实现）** 拆成可独立安装又同仓协作的包，一次 `uv sync` 装进同一套虚拟环境。

## `langgraph.json` 是干什么的？

给 **LangGraph CLI**（如 `langgraph dev`）用的配置，典型包括：

- **图入口**：从哪个 Python 模块的哪个函数构建哪张图。
- **依赖 / Python 版本**：如何安装当前项目。
- **checkpointer**：状态持久化用哪个工厂函数。
- **env**：加载哪个 `.env`。

没有它，CLI 就不知道加载哪张图、checkpoint 用哪套实现。

## **Checkpoint** 是什么？

- 在 LangGraph 里：**把图运行过程中的状态持久化**，并能在之后**恢复**（多轮对话、中断续跑、人机协作暂停等）。
- **不是**业务流程里的「审批检查点」那个中文习惯用法。
- 实现上可对接 SQLite、Postgres 等；`langgraph.json` 里的 `checkpointer` 指向**如何创建**具体后端。

## 「鉴权、路由、衔接」

- **鉴权**：确认请求者身份与权限。
- **路由**：不同 URL 对应不同处理（聊天、上传等）。
- **衔接**：把 **HTTP JSON** 转成 **LangGraph / SDK 需要的调用**，把 **运行结果** 再转成 **浏览器能消费的 SSE / JSON**。

下一篇：**[04-文件格式与学习边界.md](./04-文件格式与学习边界.md)**
