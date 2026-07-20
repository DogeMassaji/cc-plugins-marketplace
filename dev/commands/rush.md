---
description: Rush 流水线——小变更快速交付：BUILD → REVIEW → FIX 单轮
---

Rush 模式（小变更 ≤50 LOC）。审查后修复，单轮结束。

| 步骤 | 阶段 | 方式 |
|------|------|------|
| 1 | BUILD | 起 `dev:full-stack-developer` subagent，执行 `dev:build` |
| 2 | REVIEW | 起 `dev:planner` subagent，执行 `dev:review`，五轴审查 → 输出修复建议 |
| 3 | FIX | 起 `dev:full-stack-developer` subagent，执行 `dev:diagnose` |

**产物归档：** `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`

---

## 步骤 1——BUILD

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

Agent 执行 `dev:build`：理解需求 → 读相关代码 → 实现变更 → 编译检查 → 提交。不生成 spec/plan/todo。

---

## 步骤 2——REVIEW

起 `dev:planner` subagent，指定 `model: "opus"`。

Agent 执行 `dev:review`：

- 读 git diff + 用户原始需求
- `dev:code-review-and-quality` 五轴审查
- 涉安全问题 → `dev:security-and-hardening`
- 输出 REVIEW.md
- `dev:git-commit` 提交

---

## 步骤 3——FIX

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`，传 REVIEW.md 路径。

Agent 执行 `dev:diagnose`：读修复清单 → 按严重级排序 → 逐项修复+验证+提交（commit 标注 `#review`）→ 单轮结束。

---

## 规则

1. **subagent-only**——每阶段必须起对应 subagent 执行，主线程不得 inline 跑任何技能
2. 严格串行——步骤 1 → 2 → 3，不得跳过
3. 任一 subagent 失败则停流水线
4. 审查产物写入 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`
5. 单轮审查 + 单轮修复——无重新审查循环
6. 无策划产物——不生成 SPEC.md / PLAN.md / TODO.md
