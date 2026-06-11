---
description: 串行流水线——启动 senior-developer-planner (opus) → senior-developer (sonnet) → senior-reviewer (opus)，阶段之间设有人工确认检查点
---

调用三个专职 subagent 完成完整开发流水线，每个 subagent 显式指定 model 参数：

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `senior-developer-planner` | `opus` | DEFINE → PLAN |
| 2 | `senior-developer` | `sonnet` | BUILD |
| 3 | `senior-reviewer` | `opus` | REVIEW |

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

## 步骤 1——启动 senior-developer-planner（opus）

启动 `senior-developer-planner` subagent，传入用户的需求描述，显式指定 `model: "opus"`。

该 subagent 将依次执行：
- DEFINE：生成 `spec.md` 并提交，自动进入 PLAN
- PLAN：生成 `plan.md` 和 `todo.md` 并提交

等待 subagent 完成。读取产物，向用户展示策划结果。

## 步骤 2——启动 senior-developer（sonnet）

`senior-developer-planner` 完成后，立即启动 `senior-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 plan.md + todo.md，按任务顺序逐项实现、验证并提交
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示构建结果。

## 步骤 3——启动 senior-reviewer（opus）

`senior-developer` 完成后，立即启动 `senior-reviewer` subagent，显式指定 `model: "opus"`。

该 subagent 将执行：
- REVIEW：覆盖五个维度审查所有变更，必要时深入执行安全加固
- 生成 `review.md` 并提交，若有 Critical 问题询问用户处理方式

等待 subagent 完成。向用户展示审查报告。

## 跳过入口

- 用户说"从 plan 开始"：启动 `senior-developer-planner` 时告知从 PLAN 阶段入口开始
- 用户说"直接构建"：跳过步骤 1，启动 `senior-developer` 时告知从 BUILD 阶段入口开始
- 用户说"直接审查"：跳过步骤 1 和 2，直接启动 `senior-reviewer`

## 规则

1. 三个 subagent 严格串行——每个完成后立即启动下一个，无需人工确认
2. **model 参数必须显式指定**
3. 产物驱动——subagent 之间通过文件产物传递信息，不依赖上下文记忆
4. 任一 subagent 失败则停止流水线，不得继续
5. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录
