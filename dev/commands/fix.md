---
description: Fix 流水线——从问题描述到修复验证：全栈工程师诊断 (sonnet) → 用户确认 → 全栈工程师修复 (sonnet) → 初级审查者 (sonnet)，DIAGNOSE → CONFIRM → FIX → REVIEW 单轮
---

Fix — 精准修复。从用户描述 bug 出发，诊断 → 修复 → 审查。开发周期中间切入的轻量工作流。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:full-stack-developer` | `sonnet` | DIAGNOSE — 定位根因 → 输出诊断报告 |
| — | 用户确认 | — | 展示诊断结果 → 用户确认修复方案 |
| 2 | `dev:full-stack-developer` | `sonnet` | FIX — 按诊断结果修复 → 编译检查 → 提交 |
| 3 | `dev:junior-reviewer` | `sonnet` | REVIEW — 验证修复正确性 → 检查回归 |

**产物归档：** 诊断和审查报告存于 `.artifacts/<yyyymmdd>/<任务简述>/`。

---

## 前置条件

用户提供 bug 描述或异常现象，如：
- "点击提交按钮后页面无响应"
- "API 返回 500，日志看到 NullPointerException"
- "用户登录后 token 未刷新"

无需 REVIEW.md 或 TODO.md。

---

## 步骤 1——起 full-stack-developer 诊断

起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

传用户描述 bug，该 agent 执行：
- 读相关代码，理解上下文
- 复现/定位根因
- 输出诊断报告 `DIAGNOSE.md` 并提交：
  - 问题现象（复现步骤）
  - 根因分析（路径、行号、调用链）
  - 修复方案（改什么、为什么）
  - 影响范围评估
- **不修改代码**——仅诊断

等 subagent 完成。展示诊断结果，**等用户确认再修复**。用户可能：
- 认可 → 进步骤 2
- 补充调整 → 填入步骤 2 prompt
- 不认可 → 自行决定，可终止

---

## 步骤 2——修复

收到用户确认后，传：
- `DIAGNOSE.md` 路径
- 用户补充信息（如有）

执行：
- 读 DIAGNOSE.md 修复方案
- 逐项修复
- 编译/类型检查
- 提交（commit 标注 `#fix`）

等 subagent 完成。展示修复结果。

---

## 步骤 3——起 junior-reviewer 审查

修复完成后，起 `dev:junior-reviewer` subagent，指定 `model: "sonnet"`，传：
- git diff 范围（最近提交）
- `DIAGNOSE.md` 路径
- 用户原始 bug 描述

执行：
1. **验证修复正确性** — 确认修复解决诊断问题
2. **回归检查** — 确认没引入新问题
3. **生成审查建议** — `- [ ]` 格式在 `REVIEW.md` 列出
4. 输出 `REVIEW.md` 并提交

等 subagent 完成。展示审查报告，流水线结束。

**单轮审查，无重新修复。** 审查建议由用户决定是否采纳。

---

## 规则

1. **严格串行**——步骤 1 → 用户确认 → 2 → 3，不得跳过
2. **用户确认不可省略**——诊断结果必须展示，收明确信号才修复
3. **model 必须显式指定**——不依赖 agent frontmatter model 字段
4. 任一 subagent 失败则停流水线
5. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`
6. **单轮诊断 + 单轮修复 + 单轮审查**——无循环
7. **诊断阶段不修改代码**——只读分析
