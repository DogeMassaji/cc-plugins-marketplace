---
name: build
description: BUILD 阶段——按 PLAN.md + TODO.md 逐任务实现代码。跑 incremental-implementation，编译检查，回归验证。不生成新测试（由 dev:test 处理）。触发：build、实现、构建、implement。
---

# Build — 代码实现

## 概述

读 PLAN.md + TODO.md，逐任务实现、验证、提交。严格按计划驱动——不擅自设计方案变更。不生成新测试（由 `dev:test` 独立处理）。

## 何时用

- PLAN.md + TODO.md（或 TODO_BACKEND.md / TODO_FRONTEND.md）已存在
- 用户说"build"、"实现"、"构建"、"implement"

## 流程

### 1. 读计划

- 读 `PLAN.md` 理解方案和架构决策
- 读 `TODO.md`（或 `TODO_BACKEND.md` / `TODO_FRONTEND.md`）取任务列表
- 若存在 `API.md`（分离模式前端），先理解所有接口

### 2. 逐任务实现

每个任务严格按序执行：

1. **读上下文**——加载相关代码和依赖
2. **实现**——跑 `dev:incremental-implementation`，按验收标准实现
3. **编译检查**——编译/类型检查通过
4. **回归验证**——跑已有测试确认无回归（**不生成新测试**）
5. **提交**——跑 `dev:git-commit`

提交 message 规范：

| 项目类型 | message |
|---------|---------|
| 单体 | `feat: <任务简述>` |
| 后端 | `feat(backend): <任务简述>` |
| 前端 | `feat(frontend): <任务简述>` |

任务失败即停，不跳过。

### 3. 接口文档（后端有 HTTP API 时）

全部任务完成后：

- 检测项目是否有 HTTP API
- 有则跑 `dev:api-doc-generator`，输出至 `.artifacts/<yyyymmdd>/<任务简述>/API.md`
- 提交，message：`docs(backend): add API docs for <任务简述>`

### 4. 汇报

- 已完成任务列表
- 提交记录摘要
- 失败任务及原因

## ARCHIVE

代码直接提交仓库。额外产物：

```
.artifacts/<yyyymmdd>/<任务简述>/API.md   ← 仅后端
```

## 规则

1. **计划驱动**——严格按 PLAN.md + TODO.md 实现
2. **失败即停**——任务失败停止，不跳过
3. **逐任务提交**——每个任务独立提交
4. **不生成测试**——测试生成由 `dev:test` 独立处理
5. **不越界**——不分析需求、拆解任务、设计方案。发现问题反馈，不擅自修
6. **接口文档必产**——后端有 HTTP API 则跑 `dev:api-doc-generator`

## 验证

- [ ] 所有任务编译通过
- [ ] 已有测试通过（无回归）
- [ ] 接口文档已生成（如适用）
- [ ] 提交记录完整
