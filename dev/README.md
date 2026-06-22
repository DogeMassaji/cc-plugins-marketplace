# Agent Skills

**AI 编码 Agent 的工程工作流 Skill 集合。**

覆盖 Define → Plan → Build → Review → Fix → Re-Review 全开发阶段。

---

## 命令

| 命令 | 阶段 | 说明 |
|------|------|------|
| `/spec` | Define | 编写结构化 Spec |
| `/plan` | Plan | 拆分为可验证的小任务 |
| `/build` | Build | 增量实现 |
| `/review` | Review | 五轴代码审查 |
| `/fix` | Fix | 按审查清单修复 |
| `/re-review` | Re-Review | 验证修复结果 |
| `/doc` | Doc | 文档归档整理 |
| `/workflow` | 全流程 | 串行：planner → backend → frontend → reviewer → fix → re-review |

---

## 全部 11 个 Skill

### Define — 明确要构建什么

- **interview-me** — 一次一问，挖掘真实意图；发散/收敛思维精炼模糊想法
- **spec-driven-development** — 编写覆盖六大领域的结构化 Spec

### Plan — 拆分任务

- **planning-and-task-breakdown** — 纵向切片分解，每任务附带验收标准

### Build — 写代码

- **incremental-implementation** — 垂直切片：实现 → 测试 → 验证 → 提交
- **backend-test-generator** — 根据变更自动生成后端测试用例
- **api-doc-generator** — 扫描后端路由/控制器自动生成 API 文档

### Review — 合并前质量门禁

- **code-review-and-quality** — 五轴审查 + 变更大小控制
- **security-and-hardening** — 威胁建模、OWASP 防御、AI/LLM 安全

### Meta

- **using-agent-skills** — Skill 发现与路由，核心行为准则
- **git-commit** — 中文 Conventional Commit 提交
- **doc-archiver** — 按编号目录归档项目文档

---

## 项目结构

```
skills/                    # 11 个 Skill（每目录一个 SKILL.md）
├── interview-me/
├── spec-driven-development/
├── planning-and-task-breakdown/
├── incremental-implementation/
├── backend-test-generator/
├── api-doc-generator/
├── code-review-and-quality/
├── security-and-hardening/
├── using-agent-skills/
├── git-commit/
└── doc-archiver/
commands/                  # 8 个 Slash Command
references/                # 安全检查清单
hooks/                     # Session 生命周期 Hooks
agents/                    # 5 个专职 Agent（planner, developer, backend, frontend, reviewer）
docs/                      # skill-anatomy 格式说明 + 快速入门
```

## 参考清单

- [security-checklist.md](references/security-checklist.md) — 认证、授权、输入验证、安全头、CORS、OWASP、AI/LLM

## 许可

MIT
