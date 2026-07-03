---
description: Popping 流水线（S 模式）——小变更快速交付：全栈工程师 (sonnet) → 初级审查者 (sonnet) → 全栈工程师修复 (sonnet)，BUILD → REVIEW → FIX 单轮
---

Popping — 短促精准。S 模式（小变更 ≤50 LOC）专用轻量流水线。启动三个 subagent 串行执行，审查后自动修复，单轮即结束。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:full-stack-developer` | `sonnet` | BUILD — 理解需求 → 实现 → 编译检查 → 提交 |
| 2 | `dev:junior-reviewer` | `sonnet` | REVIEW — 五轴审查 → 输出修复建议 |
| 3 | `dev:full-stack-developer` | `sonnet` | FIX — 按修复清单逐项修复 |

**产物归档：** 审查报告存放在 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`。

**适用场景：**
- ≤50 LOC 变更
- 单文件改动
- typo 修复、配置修改、简单 bug 修复、单函数重构
- 无架构影响

**不适用场景（建议升级到 `/flow` 或 `/workflow`）：**
- 新功能、新模块
- 跨多文件/多模块变更
- API 契约变更
- 数据库 schema 变更

---

## 步骤 1——启动 dev:full-stack-developer（sonnet）

启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：理解用户需求 → 读取相关代码 → 实现变更
- 编译/类型检查，不通过则修复
- 提交（中文 Conventional Commit 格式）
- 不生成 spec/plan/todo 产物文件
- **跳过测试生成**——不调用 `dev:backend-test-generator`，减少不必要的测试代码产出

等待 subagent 完成。向用户展示构建结果。

---

## 步骤 2——启动 dev:junior-reviewer（sonnet）

构建完成后，启动 `dev:junior-reviewer` subagent，显式指定 `model: "sonnet"`，传入：
- git diff 范围（最近提交）
- 用户原始需求描述

该 subagent 将执行：
1. **REVIEW** — 覆盖五个维度审查变更，必要时执行安全加固
2. **生成修复建议** — 以 `- [ ]` 格式在 `REVIEW.md` 中列出，标注文件路径、行号和严重级别
3. 输出 `REVIEW.md` 并提交

等待 subagent 完成。向用户展示审查报告。

---

## 步骤 3——启动 dev:full-stack-developer 修复（sonnet）

审查完成后，再次启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`，传入：
- `REVIEW.md` 路径（包含修复清单）

该 subagent 将执行：
- FIX：读取 REVIEW.md 中的修复清单（`- [ ]` 项）
- 按严重级别排序（Critical → Important → Suggestion）
- 逐项修复、验证、提交（commit message 标注 `#review`）
- 不修改 REVIEW.md

等待 subagent 完成。向用户展示修复结果，流水线结束。

**单轮修复，无重新审查。** 修复完成即结束，由用户自行验证。

---

## Agent tool call 示例

```
步骤 1: Agent(description="实现变更", subagent_type="dev:full-stack-developer", model="sonnet", prompt="实现以下小变更: ...")
步骤 2: Agent(description="审查代码", subagent_type="dev:junior-reviewer", model="sonnet", prompt="审查最近提交的变更: ...")
步骤 3: Agent(description="修复审查问题", subagent_type="dev:full-stack-developer", model="sonnet", prompt="审查反馈修复: 读取 .artifacts/.../REVIEW.md，逐项修复并提交")
```

## 规则

1. **严格串行**——步骤 1 → 2 → 3 顺序执行，不得跳过
2. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
3. 产物驱动——subagent 之间通过 git diff / 产物文件传递信息
4. 任一 subagent 失败则停止流水线，不得继续
5. 审查产物写入 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`
6. **单轮审查 + 单轮修复**——无重新审查循环，修复后即结束
7. **无策划产物**——S 模式不生成 SPEC.md / PLAN.md / TODO.md
