---
name: fix
description: FIX 阶段——按 REVIEW.md 或 DIAGNOSE.md 修复清单逐项修复。按严重级排序，每项定位→修复→验证→提交。不修改 REVIEW.md 或 TODO.md。触发：fix、修复、fix review issues。
---

# Fix — 问题修复

## 概述

读 REVIEW.md 或 DIAGNOSE.md 中的修复项，按严重级逐项修复、验证、提交。严格按清单执行——不扩大范围。

## 何时用

- REVIEW.md 已存在，含 `- [ ]` 修复清单
- DIAGNOSE.md 已存在（fix 命令诊断后）
- 用户说"fix"、"修复"、"fix review issues"

## 入口

| 入口 | 输入 | 场景 |
|------|------|------|
| 审查修复 | REVIEW.md | ramble/rush 流水线 REVIEW 后 |
| 诊断修复 | DIAGNOSE.md | fix 命令 DIAGNOSE 后 |

两种入口处理逻辑相同：提取修复项 → 排序 → 修复。

## 流程

### 1. 读修复文档

- 读 `REVIEW.md`，提取 `- [ ]` 清单项
- 或读 `DIAGNOSE.md`，提取修复方案

### 2. 排序

按严重级别排序：Critical → Important → Suggestion

### 3. 逐项修复

每项：

1. **定位**——按文件路径和行号找到问题代码
2. **修复**——应用修复
3. **验证**——编译检查 + 跑已有测试确认不破坏
4. **提交**——跑 `dev:git-commit`

提交 message 规范：

| 场景 | message |
|------|---------|
| 审查修复（单体） | `fix: <问题简述> #review` |
| 审查修复（后端） | `fix(backend): <问题简述> #review` |
| 审查修复（前端） | `fix(frontend): <问题简述> #review` |
| 诊断修复 | `fix: <问题简述> #fix` |

### 4. 汇报

- 已修复项（附路径）
- 无法修复项及原因
- 提交记录

## 重要

**不修改 REVIEW.md 和 TODO.md**——checkbox 由 reviewer 更新。

## 规则

1. **清单即契约**——REVIEW.md `- [ ]` 是修复唯一依据，不自行扩大范围
2. **按级排序**——Critical → Important → Suggestion
3. **逐项验证**——每项修复后编译+测试
4. **不越界**——不更新 REVIEW.md 或 TODO.md（reviewer 负责）
5. **无法修复汇报**——无法处理的项目标注原因，不默默跳过

## 验证

- [ ] 所有修复项已处理或标注无法修复
- [ ] 编译/类型检查通过
- [ ] 已有测试通过（无回归）
- [ ] 提交记录完整
