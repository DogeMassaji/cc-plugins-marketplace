---
name: security-and-hardening
description: Hardens code against vulnerabilities. Use when handling user input, authentication, data storage, or external integrations. Use when building any feature that accepts untrusted data, manages user sessions, or interacts with third-party services.
---

# 安全加固

## 概述

安全优先开发。每个外部输入当敌对，每个密钥当神圣，每个授权检查当强制。安全非阶段——它是触用户数据、认证或外部系统的每个行代码上约束。

## 何时用

- 建任何接受用户输入功能
- 实现认证或授权
- 存储或传输敏感数据
- 集成外部 API/服务
- 添加文件上传、webhook、回调
- 处理支付或 PII

## 流程：先威胁建模

无威胁模型的防控是猜测。加固前，花 5 分钟像攻击者思考：

1. **绘信任边界。** 不信任数据从哪进？HTTP、表单、上传、webhook、第三方 API、消息队列、**LLM 输出**。每个边界是攻击面。
2. **命名资产。** 什么值得窃取？凭证、PII、支付、管理员操作。
3. **每个边界跑 STRIDE：**

| 威胁 | 问题 | 缓解 |
|------|------|------|
| **S**poofing（仿冒）| 能冒充用户/服务？| 认证、签名 |
| **T**ampering（篡改）| 数据能被改？| 参数化查询、HTTPS |
| **R**epudiation（抵赖）| 操作能否认？| 审计日志 |
| **I**nfo disclosure（泄露）| 数据泄露？| 加密、白名单、通用错误 |
| **D**enial of service（DoS）| 被压垮？| 速率限制、输入上限、超时 |
| **E**levation of privilege（提权）| 得不应有权限？| 授权检查、最小权限 |

4. **每个用例旁写出滥用案例。** "如何滥用？"——让它成为第一个测试。

说不出信任边界，就没准备好加固。这就是 OWASP A04: Insecure Design。

## OWASP 核心防御

### 注入（SQL、NoSQL、OS）

参数化查询或 ORM。永不拼用户输入。OS 命令用白名单参数验证。

### 认证缺陷

bcrypt（salt ≥ 12）。Session cookie：`httpOnly`、`secure`、`sameSite: lax`，24h 过期。认证端点速率限制（15min 10次）。

### XSS

框架内置输出编码。必须渲染不信任原始标记时，用流行清理库。

### 访问控制缺陷

每个受保护端点检授权，非仅认证。

### 安全配置错误

设安全头。CORS 限已知来源——永不 `*` 带 credentials。

### 敏感数据暴露

从 API 响应剥离敏感字段。密钥来自环境变量，永不在代码中。

### SSRF

任何服务端受用户影响 URL 获取都是攻击面。白名单 scheme 和 host，拒私有/保留 IP，禁重定向。注意 DNS 重绑定是 TOCTOU 风险。

完整 OWASP 速查：`references/security-checklist.md`。

## 依赖审计分级

```
依赖漏洞
├── Critical/High → 生产可达？→ 立即修。仅开发/不可达？→ 尽快
├── Moderate → 下周期修
└── Low → 跟踪，常规更新修
```

延修则记录原因并设复查日。

### 供应链

- **提交 lockfile**，CI 用干净安装
- **新依赖加前审查**——维护、下载量、是否值。每个依赖是攻击面
- **警惕 postinstall 脚本**和拼写仿冒

## 密钥管理

```
.env.example → 已提交（模板）
.env → 不提交（含真实密钥）
.gitignore：.env, .env.local, *.pem, *.key
```

**提交前：** `git diff --cached | grep -i "password\|secret\|api_key\|token"`

**若密钥已提交，立即轮换。** 删/重写历史不够——假设泄密。先撤销重签，再清历史。

## 保护 AI / LLM

应用调 LLM 则继承新攻击面：

- **模型输出视为不信任输入。** 永不传 eval/SQL/shell/未转义输出/文件路径
- **假设提示词可劫持。** 系统提示词非安全边界；代码中强制权限
- **密钥/其他用户数据不入提示词**
- **约束工具权限。** 最小范围，破坏性操作需确认，验证每个工具参数
- **限制消费。** Token、速率、循环/递归深度上限
- **隔离检索数据。** RAG 中按租户分 embedding，索引进验证文档

## 警告

- 用户输入直接传查询/shell/未转义输出
- 密钥在源码或 commit 历史
- API 端点无认证/授权
- 依赖有已知严重漏洞
- LLM 输出传入代码执行

## 验证

- [ ] 依赖审计无 Critical/High
- [ ] 源码/git 中无密钥
- [ ] 用户输入在边界验证
- [ ] 每个受保护端点检认证和授权
- [ ] 安全头存在
- [ ] 错误不暴露内部
- [ ] 认证端点有速率限制
- [ ] 服务端 URL 获取已白名单验证（无 SSRF）
- [ ] LLM 输出使用前验证和编码
