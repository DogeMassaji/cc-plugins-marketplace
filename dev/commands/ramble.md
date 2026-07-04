---
description: Ramble 流水线——大型变更完整交付：策划者 (opus) → 用户确认 → 按单体/前后端分离两条路径执行构建 → 审查者 (opus) → 修复
---

Ramble 模式（大型变更 >100 LOC）完整流水线。调用专职 subagent 完成全部阶段，审查后自动修复，单轮即结束。

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

**产物归档：** 所有阶段产物统一存放在 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下。

---

## 步骤 1——启动 dev:senior-engineer 策划

启动 `dev:senior-engineer` subagent，传入用户的需求描述，显式指定 `model: "opus"`。

该 subagent 将依次执行（同一个 agent，不拆分）：
1. **DEFINE** — 判断是否前后端分离项目，生成 `SPEC.md` 并提交
2. **PLAN** — 生成 `PLAN.md` + 任务列表并提交
   - 非前后端分离 → `TODO.md`
   - 前后端分离 → `TODO_BACKEND.md` + `TODO_FRONTEND.md`

等待 subagent 完成。**向用户展示所有策划产物（SPEC.md、PLAN.md、TODO.md），等待用户确认后再进入构建。** 用户可能：
- 认可策划方案 → 继续进入构建
- 要求调整 → 修改后重新提交确认
- 不认可 → 流水线终止

保持生命周期，等待 **BUILD** 过程结束后，进行 **REVIEW** 等后续阶段

---

## 单体项目路径

### 步骤 2-A——启动 dev:full-stack-developer

收到用户确认后，启动 `dev:full-stack-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 PLAN.md + TODO.md，按任务顺序逐项实现、验证并提交
- 若项目有 HTTP API → 可选运行 `dev:api-doc-generator`
- 任意任务失败立即停止并回报

等待 subagent 完成，保持生命周期，并向用户展示构建结果。跳至**步骤 4**。

---

## 前后端分离路径

### 步骤 2-B——启动 dev:backend-developer

收到用户确认后，启动 `dev:backend-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 PLAN.md + TODO_BACKEND.md，按任务顺序逐项实现、验证并提交
- 全部任务完成后自动运行 `dev:api-doc-generator`，输出 `.artifacts/<yyyymmdd>/<任务简述>/API.md`
- 任意任务失败立即停止并回报

等待 subagent 完成，保持生命周期，并向用户展示后端构建结果和接口文档摘要。

### 步骤 3-B——启动 dev:frontend-developer

后端构建完成且 `API.md` 生成后，启动 `dev:frontend-developer` subagent，显式指定 `model: "sonnet"`。

该 subagent 将执行：
- BUILD：读取 `API.md`（后端接口文档）→ 读取 PLAN.md + TODO_FRONTEND.md → 逐任务实现
- API 调用层严格按 API.md 定义实现，确保前后端接口对齐
- 任意任务失败立即停止并回报

**设计意图：** 后端先完成并产出接口文档，前端基于真实后端接口实现，杜绝前后端接口不一致。

等待 subagent 完成，保持生命周期，并向用户展示前端构建结果。

---

## 步骤 4——启动 dev:senior-engineer（审查）

构建阶段全部完成后，再次启动步骤 1 的 `dev:senior-engineer` subagent，传入以下信息：
- 构建阶段使用的 todo 文件路径（`TODO.md` 或 `TODO_BACKEND.md` + `TODO_FRONTEND.md`）
- 项目类型（单体/前后端分离）

该 subagent 将执行：
1. **TODO 状态检查** — 读取 todo 文件，对照 git diff 逐项标注完成状态，**更新 TODO.md** 中的实际标记
2. **REVIEW** — 覆盖五个维度审查所有变更，必要时深入执行安全加固
3. **生成修复清单** — 以 `- [ ]` 格式在 `REVIEW.md` 中列出待修复项，每个问题标注文件路径和行号
4. 输出 `REVIEW.md` 并提交（含 TODO.md 变更）

等待 subagent 完成。向用户展示审查报告。若无任何修复项，流水线结束。若有修复项，自动进入下一步。

---

## 步骤 5——修复审查问题

读取 `REVIEW.md` 中的修复清单，选择对应的开发 agent 进行修复：

| 项目类型 | 修复 agent |
|----------|-----------|
| 单体项目 | `dev:full-stack-developer` |
| 前后端分离 — 仅有后端问题 | `dev:backend-developer` |
| 前后端分离 — 仅有前端问题 | `dev:frontend-developer` |
| 前后端分离 — 两端都有问题 | 先启动后端修复 agent，完成后再启动前端修复 agent |

启动对应步骤 2 或步骤 3 的 dev subagent，prompt 中包含：
- 指向 `REVIEW.md` 的路径
- 明确指令：读取 REVIEW.md 中的修复清单，逐项修复代码（不修改 REVIEW.md，checkbox 由 reviewer 在下一轮更新）
- 项目类型和使用的 todo 文件路径

等待 subagent 完成。向用户展示修复结果，流水线结束。

**单轮修复，无重新审查。** 修复完成即结束，由用户自行验证。

## 跳过入口

| 用户意图 | 动作 |
|----------|------|
| "从 plan 开始" | 启动 `dev:senior-engineer`，告知从 PLAN 入口 |
| "直接构建后端" | 跳过步骤 1，直接执行步骤 2-B |
| "直接构建前端" | 跳过步骤 1 和 2-B，直接执行步骤 3-B |
| "直接审查" | 跳过所有构建步骤，直接执行步骤 4 |
| "直接修复" | 跳过步骤 1-4，传入 REVIEW.md 路径直接执行步骤 5（修复） |

---

## 规则

1. **步骤 1 是分支点**——senior-engineer 内部完成项目类型判断和全部策划产出，向用户展示后确认进入构建
2. **用户确认不可省略**——策划结果必须展示给用户，收到明确信号后才进入构建
3. **后端先于前端**——前后端分离路径中，步骤 2-B 完成并产出 API.md 后才能执行步骤 3-B
4. **接口文档是契约**——API.md 是前后端之间的接口契约，前端严格对照实现
5. 同路径内严格串行——每个完成后立即启动下一个，无需人工确认
6. **model 参数必须显式指定**——不依赖 agent frontmatter 的 model 字段
7. 任一 subagent 失败则停止流水线，不得继续
8. 所有产物写入 `.artifacts/<yyyymmdd>/<任务简述>/`，不得散落在项目根目录
9.  **修复清单即契约**——REVIEW.md 中的 `- [ ]` 修复项是 dev agent 修复的唯一依据，不得自行扩大修复范围
