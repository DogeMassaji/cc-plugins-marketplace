---
description: 增量实现下一个任务——编码、验证、提交
---

调用 dev:incremental-implementation skill。

从 `.artifacts/<yyyymmdd>/<任务简述>/todo.md` 中选取下一个待办任务。对每个任务：

1. 阅读任务的验收标准
2. 加载相关上下文（已有代码、模式、类型）
3. 实现代码
4. 运行构建验证编译
5. 用描述性 message 提交
6. 标记任务完成，进入下一个
