---
description: 重新审查——验证修复清单中的问题是否已修复
---

调用 dev:senior-reviewer agent 进入 re-review 模式，增量验证修复结果。

**前置条件**：`.artifacts/<yyyymmdd>/<任务简述>/review.md` 存在，包含 `- [ ]` 修复清单；或用户直接指定相关代码或 git 提交历史

**流程：**
1. 读取 review.md 中的修复清单（`- [ ]` 项）
2. 逐项检查文件和行号，确认修复是否到位
3. 已修复项在 review.md 中标注 `- [x]`
4. 未完全修复项追加 `(未完全修复: <原因>)`
5. 检查修复是否引入回归问题
6. 同步更新 todo.md 完成状态（若之前未完成的任务因修复完成)
7. 更新 review.md 和 todo.md 并提交

启动 dev:senior-reviewer（opus），传入：
- `review.md` 路径
- 模式标记：`re-review`
