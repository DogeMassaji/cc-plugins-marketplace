---
description: 五轴代码审查——正确性、可读性、架构、安全性、性能
---

调用 dev:code-review-and-quality skill。

审查当前变更（已暂存或最近提交），覆盖全部五个维度：

1. **正确性**——匹配 spec 吗？边界情况覆盖了吗？测试充分吗？
2. **可读性**——命名清晰吗？逻辑直接吗？结构合理吗？
3. **架构**——遵循已有模式吗？模块边界干净吗？抽象层级合适吗？
4. **安全性**——输入验证了吗？密钥安全吗？认证检查到位吗？（使用 dev:security-and-hardening skill）
5. **性能**——有 N+1 查询吗？有无界操作吗？

将发现分为 Critical、Important、Suggestion 三级。
输出结构化审查报告到 `.artifacts/<yyyymmdd>/<任务简述>/review.md`，以及在 `.artifacts/<yyyymmdd>/<任务简述>/todo.md` 标记已完成项，附带具体的 file:line 引用和修复建议。
