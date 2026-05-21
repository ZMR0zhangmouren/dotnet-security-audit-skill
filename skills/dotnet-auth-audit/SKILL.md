---
name: dotnet-auth-audit
description: .NET 认证与授权审计工具。识别 ASP.NET Core Identity、Cookie、JWT、Policy、Role、Claim 与资源归属校验，输出鉴权映射与 AUTH 风险分析。
---

# .NET Auth Audit

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与认证方案、授权策略、声明映射、匿名例外、外部登录和会话态相关的共享抓手
- 若项目存在 AuthService、PermissionChecker、PolicyResolver、TokenHelper 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 模式

- STATIC_MAPPING：只生成 auth_mapping 与 P0/P1 分桶
- TRACE_AUDIT：在 trace 存在时输出完整 AUTH 风险分析

## 输入依赖

- 必需：`source_path`
- `STATIC_MAPPING` 模式可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于统一入口、参数来源与 `route_id`）
- `TRACE_AUDIT` 模式必需：`route_tracer/{route_id}/trace_{timestamp}.md`
- `TRACE_AUDIT` 模式可选：`route_tracer/{route_id}/trace_summary_{timestamp}.md`
- 如通过 `dotnet-audit-pipeline` 执行，`TRACE_AUDIT` 模式中的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- [Authorize]、AllowAnonymous、Policy、Role、Claim、AuthenticationSchemes 与全局过滤器之间的优先级和覆盖关系
- JWT、Cookie、Session、Identity、Windows/Auth、OpenID Connect、API Key、自定义签名票据的解析与校验流程
- TokenValidationParameters、签名算法、密钥轮换、issuer/audience/clock skew、ticket persistence 与滑动过期配置
- 资源归属校验、租户隔离、organizationId/tenantId/userId 与 route/body 参数之间的一致性，识别 IDOR 和越权
- Controller、Minimal API、Hub、GraphQL Resolver、Background API、WCF Service 等不同入口的自定义鉴权链和遗漏点
- 外部身份源映射、本地角色缓存、claim 转换、 impersonation、sudo/admin switch 和后门角色逻辑
- 登录、登出、密码重置、二次验证、remember me、设备信任和异常回退分支是否产生绕过面

## 枚举规则（强制细化）

- 枚举所有 [Authorize]、AllowAnonymous、Policy、Roles、AuthenticationSchemes、RequireAuthorization、MapGroup 级授权定义
- 优先搜索 AddAuthentication、AddAuthorization、TokenValidationParameters、CookieAuthenticationOptions、IdentityOptions、IAuthorizationHandler
- 枚举 Controller、Minimal API、Hub、GraphQL Resolver、WCF Service、后台管理 API 和内部 API 的认证授权入口
- 枚举登录、登出、注册、密码重置、2FA、外部登录、设备信任、租户切换、管理员代登等身份流转接口
- 对每个授权点记录：入口类型、认证方案、权限判断来源、资源归属字段、异常/回退分支、默认允许路径

## 证据引用（TRACE_AUDIT）

- EVID_AUTH_PATH_PROTECTED_MATCH
- EVID_AUTH_TOKEN_DECODE_JUDGMENT
- EVID_AUTH_PERMISSION_CHECK_EXEC
- EVID_AUTH_IDOR_OWNERSHIP_CONDITION

## 输出

- STATIC_MAPPING：auth_audit/auth_mapping_{timestamp}.md
- TRACE_AUDIT：auth_audit/auth_audit_report_{timestamp}.md
