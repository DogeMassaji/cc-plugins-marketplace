---
name: using-agent-skills
description: Discovers and invokes agent skills. Use when starting a session or when you need to discover which skill applies to the current task. This is the meta-skill that governs how all other skills are discovered and invoked.
---

# Using Agent Skills

## 概述

Agent Skills 是按开发阶段组织的工程工作流集合。每个 Skill 编码了资深工程师遵循的特定流程。本 meta-skill 帮助你发现和应用正确的 Skill。

## Skill 发现

任务到达时，识别开发阶段并应用对应 Skill：

```
任务到达
    │
    ├── 需求不清晰？──────────────→ dev:interview-me
    ├── 新项目/新功能/变更？──────→ dev:spec-driven-development
    ├── 有 Spec，需要任务拆分？───→ dev:planning-and-task-breakdown
    ├── 实现代码？───────────────→ dev:incremental-implementation
    └── 审查代码？───────────────→ dev:code-review-and-quality
        └── 安全问题？───────────→ dev:security-and-hardening
```

## 核心行为准则

以下准则在所有 Skill 中始终适用。

### 0. 中文输出

所有面向人类的文档、报告、Spec、计划、任务列表、审查反馈和 commit message 必须使用中文。代码标识符、技术术语和配置键保留原文。

### 1. 暴露假设

在实现之前，明确陈述你的假设。不要默默填充模糊需求。最常见的失败模式是做出错误假设并不加验证地推进。

### 2. 管理困惑

遇到不一致或冲突需求时：停止 → 明确指出困惑点 → 提出权衡或询问 → 等待解决后再继续。不要猜。

### 3. 必要时提出异议

当方案存在明显问题时，直接指出、量化影响、提出替代方案。诚实的技术分歧比虚假的认同更有价值。

### 4. 强制简洁

能用更少行数完成就不要多写。不要过早抽象——第三个用例出现之前不需要泛化。选择无聊、显而易见的方案。

### 5. 保持范围控制

只改任务要求的内容。不做无关的"顺手清理"、不删除不完全理解的代码、不添加 spec 之外的功能。

### 6. 验证，不要假设

每个 Skill 都有验证步骤。验证通过之前任务不算完成。必须有证据，不能只是"看起来正确"。

## 生命周期

```
DEFINE → dev:interview-me, dev:spec-driven-development
PLAN   → dev:planning-and-task-breakdown
BUILD  → dev:incremental-implementation
REVIEW → dev:code-review-and-quality, dev:security-and-hardening
```

完整特性流程：dev:interview-me → dev:spec-driven-development → dev:planning-and-task-breakdown → dev:incremental-implementation → dev:code-review-and-quality

## Skill 规则

1. 开始工作前检查是否有适用的 Skill
2. Skill 是工作流，不是建议——按顺序执行，不要跳过验证
3. 多个 Skill 可以组合使用，按需串联
4. 不确定时，从 spec 开始
