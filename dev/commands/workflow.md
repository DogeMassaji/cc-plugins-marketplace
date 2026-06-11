---
description: 串行流水线——启动 senior-developer-planner (opus) → 判断项目类型 → 按单体/前后端分离两条路径执行构建 → senior-reviewer (opus)
---

调用专职 subagent 完成完整开发流水线，每个 subagent 显式指定 model 参数：

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `senior-developer-planner` | `opus` | DEFINE → PLAN |
| 2 | — | — | 判断项目类型（读取产物） |
| 3-A | `senior-developer` | `sonnet` | BUILD（单体） |
| 3-B | `senior-developer-backend` | `sonnet` | BUILD（后端） |
| — | 产出 `api.md` | — | 接口文档 |
| 4-B | `senior-developer-frontend` | `sonnet` | BUILD（前端） |
| 5 | `senior-reviewer` | `opus` | REVIEW |

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

---

## 步骤 1——启动 senior-developer-planner（opus）

启动 `senior-developer-planner` subagent，传入用户的需求描述，显式指定 `model: "opus"`。

该 subagent 将依次执行：
- 判断是否前后端分离项目
- DEFINE：生成 `spec.md` 并提交，自动进入 PLAN
- PLAN：生成 `plan.md` + 任务列表并提交
  - 非前后端分离 → `todo.md`
  - 前后端分离 → `todo_backend.md` + `todo_frontend.md`

等待 subagent 完成。向用户展示策划结果。

---

## 步骤 2——判断项目类型

读取 planner 产出的产物，根据文件存在性判定：

| 产物 | 判定 |
|------|------|
| 仅存在 `todo.md` | → **单体项目路径（步骤 3-A）** |
| 存在 `todo_backend.md` + `todo_frontend.md` | → **前后端分离路径（步骤 3-B）** |

---

## 单体项目路径

### 步骤 3-A——启动 senior-developer（sonnet）

启动 `senior-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 plan.md + todo.md，按任务顺序逐项实现、验证并提交
- 若项目有 HTTP API → 可选运行 `api-doc-generator`
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示构建结果。跳至**步骤 5**。

---

## 前后端分离路径

### 步骤 3-B——启动 senior-developer-backend（sonnet）

启动 `senior-developer-backend` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 plan.md + todo_backend.md，按任务顺序逐项实现、验证并提交
- 全部任务完成后自动运行 `api-doc-generator`，输出 `.artifacts/<yyyymmdd>/<任务简述>/api.md`
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示后端构建结果和接口文档摘要。

### 步骤 4-B——启动 senior-developer-frontend（sonnet）

后端构建完成且 `api.md` 生成后，启动 `senior-developer-frontend` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 `api.md`（后端接口文档）→ 读取 plan.md + todo_frontend.md → 逐任务实现
- API 调用层严格按 api.md 定义实现，确保前后端接口对齐
- 任意任务失败立即停止并回报

**设计意图：** 后端先完成并产出接口文档，前端基于真实后端接口实现，杜绝前后端接口不一致。

等待 subagent 完成。向用户展示前端构建结果。

---

## 步骤 5——启动 senior-reviewer（opus）

构建阶段全部完成后，启动 `senior-reviewer` subagent，显式指定 `model: "opus"`。

该 subagent 将执行：
- REVIEW：覆盖五个维度审查所有变更（前后端分离项目含两端代码），必要时深入执行安全加固
- 生成 `review.md` 并提交，若有 Critical 问题询问用户处理方式

等待 subagent 完成。向用户展示审查报告。

---

## 跳过入口

| 用户意图 | 动作 |
|----------|------|
| "从 plan 开始" | 启动 `senior-developer-planner`，告知从 PLAN 入口 |
| "直接构建后端" | 跳过步骤 1-2，直接执行步骤 3-B |
| "直接构建前端" | 跳过步骤 1-2 和 3-B，直接执行步骤 4-B |
| "直接审查" | 跳过所有构建步骤，直接执行步骤 5 |

---

## Agent tool call 示例

```
步骤 1: Agent(description="策划需求", subagent_type="senior-developer-planner", model="opus", prompt="...")

单体路径：
  步骤 3-A: Agent(description="实现代码", subagent_type="senior-developer", model="sonnet", prompt="...")

前后端分离路径：
  步骤 3-B: Agent(description="实现后端", subagent_type="senior-developer-backend", model="sonnet", prompt="...")
  步骤 4-B: Agent(description="实现前端", subagent_type="senior-developer-frontend", model="sonnet", prompt="...")

步骤 5: Agent(description="审查代码", subagent_type="senior-reviewer", model="opus", prompt="...")
```

## 规则

1. **步骤 2 是分支点**——读取产物判定路径后，两条路径互斥，不得同时执行
2. **后端先于前端**——前后端分离路径中，步骤 3-B 完成并产出 api.md 后才能执行步骤 4-B
3. **接口文档是契约**——api.md 是前后端之间的接口契约，前端严格对照实现
4. 同路径内严格串行——每个完成后立即启动下一个，无需人工确认
5. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
6. 产物驱动——subagent 之间通过文件产物传递信息，不依赖上下文记忆
7. 任一 subagent 失败则停止流水线，不得继续
8. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录
