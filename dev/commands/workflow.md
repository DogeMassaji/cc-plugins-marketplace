---
description: 串行流水线——启动 senior-developer-planner (opus) → 判断项目类型 → 按单体/前后端分离两条路径执行构建 → senior-reviewer (opus) → 修复 → 重新审查验证
---

调用专职 subagent 完成完整开发流水线，每个 subagent 显式指定 model 参数：

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `senior-developer-planner` | `opus` | DEFINE → PLAN |
| 2 | — | — | 判断项目类型（读取产物） |
| 3-A | `senior-developer` | `sonnet` | BUILD（单体） |
| 3-B | `senior-developer-backend` | `sonnet` | BUILD（后端） |
| — | 产出 `api.md` | — | 接口文档 |
| 4-B | `senior-developer-frontend` | `sonnet` | BUILD（前端） |
| 5 | `senior-reviewer`（第一轮） | `opus` | REVIEW → 标注 TODO 状态 → 生成修复清单 |
| 6 | 对应 dev agent | `sonnet` | FIX — 按修复清单逐项修复 |
| 7 | `senior-reviewer`（重新审查） | `opus` | RE-REVIEW → 验证修复 → 更新 TODO 状态 |

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

---

## 步骤 1——启动 senior-developer-planner（opus）

启动 `senior-developer-planner` subagent，传入用户的需求描述，显式指定 `model: "opus"`。

该 subagent 将依次执行：
- 判断是否前后端分离项目
- DEFINE：生成 `spec.md` 并提交，自动进入 PLAN
- PLAN：生成 `plan.md` + 任务列表并提交
  - 非前后端分离 → `todo.md`
  - 前后端分离 → `todo_backend.md` + `todo_frontend.md`

等待 subagent 完成。向用户展示策划结果。

---

## 步骤 2——判断项目类型

读取 planner 产出的产物，根据文件存在性判定：

| 产物 | 判定 |
|------|------|
| 仅存在 `todo.md` | → **单体项目路径（步骤 3-A）** |
| 存在 `todo_backend.md` + `todo_frontend.md` | → **前后端分离路径（步骤 3-B）** |

---

## 单体项目路径

### 步骤 3-A——启动 senior-developer（sonnet）

启动 `senior-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 plan.md + todo.md，按任务顺序逐项实现、验证并提交
- 若项目有 HTTP API → 可选运行 `api-doc-generator`
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示构建结果。跳至**步骤 5**。

---

## 前后端分离路径

### 步骤 3-B——启动 senior-developer-backend（sonnet）

启动 `senior-developer-backend` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 plan.md + todo_backend.md，按任务顺序逐项实现、验证并提交
- 全部任务完成后自动运行 `api-doc-generator`，输出 `.artifacts/<yyyymmdd>/<任务简述>/api.md`
- 任意任务失败立即停止并回报

等待 subagent 完成。向用户展示后端构建结果和接口文档摘要。

### 步骤 4-B——启动 senior-developer-frontend（sonnet）

后端构建完成且 `api.md` 生成后，启动 `senior-developer-frontend` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 `api.md`（后端接口文档）→ 读取 plan.md + todo_frontend.md → 逐任务实现
- API 调用层严格按 api.md 定义实现，确保前后端接口对齐
- 任意任务失败立即停止并回报

**设计意图：** 后端先完成并产出接口文档，前端基于真实后端接口实现，杜绝前后端接口不一致。

等待 subagent 完成。向用户展示前端构建结果。

---

## 步骤 5——启动 senior-reviewer（第一轮审查）

构建阶段全部完成后，启动 `senior-reviewer` subagent，显式指定 `model: "opus"`，传入以下信息：
- 构建阶段使用的 todo 文件路径（`todo.md` 或 `todo_backend.md` + `todo_frontend.md`）
- 项目类型（单体/前后端分离）

该 subagent 将执行：
1. **TODO 状态检查** — 读取 todo 文件，对照 git diff 逐项标注完成状态，**更新 todo.md** 中的实际标记
2. **REVIEW** — 覆盖五个维度审查所有变更，必要时深入执行安全加固
3. **生成修复清单** — 以 `- [ ]` 格式在 `review.md` 中列出待修复项，每个问题标注文件路径和行号
4. 输出 `review.md` 并提交（含 todo.md 变更）

等待 subagent 完成。向用户展示审查报告。若无任何修复项，流水线结束。

---

## 步骤 6——修复审查问题

读取 `review.md` 中的修复清单，选择对应的开发 agent 进行修复：

| 项目类型 | 修复 agent |
|----------|-----------|
| 单体项目 | `senior-developer`（sonnet） |
| 前后端分离 — 仅有后端问题 | `senior-developer-backend`（sonnet） |
| 前后端分离 — 仅有前端问题 | `senior-developer-frontend`（sonnet） |
| 前后端分离 — 两端都有问题 | 先启动后端修复 agent，完成后再启动前端修复 agent |

启动对应的 dev subagent，显式指定 `model: "sonnet"`，prompt 中包含：
- 指向 `review.md` 的路径
- 明确指令：读取 review.md 中的修复清单，逐项修复代码（不修改 review.md，checkbox 由 reviewer 在下一轮更新）
- 项目类型和使用的 todo 文件路径

等待 subagent 完成。若无修复 agent 被调用（全部为 suggestion 级别），跳至步骤 7。

## 步骤 7——重新审查验证

再次启动 `senior-reviewer` subagent，显式指定 `model: "opus"`，传入：
- `review.md` 路径（包含修复清单）
- 模式标记：`re-review`

该 subagent 将执行：
1. **增量验证** — 读取 review.md 中的修复清单，逐项检查是否已修复
2. **更新 TODO 状态** — 已修复项标注 `[x]`，未完全修复项追加备注 `(未完全修复: ...)`，同步更新 todo.md
3. **检查回归** — 确认修复没有引入新问题
4. 更新 `review.md` 和 `todo.md` 并提交

等待 subagent 完成。向用户展示重新审查报告。

**循环上限**：修复-审查循环最多执行 2 轮。若第 2 轮仍有未修复项，报告用户决策。

## 跳过入口

| 用户意图 | 动作 |
|----------|------|
| "从 plan 开始" | 启动 `senior-developer-planner`，告知从 PLAN 入口 |
| "直接构建后端" | 跳过步骤 1-2，直接执行步骤 3-B |
| "直接构建前端" | 跳过步骤 1-2 和 3-B，直接执行步骤 4-B |
| "直接审查" | 跳过所有构建步骤，直接执行步骤 5（第一轮审查） |
| "直接修复" | 跳过步骤 1-5，传入 review.md 路径直接执行步骤 6（修复） |
| "重新审查" | 跳过步骤 1-6，传入 review.md 路径直接执行步骤 7（重新审查） |

---

## Agent tool call 示例

```
步骤 1: Agent(description="策划需求", subagent_type="senior-developer-planner", model="opus", prompt="...")

单体路径：
  步骤 3-A: Agent(description="实现代码", subagent_type="senior-developer", model="sonnet", prompt="...")

前后端分离路径：
  步骤 3-B: Agent(description="实现后端", subagent_type="senior-developer-backend", model="sonnet", prompt="...")
  步骤 4-B: Agent(description="实现前端", subagent_type="senior-developer-frontend", model="sonnet", prompt="...")

步骤 5 (第一轮审查): Agent(description="审查代码", subagent_type="senior-reviewer", model="opus", prompt="...")
步骤 6 (修复):       Agent(description="修复审查问题", subagent_type="senior-developer", model="sonnet", prompt="审查反馈修复: 读取 .artifacts/.../review.md，逐项修复并提交")
步骤 7 (重新审查):   Agent(description="重新审查", subagent_type="senior-reviewer", model="opus", prompt="[re-review] 检查修复: review.md 路径为 .artifacts/.../review.md")
```

## 规则

1. **步骤 2 是分支点**——读取产物判定路径后，两条路径互斥，不得同时执行
2. **后端先于前端**——前后端分离路径中，步骤 3-B 完成并产出 api.md 后才能执行步骤 4-B
3. **接口文档是契约**——api.md 是前后端之间的接口契约，前端严格对照实现
4. 同路径内严格串行——每个完成后立即启动下一个，无需人工确认
5. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
6. 产物驱动——subagent 之间通过文件产物传递信息，不依赖上下文记忆
7. 任一 subagent 失败则停止流水线，不得继续
8. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录
9. **修复-审查循环上限 2 轮**——第 2 轮仍有未修复项则报告用户决策，不得无限循环
10. **修复清单即契约**——review.md 中的 `- [ ]` 修复项是 dev agent 修复的唯一依据，不得自行扩大修复范围
11. **重新审查必须增量**——只检查修复项和回归，不重新审查所有代码
