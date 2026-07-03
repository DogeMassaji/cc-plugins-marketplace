---
name: quick-check
description: Pre-merge sanity check — compile, security scan, convention check. Read-only, no code modification.
---

# 快速预检

## 概述

合并前快速 sanity check。只读模式，不修改任何代码。终端直接输出结果，不生成产物文件。

## 何时使用

- 合并任何 PR 或变更之前
- 提交前快速验证
- 与 `dev:code-review-and-quality` 配合使用——预检通过后再进入完整审查

**何时不使用：** 需要实际修改代码的场景（走 `dev:fix`）。

## 流程

### 1. 读取变更

读取 git diff（已暂存或最近一次提交）。

### 2. 编译检查

检测项目语言，运行编译/类型检查（如适用）。常见命令：

| 语言 | 命令 |
|------|------|
| Go | `go build ./...` |
| TypeScript | `npx tsc --noEmit` |
| Rust | `cargo check` |
| Python | `python -m py_compile <files>` |

### 3. 快速安全扫描

逐文件检查 diff 中的新增行：

- **硬编码密钥/令牌** — 匹配 `api_key\s*=`、`secret\s*=`、`password\s*=`、`token\s*=`、`-----BEGIN.*PRIVATE KEY-----` 等模式
- **SQL/命令注入** — 字符串拼接构建 SQL（`"SELECT.*\+` / `f"SELECT.*{` / `"SELECT.*${`）
- **未验证输入** — 直接从 request/params 取值后未做校验即使用

### 4. 规范检查

- 文件路径是否遵循项目约定
- 是否有明显的不一致命名

## 输出格式

```
file:line: [级别] 问题描述
```

级别：
- 🔴 **阻塞** — 合并前必须修复（密钥泄露、注入漏洞）
- 🟡 **警告** — 建议修复（可能的边界问题、不规范写法）
- ℹ️ **建议** — 可选改进

无问题时输出：`✅ 未发现阻塞问题`

## 规则

1. 只读——不修改任何代码
2. 不生成产物文件——终端直接输出
3. 假阳性优先报告为警告而非阻塞
4. 不检查已有代码——仅检查 diff 新增行
5. 不确定时报警告，不作阻塞

## 验证

- [ ] 编译/类型检查通过
- [ ] 无阻塞级安全问题
- [ ] 命名和路径符合项目约定
