---
description: Rush 流水线——小变更快速交付：全栈工程师 (sonnet) → 初级审查者 (sonnet) → 全栈工程师修复 (sonnet)，BUILD → REVIEW → FIX 单轮
---

Rush 模式（小变更 ≤50 LOC）。起两个 subagent 串行，审查后修复，单轮结束。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:full-stack-developer` | `sonnet` | BUILD — 理解需求 → 实现 → 编译检查 → 提交 |
| 2 | `dev:junior-reviewer` | `sonnet` | REVIEW — 五轴审查 → 输出修复建议 |
| 3 | `dev:full-stack-developer` | `sonnet` | FIX — 按修复清单逐项修复 |

**产物归档：** 审查报告存于 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`。

---

## 步骤 1——起 full-stack-developer

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

执行：
- BUILD：理解需求 → 读相关代码 → 实现变更
- 编译/类型检查
- 提交（中文 Conventional Commit）
- 不生成 spec/plan/todo
- **跳过测试生成**——不调 `dev:backend-test-generator`

等 subagent 完成，展示构建结果。

---

## 步骤 2——起 junior-reviewer

构建完成，起 `dev:junior-reviewer` subagent，指定 `model: "sonnet"`，传：
- git diff 范围（最近提交）
- 用户原始需求

执行：
1. **REVIEW** — 五维度审查，必要时安全加固
2. **生成修复建议** — `- [ ]` 在 `REVIEW.md` 列出，标注路径、行号、严重级别
3. 输出 `REVIEW.md` 并提交

等 subagent 完成。展示审查报告。

---

## 步骤 3——起 full-stack-developer 修复

审查完成，再次起步骤 1 的 `dev:full-stack-developer` subagent，传：
- `REVIEW.md` 路径

执行：
- FIX：读 REVIEW.md 修复清单（`- [ ]` 项）
- 按严重级排序（Critical → Important → Suggestion）
- 逐项修复、验证、提交（commit 标注 `#review`）
- 不修改 REVIEW.md

等 subagent 完成。展示修复结果，流水线结束。

**单轮修复，无重新审查。** 修复完成即结束，由用户自行验证。

---

## 规则

1. **严格串行**——步骤 1 → 2 → 3，不得跳过
2. **model 必须显式指定**——不依赖 agent frontmatter model 字段
3. 任一 subagent 失败则停流水线
4. 审查产物写入 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`
5. **单轮审查 + 单轮修复**——无重新审查循环
6. **无策划产物**——不生成 SPEC.md / PLAN.md / TODO.md
