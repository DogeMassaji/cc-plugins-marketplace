---
name: using-agent-skills
description: Discovers and invokes agent skills. Use when starting a session or when you need to discover which skill applies to the current task. This is the meta-skill that governs how all other skills are discovered and invoked.
---

# Using Agent Skills

## 概述

Agent Skills 是按开发阶段组织的工作流集合。每 Skill 编码资深工程师的特有流程。本 meta-skill 助你发现和应用正确 Skill。

## Skill 发现

```
任务到达
    ├── 需求不清晰？→ dev:interview-me
    ├── 新项目/功能？→ dev:spec-driven-development
    ├── 有 Spec，需拆分？→ dev:planning-and-task-breakdown
    ├── 实现代码？→ dev:incremental-implementation
    └── 审查代码？→ dev:code-review-and-quality
        └── 安全问题？→ dev:security-and-hardening
```

## 核心行为

所有 Skill 始终适用。

### 0. 中文输出

文档、报告、Spec、计划、审查、commit 用中文。标识符、术语、配置键保留原文。

### 1. 暴露假设

实现前明确陈述假设。不默默填充模糊需求。最常见失败：错假设不验证推进。

### 2. 管理困惑

遇冲突/不一致：停→指困惑点→提权衡/问→等解决再继。不猜。

### 3. 必要时异议

方案明显有问题时，直接指、量化影响、提替代。诚实分歧比假认同更有价值。

### 4. 强制简洁

能少写则少写。不早抽象——第三个用例前不泛化。选无聊显而易见的方案。

### 5. 范围控制

只改任务要求。不做"顺手清理"、不删不完全理解代码、不加 spec 外功能。

### 6. 验证，不假设

每 Skill 有验证步。通过前任务不算完成。需证据，非"看起来对"。

## 生命周期

```
DEFINE → dev:interview-me, dev:spec-driven-development
PLAN   → dev:planning-and-task-breakdown
BUILD  → dev:incremental-implementation
REVIEW → dev:code-review-and-quality, dev:security-and-hardening
```

完整流程：interview-me → spec-driven-development → planning-and-task-breakdown → incremental-implementation → code-review-and-quality。

## Skill 规则

1. 开始前检有否适用 Skill
2. Skill 是工作流非建议——顺序执行，不跳验证
3. 多 Skill 可组合串联
4. 不确定时从 spec 开始
