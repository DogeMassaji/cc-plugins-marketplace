# Skill 格式说明

本文档描述 my-agent-skills 的 SKILL.md 文件结构和格式，供创建新 Skill 或理解现有 Skill 时参考。

## 文件位置

每个 Skill 位于 `skills/` 下自己的目录中：

```
skills/
  skill-name/
    SKILL.md           # 必需：Skill 定义
    scripts/           # 可选：Skill 工作流使用的可运行辅助脚本
    supporting-file.md # 可选：按需加载的参考材料
```

`SKILL.md` 是唯一需要的文件。只有当 Skill 确实附带可运行辅助工具时才添加 `scripts/`，纯 markdown 的 Skill 完全不需要该目录。

## SKILL.md 格式

### Frontmatter（必需）

```yaml
---
name: skill-name-with-hyphens
description: Guides agents through [task/workflow]. Use when [specific trigger conditions].
---
```

**规则：**
- `name`：小写，连字符分隔。必须匹配目录名。
- `description`：以第三人称描述 Skill 的功能，然后包含一个或多个清晰的 "Use when" 触发条件。涵盖*做什么*和*何时用*。最长 1024 字符。

**为什么重要：** Agent 通过读取描述来发现 Skill。描述会被注入到 system prompt 中，因此必须让 Agent 知道这个 Skill 提供什么、何时激活。不要总结工作流——如果描述包含流程步骤，Agent 可能按照摘要执行而非阅读完整的 Skill。

### 标准章节（推荐模式）

上述 frontmatter 约定是必需的。以下章节布局是推荐模式，非硬性模板：等价标题只要能清楚地达到相同目的，都可以接受。

```markdown
# Skill 标题

## 概述
一两句解释这个 Skill 做什么、为什么重要。

## 何时使用
- 触发条件列表（症状、任务类型）
- 何时不使用（排除条件）

## 核心流程
主要工作流，分解为编号步骤或阶段。
在有用处包含代码示例。
在决策点使用流程图（ASCII）。

## 具体技术 / 模式
特定场景的详细指导。
代码示例、模板、配置。

## 警告信号
- 表明 Skill 被违反的行为模式
- 审查时需要关注的事项

## 验证
完成 Skill 流程后，确认：
- [ ] 退出条件检查清单
- [ ] 证据要求
```

## 章节目的

### 概述
Skill 的"一段话概括"。应回答：这个 Skill 做什么，Agent 为什么应该遵循它？

### 何时使用
帮助 Agent 和人类判断此 Skill 是否适用于当前任务。包含正向触发条件（"When use when X"）和负向排除（"NOT for Y"）。

### 核心流程
Skill 的核心。这是 Agent 遵循的逐步工作流。必须具体且可执行——不是模糊建议。

**好的例子：** "运行 `<项目测试命令>` 并验证所有测试通过"
**坏的例子：** "确保测试没问题"

### 警告信号
可观察的 Skill 被违反的迹象。在代码审查和自我监控时有用。

### 验证
退出标准。Agent 用来确认 Skill 流程已完成的检查清单。每个勾选项应可通过证据验证（测试输出、构建结果、截图等）。

## 辅助文件

仅在以下情况创建辅助文件：
- 参考材料超过 100 行（保持主 SKILL.md 聚焦）
- 需要代码工具或脚本
- 检查清单足够长，值得拆分独立文件

低于 50 行的模式和原则保持内联在 SKILL.md 中。

如果 Skill 不需要可运行辅助工具，不要为了与其他 Skill 保持一致而创建空的 `scripts/` 目录。空目录只增加噪音，不改变 Skill 的工作方式。

## 编写原则

1. **流程优于知识。** Skill 是工作流，不是参考文档。是步骤，不是事实。
2. **具体优于泛泛。** "运行 `<项目测试命令>`"优于"验证测试"。
3. **证据优于假设。** 每个验证勾选项都需要证明。
4. **渐进披露。** 主 SKILL.md 是入口。辅助文件仅在需要时加载。
5. **节省 Token。** 每个章节都必须证明其存在的价值。如果去掉它不会改变 Agent 行为，就去掉它。

## 命名约定

- Skill 目录：`lowercase-hyphen-separated`
- Skill 文件：`SKILL.md`（始终大写）
- 辅助文件：`lowercase-hyphen-separated.md`
- 参考材料：存储在项目根目录的 `references/` 中，不放在 skill 目录内

## 跨 Skill 引用

按名称引用其他 Skill：

```markdown
如果构建失败，先定位并修复根因再继续。
```

不要在不同 Skill 之间重复内容——改为引用和链接。

## 必需 vs 推荐

必需：
- 一个 `skills/<skill-name>/SKILL.md` 文件
- 有效的 YAML frontmatter，含 `name` 和 `description`
- description 同时包含 Skill 做什么和何时使用

推荐：
- 上述标准章节流程
- 等价标题如 `How It Works`、`Core Process` 或 `Workflow` 在更自然时使用
- 仅在辅助文件能保持主 SKILL.md 聚焦时才创建
