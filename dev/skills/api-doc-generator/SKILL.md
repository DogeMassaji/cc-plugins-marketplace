---
name: api-doc-generator
description: 根据后端代码自动生成 API 接口文档。自动检测项目语言和框架，扫描路由/控制器，提取接口定义（路径、方法、参数、响应）并生成文档。触发条件：用户说"生成接口文档"、"写 API 文档"、"导出接口文档"、"生成 OpenAPI"、"生成 Swagger 文档"、"接口文档"、"API docs"。
---

# API 接口文档生成器

## 核心原则

**从代码提取，不是从想象编写。** 文档必须精确反映代码中实际存在的接口定义，不杜撰不存在的端点、不猜测参数类型。发现文档与代码不一致时以代码为准。

## 框架自动检测

扫描项目文件，识别后端框架：

| 语言 | 检测信号 | 框架 |
|------|----------|------|
| Go | `go.mod` + `gin`/`echo`/`chi`/`fiber`/`mux` import | Gin / Echo / Chi / Fiber / Gorilla Mux |
| Go | `go.mod` + 无额外路由库 | `net/http` 标准库 |
| Python | `FastAPI` / `fastapi` import | FastAPI |
| Python | `flask` / `Flask` import | Flask |
| Python | `django` / `rest_framework` import | Django REST Framework |
| Node.js | `express` import/dependency | Express |
| Node.js | `koa` import/dependency | Koa |
| Node.js | `fastify` import/dependency | Fastify |
| Node.js | `@nestjs/core` dependency | NestJS |
| Java | `spring-boot-starter-web` / `@RestController` 注解 | Spring Boot |
| Java | `@Path` / `@GET` JAX-RS 注解 | JAX-RS / Jersey |
| PHP | `Laravel` / `laravel` in composer.json | Laravel |
| PHP | `symfony` in composer.json + `#[Route]` | Symfony |
| Rust | `actix-web` / `actix_web` in Cargo.toml | Actix Web |
| Rust | `axum` / `rocket` in Cargo.toml | Axum / Rocket |

## 输入形式

| 输入 | 识别方式 |
|------|----------|
| 全量生成 | 扫描整个项目路由文件 |
| 变更文件 | `git diff` / `git diff --cached` 变更的控制器/路由文件 |
| 指定文件/目录 | 用户直接给路径 |
| 指定 commit 范围 | `git diff <commit1> <commit2>` |

## 工作流程

### Step 1: 检测项目环境

1. 扫描项目根目录，识别语言和框架
2. 读取 `CLAUDE.md` 了解项目约定的接口规范和文档目录
3. 定位路由文件/控制器目录（各框架典型位置见附录）
4. 定位 DTO/Request/Response 类型定义文件

### Step 2: 提取接口定义

逐文件扫描，提取每个接口：

| 提取维度 | 来源 |
|----------|------|
| HTTP 方法 | 路由注册、方法装饰器/注解 |
| URL 路径 | 路由前缀 + 子路径 |
| 路径参数 | URL 中的动态段 `/:id`、`{id}`、`<int:id>` |
| 查询参数 | 函数签名中的 query params、`@Query`、`request.args` |
| 请求体 | DTO class/struct、`@Body` 装饰器、Pydantic model |
| 请求头 | `@Header`、middleware 鉴权 |
| 响应结构 | return 类型、序列化 schema、response DTO |
| 鉴权方式 | middleware、`@UseGuards`、`requireAuth` |
| 错误码 | `@ApiResponse`、HTTP exception、error handler |

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
| email | string | 是 | 邮箱 |

**响应 (200)：**
| 字段 | 类型 | 说明 |
|------|------|------|
| id | integer | 用户 ID |
| name | string | 用户名 |

**错误码：** 400 参数校验失败 | 409 邮箱已存在
```

### Step 4: 确定输出格式

根据用户指令或项目约定选择格式：

| 格式 | 适用场景 |
|------|----------|
| Markdown | 人类阅读、团队内部文档 |
| OpenAPI 3.0 (YAML/JSON) | 导入 Swagger/Postman/Apifox、自动生成 SDK |
| 中文表格 Markdown | 对接前端、给非技术人员审查 |

优先级：若项目已有 `openapi.yaml` / `swagger.json` → 优先更新现有 OpenAPI 文件；若 `CLAUDE.md` 指定格式 → 按约定；其余默认 Markdown。

### Step 5: 生成文档

**Markdown 输出模板：**

```markdown
# <项目名> API 接口文档

> 自动生成于 <YYYY-MM-DD HH:MM> | 框架: <框架名> | 接口总数: N

## 接口分组

### <分组名：如 用户管理>

#### POST /api/users
...
```

**OpenAPI 3.0 输出模板：**

```yaml
openapi: 3.0.3
info:
  title: <项目名> API
  version: <从 package.json/go.mod/Cargo.toml 提取>
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
          description: 用户名
        email:
          type: string
          description: 邮箱
```

### Step 6: 保存文档

- Markdown: `docs/api.md` 或 `.artifacts/<yyyymmdd>/<任务简述>/api.md`
- OpenAPI: `docs/openapi.yaml` 或项目根目录 `openapi.yaml`
- 若项目已有文档文件，增量更新而非覆盖
- 对于 `git diff` 增量模式，仅更新变更涉及的接口

### Step 7: 输出摘要

生成后输出：

> API 文档生成完成。
> - 框架: <框架名>
> - 扫描文件: N 个
> - 提取接口: N 个 (GET: N, POST: N, PUT: N, DELETE: N)
> - 输出: <文件路径>
> - 格式: Markdown / OpenAPI 3.0

## 硬性禁止

1. **禁止杜撰接口** — 必须从代码实际提取，不猜测"应该有"的端点
2. **禁止参数类型臆断** — 参数类型必须来自代码中的类型定义，不凭字段名猜测
3. **禁止忽略鉴权** — 每个接口必须标注是否需要鉴权及鉴权方式
4. **禁止缺少错误码** — HTTP 错误响应必须列出（至少 400/401/403/404/500）
5. **禁止覆盖手写文档** — 若项目已有手写 API 文档且无明确指示，增量更新而非全量覆盖
6. **禁止跳过 DTO schema** — 请求/响应体必须展开字段详情，不能只写类型名

## 硬性要求

1. **代码为准** — 文档与代码不一致时以代码为准，不修改代码
2. **字段展开** — 嵌套对象递归展开（最多 3 层），数组标注元素类型
3. **枚举标注** — enum/const 类型列出所有可能值
4. **版本标注** — 若项目有多版本 API（v1/v2），标注版本号
5. **分组清晰** — 接口按模块/控制器分组，不扁平罗列
6. **增量友好** — 变更模式下只更新涉及的接口，不改无关部分

## 附录：各框架路由/控制器典型位置

| 框架 | 路由文件 | 控制器目录 | DTO 目录 |
|------|----------|-----------|----------|
| Gin | `router/` `routes/` `main.go` 内 `r.GET` | `handler/` `controller/` | `dto/` `model/` `types/` |
| Echo | `routes.go` `main.go` | `handler/` `controller/` | `dto/` `model/` |
| Chi | `main.go` `routes.go` | `handler/` `controller/` | `dto/` `types/` |
| FastAPI | `main.py` `app.py` `routers/` | `routers/` `api/` | `schemas.py` `models.py` |
| Flask | `app.py` `routes/` `views/` | `views/` `blueprints/` | `schemas/` `models/` |
| Django | `urls.py` | `views.py` `viewsets.py` | `serializers.py` |
| Express | `routes/` `app.js` `index.js` | `controllers/` `routes/` | `dto/` `types/` |
| NestJS | Controller `*.controller.ts` 内装饰器 | 同上（路由在 controller 内） | `dto/` `entities/` |
| Spring Boot | Controller 类 `@RequestMapping` | `controller/` | `dto/` `vo/` `pojo/` |
| Laravel | `routes/api.php` `routes/web.php` | `app/Http/Controllers/` | `app/Http/Requests/` `app/Http/Resources/` |
| Symfony | `#[Route]` 注解 + `config/routes.yaml` | `src/Controller/` | `src/Dto/` `src/Entity/` |
| Actix Web | `main.rs` 内 `web::resource` `route` macro | `handlers/` `controllers/` | `models/` `types/` `dto/` |
| Axum | `main.rs` 内 `Router::new().route()` | `handlers/` `routes/` | `models/` `types/` |
