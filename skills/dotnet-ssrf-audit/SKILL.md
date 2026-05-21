---
name: dotnet-ssrf-audit
description: .NET SSRF 审计工具。识别用户可控 URL 进入 HttpClient、WebRequest、Webhook 或内部调用客户端的风险路径。
---

# .NET SSRF 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 URL 来源、HTTP 客户端、Webhook、回调、代理和协议限制相关的共享抓手
- 若项目存在 ApiClient、WebhookSender、HttpGateway、CallbackService 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐请求发起入口、URL 来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充代理、宿主、网络边界与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- HttpClient、HttpRequestMessage、SocketsHttpHandler、WebRequest、RestSharp 或第三方 SDK 的请求发起点
- URL 从 Route、Query、Body、配置、数据库、消息队列、GraphQL 参数或 Hub 消息中进入请求构造的路径
- Uri、UriBuilder、Path/Query 拼接、协议相对地址、双重编码、用户名密码段、IPv6、十进制/八进制 IP 表示处理
- 自动跟随重定向、代理配置、DNS 解析、Host 头重写、X-Forwarded-* 信任边界与二次跳转
- webhook 回调、开放抓取、文件/图片下载、OpenID/OAuth discovery、S3/Blob 同步、GraphQL schema introspection 拉取等业务场景
- 对 localhost、169.254.169.254、云元数据、内网 CIDR、Unix socket、gopher/file 等危险协议或地址的限制是否有效
- 结果回显、错误信息、下载内容转存、后续解析或服务端探测能力是否扩大利用影响

## 枚举规则（强制细化）

- 枚举 HttpClient、HttpRequestMessage、WebRequest、RestSharp、Refit、GraphQL 客户端、Webhook 客户端、自定义下载器
- 优先搜索 URL、Uri、BaseAddress、RequestUri、Location、callback、webhook、avatar、download、proxy 等关键字段
- 枚举所有外部请求发起前的 URL 拼接、规范化、redirect 跟随、DNS 解析与代理配置逻辑
- 对每个请求点记录：来源参数、协议限制、host allowlist、内网/metadata 防护、最终返回内容去向

## 证据引用

- EVID_SSRF_URL_NORMALIZATION
- EVID_SSRF_FINAL_URL_HOST_PORT
- EVID_SSRF_DNSIP_AND_INNER_BLOCK

## 输出

- vuln_audit/ssrf_{timestamp}.md
- 可选：vuln_poc/ssrf_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
