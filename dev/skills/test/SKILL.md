---
name: test
description: TEST 阶段——为变更代码生成并执行测试。自动检测语言/框架，后端走 backend-test-generator，前端基于 testing-library/vitest/cypress 生成。输出 TEST_REPORT.md。触发：test、生成测试、测试、test report。
---

# Test — 测试生成

## 概述

分析变更代码，生成测试用例，执行并输出报告。后端变更复用 `dev:backend-test-generator`，前端变更基于对应测试框架直接生成。

## 何时用

- BUILD 完成后，需补充测试覆盖
- 用户说"test"、"生成测试"、"测试"、"test report"
- 代码变更后需验证质量

## 流程

### 1. 检测环境

自动检测语言/框架/测试框架：

| 类型 | 检测方式 | 测试框架 |
|------|----------|----------|
| 后端 | 复用 `dev:backend-test-generator` 检测逻辑 | 依项目而定（Jest、pytest、Go testing 等） |
| 前端 React | `package.json` 含 react | testing-library + vitest/jest |
| 前端 Vue | `package.json` 含 vue | vitest + @vue/test-utils |
| 前端 Svelte | `package.json` 含 svelte | vitest + @testing-library/svelte |

### 2. 识别变更范围

- 通过 git diff / commit 范围 / 指定路径确定变更文件
- 排除测试文件、配置文件、文档

### 3. 分析变更代码

对每个变更文件分析：

- 输入/输出/依赖/异常路径
- 导出函数/组件/接口
- 纯逻辑 or 副作用（I/O、网络、DOM）

### 4. 设计测试用例

| 场景 | 覆盖 |
|------|------|
| 正常路径 | 核心逻辑正向验证 |
| 参数校验 | 边界值、null、""、负数 |
| 业务拒绝 | 权限不足、状态不符 |
| 异常路径 | 网络失败、超时、格式错误 |

### 5. 生成测试

- **后端**：跑 `dev:backend-test-generator`，按该 skill 的检测→生成→执行→修复→提交流程
- **前端**：基于检测到的测试框架直接生成测试代码，覆盖组件渲染、交互、API 调用 mock

### 6. 执行测试

- 运行生成的测试套件
- 失败则修复（最多 3 次重试）
- 确认全部通过

### 7. 输出报告

生成 `TEST_REPORT.md`：

```markdown
# Test Report: <任务简述>

## 摘要
- 测试框架：<框架>
- 测试文件数：<N>
- 测试用例数：<N>
- 通过率：<X>%

## 场景覆盖
| 场景 | 状态 | 覆盖文件 |
|------|------|----------|

## 跳过
| 文件 | 原因 |
|------|------|
```

### 8. 提交

跑 `dev:git-commit`，message：`test: add tests for <任务简述>`

## 产物

```
.artifacts/<yyyymmdd>/<任务简述>/TEST_REPORT.md
```

## 规则

1. **先检测再生成**——不确定框架就问，不猜
2. **行为优先**——测行为非实现细节
3. **失败重试**——最多 3 次修复重试
4. **覆盖关键路径**——至少覆盖正常路径+主要异常
5. **不跳过不解释**——无法测试的文件注明原因
6. **后端复用**——后端走 `dev:backend-test-generator`，不重复发明

## 验证

- [ ] 测试框架已正确检测
- [ ] 所有生成测试通过
- [ ] TEST_REPORT.md 已输出
- [ ] 提交记录完整
