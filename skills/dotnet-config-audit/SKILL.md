---
name: dotnet-config-audit
description: .NET 配置安全审计工具。识别 appsettings、web.config、launchSettings、middleware 配置中的错误暴露、CORS、安全头、调试与危险开关风险。
---

# .NET 配置安全审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 appsettings、web.config、环境变量、Options 绑定、运行时覆盖和安全开关相关的共享抓手
- 若项目存在 ConfigHelper、FeatureFlagService、TenantSettingsProvider、SecretResolver 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于关联受影响入口、功能面与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于关联匿名入口、P0/P1 分桶与鉴权边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可作为配置影响到具体入口的交叉证据，不是前置硬依赖）

重点检查：

- DeveloperExceptionPage、DetailedErrors、Swagger/OpenAPI、GraphQL Playground、Hangfire Dashboard、health check、metrics 与诊断页面暴露
- CORS allow-any、AllowCredentials、Security Headers、HSTS、HTTPS 重定向、CookiePolicy、StaticFile 配置与缓存头
- appsettings.*、appsettings.{Environment}.json、web.config、launchSettings、User Secrets、环境变量、KeyVault/Secrets Manager 合并与覆盖顺序
- 反向代理、ForwardedHeaders、KnownProxies/KnownNetworks、PathBase、Host 头信任、TLS 终止与真实客户端 IP 判断边界
- DataProtection key 存储、连接字符串、日志级别、Feature Flag、实验开关、租户配置和默认账号配置
- Debug/trace/sandbox/test endpoint、文件浏览器、目录列表、静态文件映射、Razor Runtime Compilation 等危险开关
- 配置是否仅在开发环境生效，还是因环境判断错误、容器变量错配、流水线发布失误导致生产暴露

## 枚举规则（强制细化）

- 枚举 appsettings*.json、web.config、launchSettings、User Secrets、环境变量、KeyVault/Secret Manager、Helm/容器配置映射
- 优先搜索 UseDeveloperExceptionPage、UseSwagger、AllowAnyOrigin、ForwardedHeaders、Kestrel、DataProtection、StaticFiles、FeatureFlag 等关键配置项
- 枚举 Program.cs / Startup.cs / 扩展方法中的运行时配置覆盖、环境分支和默认值
- 对每个配置点记录：配置位置、运行时代码、影响面、环境限定条件、是否属于安全开关

证据引用：

- EVID_CFG_CONFIG_LOCATION
- EVID_CFG_RUNTIME_SETTING_CODE
- EVID_CFG_IMPACT_ASSOCIATION
- EVID_CFG_SECURITY_SWITCH_EVIDENCE

## 输出

- vuln_audit/cfg_{timestamp}.md
- 可选：vuln_poc/cfg_{timestamp}.md（如能构造复现请求或明确验证步骤）
