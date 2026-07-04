---
name: backend-test-generator
description: Auto-generates backend test cases based on code changes. Detects language and test framework, generates integration/unit tests, executes tests and outputs reports. Triggers: user says "generate tests", "write test cases", "run tests", "test report", "write unit tests for X".
---

# 后端测试生成器

## 核心原则

**测行为，不测代码写法。** 每测试验证可观测行为结果，非内部实现。测试职责：找 bug，非为通过而通过。

## 输入

| 输入 | 识别方式 |
|------|----------|
| 未提交代码 | `git diff` + `git diff --cached` |
| Git 历史 | 指定 commit hash 或范围 |
| 指定路径 | 用户直接给 |
| 文档 | 业务/接口文档 |

## 流程

### Step 1: 检测环境

自动检测语言、框架、测试框架：

```
Go       → go.mod         → go test / testify
Python   → pyproject.toml → pytest / unittest
Node.js  → package.json   → jest / vitest / mocha
Java     → pom.xml        → JUnit / TestNG
PHP      → composer.json  → PHPUnit / Pest
Rust     → Cargo.toml     → cargo test
```

读 `CLAUDE.md` 了解约定和测试目录结构。

### Step 2: 识别变更范围

- `git diff` → 列出变更文件
- commit 范围 → 列出涉变文件
- 用户指定路径 → 直接用

过滤：只关注后端代码（控制器、服务层、模型、工具），跳过视图模板、静态资源、配置、构建产物、`.artifacts/` 和 `docs/`。

### Step 3: 分析变更

逐个读文件，分析：

| 维度 | 来源 |
|------|------|
| 输入参数 | 函数签名、请求体、查询参数 |
| 参数校验 | 验证规则、守卫语句 |
| 返回值 | return 语句、响应封装 |
| 依赖 | 数据库、外部 API、消息队列、缓存 |
| 异常 | catch、rollback、error 返回 |

分类：新增→新测试；修改→回归测试；删除→记录不入测试。

### Step 4: 设计用例

每个方法覆盖：

| 场景 | 说明 | 必选 |
|------|------|------|
| 正常路径 | 合法输入→期望输出。DB 操作验证落库 | 是 |
| 参数校验 | 非法/缺参数→期望错误 | 是 |
| 业务拒绝 | 条件不允许→期望拒绝 | 是 |
| 边界值 | 空值、零值、最大值、分页边界 | 是 |
| 异常 | 外部依赖失败、回滚、超时 | 否（视代码） |

### Step 5: 生成测试文件

- 放入约定测试目录，镜像源文件结构
- 命名遵循项目约定（CLAUDE.md）

**通用结构：**
```
1. setup — 准备测试数据（真实写入，事务包裹）
2. exercise — 调用被测方法
3. verify — 断言结果（查 DB、断言返回值）
4. teardown — 清理（回滚）
```

### Step 6: 执行测试

- Go: `go test ./... -v -count=1`
- Python: `pytest -v`
- Node.js: `npx jest --verbose`
- PHP: `php vendor/bin/phpunit --testdox`
- Java: `mvn test` / `gradle test`

收集通过/失败/跳过/错误数。

### Step 7: 修复失败用例

- `F`（断言失败）和 `E`（错误）：分析根因，修测试代码（非被测），重试最多 3 次
- `S`（Skip）：仅外部依赖确实不可 mock 时标记 skip，写明原因
- 发现被测代码 bug 时：记录报告，不修改被测代码

### Step 8: 输出报告

路径：`.artifacts/<YYMMDD_序号_业务>/TEST_REPORT.md`。

```markdown
# 测试报告

## 摘要

- **日期**: <YYYY-MM-DD HH:MM>
- **范围**: <变更范围>
- **语言/框架**: <检测结果>
- **测试框架**: <框架>
- **文件**: N
- **方法**: N total | N passed | N failed | N skipped

## 结果

| 方法 | 说明 | 状态 | 耗时 | 备注 |

## 场景覆盖

| 场景 | 已覆盖 | 应覆盖 |

## 跳过 / 已知局限
- ...

## 测试输出
```
<原始输出>
```
```

## 禁止

1. **禁止验证类/函数存在**——`assertTrue(class_exists(...))`
2. **禁止反射读源码做字符串断言**
3. **禁止反射验证注解/装饰器**
4. **禁止反射验证参数个数/默认值**
5. **禁止读源文件切方法体**
6. **禁止 `assertTrue(true)`**
7. **禁止空壳 skip**——要么写完，要么 skip 写明原因
8. **禁止不调用被测方法**
9. **禁止不验证行为结果**

## 要求

1. **测行为**——断言返回值、DB 状态、副作用，非内部实现
2. **真实交互优先**——DB 真实执行，事务回滚隔离；mock 仅不可控外部服务
3. **断言具体值**——`assertSame('paid', status)` 非 `assertNotNull(status)`
4. **不修改被测代码**——发现 bug 记入报告
5. **测试独立**——不依赖其他测试执行顺序
6. **中文注释**——每个方法简短说明被测场景
7. **事后清理**——事务回滚或 teardown
