# dev

AI 编码 Agent 的工程工作流 Skill，覆盖 define, plan, build, review 四个阶段。

## 项目结构

```
skills/                    # 核心 Skill（每目录一个 SKILL.md）
agents/                    # 自定义 Agent 定义
.claude/commands/          # Slash Command（/spec, /plan, /build, /review, /workflow）
hooks/                     # Session 生命周期 Hooks
references/                # 补充检查清单
```

## Skills by Phase

**Define:** interview-me, spec-driven-development
**Plan:** planning-and-task-breakdown
**Build:** incremental-implementation, api-doc-generator, backend-test-generator
**Review:** code-review-and-quality, security-and-hardening
**Meta:** using-agent-skills, git-commit

## 约定

- 所有面向人类的输出（文档、spec、plan、审查、commit message）使用中文
- 代码标识符、技术术语、配置键保留原文
- 每个 Skill 位于 `skills/<name>/SKILL.md`
- YAML frontmatter 包含 `name` 和 `description`
- 补充参考材料放在 `references/`，不放在 skill 目录内

## 命令

- `bash hooks/session-start-test.sh` — 测试 SessionStart Hook

## 约束

- 始终遵循 skill-anatomy.md 格式创建新 Skill
- 不要添加模糊建议而非可执行流程的 Skill
- 不要在不同 Skill 间重复内容——引用其他 Skill
