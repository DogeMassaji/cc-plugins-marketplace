---
description: 串行流水线——启动 senior-developer subagent 完成 spec → plan → build，再启动 senior-reviewer subagent 完成 review，阶段之间设有人工确认检查点
---

调用两个专职 subagent 完成完整开发流水线：

1. **`senior-developer`** — 负责 DEFINE → PLAN → BUILD（spec、plan、build 三阶段）
2. **`senior-reviewer`** — 负责 REVIEW（代码审查与安全加固）

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

## 步骤 1——启动 senior-developer

启动 `senior-developer` subagent，传入用户的需求描述。

该 subagent 将依次执行：
- DEFINE：生成 `SPEC.md`，暂停等待用户确认
- PLAN：生成 `plan.md` 和 `todo.md`，暂停等待用户确认
- BUILD：按 `todo.md` 逐任务实现并提交

等待 subagent 完成。读取产物，向用户展示构建结果，随即启动 senior-reviewer。

## 步骤 2——启动 senior-reviewer

`senior-developer` 完成后，立即启动 `senior-reviewer` subagent。

该 subagent 将执行：
- REVIEW：覆盖五个维度审查所有变更，必要时深入执行安全加固
- 生成 `review.md`，若有 Critical 问题询问用户处理方式

等待 subagent 完成。向用户展示审查报告。

## 跳过入口

- 用户说"从 plan 开始"：启动 `senior-developer` 时告知从 PLAN 阶段入口开始
- 用户说"直接构建"：启动 `senior-developer` 时告知从 BUILD 阶段入口开始
- 用户说"直接审查"：跳过步骤 1，直接启动 `senior-reviewer`

## 规则

1. 两个 subagent 严格串行——`senior-developer` 完成后立即启动 `senior-reviewer`，无需人工确认
2. 产物驱动——subagent 之间通过文件产物传递信息，不依赖上下文记忆
3. 任一 subagent 失败则停止流水线，不得继续
4. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录
