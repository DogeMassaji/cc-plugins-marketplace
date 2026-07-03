# Getting Started with dev

agent-skills works with any AI coding agent that accepts Markdown instructions. This guide covers the universal approach. For tool-specific setup, see the dedicated guides.

## How Skills Work

Each skill is a Markdown file (`SKILL.md`) that describes a specific engineering workflow. When loaded into an agent's context, the agent follows the workflow — including verification steps and exit criteria.

**Skills are not reference docs.** They're step-by-step processes the agent follows.

## Quick Start (Any Agent)

### 1. Clone the repository

```bash
git clone https://github.com/DogeMassaji/cc-plugins-marketplace.git
```

### 2. Choose a skill

Browse the `skills/` directory. Each subdirectory contains a `SKILL.md` with:
- **When to use** — triggers that indicate this skill applies
- **Process** — step-by-step workflow
- **Verification** — how to confirm the work is done
- **Red flags** — signs the skill is being violated

### 3. Load the skill into your agent

Copy the relevant `SKILL.md` content into your agent's system prompt, rules file, or conversation. The most common approaches:

**System prompt:** Paste the skill content at the start of the session.

**Rules file:** Add skill content to your project's rules file (CLAUDE.md, .cursorrules, etc.).


### 4. Use the meta-skill for discovery

Start with the `dev:using-agent-skills` skill loaded. It contains a flowchart that maps task types to the appropriate skill.

## Recommended Setup

### Minimal (Start here)

Load three essential skills into your rules file:

1. **dev:spec-driven-development** — For defining what to build
3. **dev:code-review-and-quality** — For verifying quality before merge

These three cover the most critical quality gaps in AI-assisted development.

### Full Lifecycle

For comprehensive coverage, load skills by phase:

```
Starting a project:  dev:spec-driven-development → dev:planning-and-task-breakdown
Before merge:        dev:code-review-and-quality + dev:security-and-hardening
```

### Context-Aware Loading

Don't load all skills at once — it wastes context. Load skills relevant to the current task.

## Skill Anatomy

Every skill follows the same structure:

```
YAML frontmatter (name, description)
├── Overview — What this skill does
├── When to Use — Triggers and conditions
├── Core Process — Step-by-step workflow
├── Examples — Code samples and patterns
├── Red Flags — Signs the skill is being violated
└── Verification — Exit criteria checklist
```

See [skill-anatomy.md](skill-anatomy.md) for the full specification.

## Using Commands

The `commands/` directory contains slash commands for Claude Code:

| Command | Skill Invoked |
|---------|---------------|
| `/rush` | S 模式 — 全栈工程师 → 初级审查者 → 全栈工程师修复 |
| `/ramble` | L 模式 — 全流程编排 |
| `/spec` | dev:spec-driven-development |
| `/plan` | dev:planning-and-task-breakdown |
| `/build` | dev:incremental-implementation |
| `/review` | dev:code-review-and-quality |
| `/fix` | dev:fix |
| `/doc` | dev:doc-archiver |

## Using References

The `references/` directory contains supplementary checklists:

| Reference | Use With |
|-----------|----------|
| `security-checklist.md` | dev:security-and-hardening |

Load a reference when you need detailed patterns beyond what the skill covers.

## Spec and task artifacts

The `/spec`、`/plan`、`/review` 和 `/ramble` 命令将产物保存到 `.artifacts/<yyyymmdd>/<任务简述>/` 目录下：

```
.artifacts/
  └── <yyyymmdd>/<任务简述>/
      ├── SPEC.md      # 结构化规格说明
      ├── PLAN.md      # 实现计划
      ├── TODO.md      # 任务列表
      └── REVIEW.md    # 审查报告
```

将产物视为开发过程中的**活文档**：

- 在开发期间纳入版本控制，使人和 Agent 共享一份事实来源。
- 当范围或决策变更时及时更新。
- 如果仓库不需要长期保留，合并前删除或将 `.artifacts/` 加入 `.gitignore`。

## Tips

1. **Start with dev:spec-driven-development** for any non-trivial work
3. **Don't skip verification steps** — they're the whole point
4. **Load skills selectively** — more context isn't always better
5. **Use the agents for review** — different perspectives catch different issues
