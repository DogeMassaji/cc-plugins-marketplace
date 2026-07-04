---
description: Fix 流水线——从问题描述到修复验证：全栈工程师诊断 (sonnet) → 用户确认 → 全栈工程师修复 (sonnet) → 初级审查者 (sonnet)，DIAGNOSE → CONFIRM → FIX → REVIEW 单轮
---

Fix — 精准修复。从用户描述的 bug 或现象出发，执行诊断 → 修复 → 审查。从开发周期中间切入的轻量工作流。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:full-stack-developer` | `sonnet` | DIAGNOSE — 定位 bug 根因 → 输出诊断报告 |
| — | 用户确认 | — | 展示诊断结果 → 用户确认修复方案 |
| 2 | `dev:full-stack-developer` | `sonnet` | FIX — 按诊断结果修复代码 → 编译检查 → 提交 |
| 3 | `dev:junior-reviewer` | `sonnet` | REVIEW — 验证修复正确性 → 检查回归 |

**产物归档：** 诊断报告和审查报告存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

---

## 前置条件

用户提供 bug 描述或异常现象，例如：
- "点击提交按钮后页面无响应"
- "API 返回 500，日志里看到 NullPointerException"
- "用户登录后 token 未刷新"

无需 REVIEW.md 或 TODO.md 产物文件。

---

## 步骤 1——启动 dev:full-stack-developer 诊断

启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`。

传入用户描述的 bug 现象，该 subagent 将执行：
- 读取相关代码，理解上下文
- 复现/定位 bug 根因
- 输出诊断报告 `DIAGNOSE.md` 并提交：
  - 问题现象（复现步骤）
  - 根因分析（文件路径、行号、调用链）
  - 修复方案（修改什么、为什么这样改）
  - 影响范围评估
- **不修改代码**——仅诊断

等待 subagent 完成。向用户展示诊断结果，**等待用户确认后再进入修复**。用户可能：
- 认可诊断方案 → 继续进入步骤 2
- 补充信息或调整方向 → 将补充信息填入步骤 2 的 prompt
- 不认可诊断方向 → 用户自行决定后续，流水线可终止

---

## 步骤 2——修复

收到用户确认后，传入：
- `DIAGNOSE.md` 路径
- 用户补充信息（如有）

该 subagent 将执行：
- FIX：读取 DIAGNOSE.md 中的修复方案
- 逐项修复代码
- 编译/类型检查，不通过则修复
- 提交（commit message 标注 `#fix`）

等待 subagent 完成。向用户展示修复结果。

---

## 步骤 3——启动 dev:junior-reviewer 审查（sonnet）

修复完成后，启动 `dev:junior-reviewer` subagent，显式指定 `model: "sonnet"`，传入：
- git diff 范围（最近提交）
- `DIAGNOSE.md` 路径（原始问题描述）
- 用户原始 bug 描述

该 subagent 将执行：
1. **验证修复正确性** — 确认修复确实解决了诊断中描述的问题
2. **回归检查** — 确认修复没有引入新问题
3. **生成审查建议** — 以 `- [ ]` 格式在 `REVIEW.md` 中列出
4. 输出 `REVIEW.md` 并提交

等待 subagent 完成。向用户展示审查报告，流水线结束。

**单轮审查，无重新修复。** 审查建议由用户自行决定是否采纳。

---

## 规则

1. **严格串行**——步骤 1 → 用户确认 → 步骤 2 → 步骤 3 顺序执行，不得跳过
2. **用户确认不可省略**——诊断结果必须展示给用户，收到明确信号后才进入修复
3. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
4. 任一 subagent 失败则停止流水线，不得继续
5. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`
6. **单轮诊断 + 单轮修复 + 单轮审查**——无循环
7. **诊断阶段不修改代码**——只读分析
