# AGENTS.md

本文件为 AI 编码 Agent（Claude Code、Cursor、Copilot 等）提供指导。

## 仓库概述

AI 编码 Agent 的 Skill 集合，覆盖 define, plan, build, review 四个开发阶段。

## 三模式流水线

| 模式 | 命令 | 规模 | 流程 |
|------|------|------|------|
| **S** | `/popping` | ≤50 LOC | 全栈工程师 → 初级审查者 → 全栈工程师修复（BUILD → REVIEW → FIX，单轮） |
| **M** | `/locking` | 50-200 LOC | 策划者 → 全栈工程师 → 初级审查者 → 全栈工程师修复（SPEC → PLAN → BUILD → REVIEW → FIX，单轮） |
| **L** | `/breaking` | >200 LOC | 策划者 → 构建（单体/前后端分离）→ 审查者 → 修复（单轮） |

## Agent 发现

```
任务到达
    │
    ├── S 模式 — 小改动/快速修复？────→ /popping
    │         ├── BUILD → dev:full-stack-developer (sonnet)
    │         ├── REVIEW → dev:junior-reviewer (sonnet)
    │         └── FIX → dev:full-stack-developer (sonnet)
    │
    ├── M 模式 — 标准变更？────────────→ /locking
    │         ├── DEFINE + PLAN → dev:senior-engineer (opus)
    │         ├── BUILD → dev:full-stack-developer (sonnet)
    │         ├── REVIEW → dev:junior-reviewer (sonnet)
    │         └── FIX → dev:full-stack-developer (sonnet)
    │
    ├── L 模式 — 大型变更/全流程？─────→ /breaking
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

## Skill 发现

```
任务到达
    │
    ├── 需求不清晰？──────────────→ dev:interview-me
    ├── 新项目/新功能/变更？──────→ dev:spec-driven-development
    ├── 有 Spec，需要任务拆分？───→ dev:planning-and-task-breakdown
    ├── 实现代码？───────────────→ dev:incremental-implementation
    ├── 生成后端测试？───────────→ dev:backend-test-generator
    ├── 生成接口文档？───────────→ dev:api-doc-generator
    └── 审查代码？───────────────→ dev:code-review-and-quality
        └── 安全问题？───────────→ dev:security-and-hardening
```

## 生命周期

```
POPPING  (S) → /popping  → full-stack-developer → junior-reviewer → full-stack-developer
LOCKING  (M) → /locking  → senior-engineer → full-stack-developer → junior-reviewer → full-stack-developer
BREAKING (L) → /breaking → senior-engineer → build(单体/fullstack) → senior-engineer → fix
DEFINE    → dev:interview-me, dev:spec-driven-development
PLAN      → dev:planning-and-task-breakdown
BUILD     → dev:incremental-implementation
REVIEW    → dev:code-review-and-quality, dev:security-and-hardening
CHECK     → /check    → 只读快速扫描（编译+安全+规范）
FIX       → /fix      → 按 REVIEW.md 修复清单逐项修复
DOC       → /doc      → dev:doc-archiver
```

## 核心规则

- 如果任务匹配某个 Skill，通过 `Skill` 工具调用
- Skill 位于 `skills/<skill-name>/SKILL.md`
- Skill 是工作流，不是建议——按顺序执行
- 所有面向人类的文档、Spec、计划、任务列表、审查反馈必须使用中文
