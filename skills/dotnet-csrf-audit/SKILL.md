---
name: dotnet-csrf-audit
description: .NET CSRF 审计工具。识别状态变更入口的 Antiforgery 保护缺失、校验绕过与不一致配置风险。
---

# .NET CSRF 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 Antiforgery、Cookie 认证、跨站提交、状态变更入口和例外白名单相关的共享抓手
- 若项目存在 CsrfHelper、AjaxWrapper、MutationGateway、AdminActionService 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐状态变更入口、参数来源与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于补充认证态、匿名入口与权限边界）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- ValidateAntiForgeryToken、AutoValidateAntiforgeryToken、IgnoreAntiforgeryToken、Endpoint Filter 与 Minimal API 自定义校验逻辑
- Cookie 认证、Session 认证、OpenID Connect 回调与 API / SPA 混用场景下的跨站请求保护边界
- SignalR、GraphQL Mutation、Blazor Server 事件回调、文件上传、管理后台 Ajax 与预签名操作接口的状态变更路径
- Antiforgery token 的生成、存储、传输、Header/Form 字段名、SameSite 配置以及子域共享行为
- JSON API、multipart、text/plain、自定义 content-type、iframe、跨源重定向和预检请求对校验分支的影响
- 例外白名单、路径忽略、环境分支、代理/CDN 重写和前端框架封装是否导致部分入口未覆盖

## 枚举规则（强制细化）

- 枚举所有状态变更接口：POST、PUT、PATCH、DELETE、GraphQL Mutation、Blazor Server 事件、副作用 Hub 方法
- 优先搜索 ValidateAntiForgeryToken、AutoValidateAntiforgeryToken、IgnoreAntiforgeryToken、IAntiforgery、cookie auth + API 组合使用位置
- 枚举前端表单、Ajax、SPA、GraphQL、SignalR、iframe 提交、文件上传和多部件请求路径
- 对每个状态变更入口记录：认证方式、token 来源、校验位置、例外规则、content-type 分支、是否跨站可触达

证据引用：

- EVID_CSRF_STATE_CHANGE_HANDLER_EXEC
- EVID_CSRF_TOKEN_SOURCE
- EVID_CSRF_TOKEN_VERIFY
- EVID_CSRF_BYPASS_BRANCH

## 输出

- vuln_audit/csrf_{timestamp}.md
- 可选：vuln_poc/csrf_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
