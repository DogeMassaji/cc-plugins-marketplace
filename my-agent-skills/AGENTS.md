# AGENTS.md

本文件为 AI 编码 Agent（Claude Code、Cursor、Copilot 等）提供指导。

## 仓库概述

AI 编码 Agent 的 Skill 集合，覆盖 define, plan, build, review 四个开发阶段。

## Skill 发现

```
任务到达
    │
    ├── 需求不清晰？──────────────→ interview-me
    ├── 新项目/新功能/变更？──────→ spec-driven-development
    ├── 有 Spec，需要任务拆分？───→ planning-and-task-breakdown
    ├── 实现代码？───────────────→ incremental-implementation
    └── 审查代码？───────────────→ code-review-and-quality
        └── 安全问题？───────────→ security-and-hardening
```

## 生命周期

```
DEFINE → interview-me, spec-driven-development
PLAN   → planning-and-task-breakdown
BUILD  → incremental-implementation
REVIEW → code-review-and-quality, security-and-hardening
```

## 核心规则

- 如果任务匹配某个 Skill，通过 `Skill` 工具调用
- Skill 位于 `skills/<skill-name>/SKILL.md`
- Skill 是工作流，不是建议——按顺序执行
- 所有面向人类的文档、Spec、计划、任务列表、审查反馈必须使用中文
