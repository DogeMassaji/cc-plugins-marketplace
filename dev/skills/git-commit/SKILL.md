---
name: git-commit
description: Creates well-formatted git commits following conventional commit standards. Use after completing a task or producing an artifact. Generates commit messages in Chinese.
---

## 用法
```
/commit
```

## 行为
1. 通过 `git diff --staged` 分析暂存的变更
2. 生成中文 conventional commit message
3. 以正确格式创建提交

## 提交格式
```
<type>(<scope>): <中文描述>

[optional body]

[optional footer]
```

## 类型
- feat: 新功能
- fix: 缺陷修复
- docs: 文档变更
- style: 代码风格变更
- refactor: 代码重构
- test: 测试补充或修改
- chore: 维护任务

## 示例输出
```
feat(auth): 增加密码重置功能

- 增加忘记密码表单
- 实现邮箱验证流程
- 增加密码重置接口
```