---
name: plan
description: PLAN 阶段——将 Spec 拆解为可执行任务列表。自动检测前后端分离，生成 PLAN.md + TODO.md。触发：plan、任务拆分、计划、task breakdown。
---

# Plan — 任务规划

## 概述

读 SPEC.md，判断项目类型，拆解为带验收标准和依赖排序的任务列表。不写代码——只产出 Plan 文档。

## 何时用

- SPEC.md 已存在，用户说"plan"、"任务拆分"、"计划"
- 已有需求文档，需拆解为可执行任务

## 流程

### 1. 读 Spec

读 `.artifacts/<yyyymmdd>/<任务简述>/SPEC.md`，理解需求和约束。

### 2. 判断前后端分离

扫描项目结构，检查以下信号：

- 存在 `frontend/` `backend/` `client/` `server/` `web/` `api/` 等独立目录
- 各自独立包管理文件（如 `frontend/package.json` + `backend/go.mod`）
- monorepo 结构（`packages/frontend` + `packages/backend`）
- CLAUDE.md 或 README 中描述的分离架构

### 3. 任务拆解

跑 `dev:planning-and-task-breakdown`：

- 识别组件依赖图
- 垂直切片拆解，每项附验收标准
- 关键节点设置检查点
- 按依赖排序，非感知重要性
- 单任务 ≤~5 文件

### 4. 输出产物

**非分离：**

- `PLAN.md`（整体技术方案 + 架构决策）
- `TODO.md`（任务列表，含验收标准和依赖）

**前后端分离：**

- `PLAN.md`（整体方案 + 前后端子方案）
- `TODO_BACKEND.md`（后端任务，按依赖排序）
- `TODO_FRONTEND.md`（前端任务，按依赖排序）
- 跨端联调任务放入 TODO_BACKEND.md 末尾或最后完成端

### 5. 提交

跑 `dev:git-commit`，message：`docs: add plan for <任务简述>`

## 产物

```
.artifacts/<yyyymmdd>/<任务简述>/
├── SPEC.md             ← 来自 define
├── PLAN.md             ← plan 输出
├── TODO.md             ← 非分离
└── TODO_BACKEND.md     ← 分离
└── TODO_FRONTEND.md    ← 分离
```

## 规则

1. **Spec 驱动**——先读 SPEC.md 再规划
2. **自动检测分离**——扫描项目结构，不靠用户说
3. **依赖排序**——按依赖，非感知重要
4. **任务可控**——单任务 ≤~5 文件，附验收标准
5. **不写代码**——只产出文档

## 验证

- [ ] 前后端分离判定已记录
- [ ] 任务列表按依赖排序
- [ ] 每项含验收标准
- [ ] 已提交到产物目录
