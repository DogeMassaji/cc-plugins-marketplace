# dev 精简参考

命令、Skill、Hook 说明。

---

## 一、命令总览（8 个）

| 命令 | 开发阶段 | 核心 Skill | 辅助 Skill |
|------|----------|-----------|-----------|
| `/spec` | Define | `dev:spec-driven-development` | — |
| `/plan` | Plan | `dev:planning-and-task-breakdown` | — |
| `/build` | Build | `dev:incremental-implementation` | — |
| `/review` | Review | `dev:code-review-and-quality` | `dev:security-and-hardening` |
| `/fix` | Fix | `dev:incremental-implementation` | 按 review.md 清单逐项修复 |
| `/re-review` | Re-Review | `dev:code-review-and-quality` | 增量验证修复清单 |
| `/doc` | Doc | `dev:doc-archiver` | 归档/整理项目文档 |
| `/workflow` | 全流程 | 串行编排：planner (opus) → backend (sonnet) → api.md → frontend (sonnet) → reviewer (opus) → fix → re-review | 前后端分离时后端先完成再前端 |

---

## 二、命令详情

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

### `/workflow` — 串行全流程流水线

前后端分离：`dev:senior-developer-planner` (opus: DEFINE → PLAN) → `dev:senior-developer-backend` (sonnet: BUILD 后端) → 产出 `api.md` → `dev:senior-developer-frontend` (sonnet: BUILD 前端 → 消费 api.md) → `dev:senior-reviewer` (opus: REVIEW) → FIX → RE-REVIEW。后端先完成并生成接口文档，前端基于真实文档实现。

单体项目：`dev:senior-developer-planner` → `dev:senior-developer` → `dev:senior-reviewer` → FIX → RE-REVIEW。

每个 subagent 调用时显式指定 `model` 参数，不依赖 frontmatter。所有产物归档到 `.artifacts/<yyyymmdd>/<任务简述>/`。支持跳过入口：从 plan / build-backend / build-frontend / review 直接开始。修复-审查循环上限 2 轮。

- 命令：`commands/workflow.md`

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

## 六、生命周期

```
DEFINE   → /spec      → dev:spec-driven-development
         → (描述需求)  → dev:interview-me
PLAN     → /plan      → dev:planning-and-task-breakdown
BUILD    → /build     → dev:incremental-implementation
REVIEW   → /review    → dev:code-review-and-quality + dev:security-and-hardening
FIX      → /fix       → 按 review.md 清单逐项修复
RE-REVIEW→ /re-review → 增量验证修复结果
DOC      → /doc       → dev:doc-archiver
```

快捷方式：`/workflow` 提供 dev:senior-developer-planner (opus) → dev:senior-developer-backend (sonnet) → api.md → dev:senior-developer-frontend (sonnet) → dev:senior-reviewer (opus) → fix → re-review 串行编排（前后端分离），或单体项目 dev:senior-developer-planner → dev:senior-developer → dev:senior-reviewer。

---

## 七、文件索引

### 命令定义
| 文件 | 说明 |
|------|------|
| `commands/spec.md` | `/spec` |
| `commands/plan.md` | `/plan` |
| `commands/build.md` | `/build` |
| `commands/review.md` | `/review` |
| `commands/fix.md` | `/fix` |
| `commands/re-review.md` | `/re-review` |
| `commands/doc.md` | `/doc` |
| `commands/workflow.md` | `/workflow` |

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
