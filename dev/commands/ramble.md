---
description: Ramble 流水线——大型变更完整交付：DEFINE → PLAN → BUILD → REVIEW → FIX
---

Ramble 模式（大型变更 >100 LOC）。审查后修复，单轮结束。

| 步骤 | 阶段 | 方式 |
|------|------|------|
| 1 | DEFINE + PLAN | 执行 `dev:spec` → `dev:plan`，产出 SPEC.md + PLAN.md + TODO.md |
| — | 用户确认 | 展示策划结果 → 用户确认后进入构建 |
| 2-A | BUILD（单体） | 起 `dev:full-stack-developer` subagent，执行 `dev:build` |
| 2-B | BUILD（后端） | 起 `dev:backend-developer` subagent，执行 `dev:build`，产出 API.md |
| 3-B | BUILD（前端） | 起 `dev:frontend-developer` subagent，执行 `dev:build` |
| 4 | REVIEW | 执行 `dev:review`，产出 REVIEW.md + 更新 TODO.md |
| 5 | FIX | 起对应 dev subagent，执行 `dev:fix` |

**产物归档：** `.artifacts/<yyyymmdd>/<任务简述>/`

---

## 步骤 1——策划

执行 `dev:spec`：

- 需求清晰度判断 → 可选 `dev:interview-me` → `dev:spec-driven-development`
- 输出 SPEC.md，`dev:git-commit` 提交

执行 `dev:plan`：

- 读 SPEC.md → 判前后端分离 → `dev:planning-and-task-breakdown`
- 输出 PLAN.md + TODO.md（或 `TODO_BACKEND.md` + `TODO_FRONTEND.md`）
- `dev:git-commit` 提交

**展示所有策划产物给用户，等确认再进构建。**

---

## 单体路径

### 步骤 2-A——BUILD

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

Agent 执行 `dev:build`：读 PLAN.md + TODO.md → 逐任务实现 → 编译验证 → 回归测试 → 提交。

等 subagent 完成，跳**步骤 4**。

---

## 前后端分离路径

### 步骤 2-B——BUILD 后端

起 `dev:backend-developer` subagent，指定 `model: "sonnet"`。

Agent 执行 `dev:build`：读 PLAN.md + TODO_BACKEND.md → 逐任务实现 → 完成跑 `dev:api-doc-generator`。

### 步骤 3-B——BUILD 前端

起 `dev:frontend-developer` subagent，指定 `model: "sonnet"`。

Agent 执行 `dev:build`：读 API.md → PLAN.md + TODO_FRONTEND.md → 逐任务实现。

---

## 步骤 4——审查

执行 `dev:review`：

- TODO 状态检查 → `dev:quick-check` → `dev:code-review-and-quality`（五轴）
- 涉安全问题 → `dev:security-and-hardening`
- 输出 REVIEW.md + 更新 TODO.md
- `dev:git-commit` 提交

展示审查报告。无修复项则结束。

---

## 步骤 5——修复

读 REVIEW.md 修复清单，选对应 dev agent：

| 项目类型 | 修复 agent |
|----------|-----------|
| 单体 | `dev:full-stack-developer` |
| 分离 — 仅后端 | `dev:backend-developer` |
| 分离 — 仅前端 | `dev:frontend-developer` |
| 分离 — 两端都有 | 先后端，完成后前端 |

起对应 subagent，指定 `model: "sonnet"`。Agent 执行 `dev:fix`：读 REVIEW.md → 按严重级排序 → 逐项修复+验证+提交 → 单轮结束。

---

## 跳过入口

| 用户意图 | 动作 |
|----------|------|
| "从 plan 开始" | 执行 `dev:plan` |
| "直接构建后端" | 跳过步骤 1，直接 2-B |
| "直接构建前端" | 跳过 1 和 2-B，直接 3-B |
| "直接审查" | 跳过构建，直接步骤 4 |
| "直接修复" | 跳过 1-4，传 REVIEW.md 直接步骤 5 |

---

## 规则

1. **用户确认不可省略**——策划结果必须展示，收明确信号才进构建
2. **后端先于前端**——分离路径中，2-B 完成且产出 API.md 后才能 3-B
3. **接口文档是契约**——API.md 是前后端接口契约，前端严格对照
4. 同路径内严格串行——失败即停
5. 修复清单即契约——REVIEW.md `- [ ]` 是修复唯一依据，不扩大范围
6. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`
