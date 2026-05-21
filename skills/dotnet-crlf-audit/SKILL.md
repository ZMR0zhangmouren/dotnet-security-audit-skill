---
name: dotnet-crlf-audit
description: .NET CRLF 与响应分割审计工具。识别 Response.Headers、Set-Cookie、Location 等输出点中的控制字符注入风险。
---

# .NET CRLF 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 Header、Cookie、日志换行、响应分割和代理透传相关的共享抓手
- 若项目存在 ResponseHelper、HeaderBuilder、CookieManager、LogFormatter 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐输出入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主服务器、代理边界与响应处理控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- Response.Headers.Append/Add、AppendCommaSeparatedValues、WriteAsync 前设置 header、异常处理中写 header 的调用点
- Set-Cookie、Location、Content-Disposition、X-Accel-Redirect、X-Sendfile、自定义审计 header 等高风险响应头构造
- 用户输入进入 header 名、header 值、文件名、下载名、重定向地址、cookie 值或日志关联 ID 的映射
- \r、\n、%0d%0a、Unicode 分隔符、折行、双重解码和代理层标准化对过滤逻辑的影响
- 框架是否在 Kestrel、IIS、反向代理或第三方库层面对非法 header 做了硬拦截，以及是否可被绕过
- CRLF 注入后是否可造成响应分割、缓存污染、下载名欺骗、日志伪造或安全策略头覆盖

## 枚举规则（强制细化）

- 枚举所有 Response.Headers 写入、Set-Cookie、Location、Content-Disposition、自定义下载和反向代理透传 header 逻辑
- 优先搜索 HeaderDictionary、Append、Add、SetCookieHeaderValue、FileDownloadName、redirect location 等关键调用
- 枚举来自用户输入、文件名、日志 ID、代理头、数据库字段进入 header 的位置
- 对每个 header 输出点记录：字段名、值来源、过滤逻辑、宿主服务器拦截行为、可能造成的响应污染类型

证据建议：重点记录输出点、用户输入映射和控制字符过滤行为。

## 输出

- vuln_audit/crlf_{timestamp}.md
- 可选：vuln_poc/crlf_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
