---
description: 按 review.md 修复清单逐项修复代码
---

修复流程：读取 review.md 修复清单 → 选择对应 dev agent → 逐项修复。

**前置条件**：`.artifacts/<yyyymmdd>/<任务简述>/review.md` 存在，包含 `- [ ]` 修复清单；或用户直接指定清单文件或 bug 描述

**根据项目类型选择修复 agent：**

| 场景 | agent |
|------|-------|
| 单体项目 | `dev:senior-developer`（sonnet） |
| 后端问题 | `dev:senior-developer-backend`（sonnet） |
| 前端问题 | `dev:senior-developer-frontend`（sonnet） |

启动 agent 时传入：
- `review.md` 路径
- `todo.md` 路径
- 指令：读取修复清单，逐项修复代码，不修改 review.md 和 todo.md（checkbox 由 reviewer 更新）
- 每修复一项在 commit message 中标注 `#review`
