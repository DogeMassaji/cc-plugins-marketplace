---
description: 将工作拆分为小而可验证的任务，附带验收标准和依赖排序
---

调用 my-agent-skills:planning-and-task-breakdown skill。

阅读已有 Spec（`.artifacts/<yyyymmdd>/<任务简述>/SPEC.md` 或等效文档）和相关代码库部分。然后：

1. 进入 plan mode——只读，不写代码
2. 识别组件间的依赖图
3. 垂直切片（每个任务一条完整路径，而非水平分层）
4. 编写任务，附带验收标准和验证步骤
5. 在阶段之间添加检查点
6. 提交计划供人工审查

将计划保存到 `.artifacts/<yyyymmdd>/<任务简述>/plan.md`，任务列表保存到 `.artifacts/<yyyymmdd>/<任务简述>/todo.md`。
