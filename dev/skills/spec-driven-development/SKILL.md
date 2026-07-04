---
name: spec-driven-development
description: Creates specs before coding. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea.
---

# Spec 驱动开发

## 概述

写代码前写结构化 Spec。Spec 是你和工程师间共享真理——定义建什么、为何建、如何算完成。无 spec 代码即猜测。

## 何时用

- 开始新项目/功能
- 需求模糊不完整
- 变更涉多文件/模块
- 即将做架构决策
- 任务预估 >30 分钟

**不用时：** 单行修复、拼写纠正、需求明确的独立变更。

## 分阶段门控流程

四阶段。当前阶段验证通过前不进下一。

```
SPECIFY → PLAN → TASKS → IMPLEMENT
   ↓       ↓       ↓       ↓
 人工审   人工审   人工审   人工审
```

### 阶段 1：Specify

从高层愿景开始，向人提问澄清直到需求具体。

**立即暴露假设。** 写 spec 内容前列出：

```
我当前假设：
1. 这是 Web（非原生）
2. 认证用 session cookie（非 JWT）
3. 数据库 PostgreSQL（基于 Prisma schema）
4. 目标现代运行时
→ 现在纠正我，否则基于这些继续。
```

不默默填充模糊需求。Spec 目的就是写代码*前*揭示误解——假设最危险。

**覆盖六大核心领域：**

1. **目标** — 建什么，为何？用户是谁？成功什么样？
2. **命令** — 完整命令：
   ```
   Build: <构建命令>
   Test: <测试命令>
   Lint: <lint 命令>
   Dev: <开发命令>
   ```
3. **项目结构** — 源码、测试、文档位置。
4. **代码风格** — 真实示例展示风格，胜过三段描述。含命名、格式、好输出样例。
5. **测试策略** — 框架、位置、覆盖率、级别。
6. **边界** — 三层：
   - **Always do:** 提交前跑测试、遵循命名、验证输入
   - **Ask first:** DB schema 变更、加依赖、改 CI
   - **Never do:** 提交密钥、编 vendor、删失败测试无批准

**Spec 模板：**

```markdown
# Spec: [名称]

## 目标

## 技术栈

## 命令
[build、test、lint、dev]

## 项目结构
[目录布局]

## 代码风格
[示例 + 约定]

## 测试策略
[框架、位置、覆盖率]

## 边界
- Always: [...]
- Ask first: [...]
- Never: [...]

## 成功标准
[具体可测条件]

## 待定问题
[需人输入]
```

**将指令重构为成功标准。** 模糊需求转具体条件：

```
需求："让 dashboard 更快"

成功标准：
- 首次内容绘制 < 2.5s（4G）
- 初始数据加载 < 500ms
- 无布局偏移（CLS < 0.1）
→ 这些正确？
```

这使你循环直到明确，而非猜"更快"。

### 阶段 2：Plan

基于已验证 Spec，生技术方案：
1. 识别主组件及依赖
2. 定实现顺序
3. 标风险和缓解策略
4. 识别可并行 vs. 必须串行
5. 定阶段间检查点

计划应可审查：人能读并说"对"或"改 X"。

### 阶段 3：Tasks

计划拆离散可实施任务：
- 每个任务单 session 完成
- 每个任务有验收标准
- 每个任务含验证步骤
- 按依赖排序，非感知重要
- 无任务涉 >~5 文件

**模板：**
```markdown
- [ ] 任务: [描述]
  - 验收: [完成时必须真]
  - 验证: [命令、构建、手动]
  - 文件: [哪些改动]
```

### 阶段 4：Implement

一次一任务，按 `dev:incremental-implementation`。

## 保持 Spec 更新

活文档，非一次性：
- **决策变则更新** — 先改 spec 再实现
- **范围变则更新** — 新增移除反映在 spec
- **提交 spec** — 与代码同版控
- **PR 中引用** — 链接到 spec 部分

## 警告

- 无书面需求就开始代码
- "直接开始建"但"完成"含义不清
- 实现 spec/task list 未提功能

## 验证

进实现前确认：
- [ ] Spec 覆盖六大核心领域
- [ ] 用户已审查批准
- [ ] 成功标准具体可测
- [ ] 边界（Always/Ask/Never）已定义
- [ ] Spec 已存到仓库文件
