# Agent Skills

**AI 编码 Agent 的工程工作流 Skill 集合。**

覆盖 Define → Plan → Build → Review → Fix 全开发阶段。

---

## 双模式流水线

| 命令 | 模式 | 规模 | 流程 |
|------|------|------|------|
| `/rush` | **S** | ≤50 LOC | 全栈工程师 → 初级审查者 → 全栈工程师修复（BUILD → REVIEW → FIX） |
| `/ramble` | **L** | >200 LOC | 策划者 → 构建（支持前后端分离）→ 审查 → 修复（单轮） |

## Agent 发现

```
任务到达
    │
    ├── S 模式 — 小改动/快速修复？────→ /rush
    │         ├── BUILD → dev:full-stack-developer (sonnet)
    │         ├── REVIEW → dev:junior-reviewer (sonnet)
    │         └── FIX → dev:full-stack-developer (sonnet)
    │
    ├── L 模式 — 大型变更/全流程？─────→ /ramble
    │         ├── DEFINE + PLAN → dev:senior-engineer (opus)
    │         ├── BUILD（前后端分离 - 后端）→ dev:backend-developer (sonnet)
    │         │       └── BUILD（前后端分离 - 前端）→ dev:frontend-developer (sonnet)
    │         ├── BUILD（单体）→ dev:full-stack-developer (sonnet)
    │         ├── REVIEW → dev:senior-engineer (opus)
    │         └── FIX → dev:full-stack-developer / dev:backend-developer / dev:frontend-developer (sonnet)
    │
    ├── 仅 DEFINE + PLAN？────────→ dev:senior-engineer (opus)
    ├── 仅 BUILD 后端？──────────→ dev:backend-developer (sonnet)
    ├── 仅 BUILD 前端？──────────→ dev:frontend-developer (sonnet)
    ├── 仅 BUILD 单体？──────────→ dev:full-stack-developer (sonnet)
    ├── 仅 REVIEW（完整）？──────→ dev:senior-engineer (opus)
    └── 仅 REVIEW（单轮）？──────→ dev:junior-reviewer (sonnet)
```

## 单阶段命令

| 命令 | 开发阶段 | 说明 |
|------|----------|------|
| `/spec` | Define | 编写结构化 Spec |
| `/plan` | Plan | 拆分为可验证的小任务 |
| `/build` | Build | 增量实现 |
| `/review` | Review | 五轴代码审查 |
| `/fix` | Fix | 按审查清单修复 |
| `/doc` | Doc | 文档归档整理 |

## 全部 11 个 Skill

### Define — 明确要构建什么

- **interview-me** — 一次一问，挖掘真实意图
- **spec-driven-development** — 编写覆盖六大领域的结构化 Spec

### Plan — 拆分任务

- **planning-and-task-breakdown** — 纵向切片分解，每任务附带验收标准

### Build — 写代码

- **incremental-implementation** — 垂直切片：实现 → 测试 → 验证 → 提交
- **backend-test-generator** — 根据变更自动生成后端测试用例
- **api-doc-generator** — 扫描后端路由/控制器自动生成 API 文档

### Review — 合并前质量门禁

- **code-review-and-quality** — 五轴审查
- **security-and-hardening** — 威胁建模、OWASP 防御、AI/LLM 安全

### Meta

- **using-agent-skills** — Skill 发现与路由，核心行为准则
- **git-commit** — 中文 Conventional Commit 提交
- **doc-archiver** — 按编号目录归档项目文档

## 全部 5 个 Agent

| Agent | model | 职责 |
|-------|-------|------|
| `senior-engineer` | opus | DEFINE + PLAN + REVIEW 全流程 |
| `full-stack-developer` | sonnet | BUILD 全栈实现 |
| `backend-developer` | sonnet | BUILD 后端实现 |
| `frontend-developer` | sonnet | BUILD 前端实现 |
| `junior-reviewer` | sonnet | 单轮 REVIEW（S 模式） |

## 生命周期

```
RUSH    (S) → /rush    → full-stack-developer → junior-reviewer → full-stack-developer
RAMBLE  (L) → /ramble  → senior-engineer → build → senior-engineer → fix
DEFINE    → interview-me, spec-driven-development
PLAN      → planning-and-task-breakdown
BUILD     → incremental-implementation
REVIEW    → code-review-and-quality, security-and-hardening
FIX       → /fix      → 按审查清单逐项修复
DOC       → /doc      → 文档归档
```

## 核心规则

- 如果任务匹配某个 Skill，通过 `Skill` 工具调用
- Skill 位于 `skills/<skill-name>/SKILL.md`
- Skill 是工作流，不是建议——按顺序执行
- 所有面向人类的文档、Spec、计划、任务列表、审查反馈必须使用中文
- 产物统一存放于 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下

## 项目结构

```
skills/                    # 11 个 Skill（每目录一个 SKILL.md）
agents/                    # 5 个专职 Agent
commands/                  # Slash Command 定义
references/                # 安全检查清单
hooks/                     # Session 生命周期 Hooks
docs/                      # skill-anatomy 格式说明 + 快速入门
```

## 参考清单

- [security-checklist.md](references/security-checklist.md) — 认证、授权、输入验证、安全头、CORS、OWASP、AI/LLM

## 许可

MIT
