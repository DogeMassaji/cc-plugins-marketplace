---
name: api-doc-generator
description: Auto-generates API documentation from backend code. Detects language and framework, scans routes/controllers, extracts endpoint definitions (path, method, parameters, responses) and generates docs. Triggers: user says "generate API docs", "write API documentation", "export API docs", "generate OpenAPI", "generate Swagger".
---

# API 接口文档生成器

## 核心原则

**从代码提取，不从想象编写。** 文档必须精确反映代码中实际接口，不杜撰端点、不猜测参数类型。文档与代码不一致时以代码为准。

## 框架自动检测

扫描项目文件，识别后端框架：

| 语言 | 信号 | 框架 |
|------|------|------|
| Go | `go.mod` + `gin`/`echo`/`chi`/`fiber`/`mux` | Gin / Echo / Chi / Fiber / Gorilla Mux |
| Go | `go.mod` + 无路由库 | `net/http` |
| Python | `FastAPI` / `fastapi` | FastAPI |
| Python | `flask` / `Flask` | Flask |
| Python | `django` / `rest_framework` | Django REST Framework |
| Node.js | `express` | Express |
| Node.js | `koa` | Koa |
| Node.js | `fastify` | Fastify |
| Node.js | `@nestjs/core` | NestJS |
| Java | `spring-boot-starter-web` / `@RestController` | Spring Boot |
| Java | `@Path` / `@GET` JAX-RS | JAX-RS / Jersey |
| PHP | `Laravel` / `laravel` in composer.json | Laravel |
| PHP | `symfony` + `#[Route]` | Symfony |
| Rust | `actix-web` / `actix_web` | Actix Web |
| Rust | `axum` / `rocket` | Axum / Rocket |

## 输入

| 输入 | 识别方式 |
|------|----------|
| 全量生成 | 扫整个项目路由文件 |
| 变更文件 | `git diff` / `git diff --cached` |
| 指定路径 | 用户直接给 |
| commit 范围 | `git diff <c1> <c2>` |

## 流程

### Step 1: 检测环境

1. 扫根目录，识别语言和框架
2. 读 `CLAUDE.md` 了解接口规范和文档目录
3. 定位路由/控制器目录
4. 定位 DTO/Request/Response 类型定义

### Step 2: 提取接口定义

逐文件扫，提取每个接口：

| 维度 | 来源 |
|------|------|
| HTTP 方法 | 路由注册、装饰器/注解 |
| URL 路径 | 路由前缀 + 子路径 |
| 路径参数 | URL 动态段 `/:id`、`{id}` |
| 查询参数 | query params、`@Query` |
| 请求体 | DTO class/struct、`@Body`、Pydantic model |
| 请求头 | `@Header`、middleware 鉴权 |
| 响应结构 | return 类型、序列化 schema |
| 鉴权方式 | middleware、`@UseGuards` |
| 错误码 | `@ApiResponse`、error handler |

### Step 3: 设计文档结构

每个接口至少包含：

```markdown
### POST /api/users
**说明：** 创建用户
**鉴权：** Bearer Token

**请求体：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 用户名 |

**响应 (200)：**
| 字段 | 类型 | 说明 |
|------|------|------|
| id | integer | 用户 ID |

**错误码：** 400 参数校验失败 | 409 邮箱已存在
```

### Step 4: 确定输出格式

| 格式 | 场景 |
|------|------|
| Markdown | 人类阅读、内部文档 |
| OpenAPI 3.0 (YAML/JSON) | 导入 Swagger/Postman/Apifox |
| 中文表格 Markdown | 对接前端、非技术人员 |

优先：项目已有 OpenAPI 文件→更新；CLAUDE.md 指定→按约定；否则 Markdown。

### Step 5: 生成文档

**Markdown 模板：**

```markdown
# <项目名> API 接口文档

> 自动生成于 <YYYY-MM-DD HH:MM> | 框架: <框架名> | 接口: N

## 接口分组

### <分组名>
...
```

**OpenAPI 3.0 模板：**

```yaml
openapi: 3.0.3
info:
  title: <项目名>
  version: <从包管理文件提取>
paths:
  /api/users:
    post:
      summary: 创建用户
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
  schemas:
    CreateUserRequest:
      type: object
      required: [name, email]
      properties:
        name:
          type: string
        email:
          type: string
```

### Step 6: 保存

- Markdown: `docs/API.md` 或 `.artifacts/<yyyymmdd>/<任务简述>/API.md`
- OpenAPI: `docs/openapi.yaml` 或根目录
- 已有文件则增量更新
- git diff 增量模式仅更新涉变接口

### Step 7: 输出摘要

> API 文档生成完成。
> - 框架: <框架名>
> - 文件: N
> - 接口: N (GET: N, POST: N, PUT: N, DELETE: N)
> - 输出: <路径>
> - 格式: Markdown / OpenAPI 3.0

## 禁止

1. **禁止杜撰接口**——从代码提取，不猜测
2. **禁止臆断参数类型**——来自代码类型定义
3. **禁止忽略鉴权**——每个接口标注鉴权方式
4. **禁止缺错误码**——至少列出 400/401/403/404/500
5. **禁止覆盖手写文档**——无明确指示则增量更新
6. **禁止跳过 DTO schema**——展开字段详情，非仅类型名

## 要求

1. **代码为准**——不一致时以代码为准，不改代码
2. **字段展开**——嵌套递归最多 3 层，数组标元素类型
3. **枚举标注**——列出所有可能值
4. **版本标注**——多版本 API 标版本号
5. **分组清晰**——按模块/控制器分组
6. **增量友好**——变更模式下只更新涉变接口

## 附录：各框架路径

| 框架 | 路由文件 | 控制器目录 | DTO 目录 |
|------|----------|-----------|----------|
| Gin | `router/` `routes/` `main.go` | `handler/` `controller/` | `dto/` `model/` |
| Echo | `routes.go` `main.go` | `handler/` `controller/` | `dto/` `model/` |
| Chi | `main.go` `routes.go` | `handler/` `controller/` | `dto/` `types/` |
| FastAPI | `routers/` | `routers/` `api/` | `schemas.py` |
| Flask | `app.py` `routes/` | `views/` `blueprints/` | `schemas/` |
| Django | `urls.py` | `views.py` | `serializers.py` |
| Express | `routes/` | `controllers/` | `dto/` `types/` |
| NestJS | Controller decorators | `controllers/` | `dto/` |
| Spring Boot | `@RequestMapping` | `controller/` | `dto/` `vo/` |
| Laravel | `routes/api.php` | `app/Http/Controllers/` | `app/Http/Requests/` |
| Symfony | `#[Route]` | `src/Controller/` | `src/Dto/` |
| Actix Web | `main.rs` resource | `handlers/` | `models/` |
| Axum | `main.rs` `.route()` | `handlers/` | `models/` |
