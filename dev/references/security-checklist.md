# 安全检查清单

Web 应用安全快速参考。配 `dev:security-and-hardening` 用。

## 威胁建模

- [ ] 信任边界已绘制（请求、上传、webhook、第三方 API、LLM 输出）
- [ ] 资产已命名（凭证、PII、支付、管理员操作）
- [ ] 每个边界跑 STRIDE（仿冒、篡改、抵赖、泄露、DoS、提权）
- [ ] 滥用案例写在用例旁（"如何滥用？"）

## 提交前检查

- [ ] 代码中无密钥（`git diff --cached | grep -i "password\|secret\|api_key\|token"`）
- [ ] `.gitignore` 覆盖：`.env`、`.env.local`、`*.pem`、`*.key`
- [ ] `.env.example` 用占位符值（非真实密钥）

## 认证

- [ ] 密码用 bcrypt（≥12 rounds）、scrypt 或 argon2
- [ ] Session cookie：`httpOnly`、`secure`、`sameSite: 'lax'`
- [ ] Session 过期已配置（合理 max-age）
- [ ] 登录速率限制（15min ≤10次）
- [ ] 密码重置 token：有时限≤1h、一次性
- [ ] 多次失败后账户锁定（可选，附通知）
- [ ] 敏感操作支持 MFA（可选但推荐）

## 授权

- [ ] 每个受保护端点检查认证
- [ ] 每个资源访问检查所有权/角色（防 IDOR）
- [ ] 管理员端点需角色验证
- [ ] API 密钥限最小权限
- [ ] JWT 验证（签名、过期、签发者）

## 输入验证

- [ ] 用户输入在系统边界验证（API 路由、表单）
- [ ] 用白名单非黑名单
- [ ] 字符串长度受限
- [ ] 数值范围已验证
- [ ] Email、URL、日期格式用库验证
- [ ] 文件上传：类型受限、大小受限、内容验证
- [ ] SQL 参数化查询（无字符串拼接）
- [ ] HTML 输出编码（框架自动转义）
- [ ] URL 重定向验证（防开放重定向）
- [ ] 服务端 URL 获取白名单校验；阻塞私有/保留 IP（防 SSRF）

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

## CORS

- origin 限已知域名列表
- methods 限实际需要
- credentials: true 时不允 `origin: '*'`
- 生产不通用配符

## 数据保护

- [ ] API 响应排除敏感字段（`passwordHash`、`resetToken`）
- [ ] 敏感数据不入日志（密码、token、完整信用卡号）
- [ ] PII 静态加密
- [ ] 外部通信全 HTTPS
- [ ] 数据库备份加密

## 依赖安全

- [ ] 每次发布前跑依赖审计
- [ ] Lockfile 已提交，CI 用干净安装
- [ ] 新依赖加前审查（维护、下载量、postinstall 脚本）
- [ ] 警惕拼写仿冒（`cross-env` vs `crossenv`）
- [ ] 已知严重漏洞生产可达时立即修

## AI / LLM 安全

调用 LLM 功能（聊天、摘要、Agent、RAG）：

- [ ] 模型输出视为不信任——绝不传入 eval/SQL/shell/未转义输出/文件路径
- [ ] 假定 prompt injection 存在；权限在代码中强制，不在系统提示词
- [ ] 密钥、跨租户数据、完整提示词不入上下文
- [ ] 工具/Agent 权限限范围；破坏性操作需确认
- [ ] Token、速率、循环/递归深度设上限

## 错误处理

- 生产返回通用错误，不暴露内部
- 不返回 `err.message`、`err.stack`、SQL 等
- 用错误码（如 `INTERNAL_ERROR`）非原始错误文本

## OWASP Top 10

| # | 漏洞 | 预防 |
|---|---|---|
| 1 | 访问控制缺陷 | 每个端点检查认证、所有权 |
| 2 | 加密失败 | HTTPS、强哈希、代码中无密钥 |
| 3 | 注入 | 参数化查询、输入验证 |
| 4 | 不安全设计 | 威胁建模、spec 驱动 |
| 5 | 安全配置错误 | 安全头、最小权限、审计依赖 |
| 6 | 脆弱组件 | 依赖审计、保持更新 |
| 7 | 认证失败 | 强密码、速率限制、session 管理 |
| 8 | 数据完整性 | 验证更新/依赖、签名产物 |
| 9 | 日志失败 | 记录安全事件、不记密钥 |
| 10 | SSRF | 验证 URL、限制出站 |

## OWASP LLM Top 10

参考 [OWASA GenAI Security](https://genai.owasp.org/llm-top-10/)。

| ID | 风险 | 预防 |
|---|---|---|
| LLM01 | Prompt Injection | 不信任系统提示词为安全边界；代码中强制权限 |
| LLM02 | 敏感信息泄露 | 密钥/PII 不入提示词；过滤输出 |
| LLM03 | 供应链 | 审查模型、数据集和插件如任何依赖 |
| LLM04 | 数据与模型投毒 | 用可信模型来源、校验完整性；审查微调和 RAG 数据 |
| LLM05 | 不当输出处理 | 模型输出视为不信任；验证、编码 |
| LLM06 | 过度代理 | 限定工具权限；破坏性操作需确认 |
| LLM07 | 提示词泄露 | 假定提示词可能泄露；不放密钥 |
| LLM08 | Vector/Embedding | 按租户分区 RAG embedding；索引进验证文档 |
| LLM09 | 误导信息 | 引用支撑答案；验证关键声明；人机协同 |
| LLM10 | 无界消耗 | 设 token、速率、循环/递归上限 |
