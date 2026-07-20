---
name: quick-check
description: Pre-merge sanity check — compile, security scan, convention check. Read-only, no code modification.
---

# 快速预检

## 概述

合并前快速 sanity check。只读模式，不修改代码。终端输出，不生成产物。

## 何时用

- 合并 PR 或变更前
- 提交前快速验证
- 配合 `dev:code-review-and-quality`——预检过再进完整审查

**不用时：** 需改代码的场景（走 `dev:diagnose`）。

## 流程

### 1. 读变更

读 git diff（暂存或最近提交）。

### 2. 编译检查

检测项目语言，跑编译/类型检查：

| 语言 | 命令 |
|------|------|
| Go | `go build ./...` |
| TypeScript | `npx tsc --noEmit` |
| Rust | `cargo check` |
| Python | `python -m py_compile <files>` |

### 3. 快速安全扫描

逐文件检 diff 新增行：

- **密钥硬编码** — `api_key\s*=`、`secret\s*=` 等
- **SQL/命令注入** — 字符串拼 SQL（`"SELECT.*\+` / `f"SELECT.*{`）
- **未验证输入** — 从 request/params 取值未校验即用

### 4. 规范检查

- 文件路径遵循约定？
- 命名一致？

## 输出

```
file:line: [级别] 问题
```

级别：
- 🔴 **阻塞** — 合并前必须修（密钥、注入）
- 🟡 **警告** — 建议修
- ℹ️ **建议** — 可选

无问题：`✅ 未发现阻塞问题`

## 规则

1. 只读——不修代码
2. 无产物——终端输出
3. 假阳性报警告非阻塞
4. 仅检 diff 新增行
5. 不确定报警告，不作阻塞

## 验证

- [ ] 编译/类型检查通过
- [ ] 无阻塞级安全问题
- [ ] 命名和路径符合约定
