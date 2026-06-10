---
description: 串行流水线——spec → plan → build → review，各自在独立 subagent session 中运行，阶段之间设有人工确认检查点
---

调用完整串行流水线：`spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` → `code-review-and-quality`。

这是**串行编排器**。与 `/ship`（并行）不同，`/workflow` 按顺序运行四个阶段，每个阶段在全新的 subagent 上下文中执行。每个阶段产出一个文件产物，供下一阶段消费。主 Agent 读取每个产物并在启动下一个 subagent 前暂停，等待用户确认。

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

## 阶段 A——Spec

启动 subagent 执行 `spec-driven-development`：

- 就目标、目标用户、核心功能、技术栈、约束、边界提出澄清问题
- 生成覆盖全部六大领域的结构化 spec（目标、命令、项目结构、代码风格、测试策略、边界）
- 保存为 `.artifacts/<yyyymmdd>/<任务简述>/SPEC.md`

等待 subagent 完成。读取 SPEC.md。向用户展示摘要并请求确认。

**在用户确认 spec 之前，不要进入阶段 B。**

## 阶段 B——Plan

用户确认 spec 后，启动新的 subagent 执行 `planning-and-task-breakdown`：

- 读取 SPEC.md 和相关代码库部分（只读）
- 识别组件间的依赖图
- 垂直切片（每个任务一条完整路径，而非水平分层）
- 编写任务，附带验收标准和验证步骤
- 在阶段之间添加检查点
- 将计划保存到 `.artifacts/<yyyymmdd>/<任务简述>/plan.md`，任务列表保存到 `.artifacts/<yyyymmdd>/<任务简述>/todo.md`

等待 subagent 完成。读取 plan.md 和 todo.md。向用户展示任务拆分并请求确认。

**在用户确认计划之前，不要进入阶段 C。**

## 阶段 C——Build

用户确认计划后，启动新的 subagent 执行 `incremental-implementation`：

- 从 `.artifacts/<yyyymmdd>/<任务简述>/todo.md` 中选取第一个待办任务
- 对每个任务：
  1. 阅读验收标准
  2. 加载相关上下文（已有代码、模式、类型）
  3. 实现代码
  4. 运行构建验证编译
  5. 用描述性 message 提交
  6. 标记任务完成，进入下一个
- 任何任务失败时停止并回报

等待 subagent 完成。报告结果：已完成的任务、提交记录、任何失败。

**在用户确认构建结果之前，不要进入阶段 D。**

## 阶段 D——Review

用户确认构建结果后，启动新的 subagent 执行 `code-review-and-quality`：

- 审查全部已完成任务的变更（staged 或最近提交）
- 覆盖五个维度：正确性、可读性、架构、安全性、性能
- 将发现分为 Critical、Important、Suggestion 三级
- 输出结构化审查报告到 `.artifacts/<yyyymmdd>/<任务简述>/review.md`

等待 subagent 完成。向用户展示审查报告。如有 Critical 问题，询问用户是否立即修复或记录为后续任务。

## 规则

1. 每个阶段在独立的 subagent session 中运行——全新上下文，无残留。
2. 阶段严格串行——每个阶段依赖前一阶段的文件产物。
3. 主 Agent 读取产物并在阶段之间与用户确认。
4. 如果用户拒绝某阶段的输出，带着反馈重新运行该阶段后再继续。
5. 任何阶段失败则停止流水线——不得继续。
6. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录。

## 跳过

如果对应的 `.artifacts/<yyyymmdd>/<任务简述>/SPEC.md` 已存在且用户说"从 spec 继续"或"从 plan 继续"，从阶段 B 开始。
如果对应的 `.artifacts/<yyyymmdd>/<任务简述>/plan.md` 已存在且用户说"直接构建"，从阶段 C 开始。
如果所有任务已构建完成且用户说"直接审查"，从阶段 D 开始。
