# Agent Skills

**AI 编码 Agent 的工程工作流 Skill 集合。**

覆盖 Define → Plan → Build → Test → Review → Fix 全开发阶段。

---

## 三模式流水线

| 命令 | 规模 | 流程 |
|------|------|------|
| `/rush` | ≤100 LOC | BUILD（subagent）→ REVIEW（inline）→ FIX（subagent） |
| `/ramble` | >100 LOC | DEFINE + PLAN（inline）→ BUILD（subagent，支持前后端分离）→ REVIEW（inline）→ FIX（subagent） |
| `/fix` | 不限 | DIAGNOSE（subagent）→ FIX（subagent）→ REVIEW（inline） |

## 阶段执行方式

```
任务到达
    │
    ├── 小改动/快速修复？────────→ /rush
    │         ├── BUILD → subagent: dev:full-stack-developer
    │         ├── REVIEW → inline: dev:review
    │         └── FIX → subagent: dev:full-stack-developer
    │
    ├── 大型变更/全流程？────────→ /ramble
    │         ├── DEFINE + PLAN → inline: dev:spec → dev:plan
    │         ├── BUILD（前后端分离 - 后端）→ subagent: dev:backend-developer
    │         │       └── BUILD（前后端分离 - 前端）→ subagent: dev:frontend-developer
    │         ├── BUILD（单体）→ subagent: dev:full-stack-developer
    │         ├── REVIEW → inline: dev:review
    │         └── FIX → subagent: dev:full-stack-developer / dev:backend-developer / dev:frontend-developer
    │
    ├── 有 bug 需要诊断修复？────────→ /fix
    │         ├── DIAGNOSE → subagent: dev:full-stack-developer
    │         ├── FIX → subagent: dev:full-stack-developer
    │         └── REVIEW → inline: dev:review
    │
    ├── 仅 DEFINE + PLAN？────────→ inline: dev:spec → dev:plan
    ├── 仅 BUILD 后端？──────────→ subagent: dev:backend-developer
    ├── 仅 BUILD 前端？──────────→ subagent: dev:frontend-developer
    ├── 仅 BUILD 单体？──────────→ subagent: dev:full-stack-developer
    └── 仅 REVIEW？──────────────→ inline: dev:review
```

## 单阶段命令

| 命令 | 开发阶段 | 说明 |
|------|----------|------|
| `/doc` | Doc | 文档归档整理 |

## Phase Skills（6 个）

| Skill | 阶段 | 说明 |
|-------|------|------|
| `dev:spec` | DEFINE | 需求 → SPEC.md |
| `dev:plan` | PLAN | SPEC.md → PLAN.md + TODO.md |
| `dev:build` | BUILD | TODO.md → 实现代码 |
| `dev:test` | TEST | 变更 → 测试 + TEST_REPORT.md |
| `dev:review` | REVIEW | diff → REVIEW.md + TODO 更新 |
| `dev:diagnose` | FIX | REVIEW.md → 修复代码 |

## Utility Skills

- **interview-me** — 一次一问，挖掘真实意图
- **spec-driven-development** — 编写覆盖六大领域的结构化 Spec
- **planning-and-task-breakdown** — 纵向切片分解，每个任务附带验收标准
- **incremental-implementation** — 垂直切片：实现 → 测试 → 验证 → 提交
- **backend-test-generator** — 根据变更自动生成后端测试用例
- **api-doc-generator** — 扫描后端路由/控制器自动生成 API 文档
- **code-review-and-quality** — 五轴审查
- **security-and-hardening** — 威胁建模、OWASP 防御、AI/LLM 安全
- **using-agent-skills** — Skill 发现与路由，核心行为准则
- **git-commit** — 中文 Conventional Commit 提交
- **doc-archiver** — 按编号目录归档项目文档
- **quick-check** — 合并前编译/安全/规范预检

## 全部 3 个 Agent

| Agent | model | 职责 |
|-------|-------|------|
| `full-stack-developer` | sonnet | BUILD + FIX（单体） |
| `backend-developer` | sonnet | BUILD + FIX（后端） |
| `frontend-developer` | sonnet | BUILD + FIX（前端） |

## 生命周期

```
RUSH    (S) → /rush    → subagent: full-stack-developer → inline: review → subagent: full-stack-developer
RAMBLE  (L) → /ramble  → inline: spec → plan → subagent: build → inline: review → subagent: fix
PATCH   (FIX) → /fix  → subagent: diagnose → subagent: fix → inline: review
DEFINE    → dev:spec → interview-me, spec-driven-development
PLAN      → dev:plan → planning-and-task-breakdown
BUILD     → dev:build → incremental-implementation
TEST      → dev:test → backend-test-generator
REVIEW    → dev:review → code-review-and-quality, security-and-hardening
FIX       → dev:diagnose
DOC       → /doc    → 文档归档
```

## 核心规则

- Skill 是工作流，不是建议——按顺序执行
- 所有面向人类的文档、Spec、计划、任务列表、审查反馈必须使用中文
- 产物统一存放于 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下

## 项目结构

```
skills/                    # 17 个 Skill（6 phase + 11 utility）
agents/                    # 3 个专职 Agent（BUILD/FIX）
commands/                  # 4 个 Slash Command
references/                # 安全检查清单
hooks/                     # Session 生命周期 Hooks
```

## 参考清单

- [security-checklist.md](references/security-checklist.md) — 认证、授权、输入验证、安全头、CORS、OWASP、AI/LLM

## 许可

MIT
