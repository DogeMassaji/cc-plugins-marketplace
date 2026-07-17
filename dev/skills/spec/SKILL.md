---
name: spec
description: DEFINE 阶段——从模糊需求到结构化 Spec。需求不清晰时先 interview，再跑 spec-driven-development 生成 SPEC.md。触发：spec、define、需求定义、写 spec。
---

# Define — 需求定义

## 概述

将用户需求转化为结构化 SPEC.md。需求模糊时先澄清再写 spec。不写代码、不拆任务——只产出 Spec 文档。

## 何时用

- 开始新项目/功能，需求存在但无 SPEC.md
- 用户说"define"、"spec"、"需求定义"、"写 spec"
- 需求模糊不完整，需先澄清

## 流程

### 1. 判断需求清晰度

置信度 < 95%（需求缺用户/目的/成功标准/约束）：

- 跑 `dev:interview-me`，一次一个问题澄清
- 直到覆盖：用户是谁、要做什么、为什么现在做、成功标准是什么、技术约束

### 2. 生成 Spec

跑 `dev:spec-driven-development`：

- **暴露所有假设**，列出当前假设请用户纠正
- 覆盖六大核心领域：目标、命令、项目结构、代码风格、测试策略、边界
- 生成结构化 SPEC.md

### 3. 提交产物

- 保存至 `.artifacts/<yyyymmdd>/<任务简述>/SPEC.md`
- 跑 `dev:git-commit`，message：`docs: add spec for <任务简述>`

## 产物

```
.artifacts/<yyyymmdd>/<任务简述>/SPEC.md
```

## 规则

1. **需求不清不写**——置信度不够先 interview，不默默填充
2. **假设透明**——立即暴露所有假设，不藏
3. **范围限定**——不写代码、不拆任务、不设计方案
4. **Spec 是活文档**——决策变则更新 spec

## 验证

- [ ] SPEC.md 覆盖六大核心领域
- [ ] 假设已暴露并确认
- [ ] 成功标准具体可测
- [ ] 已提交到产物目录
