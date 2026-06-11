# dev 精简参考

命令、Skill、Hook 说明。

---

## 一、命令总览（5 个）

| 命令 | 开发阶段 | 核心 Skill | 辅助 Skill |
|------|----------|-----------|-----------|
| `/spec` | Define | `spec-driven-development` | — |
| `/plan` | Plan | `planning-and-task-breakdown` | — |
| `/build` | Build | `incremental-implementation` | — |
| `/review` | Review | `code-review-and-quality` | `security-and-hardening` |
| `/workflow` | 全流程 | 串行编排：planner (opus) → backend (sonnet) → api.md → frontend (sonnet) → reviewer (opus) | 前后端分离时后端先完成再前端 |

---

## 二、命令详情

### `/spec` — 编写结构化 Spec 说明

1. 向用户提问，澄清：目标用户、核心功能与验收标准、技术栈偏好与约束、边界条件
2. 生成涵盖六大领域的结构化 Spec 说明
3. 保存到 `.artifacts/<yyyymmdd>/<任务简述>/spec.md`，与用户确认

- 命令：`.claude/commands/spec.md`
- Skill：`skills/spec-driven-development/SKILL.md`

### `/plan` — 将工作拆分为可验证的小任务

1. 读取 Spec，进入 Plan Mode（只读）
2. 垂直切片，每任务附带验收标准
3. 保存到 `.artifacts/<yyyymmdd>/<任务简述>/plan.md` + `todo.md`

- 命令：`.claude/commands/plan.md`
- Skill：`skills/planning-and-task-breakdown/SKILL.md`

### `/build` — 增量实现

1. 从 `.artifacts/<yyyymmdd>/<任务简述>/todo.md` 取下一个任务
2. 实现代码 → 运行构建验证 → 提交
3. 失败时输出错误，等待用户指导

- 命令：`.claude/commands/build.md`
- Skill：`skills/incremental-implementation/SKILL.md`

### `/review` — 五轴代码审查

正确性、可读性、架构、安全性、性能。输出 Critical / Important / Suggestion 分类报告到 `.artifacts/<yyyymmdd>/<任务简述>/review.md`。

- 命令：`.claude/commands/review.md`
- Skill：`skills/code-review-and-quality/SKILL.md`

### `/workflow` — 串行全流程流水线

前后端分离：`senior-developer-planner` (opus: DEFINE → PLAN) → `senior-developer-backend` (sonnet: BUILD 后端) → 产出 `api.md` → `senior-developer-frontend` (sonnet: BUILD 前端 → 消费 api.md) → `senior-reviewer` (opus: REVIEW)。后端先完成并生成接口文档，前端基于真实文档实现。

单体项目：`senior-developer-planner` → `senior-developer` → `senior-reviewer`。

每个 subagent 调用时显式指定 `model` 参数，不依赖 frontmatter。所有产物归档到 `.artifacts/<yyyymmdd>/<任务简述>/`。支持跳过入口：从 plan / build-backend / build-frontend / review 直接开始。

- 命令：`.claude/commands/workflow.md`

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

## 四、完整 Skill 清单（9 个）

| Skill | 阶段 | 命令 | 说明 |
|-------|------|------|------|
| `interview-me` | Define | — | 需求访谈 + 想法精炼 |
| `spec-driven-development` | Define | `/spec` | 结构化 Spec 说明 |
| `planning-and-task-breakdown` | Plan | `/plan` | 垂直切片，任务拆分 |
| `incremental-implementation` | Build | `/build` | 逐个任务增量实现 |
| `backend-test-generator` | Build | — | 根据变更自动生成后端测试用例并执行 |
| `api-doc-generator` | Build | — | 扫描后端路由/控制器自动生成 API 接口文档 |
| `code-review-and-quality` | Review | `/review` | 五轴审查 |
| `security-and-hardening` | Review | `/review` 安全维度 | OWASP 防护、威胁建模、AI/LLM |
| `using-agent-skills` | Meta | session 自动注入 | Skill 发现与路由，核心行为规范 |

---

## 五、Hooks（2 个）

| Hook | 文件 | 触发 | 用途 |
|------|------|------|------|
| SessionStart | `hooks/session-start.sh` | session 启动 | 注入 `using-agent-skills` |
| SessionStart Test | `hooks/session-start-test.sh` | 手动 | 验证 SessionStart payload |

---

## 六、生命周期

```
DEFINE → /spec   → spec-driven-development
       → (描述需求)  → interview-me
PLAN   → /plan   → planning-and-task-breakdown
BUILD  → /build  → incremental-implementation
REVIEW → /review → code-review-and-quality + security-and-hardening
```

快捷方式：`/workflow` 提供 senior-developer-planner (opus) → senior-developer-backend (sonnet) → api.md → senior-developer-frontend (sonnet) → senior-reviewer (opus) 串行编排（前后端分离），或单体项目 senior-developer-planner → senior-developer → senior-reviewer。

---

## 七、文件索引

### 命令定义
| 文件 | 说明 |
|------|------|
| `.claude/commands/spec.md` | `/spec` |
| `.claude/commands/plan.md` | `/plan` |
| `.claude/commands/build.md` | `/build` |
| `.claude/commands/review.md` | `/review` |
| `.claude/commands/workflow.md` | `/workflow` |

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
