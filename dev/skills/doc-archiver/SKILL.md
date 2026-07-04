---
name: doc-archiver
description: Archives project documents following numbered directory + Chinese filename convention. Supports creating new docs, organizing existing docs, deprecating old docs, and maintaining TODO lists. Triggers: user says "archive docs", "organize docs", "write documentation", "create doc", "deprecate doc", "update TODO".
---

# 文档归档整理器

按标准化目录结构归档项目文档，保文档可发现、可追溯、可维护。

## 目录结构

```
docs/
├── 01-项目概述/        # 项目级文档：构思、架构、TODO
├── 02-技术验证/        # 技术验证、实验、调研
├── 03-设计决策/        # ADR、取舍
├── ...                # 按需新增
└── 99-已废弃/          # 已废弃方案、旧文档
```

**编号：** `01`–`98` 活跃分类，`99` 固定废弃。新分类取最大编号+1。

**命名：** `NN-中文分类名`，2-4 字。

## 文件格式

### 命名

中文描述性文件名，无编号前缀。例：
- `项目构思.md`
- `CDP事件isTrusted验证.md`

### 头部

```markdown
# 标题

> 日期: YYYY-MM-DD | 状态: 草稿/已验证/已废弃
```

### 正文

```markdown
---

## 一、概述/背景

---

## 二、核心内容

---

## 验证/结论
```

- `---` 分大章节
- 中文数字编号大标题
- 表格对比/技术选型
- 代码块标语言
- 末附验证清单

### 废弃

废文头部加废因，移入 `99-已废弃/`：

```markdown
> **废弃日期**: YYYY-MM-DD
> **废弃原因**: 一句话说明
> **替代方案**: 见 `docs/NN-分类/新文档.md`
```

## TODO.md

每个活跃分类目录下可有：

```markdown
# TODO

## YYYY-MM-DD

- [x] 已完成项 (commit_hash)
- [ ] 待办项
```

- 日期倒序，最新在上
- 已完成标 commit hash

## 流程

### Step 1: 确定位置

1. 读 `docs/` 目录树，了解现有分类
2. 判断新文档归属
3. 无匹配则建新编号目录
4. 涉废弃旧方案，同步规划迁移

### Step 2: 创建/更新

1. 按格式写文档
2. 代码引用标路径和行号
3. 外部引用用 markdown 链接
4. 涉技术验证附脚本路径和结果

### Step 3: 更新 TODO

1. 任务相关则更新 TODO.md
2. 已完成标 commit hash
3. 新待办追加当天段

### Step 4: 输出

```
文档归档完成。
- 新增: docs/02-技术验证/xxx.md
- 更新: docs/01-项目概述/TODO.md
- 废弃: docs/99-已废弃/旧方案.md
```

## 要求

1. **编号目录**——所有文档在编号目录下，不散放 `docs/` 根
2. **中文文件名**——用中文描述（代码/技术术语除外）
3. **头部元数据**——每个文档有 `# 标题` 和 `> 日期 | 状态`
4. **废文入 99**——废弃文档移入 `99-已废弃/`，不原地留
5. **TODO 同步**——文档内容涉任务必须在 TODO.md 有对应

## 禁止

1. **禁止英文文件名**——除代码标识符外
2. **禁止无头文档**——缺日期和状态
3. **禁止废文原地留**——不留在活跃目录
4. **禁止空目录**——新分类至少有一个 `.md`
5. **禁止散放**——文档必须归目录

## 验证

- [ ] 文档在编号目录下
- [ ] 每个文档有日期和状态头
- [ ] 文件名中文
- [ ] 废弃文档在 `99-已废弃/` 且标废因
- [ ] TODO.md 已同步
