---
description: Locking 流水线（M 模式）——标准变更交付：策划者 (opus) → 全栈工程师 (sonnet) → 初级审查者 (sonnet) → 全栈工程师修复 (sonnet)，SPEC → PLAN → BUILD → REVIEW → FIX 单轮
---

Locking — 卡点锁步。M 模式（中等变更 50-200 LOC）标准流水线。启动四个 subagent 串行执行，审查后自动修复，单轮即结束，无重新审查循环。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:senior-developer` | `opus` | DEFINE → PLAN |
| 2 | `dev:full-stack-developer` | `sonnet` | BUILD |
| 3 | `dev:junior-reviewer` | `sonnet` | REVIEW |
| 4 | `dev:full-stack-developer` | `sonnet` | FIX — 按修复清单逐项修复 |

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

**适用场景：**
- 50-200 LOC 变更
- 2-5 个文件
- 单模块功能增强、中等重构
- 需要结构化策划但无需复杂审查循环

**不适用场景（建议升级到 `/workflow`）：**
- 多模块、架构级变更
- 前后端分离项目
- 需要严格修复-验证循环的高风险变更

---

## 步骤 1——启动 dev:senior-developer（opus）

启动 `dev:senior-developer` subagent，传入用户的需求描述，显式指定 `model: "opus"`。

该 subagent 将依次执行：
- DEFINE：生成 `SPEC.md` 并提交，自动进入 PLAN
- PLAN：生成 `PLAN.md` + `TODO.md` 并提交

等待 subagent 完成。向用户展示策划结果。

---

## 步骤 2——启动 dev:full-stack-developer（sonnet）

启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 PLAN.md + TODO.md，按任务顺序逐项实现、验证并提交
- 若项目有 HTTP API → 可选运行 `dev:api-doc-generator`
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示构建结果。

---

## 步骤 3——启动 dev:junior-reviewer（sonnet）

构建阶段完成后，启动 `dev:junior-reviewer` subagent，显式指定 `model: "sonnet"`，传入：
- git diff 范围（构建阶段所有提交）
- SPEC.md / PLAN.md 路径（用于对照需求）
- TODO.md 路径（用于标注完成状态）

该 subagent 将执行：
1. **TODO 状态检查** — 读取 TODO.md，对照 git diff 逐项标注完成状态
2. **REVIEW** — 覆盖五个维度审查所有变更，必要时深入执行安全加固
3. **生成修复建议** — 以 `- [ ]` 格式在 `review.md` 中列出
4. 输出 `review.md` 并提交（含 TODO.md 变更）

等待 subagent 完成。向用户展示审查报告。

---

## 步骤 4——启动 dev:full-stack-developer 修复（sonnet）

审查完成后，再次启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`，传入：
- `review.md` 路径（包含修复清单）
- `TODO.md` 路径

该 subagent 将执行：
- FIX：读取 review.md 中的修复清单（`- [ ]` 项）
- 按严重级别排序（Critical → Important → Suggestion）
- 逐项修复、验证、提交（commit message 标注 `#review`）
- 不修改 review.md（checkbox 由 reviewer 更新）

等待 subagent 完成。向用户展示修复结果，流水线结束。

**单轮修复，无重新审查。** 修复完成即结束，由用户自行验证。

---

## Agent tool call 示例

```
步骤 1: Agent(description="策划需求", subagent_type="dev:senior-developer", model="opus", prompt="...")
步骤 2: Agent(description="实现代码", subagent_type="dev:full-stack-developer", model="sonnet", prompt="读取 .artifacts/.../PLAN.md 和 .artifacts/.../TODO.md，逐任务实现并提交")
步骤 3: Agent(description="审查代码", subagent_type="dev:junior-reviewer", model="sonnet", prompt="审查构建阶段的所有变更: SPEC.md 路径为 .artifacts/.../SPEC.md, TODO.md 路径为 .artifacts/.../TODO.md")
步骤 4: Agent(description="修复审查问题", subagent_type="dev:full-stack-developer", model="sonnet", prompt="审查反馈修复: 读取 .artifacts/.../review.md，逐项修复并提交")
```

## 规则

1. **严格串行**——步骤 1 → 2 → 3 → 4 顺序执行，不得跳过
2. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
3. 产物驱动——subagent 之间通过文件产物传递信息，不依赖上下文记忆
4. 任一 subagent 失败则停止流水线，不得继续
5. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`
6. **单轮审查 + 单轮修复**——无重新审查循环，修复后即结束
7. **无前后端分离**——M 模式仅支持单体项目，前后端分离请使用 `/breaking`
