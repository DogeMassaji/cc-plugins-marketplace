---
name: backend-test-generator
description: Auto-generates backend test cases based on code changes. Detects language and test framework, generates integration/unit tests, executes tests and outputs reports. Triggers: user says "generate tests", "write test cases", "run tests", "test report", "write unit tests for X".
---

# 后端测试生成器

## 核心原则

**测试行为，不测试代码写法。** 每个测试必须验证可观测的行为结果，而非内部实现细节。测试职责：找到现行代码的 bug，不是为了通过而通过。

## 输入形式

支持四种输入：

| 输入 | 识别方式 |
|------|----------|
| 未提交代码 | `git diff` + `git diff --cached` |
| Git 历史 | 用户指定 commit hash 或范围（如 `HEAD~3..HEAD`） |
| 指定文件/目录 | 用户直接给路径 |
| 业务文档/接口文档 | 用户提供相关文档 |

## 工作流程

### Step 1: 检测项目环境

自动检测项目语言、框架、测试框架：

```
Go       → go.mod, *.go         → go test / testify
Python   → requirements.txt, pyproject.toml → pytest / unittest
Node.js  → package.json         → jest / vitest / mocha
Java     → pom.xml, build.gradle → JUnit / TestNG
PHP      → composer.json        → PHPUnit / Pest
Rust     → Cargo.toml           → cargo test
Ruby     → Gemfile              → RSpec / Minitest
```

读取项目 `CLAUDE.md` 了解约定和测试目录结构。

### Step 2: 识别变更范围

根据输入类型：
- `git diff` / `git diff --cached` → 列出所有变更文件
- `git diff <commit1> <commit2>` → 列出 commit 间的变更文件
- 用户指定路径 → 直接使用

过滤规则：
- 只关注后端代码文件（控制器、服务层、模型、工具函数）
- 跳过视图模板、静态资源、配置文件、构建产物
- 跳过纯文档/产物目录（`.artifacts/`、`docs/`）

### Step 3: 分析变更代码

逐个读取文件，分析：

| 分析维度 | 来源 |
|----------|------|
| 输入参数 | 函数/方法签名、请求体解析、查询参数 |
| 参数校验 | 验证规则、if/throw 守卫语句 |
| 返回值 | return 语句、响应封装 |
| 依赖 | 数据库调用、外部 API、消息队列、缓存 |
| 异常路径 | catch 块、rollback、error 返回、异常抛出 |

分类：
- 新增函数/方法 → 生成新测试
- 修改的函数/方法 → 生成回归测试
- 删除的函数/方法 → 记录在报告中（不生成测试）

### Step 4: 设计测试用例

每个待测方法必须覆盖以下场景：

| 场景类型 | 说明 | 必选 |
|----------|------|------|
| 正常路径 | 合法输入 → 期望输出。数据库操作验证实际落库结果 | 是 |
| 参数校验 | 非法/缺失参数 → 期望错误响应 | 是 |
| 业务拒绝 | 不满足业务条件（状态不允许、余额不足等）→ 期望拒绝 | 是 |
| 边界值 | 空值、零值、最大值、分页边界 | 是 |
| 异常路径 | 外部依赖失败、数据库回滚、超时 | 否（视代码而定） |

### Step 5: 生成测试文件

- 测试文件放入项目约定的测试目录
- 目录结构镜像源文件结构
- 测试命名遵循项目约定（优先读取 CLAUDE.md 中的规范）

**通用测试结构：**

```
1. setup / before — 准备测试数据（真实写入，事务包裹）
2. exercise — 调用被测方法
3. verify — 断言结果（查数据库、断言返回值）
4. teardown / after — 清理（回滚事务）
```

### Step 6: 执行测试

运行项目对应的测试命令：
- Go: `go test ./... -v -count=1`
- Python: `pytest -v` / `python -m pytest -v`
- Node.js: `npx jest --verbose` / `npx vitest run`
- PHP: `php vendor/bin/phpunit --testdox`
- Java: `mvn test` / `gradle test`

收集输出：
- 通过 / 失败 / 跳过 / 错误 数量
- 失败用例的具体错误信息

### Step 7: 修复失败用例

- `F`（断言失败）和 `E`（错误）：分析根因，修复测试代码（非被测代码），重新运行，最多重试 3 次
- `S`（Skip）：仅当外部依赖确实无法 mock 时标记 skip，并写明原因
- 发现被测代码 bug 时：记录在报告中，不修改被测代码

### Step 8: 输出报告

产物存放：优先 `.artifacts/<YYMMDD_序号_业务>/test_report.md`，无当天业务时放 `.artifacts/general/test_report.md`。

```markdown
# 测试报告

## 摘要

- **日期**: <YYYY-MM-DD HH:MM>
- **范围**: <变更范围描述>
- **项目语言/框架**: <检测结果>
- **测试框架**: <使用的框架>
- **涉及文件**: N
- **测试方法**: N total | N passed | N failed | N skipped

## 结果

| 测试方法 | 说明 | 状态 | 耗时 | 备注 |
|----------|------|------|------|------|
| testXxx | 中文说明 | PASS | 0.01s | |

## 场景覆盖

| 场景类型 | 已覆盖 | 应覆盖 |
|----------|--------|--------|
| 正常路径 | 3 | 3 |
| 参数校验 | 2 | 2 |
| 边界值 | 1 | 2 |
| 业务拒绝 | 1 | 2 |

## 跳过 / 已知局限

- ...

## 测试输出

```
<原始测试命令输出>
```
```

## 硬性禁止（违反即为错误）

以下测试模式在所有语言/框架中绝对禁止：

1. **禁止仅验证类/函数存在** — `assertTrue(class_exists(...))`、`assertThat(function_exists(...))` 等
2. **禁止反射读源码做字符串断言** — 不验证"代码写了什么字符串"
3. **禁止反射验证注解/装饰器** — 不验证 annotation/attribute/decorator 的路径或参数值
4. **禁止反射验证参数个数/默认值** — 不验证方法签名细节
5. **禁止读源文件切方法体** — 不 `file_get_contents + regex` 做源码文本断言
6. **禁止 `assertTrue(true)`** — 不造假通过
7. **禁止空壳 skip** — 要么写完，要么 skip 并写明具体原因
8. **禁止不调用被测方法的测试** — 每个测试必须真实调用被测代码
9. **禁止不验证行为结果的测试** — 调完方法必须断言可观测结果

## 硬性要求（必须遵守）

1. **测试行为** — 断言返回值、数据库状态、副作用，不断言内部实现
2. **真实交互优先** — 数据库操作真实执行，用事务回滚隔离；mock 仅用于不可控的外部服务
3. **断言具体值** — `assertSame('paid', status)` 而非 `assertNotNull(status)`
4. **不修改被测代码** — 只写测试，不改业务逻辑。发现 bug 记录在报告中
5. **测试独立** — 每个测试不依赖其他测试的执行顺序
6. **中文注释** — 每个测试方法简短说明被测场景
7. **事后清理** — 测试数据不污染数据库（事务回滚或 teardown 删除）
