---
name: git-commit
description: Creates well-formatted git commits following conventional commit standards. Use after completing a task or producing an artifact. Generates commit messages in Chinese.
---

## 用法
```
/commit
```

## 行为
1. `git diff --staged` 分析暂存变更
2. 生成中文 conventional commit message
3. 创建提交

## 格式
```
<type>(<scope>): <中文描述>

[body]

[footer]
```

## 类型
- feat: 新功能
- fix: 修复
- docs: 文档
- style: 代码风格
- refactor: 重构
- test: 测试
- chore: 维护

## 示例
```
feat(auth): 增加密码重置功能

- 增加忘记密码表单
- 实现邮箱验证流程
- 增加密码重置接口
```
