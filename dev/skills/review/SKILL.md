---
name: review
description: REVIEW 阶段——多维度代码审查。TODO 状态检查 → quick-check 预检 → 五轴审查 → 安全加固。输出 REVIEW.md + 更新 TODO.md。触发：review、代码审查、code review、审查。
---

# Review — 代码审查

## 概述

对变更代码执行多维度审查：TODO 状态检查、编译/安全预检、五轴审查、安全深度分析。输出结构化 REVIEW.md 含修复清单。

## 何时用

- BUILD 完成后，合并前
- 用户说"review"、"代码审查"、"code review"、"审查"
- 其他 agent 产出代码需评估

## 流程

### 1. 读上下文

- 读 git diff / 最近提交变更
- 若有 `SPEC.md`，读取对比需求
- 若有 TODO.md（或 TODO_BACKEND.md / TODO_FRONTEND.md），确认任务完成状态

### 2. TODO 状态检查

遍历 TODO 每项，对照 git diff 判断完成状态，在 REVIEW.md 写入状态表：

```markdown
## TODO 状态
| 任务 | 状态 | 备注 |
|------|------|------|
| 任务 1 | ✅ 已实现 | 符合预期 |
| 任务 2 | ⚠️ 部分实现 | 缺边界处理 |
| 任务 3 | ❌ 未实现 | spec 中有但未编码 |
```

**同步更新原始 TODO.md** 每项前标注 `[x]` / `[ ]`。

### 3. 快速预检

跑 `dev:quick-check` — 编译检查、安全扫描、规范检查。阻塞项先修复再进审查。

### 4. 五轴审查

跑 `dev:code-review-and-quality`，覆盖：

| 维度 | 检查要点 |
|------|----------|
| 正确性 | 匹配 Spec、边界、错误路径 |
| 可读性 | 命名、控制流、代码组织、死代码 |
| 架构 | 模式一致性、模块边界、依赖方向 |
| 安全性 | 输入验证、密钥暴露、注入风险 |
| 性能 | N+1 查询、不必要重计算、内存问题 |

### 5. 安全问题升级

发现安全问题时，跑 `dev:security-and-hardening` 深度分析。

### 6. 生成修复清单

问题分类：

| 级别 | 含义 |
|------|------|
| Critical | 必须修复——bug、漏洞、数据丢失 |
| Important | 强烈建议——边界缺失、错误处理 |
| Suggestion | 可选改进——简化、命名、重构 |

每项必含：文件路径、行号、问题描述、严重级别。

### 7. 输出审查报告

保存至 `.artifacts/<yyyymmdd>/<任务简述>/REVIEW.md`：

```markdown
# Review: <任务简述>

## TODO 状态
...

## 审查发现
...

## 修复清单
### Critical
- [ ] `path:line` — 描述

### Important
- [ ] `path:line` — 描述

### Suggestion
- [ ] `path:line` — 描述

## 裁决
- [ ] 批准 — 无修复项
- [x] 请求修改 — 存在待修复项
```

### 8. 提交

跑 `dev:git-commit`，提交 REVIEW.md + 更新的 TODO.md，message：`docs: add review for <任务简述>`

## 产物

```
.artifacts/<yyyymmdd>/<任务简述>/
├── REVIEW.md         ← review 输出
└── TODO.md           ← 更新状态标记
```

## 规则

1. **单轮**——只做一轮审查，不循环
2. **全面覆盖**——五维度均检查，不跳过
3. **证据导向**——每发现必引文件+行号
4. **分级准确**——Critical 仅用于 bug、漏洞、数据丢失
5. **安全优先**——疑虑即时升级 `dev:security-and-hardening`
6. **清单可执行**——每 `- [ ]` 项是 dev agent 可直接定位修复的具体问题
7. **TODO 属实**——对照代码标注，不凭提交记录

## 验证

- [ ] TODO 状态已检查并更新
- [ ] 五轴审查全部覆盖
- [ ] 修复清单含文件路径+行号
- [ ] 裁决明确
- [ ] 已提交 REVIEW.md
