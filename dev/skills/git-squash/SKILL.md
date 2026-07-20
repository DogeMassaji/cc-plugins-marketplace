---
name: squash
description: 根据用户要求自动 squash git 提交。支持按数量、按策略合并多个 commit，适用于合并前清理历史。
---

# Squash — Git 提交合并

## 概述

将多个 git commit 合并为一个或一组。自动执行 `git rebase -i` 的 squash 操作。

## 何时用

- 合并 PR 前清理提交历史
- 将 WIP/fixup 提交合并到功能提交
- 将一组小提交合并为有意义的提交

## 流程

### 1. 确认目标

确认用户意图：
- squash 最近的 N 个提交？
- squash 从某个提交开始到 HEAD？
- 按某种策略分组合并？

### 2. 展示当前日志

```
git log --oneline --graph -20
```

展示最近 20 个提交，让用户确认范围。

### 3. 执行 squash

根据用户要求执行：

| 方式 | 命令 | 说明 |
|------|------|------|
| 按数量 | `git rebase -i HEAD~N` | 合并最近 N 个提交 |
| 指定起点 | `git rebase -i <commit>^` | 从某 commit 开始合并 |
| 自动 squash | `git reset --soft HEAD~N && git commit` | 软重置后重新提交 |
| 合并到上游 | `git rebase -i <base>` | 以 base 为根 |

**交互式执行：**
- 用 `git rebase -i` 时自动将指定范围内的 pick 改为 squash
- 保留第一个为 pick，其余改为 squash

**非交互式执行（自动批量）：**
```
GIT_SEQUENCE_EDITOR="sed -i '2,\$s/^pick/squash/'" git rebase -i HEAD~N
```

### 4. 编辑提交信息

- 提供合并后的提交信息
- 默认聚合各提交的 subject
- 允许用户自定义

### 5. 验证

```
git log --oneline -5
```

展示合并后日志，确认结果正确。

## 策略选项

```
/squash last <N>              # 合并最近 N 个提交
/squash from <commit>         # 从某 commit 开始合并到 HEAD
/squash all                   # 合并所有未推送提交（vs upstream）
/squash soft <N> ["message"]  # 软重置 N 个提交，指定提交信息
```

## 安全

1. 如果提交已推送，警告用户需 `--force` 推送
2. 不自动推送——只操作本地
3. 执行前确认用户已备份或可 force push

## 规则

1. 展示 squash 范围——不默默合并
2. 用户确认后才执行 rebase
3. 已推送提交不自动 force push——仅警告
4. squash 后展示结果日志

## 验证

- [ ] 提交数量符合预期
- [ ] 合并后的提交信息正确
- [ ] squash 结果已展示给用户
