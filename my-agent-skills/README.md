# Agent Skills

**AI 编码 Agent 的工程工作流 Skill 集合。**

覆盖 Define → Plan → Build → Review 四个开发阶段。

---

## 命令

| 命令 | 阶段 | 说明 |
|------|------|------|
| `/spec` | Define | 编写结构化 Spec |
| `/plan` | Plan | 拆分为可验证的小任务 |
| `/build` | Build | 增量实现 |
| `/review` | Review | 五轴代码审查 |
| `/workflow` | 全流程 | 串行：spec → plan → build |

---

## 全部 7 个 Skill

### Define — 明确要构建什么

- **interview-me** — 一次一问，挖掘真实意图；发散/收敛思维精炼模糊想法
- **spec-driven-development** — 编写覆盖六大领域的结构化 Spec

### Plan — 拆分任务

- **planning-and-task-breakdown** — 纵向切片分解，每任务附带验收标准

### Build — 写代码

- **incremental-implementation** — 垂直切片：实现 → 测试 → 验证 → 提交

### Review — 合并前质量门禁

- **code-review-and-quality** — 五轴审查 + 变更大小控制
- **security-and-hardening** — 威胁建模、OWASP 防御、AI/LLM 安全

### Meta

- **using-agent-skills** — Skill 发现与路由，核心行为准则

---

## 项目结构

```
skills/                    # 7 个 Skill（每目录一个 SKILL.md）
├── interview-me/
├── spec-driven-development/
├── planning-and-task-breakdown/
├── incremental-implementation/
├── code-review-and-quality/
├── security-and-hardening/
└── using-agent-skills/
.claude/commands/          # 5 个 Slash Command
references/                # 安全检查清单
hooks/                     # Session 生命周期 Hooks
docs/                      # skill-anatomy 格式说明
```

## 参考清单

- [security-checklist.md](references/security-checklist.md) — 认证、授权、输入验证、安全头、CORS、OWASP、AI/LLM

## 许可

MIT
