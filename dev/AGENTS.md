# AGENTS.md

本文件为 AI 编码 Agent（Claude Code、Cursor、Copilot 等）提供指导。

## 仓库概述

AI 编码 Agent 的 Skill 集合，覆盖 define, plan, build, review 四个开发阶段。

## Agent 发现

```
任务到达
    │
    ├── 需求 → 代码全流程？──────→ /workflow
    │         ├── DEFINE + PLAN                  → dev:senior-developer-planner (opus)
    │         ├── BUILD（前后端分离 - 后端）      → dev:senior-developer-backend (sonnet)
    │         │       └── BUILD（前后端分离 - 前端）→ dev:senior-developer-frontend (sonnet)
    │         ├── BUILD（单体 / 非前后端分离）     → dev:senior-developer (sonnet)
    │         └── REVIEW                         → dev:senior-reviewer (opus)
    │
    ├── 仅 DEFINE + PLAN？────────→ dev:senior-developer-planner (opus)
    ├── 仅 BUILD 后端？──────────→ dev:senior-developer-backend (sonnet)
    ├── 仅 BUILD 前端？──────────→ dev:senior-developer-frontend (sonnet)
    ├── 仅 BUILD 单体？──────────→ dev:senior-developer (sonnet)
    └── 仅 REVIEW？──────────────→ dev:senior-reviewer (opus)
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
DEFINE → dev:interview-me, dev:spec-driven-development
PLAN   → dev:planning-and-task-breakdown
BUILD  → dev:incremental-implementation
REVIEW → dev:code-review-and-quality, dev:security-and-hardening
```

## 核心规则

- 如果任务匹配某个 Skill，通过 `Skill` 工具调用
- Skill 位于 `skills/<skill-name>/SKILL.md`
- Skill 是工作流，不是建议——按顺序执行
- 所有面向人类的文档、Spec、计划、任务列表、审查反馈必须使用中文
