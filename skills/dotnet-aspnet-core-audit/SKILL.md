---
name: dotnet-aspnet-core-audit
description: ASP.NET Core 框架专项审计工具。针对中间件顺序、配置绑定、鉴权、CSRF、CORS、静态文件、安全头和运行时选项进行白盒审计。
---

# ASP.NET Core 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐入口、参数绑定与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐匿名入口、P0/P1 分桶与授权边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需将中间件/宿主控制面与具体入口、副作用链做交叉引用）

## 重点检查

- Middleware 顺序：UseForwardedHeaders、UsePathBase、UseRouting、UseCors、UseAuthentication、UseAuthorization、UseSession、UseEndpoints 的先后关系
- Antiforgery、CORS、静态文件、异常页、安全头、HSTS、HTTPS 重定向、Rate Limiting、Output Caching 的配置与覆盖范围
- Minimal API、Endpoint Filter、Route Group、MapFallback、MapWhen、branch pipeline 与匿名/授权混用边界
- appsettings.json、web.config、Options 绑定、IOptionsSnapshot、IOptionsMonitor、自定义配置绑定器与模型验证缺口
- Kestrel、IIS、反向代理、容器、PathBase、HostFiltering、ForwardedHeaders 的信任边界与环境差异
- Swagger、HealthChecks、Prometheus、Hangfire、SignalR、StaticFile、Razor Runtime Compilation 等功能组件的暴露面

## 枚举规则（强制细化）

- 枚举 Program.cs、Startup.cs、扩展方法和宿主初始化代码中的中间件注册顺序，记录每个 Use/Map/Run 分支的进入条件、顺序位置和是否存在短路
- 优先搜索 appsettings.json、web.config、UseRouting、UseAuthentication、UseAuthorization、UseCors、UseSession、UseEndpoints、MapGroup、MapControllers、MapFallback、MapWhen、UseWhen、AddAntiforgery、AddCors、AddAuthentication、AddAuthorization
- 枚举 Minimal API、Controller Endpoint、HealthChecks、Swagger、StaticFiles、Hangfire、SignalR、GraphQL、Prometheus 等暴露端点，记录匿名可达性、环境开关、PathBase 与反向代理前缀影响
- 枚举 Options 绑定、Configure<T>、PostConfigure<T>、BindConfiguration、自定义校验器、IValidateOptions、环境变量覆盖和 appsettings.* 分层，记录安全开关是否会在运行时被覆盖
- 对每个高风险组件记录：注册位置、保护机制、依赖配置、环境差异、前置中间件、后置中间件以及可能影响的漏洞类型

## 输出要求

- 输出文件：framework_audit/aspnet_core_{timestamp}.md
- 必须按 中间件顺序、端点暴露面、配置绑定与安全开关、环境差异与代理边界 四个章节组织结果
- 每个问题点必须写明 代码位置、触发条件、影响范围、关联端点/功能组件、建议联动复核的漏洞专项
- 必须单列 未闭合链路与环境依赖项，说明缺失的是 配置证据、部署证据、代理证据 还是运行时注册证据

## 证据引用

- EVID_CFG_RUNTIME_SETTING_CODE
- EVID_CFG_SECURITY_SWITCH_EVIDENCE
- EVID_AUTH_TOKEN_DECODE_JUDGMENT
- EVID_AUTH_PERMISSION_CHECK_EXEC

输出类型映射：AUTH、CSRF、CFG、XSS、LOGIC、SESS。
