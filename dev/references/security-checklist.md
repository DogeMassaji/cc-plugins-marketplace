# 安全检查清单

Web 应用安全快速参考。配合 `security-and-hardening` Skill 使用。

## 目录

- [威胁建模（从这里开始）](#威胁建模从这里开始)
- [提交前检查](#提交前检查)
- [认证](#认证)
- [授权](#授权)
- [输入验证](#输入验证)
- [安全头](#安全头)
- [CORS 配置](#cors-配置)
- [数据保护](#数据保护)
- [依赖安全](#依赖安全)
- [AI / LLM 安全](#ai--llm-安全)
- [错误处理](#错误处理)
- [OWASP Top 10 速查](#owasp-top-10-速查)
- [OWASP LLM Top 10 速查](#owasp-llm-top-10-速查)

## 威胁建模（从这里开始）

在加固之前，花五分钟像攻击者一样思考：

- [ ] 信任边界已绘制（请求、上传、webhook、第三方 API、LLM 输出）
- [ ] 资产已命名（凭证、PII、支付数据、管理员操作）
- [ ] 每个边界运行 STRIDE（仿冒、篡改、抵赖、信息泄露、拒绝服务、权限提升）
- [ ] 滥用案例已写在用例旁（"我如何滥用这个？"）

## 提交前检查

- [ ] 代码中无密钥（`git diff --cached | grep -i "password\|secret\|api_key\|token"`）
- [ ] `.gitignore` 覆盖：`.env`、`.env.local`、`*.pem`、`*.key`
- [ ] `.env.example` 使用占位符值（非真实密钥）

## 认证

- [ ] 密码用 bcrypt（≥12 rounds）、scrypt 或 argon2 哈希
- [ ] Session cookie：`httpOnly`、`secure`、`sameSite: 'lax'`
- [ ] Session 过期已配置（合理的 max-age）
- [ ] 登录端点速率限制（15 分钟内 ≤10 次尝试）
- [ ] 密码重置 token：有时限（≤1 小时）、一次性
- [ ] 重复失败后账户锁定（可选，附通知）
- [ ] 敏感操作支持 MFA（可选但推荐）

## 授权

- [ ] 每个受保护端点检查认证
- [ ] 每个资源访问检查所有权/角色（防止 IDOR）
- [ ] 管理员端点需要管理员角色验证
- [ ] API 密钥范围限定到最小必要权限
- [ ] JWT token 已验证（签名、过期、签发者）

## 输入验证

- [ ] 所有用户输入在系统边界验证（API 路由、表单处理器）
- [ ] 验证使用白名单（非黑名单）
- [ ] 字符串长度受限（min/max）
- [ ] 数值范围已验证
- [ ] Email、URL、日期格式用合适的库验证
- [ ] 文件上传：类型受限、大小受限、内容已验证
- [ ] SQL 查询参数化（无字符串拼接）
- [ ] HTML 输出编码（使用框架自动转义）
- [ ] URL 在重定向前验证（防止开放重定向）
- [ ] 服务端 URL 获取白名单校验；私有/保留 IP 被阻止（防止 SSRF）

## 安全头

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (禁用，依赖 CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS 配置

- origin 限制为已知域名列表（`['https://yourdomain.com']`）
- methods 限制为实际需要的 HTTP 方法
- credentials: true 时不允许 `origin: '*'`
- 生产环境永远不要使用通配符 origin

## 数据保护

- [ ] API 响应排除敏感字段（`passwordHash`、`resetToken` 等）
- [ ] 敏感数据不记入日志（密码、token、完整信用卡号）
- [ ] PII 静态加密（如法规要求）
- [ ] 所有外部通信使用 HTTPS
- [ ] 数据库备份加密

## 依赖安全

- [ ] 每次发布前运行依赖漏洞审计
- [ ] Lockfile 已提交，CI 使用干净安装（可重现构建）
- [ ] 新依赖添加前审查（维护情况、下载量、`postinstall` 脚本）
- [ ] 警惕拼写仿冒攻击（`cross-env` vs `crossenv`）
- [ ] 已知严重漏洞在生产中可达时立即修复

## AI / LLM 安全

任何调用 LLM 的功能（聊天机器人、摘要、Agent、RAG）：

- [ ] 模型输出视为不信任数据——绝不传入 `eval`/SQL/shell/未转义输出/文件路径
- [ ] 假定存在 prompt injection；权限在代码中强制，不在系统提示词中
- [ ] 密钥、跨租户数据和完整系统提示词不放入上下文窗口
- [ ] 工具/Agent 权限范围限定；破坏性或不可逆操作需要确认
- [ ] Token、速率和循环/递归深度已设上限

## 错误处理

- 生产环境返回通用错误消息，不暴露内部详情
- 不返回 `err.message`、`err.stack` 或 SQL 查询等内部信息
- 使用错误码（如 `INTERNAL_ERROR`）而非原始错误文本

## OWASP Top 10 速查

| # | 漏洞 | 预防 |
|---|---|---|
| 1 | 访问控制缺陷 | 每个端点检查认证、所有权验证 |
| 2 | 加密失败 | HTTPS、强哈希、代码中无密钥 |
| 3 | 注入 | 参数化查询、输入验证 |
| 4 | 不安全设计 | 威胁建模、spec-driven 开发 |
| 5 | 安全配置错误 | 安全头、最小权限、审计依赖 |
| 6 | 脆弱组件 | 依赖审计、保持更新、最小依赖 |
| 7 | 认证失败 | 强密码、速率限制、session 管理 |
| 8 | 数据完整性失败 | 验证更新/依赖、签名产物 |
| 9 | 日志失败 | 记录安全事件、不记密钥 |
| 10 | SSRF | 验证/白名单 URL、限制出站请求 |

## OWASP LLM Top 10 速查

针对含 LLM 功能的应用。参见 [OWASP GenAI Security Project](https://genai.owasp.org/llm-top-10/)。

| ID | 风险 | 预防 |
|---|---|---|
| LLM01 | Prompt Injection | 不信任系统提示词为安全边界；在代码中强制权限 |
| LLM02 | 敏感信息泄露 | 密钥/PII 不放入提示词；过滤输出 |
| LLM03 | 供应链 | 像审查任何依赖一样审查模型、数据集和插件 |
| LLM04 | 数据与模型投毒 | 使用可信模型来源、验证完整性；审查微调和 RAG 数据 |
| LLM05 | 不当输出处理 | 将模型输出视为不信任数据；验证、参数化、编码 |
| LLM06 | 过度代理 | 限定工具权限；破坏性操作需确认 |
| LLM07 | 系统提示词泄露 | 假定系统提示词可能泄露；不放密钥在其中 |
| LLM08 | Vector 和 Embedding 弱点 | 按租户分区 RAG embedding；索引前验证文档 |
| LLM09 | 误导信息 | 用引用支撑答案；验证关键声明；保持人机协同 |
| LLM10 | 无界消耗 | 设置 token、请求速率和循环/递归深度上限 |
