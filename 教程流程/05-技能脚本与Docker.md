# 05 — 技能目录、脚本与 Docker（后期对齐）

本章对应 DeerFlow 里 **Skills**、**Sandbox**、**Docker 生产/开发** 等「外围能力」。脚手架阶段可只建空目录，等功能主体跑通后再填充。

## 1. `skills/` 目录

上游在仓库根有 `skills/`，用于挂载或声明可被 agent 使用的技能。你复刻时建议：

- 先新建 **`skills/README.md`** 说明：技能是文件型还是注册在 DB / 配置里。
- 对照 `../deer-flow/skills` 的文件组织方式，**先模仿目录结构**，再实现加载逻辑（通常在 harness 内）。

## 2. Sandbox

DeerFlow 可与容器化 **Sandbox** 配合执行命令或隔离环境；`Makefile` 里有 `setup-sandbox`、`docker-init` 等。复刻路径建议：

1. 先 **无 Sandbox** 跑通主链路。
2. 再读上游 `docker/` 与 harness 里 sandbox 相关调用，把镜像名、权限、挂载卷一步步对齐你环境。

若暂时拉不到镜像，文档里写清楚「跳过步骤与降级行为」，避免团队其他人踩坑。

## 3. Docker 与 `make up` / `make docker-start`

上游提供生产向 **`make up`** 与开发向 **`make docker-start`** 等（见根 `Makefile`）。你复刻时：

- 先能 **本机进程** 跑通，再迁到 compose。
- `docker-compose` / `Dockerfile` 可从 `../deer-flow/docker` **对照抄思路**，镜像名与端口改成 `sakura-flow`。

## 4. 渠道（可选）

若你需要 Lark / Slack / Telegram 等 IM，上游 `backend/pyproject.toml` 里已有相关依赖。复刻时作为 **独立 milestone**：主 Web 链路稳定后再接。

## 5. 全文回顾：你的一条龙清单

按推荐的实现顺序自评：

1. 根目录 + git + `config.example.yaml` + stub `Makefile`  
2. `backend` uv workspace + `langgraph.json` + 最小 Gateway  
3. `frontend` Next + `pnpm` + 最小页面  
4. `scripts/serve.*` 把 2、3 串起来  
5. 配置 `models` 与 `.env`，跑通一次端到端  
6. 技能、Sandbox、Docker、渠道逐项对齐  

---

恭喜你读完本教程骨架。**动手时始终以 `../deer-flow` 为参考实现**，遇到版本差异以双方 `pyproject.toml` / `package.json` 为准。若你希望我把某一章（例如「最小 Gateway」或「最小 LangGraph 图」）拆成更细的逐步清单，可以在下一轮指定章号。
