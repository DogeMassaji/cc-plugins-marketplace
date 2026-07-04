---
description: Ramble 流水线——大型变更完整交付：策划者 (opus) → 用户确认 → 按单体/前后端分离两条路径执行构建 → 审查者 (opus) → 修复
---

Ramble 模式（大型变更 >100 LOC）。调 subagent 完成所有阶段，审查后修复，单轮结束。

| 步骤 | Agent | model | 阶段 |
|------|-------|-------|------|
| 1 | `dev:senior-engineer` | `opus` | PLAN — DEFINE + PLAN 合并，产出 SPEC.md + PLAN.md + 任务列表 |
| — | 用户确认 | — | 展示策划结果 → 用户确认后进入构建 |
| 2-A | `dev:full-stack-developer` | `sonnet` | BUILD（单体） |
| 2-B | `dev:backend-developer` | `sonnet` | BUILD（后端） |
| — | 产出 `API.md` | — | 接口文档 |
| 3-B | `dev:frontend-developer` | `sonnet` | BUILD（前端） |
| 4 | `dev:senior-engineer` | `opus` | REVIEW → 标注 TODO 状态 → 生成修复清单 |
| 5 | 对应 dev agent | `sonnet` | FIX — 按修复清单逐项修复 |

**产物归档：** 所有阶段产物统一存入 `.artifacts/<yyyymmdd>/<任务简述>/`。

---

## 步骤 1——起 senior-engineer 策划

起 `dev:senior-engineer` subagent，传用户需求，指定 `model: "opus"`。

依次执行（同一 agent，不拆分）：
1. **DEFINE** — 判断是否前后端分离，生成 `SPEC.md` 并提交
2. **PLAN** — 生成 `PLAN.md` + 任务列表并提交
   - 非前后端分离 → `TODO.md`
   - 前后端分离 → `TODO_BACKEND.md` + `TODO_FRONTEND.md`

等 subagent 完成。**展示所有策划产物给用户，等确认再进构建。** 用户可能：
- 认可 → 进构建
- 调整 → 改后重新确认
- 不认可 → 终止

保持生命周期，等 BUILD 完成再进行 REVIEW 等后续阶段。

---

## 单体路径

### 步骤 2-A——起 full-stack-developer

收到用户确认后，起 `dev:full-stack-developer` subagent，指定 `model: "sonnet"`。

执行：
- BUILD：读 PLAN.md + TODO.md，按序逐项实现、验证、提交
- 项目有 HTTP API → 可选跑 `dev:api-doc-generator`
- 任意任务失败即停回报

等 subagent 完成，展示构建结果。跳**步骤 4**。

---

## 前后端分离路径

### 步骤 2-B——起 backend-developer

收到用户确认后，起 `dev:backend-developer` subagent，指定 `model: "sonnet"`。

执行：
- BUILD：读 PLAN.md + TODO_BACKEND.md，按序逐项实现、验证、提交
- 任务完成自动跑 `dev:api-doc-generator`，输出 `.artifacts/<yyyymmdd>/<任务简述>/API.md`
- 任意任务失败即停回报

等 subagent 完成，展示后端构建结果和接口文档摘要。

### 步骤 3-B——起 frontend-developer

后端完成且 `API.md` 生成后，起 `dev:frontend-developer` subagent，指定 `model: "sonnet"`。

执行：
- BUILD：读 `API.md` → 读 PLAN.md + TODO_FRONTEND.md → 逐任务实现
- API 调用层严格按 API.md 实现，确保前后端接口对齐
- 任意任务失败即停回报

**设计意图：** 后端先完成并产出接口文档，前端基于真实后端接口实现，杜绝不一致。

等 subagent 完成，展示前端构建结果。

---

## 步骤 4——起 dev:senior-engineer（审查）

构建阶段全完成后，再次起步骤 1 的 `dev:senior-engineer` subagent，传：
- todo 文件路径（`TODO.md` 或 `TODO_BACKEND.md` + `TODO_FRONTEND.md`）
- 项目类型（单体/分离）

执行：
1. **TODO 状态检查** — 读 todo 文件，对照 git diff 逐项标注完成状态，**更新 TODO.md** 标记
2. **REVIEW** — 五维度审查所有变更，必要时深入安全加固
3. **生成修复清单** — `- [ ]` 格式在 `REVIEW.md` 列出，每项标注路径和行号
4. 输出 `REVIEW.md` 并提交（含 TODO.md 变更）

等 subagent 完成。展示审查报告。无修复项则结束。有修复项自动进下一步。

---

## 步骤 5——修复审查问题

读 `REVIEW.md` 修复清单，选对应 dev agent 修复：

| 项目类型 | 修复 agent |
|----------|-----------|
| 单体 | `dev:full-stack-developer` |
| 分离 — 仅后端 | `dev:backend-developer` |
| 分离 — 仅前端 | `dev:frontend-developer` |
| 分离 — 两端都有 | 先启动后端，完成后再启动前端 |

起对应 dev subagent，prompt 含：
- `REVIEW.md` 路径
- 指令：读修复清单逐项修复（不修改 REVIEW.md）
- 项目类型和 todo 文件路径

等 subagent 完成。展示修复结果，流水线结束。

**单轮修复，无重新审查。** 修复完成即结束，由用户自行验证。

## 跳过入口

| 用户意图 | 动作 |
|----------|------|
| "从 plan 开始" | 起 `dev:senior-engineer`，告知从 PLAN 入口 |
| "直接构建后端" | 跳过步骤 1，直接 2-B |
| "直接构建前端" | 跳过 1 和 2-B，直接 3-B |
| "直接审查" | 跳过所有构建，直接步骤 4 |
| "直接修复" | 跳过 1-4，传 REVIEW.md 路径直接步骤 5 |

---

## 规则

1. **步骤 1 是分支点**——senior-engineer 内完成项目类型判断和全部策划产出，展示后确认进构建
2. **用户确认不可省略**——策划结果必须展示，收明确信号才进构建
3. **后端先于前端**——分离路径中，2-B 完成并产出 API.md 后才能 3-B
4. **接口文档是契约**——API.md 是前后端接口契约，前端严格对照
5. 同路径内严格串行——每步完成自动启动下一步，无需人工确认
6. **model 必须显式指定**——不依赖 agent frontmatter model 字段
7. 任一 subagent 失败则停流水线
8. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不散落在根目录
9. **修复清单即契约**——REVIEW.md `- [ ]` 是修复唯⼀依据，不得自行扩大范围
