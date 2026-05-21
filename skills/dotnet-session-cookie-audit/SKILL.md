---
name: dotnet-session-cookie-audit
description: .NET 会话、Cookie 与 JWT 审计工具。识别 Cookie flags、认证票据轮换、JWT 校验参数和登出清理缺陷。
---

# .NET Session / Cookie / JWT 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 Session、Cookie、JWT、票据属性、续期、注销和跨站行为相关的共享抓手
- 若项目存在 TokenService、CookieHelper、SessionManager、AuthCookieFactory 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐登录、登出、票据更新入口与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于补充认证态、匿名入口与权限边界）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- Cookie flags：HttpOnly、Secure、SameSite、Domain、Path、IsEssential、Persistent、SlidingExpiration 以及跨子域共享策略
- 登录、权限提升、2FA 完成、租户切换、模拟登录后是否轮换认证票据、session id、correlation cookie 和 antiforgery token
- ASP.NET Core Identity、CookieAuthentication、SessionMiddleware、DataProtection key ring 与分布式缓存会话的一致性
- TokenValidationParameters、签名算法、kid 选择、密钥轮换、issuer/audience、exp/nbf、clock skew、refresh token 管理
- JWT、cookie、session、remember me、外部登录回调、设备信任和无状态 API 混用时的边界问题
- 登出是否清理客户端 cookie、服务端 session、刷新令牌、服务器缓存、SignalR 连接状态和后台持久会话

## 枚举规则（强制细化）

- 枚举所有 CookieAuthentication、SessionMiddleware、Identity、JwtBearer、RefreshToken、RememberMe、Persistent Cookie 配置与使用点
- 优先搜索 HttpOnly、SecurePolicy、SameSite、ExpireTimeSpan、SlidingExpiration、SaveTokens、TokenValidationParameters、RefreshToken 表/缓存
- 枚举登录、权限提升、租户切换、2FA 完成、外部登录回调、密码重置后的票据更新路径
- 对每个会话机制记录：票据类型、存储位置、轮换时机、登出清理路径、是否跨子域/跨应用共享

证据引用：

- EVID_SESS_COOKIE_FLAGS
- EVID_SESS_TICKET_REGEN_ROTATION
- EVID_SESS_JWT_VERIFY_CLAIMS
- EVID_SESS_LOGOUT_CLEAR

## 输出

- vuln_audit/sess_{timestamp}.md
- 可选：vuln_poc/sess_{timestamp}.md（如能构造复现请求、配置验证步骤或环境依赖验证链）
