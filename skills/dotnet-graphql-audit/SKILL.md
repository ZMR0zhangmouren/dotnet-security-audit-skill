---
name: dotnet-graphql-audit
description: .NET GraphQL 框架专项审计工具。针对 Query/Mutation/Subscription、Resolver 鉴权、输入绑定、复杂度限制与 introspection 暴露进行白盒审计。
---

# GraphQL 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐公开操作、变量绑定与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐 Resolver 鉴权、匿名入口与资源归属边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需将 Resolver 与下游 SQL/SSRF/文件/模板等副作用链做交叉引用）

## 重点检查

- Resolver 级鉴权、字段级授权、schema stitching、federation、租户上下文与批量查询中的授权一致性
- Query、Mutation、Subscription 参数绑定、输入对象、别名、fragment、变量和默认值对业务校验的影响
- introspection、schema 暴露、playground、Banana Cake Pop、persisted query、复杂度限制、深度限制与批量查询边界
- 自定义 scalar、DataLoader、缓存键、批处理器、N+1 优化与越权数据复用风险
- GraphQL upload、订阅连接、WebSocket 鉴权、错误返回、调试信息和 tracing 扩展暴露
- Resolver 是否进一步调用 HttpClient.SendAsync / GetAsync、HttpWebRequest / WebRequest.Create、文件、命令、模板、数据库 RawSQL 或后台作业形成复合风险

## 枚举规则（强制细化）

- 枚举 Query、Mutation、Subscription、字段 Resolver、DataLoader、Schema 配置、授权中间件、复杂度限制器和 introspection 开关
- 优先搜索 AddGraphQL、AddQueryType、AddMutationType、AddSubscriptionType、Authorize、UsePaging、UseFiltering、UseSorting、DataLoader、UploadType、Introspection
- 枚举输入对象、变量、fragment、别名、批量查询、persisted query、上传、订阅连接和 WebSocket 握手逻辑，记录字段级校验与租户约束
- 枚举 playground、Banana Cake Pop、schema 导出、错误扩展、tracing、debug 输出和 introspection 配置，记录是否在生产环境暴露
- 对每个高风险 Resolver 记录：字段名、调用类型、授权条件、输入对象、批处理行为、下游调用类型与复合风险链路

## 输出要求

- 输出文件：framework_audit/graphql_{timestamp}.md
- 必须按 Schema 暴露与调试、Resolver 授权、输入与批处理、订阅与复合风险 四个章节组织结果
- 每个问题点必须写明 字段或 Resolver、调用类型、授权模型、输入对象、下游组件调用
- 必须单列 Query、Mutation、Subscription 三类面向不同攻击面的结论，避免只给笼统 GraphQL 总结

## 证据引用

- EVID_AUTH_PERMISSION_CHECK_EXEC
- EVID_SSRF_URL_NORMALIZATION
- EVID_SQL_EXEC_POINT
- EVID_XSS_OUTPUT_POINT

输出类型映射：AUTH、LOGIC、SSRF、CFG、XSS。
