# dev 精简参考

命令、Skill、Hook 说明。

---

## 一、命令总览（11 个）

### 三模式流水线

| 命令 | 模式 | 规模 | 流程 | 审查者 |
|------|------|------|------|--------|
| `/popping` | S | ≤50 LOC | BUILD → REVIEW → FIX（单轮） | junior-reviewer (sonnet) |
| `/locking` | M | 50-200 LOC | SPEC → PLAN → BUILD → REVIEW → FIX（单轮） | junior-reviewer (sonnet) |
| `/breaking` | L | >200 LOC | SPEC → PLAN → BUILD → REVIEW → FIX → RE-REVIEW（含循环） | senior-reviewer (opus) |

### 单阶段命令

| 命令 | 开发阶段 | 核心 Skill | 辅助 Skill |
|------|----------|-----------|-----------|
| `/spec` | Define | `dev:spec-driven-development` | — |
| `/plan` | Plan | `dev:planning-and-task-breakdown` | — |
| `/build` | Build | `dev:incremental-implementation` | — |
| `/review` | Review | `dev:code-review-and-quality` | `dev:security-and-hardening` |
| `/check` | Check | 只读快速扫描 | 编译检查、安全扫描、规范检查 |
| `/fix` | Fix | `dev:incremental-implementation` | 按 review.md 清单逐项修复 |
| `/re-review` | Re-Review | `dev:code-review-and-quality` | 增量验证修复清单 |
| `/doc` | Doc | `dev:doc-archiver` | 归档/整理项目文档 |

---

## 二、命令详情

### `/popping` — Popping 流水线（S 模式）

小变更快速交付。全栈工程师实现 → 初级审查者单轮审查。

- 命令：`commands/popping.md`
- Agent：`dev:full-stack-engineer` (sonnet) → `dev:junior-reviewer` (sonnet)

### `/locking` — Locking（M 模式）

标准变更交付。策划者策划 → 全栈工程师实现 → 初级审查者审查 → 全栈工程师修复。

- 命令：`commands/locking.md`
- Agent：`dev:senior-developer-planner` (opus) → `dev:full-stack-engineer` (sonnet) → `dev:junior-reviewer` (sonnet) → `dev:full-stack-engineer` (sonnet)

### `/breaking` — Breaking 流水线（L 模式）

大型变更完整交付。策划 → 构建（支持前后端分离）→ 高级审查 → 修复 → 重新审查。

- 命令：`commands/breaking.md`
- Agent：planner (opus) → build (sonnet) → senior-reviewer (opus) → fix (sonnet) → re-review (opus)

### `/spec` — 编写结构化 Spec 说明

1. 向用户提问，澄清：目标用户、核心功能与验收标准、技术栈偏好与约束、边界条件
2. 生成涵盖六大领域的结构化 Spec 说明
3. 保存到 `.artifacts/<yyyymmdd>/<任务简述>/spec.md`，与用户确认

- 命令：`commands/spec.md`
- Skill：`skills/spec-driven-development/SKILL.md`

### `/plan` — 将工作拆分为可验证的小任务

1. 读取 Spec，进入 Plan Mode（只读）
2. 垂直切片，每任务附带验收标准
3. 保存到 `.artifacts/<yyyymmdd>/<任务简述>/plan.md` + `todo.md`

- 命令：`commands/plan.md`
- Skill：`skills/planning-and-task-breakdown/SKILL.md`

### `/build` — 增量实现

1. 从 `.artifacts/<yyyymmdd>/<任务简述>/todo.md` 取下一个任务
2. 实现代码 → 运行构建验证 → 提交
3. 失败时输出错误，等待用户指导

- 命令：`commands/build.md`
- Skill：`skills/incremental-implementation/SKILL.md`

### `/review` — 五轴代码审查

正确性、可读性、架构、安全性、性能。输出 Critical / Important / Suggestion 分类报告到 `.artifacts/<yyyymmdd>/<任务简述>/review.md`。

- 命令：`commands/review.md`
- Skill：`skills/code-review-and-quality/SKILL.md`

### `/check` — 快速预检（只读）

1. 读取 git diff（已暂存或最近提交）
2. 编译/类型检查（如适用）
3. 快速安全扫描（硬编码密钥、SQL 注入、未验证输入）
4. 基本规范检查
5. 终端直接输出，不生成产物文件

- 命令：`commands/check.md`
- 模型：haiku（轻量快速）
- 级别：🔴 阻塞 / 🟡 警告 / ℹ️ 建议

### `/fix` — 按审查清单修复

1. 读取 `review.md` 中的修复清单
2. 选择匹配的 dev agent，逐项修复
3. 通过 `#review` 标记提交

- 命令：`commands/fix.md`

### `/re-review` — 重新审查验证

1. 调用 senior-reviewer 进入 RE-REVIEW 模式
2. 增量验证修复清单中的问题是否已修复
3. 检测是否有回归问题

- 命令：`commands/re-review.md`

### `/doc` — 文档归档

1. 调用 `dev:doc-archiver` skill
2. 按编号目录 + 中文描述文件名规范归档
3. 支持创建、整理、废弃、TODO 维护

- 命令：`commands/doc.md`

---

## 三、产物归档

所有文档类产出（Spec、计划、任务列表、审查报告）统一存放在：

```
.artifacts/
  └── <yyyymmdd>/<任务简述>/
      ├── spec.md      # 结构化规格说明
      ├── plan.md      # 实现计划
      ├── todo.md      # 任务列表
      └── review.md    # 审查报告
```

---

## 四、完整 Skill 清单（11 个）

| Skill | 阶段 | 命令 | 说明 |
|-------|------|------|------|
| `dev:interview-me` | Define | — | 需求访谈 + 想法精炼 |
| `dev:spec-driven-development` | Define | `/spec` | 结构化 Spec 说明 |
| `dev:planning-and-task-breakdown` | Plan | `/plan` | 垂直切片，任务拆分 |
| `dev:incremental-implementation` | Build | `/build` | 逐个任务增量实现 |
| `dev:backend-test-generator` | Build | — | 根据变更自动生成后端测试用例并执行 |
| `dev:api-doc-generator` | Build | — | 扫描后端路由/控制器自动生成 API 接口文档 |
| `dev:code-review-and-quality` | Review | `/review` | 五轴审查 |
| `dev:security-and-hardening` | Review | `/review` 安全维度 | OWASP 防护、威胁建模、AI/LLM |
| `dev:git-commit` | Meta | — | 中文 Conventional Commit 提交 |
| `dev:doc-archiver` | Meta | `/doc` | 按编号目录归档项目文档 |
| `dev:using-agent-skills` | Meta | session 自动注入 | Skill 发现与路由，核心行为规范 |

---

## 五、Hooks（2 个）

| Hook | 文件 | 触发 | 用途 |
|------|------|------|------|
| SessionStart | `hooks/session-start.sh` | session 启动 | 注入 `dev:using-agent-skills` |
| SessionStart Test | `hooks/session-start-test.sh` | 手动 | 验证 SessionStart payload |

---

## 六、Agent 清单（6 个）

| Agent | model | 职责 |
|-------|-------|------|
| `dev:senior-developer-planner` | opus | DEFINE + PLAN 策划 |
| `dev:full-stack-engineer` | sonnet | BUILD 全栈实现 |
| `dev:senior-developer-backend` | sonnet | BUILD 后端实现 |
| `dev:senior-developer-frontend` | sonnet | BUILD 前端实现 |
| `dev:junior-reviewer` | sonnet | 单轮 REVIEW（S/M 模式） |
| `dev:senior-reviewer` | opus | 完整 REVIEW + RE-REVIEW（L 模式） |

---

## 七、生命周期

```
S: POPPING  → /popping  → full-stack-engineer (sonnet) → junior-reviewer (sonnet) → full-stack-engineer (sonnet)
M: LOCKING  → /locking  → planner (opus) → full-stack-engineer (sonnet) → junior-reviewer (sonnet) → full-stack-engineer (sonnet)
L: BREAKING → /breaking → planner (opus) → build (sonnet) → senior-reviewer (opus) → fix → re-review
DEFINE      → /spec     → dev:spec-driven-development
            → (描述需求) → dev:interview-me
PLAN        → /plan     → dev:planning-and-task-breakdown
BUILD       → /build    → dev:incremental-implementation
REVIEW      → /review   → dev:code-review-and-quality + dev:security-and-hardening
CHECK       → /check    → 只读快速扫描（编译+安全+规范）
FIX         → /fix      → 按 review.md 清单逐项修复
RE-REVIEW   → /re-review → 增量验证修复结果
DOC         → /doc      → dev:doc-archiver
```

---

## 八、文件索引

### 命令定义
| 文件 | 说明 |
|------|------|
| `commands/popping.md` | `/popping` — S 模式 |
| `commands/locking.md` | `/locking` — M 模式 |
| `commands/breaking.md` | `/breaking` — L 模式 |
| `commands/spec.md` | `/spec` |
| `commands/plan.md` | `/plan` |
| `commands/build.md` | `/build` |
| `commands/review.md` | `/review` |
| `commands/check.md` | `/check` |
| `commands/fix.md` | `/fix` |
| `commands/re-review.md` | `/re-review` |
| `commands/doc.md` | `/doc` |

### Skill 定义
| 文件 | 阶段 |
|------|------|
| `skills/interview-me/SKILL.md` | Define |
| `skills/spec-driven-development/SKILL.md` | Define |
| `skills/planning-and-task-breakdown/SKILL.md` | Plan |
| `skills/incremental-implementation/SKILL.md` | Build |
| `skills/backend-test-generator/SKILL.md` | Build |
| `skills/api-doc-generator/SKILL.md` | Build |
| `skills/code-review-and-quality/SKILL.md` | Review |
| `skills/security-and-hardening/SKILL.md` | Review |
| `skills/git-commit/SKILL.md` | Meta |
| `skills/doc-archiver/SKILL.md` | Meta |
| `skills/using-agent-skills/SKILL.md` | Meta |

### Agent 定义
| 文件 | model |
|------|-------|
| `agents/senior-developer-planner.md` | opus |
| `agents/full-stack-engineer.md` | sonnet |
| `agents/senior-developer-backend.md` | sonnet |
| `agents/senior-developer-frontend.md` | sonnet |
| `agents/junior-reviewer.md` | sonnet |
| `agents/senior-reviewer.md` | opus |

### Hooks
| 文件 | 说明 |
|------|------|
| `hooks/hooks.json` | Hook 配置 |
| `hooks/session-start.sh` | SessionStart |
| `hooks/session-start-test.sh` | SessionStart 测试 |

### 参考文档
| 文件 | 说明 |
|------|------|
| `references/security-checklist.md` | 安全检查清单 |

### 其他文档
| 文件 | 说明 |
|------|------|
| `docs/skill-anatomy.md` | Skill 格式说明 |
| `docs/getting-started.md` | 快速入门 |
