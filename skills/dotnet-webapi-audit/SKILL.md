---
name: dotnet-webapi-audit
description: ASP.NET Web API / REST API 框架专项审计工具。针对 API 鉴权、序列化、DTO 绑定、CORS、版本路由和错误返回进行白盒审计。
---

# ASP.NET Web API 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐 API 入口、绑定源与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐匿名 API、P0/P1 分桶与授权边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需把 API 入口、序列化链和下游副作用链做交叉引用）

## 重点检查

- API 认证、匿名路由、版本路由、路由约束、OpenAPI 分组与网关/反向代理前缀映射
- System.Text.Json、Newtonsoft.Json TypeNameHandling、循环引用、ReferenceHandler、类型信息、多态序列化与错误输出中的敏感字段暴露
- DTO 绑定、FromBody/FromQuery/FromRoute/FromHeader、Patch/JsonPatchDocument、批量接口和 over-posting 风险
- CORS、Swagger、ProblemDetails、异常处理中间件、模型校验失败响应与调试信息暴露
- API key、HMAC、Cookie flags / JWT validation parameters、mTLS、自定义签名头与重放保护逻辑
- 文件上传、流式下载、回调/Webhook、后台管理 API 和内部 API 的边界是否清晰

## 枚举规则（强制细化）

- 枚举所有 ApiController、Minimal API、版本路由、RoutePrefix/MapGroup、自定义约束和网关前缀映射，记录外网可达路径与版本差异
- 优先搜索 ApiController、ProducesResponseType、FromBody、FromQuery、FromRoute、FromHeader、JsonPatchDocument、ProblemDetails、EnableCors、AddSwaggerGen、AddEndpointsApiExplorer
- 枚举输入 DTO、Patch 模型、批量接口、Webhook、回调地址、文件上传和流式下载入口，记录字段来源、批量写入能力与认证方式
- 枚举序列化配置：System.Text.Json、Newtonsoft.Json、ReferenceHandler、TypeNameHandling、自定义 JsonConverter、多态解析与错误格式化逻辑
- 对每个 API 记录：路由模板、认证方式、绑定源、序列化器、错误输出、文档暴露面、是否面向内部或外部调用

## 输出要求

- 输出文件：framework_audit/webapi_{timestamp}.md
- 必须按 路由与版本、绑定与序列化、认证与跨域、错误返回与文档暴露 四个章节组织结果
- 每个问题点必须写明 受影响接口、输入绑定方式、认证上下文、响应类型、与漏洞专项的交叉点
- 必须单列 对外 API、内部 API、Webhook/回调 API 三类边界，避免将内外网假设混写

## 证据引用

- EVID_AUTH_TOKEN_DECODE_JUDGMENT
- EVID_AUTH_PATH_PROTECTED_MATCH
- EVID_DESER_OBJECT_TYPE_MAGIC_TRIGGER_CHAIN
- EVID_CFG_SECURITY_SWITCH_EVIDENCE

输出类型映射：AUTH、DESER、CFG、LOGIC、CSRF。
