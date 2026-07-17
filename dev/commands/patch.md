---
description: Patch 流水线——从问题描述到修复验证：DIAGNOSE → CONFIRM → FIX → REVIEW 单轮
---

Patch — 精准打补丁。从用户描述 bug 出发，诊断 → 修复 → 审查。

| 步骤 | 阶段 | 方式 |
|------|------|------|
| 1 | DIAGNOSE | 起 `dev:full-stack-developer` subagent，定位根因 → 输出诊断报告 |
| — | 用户确认 | 展示诊断结果 → 用户确认修复方案 |
| 2 | FIX | 起 `dev:full-stack-developer` subagent，执行 `dev:fix` |
| 3 | REVIEW | 执行 `dev:review`，验证修复正确性 → 检查回归 |

**产物归档：** `.artifacts/<yyyymmdd>/<任务简述>/`

---

## 前置条件

用户提供 bug 描述或异常现象，无需 REVIEW.md 或 TODO.md。

---

## 步骤 1——DIAGNOSE

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

执行：
- 读相关代码，理解上下文
- 复现/定位根因
- 输出 `DIAGNOSE.md` 并提交：
  - 问题现象（复现步骤）
  - 根因分析（路径、行号、调用链）
  - 修复方案（改什么、为什么）
  - 影响范围评估
- 不修改代码

等 subagent 完成。展示诊断结果，**等用户确认再修复**。

---

## 步骤 2——FIX

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`，传 DIAGNOSE.md 路径。

Agent 执行 `dev:fix`：读 DIAGNOSE.md 修复方案 → 逐项修复 → 编译检查 → 提交（commit 标注 `#fix`）。

---

## 步骤 3——REVIEW

执行 `dev:review`：

- 读 git diff + DIAGNOSE.md + 原始 bug 描述
- 验证修复正确性 + 回归检查
- 输出 REVIEW.md
- `dev:git-commit` 提交

流水线结束。

---

## 规则

1. 严格串行——步骤 1 → 用户确认 → 2 → 3，不得跳过
2. 用户确认不可省略——诊断结果必须展示
3. 任一 subagent 失败则停流水线
4. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`
5. 诊断阶段不修改代码——只读分析
