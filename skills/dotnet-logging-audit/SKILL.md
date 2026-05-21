---
name: dotnet-logging-audit
description: .NET 安全日志与监控审计工具。识别缺失审计事件、敏感数据入日志、结构化日志注入与告警链路缺陷。
---

# .NET 日志与监控审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与日志字段、异常记录、审计轨迹、监控事件、敏感信息脱敏和告警回调相关的共享抓手
- 若项目存在 AuditLogger、TelemetryService、EventPublisher、SecurityMonitor 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于关联关键业务入口、审计面与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于关联认证失败、权限拒绝和管理操作等高价值事件）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可作为日志事件与具体副作用入口的交叉证据，不是前置硬依赖）

重点检查：

- Serilog、NLog、ILogger、ETW、Application Insights、OpenTelemetry 等日志与追踪管道中的敏感字段记录
- 身份认证成功/失败、权限拒绝、管理操作、关键配置变更、账务操作、导入导出、租户切换是否有审计事件
- 结构化日志模板注入、message template 参数污染、异常堆栈与原始请求体写入日志的风险
- TraceId、CorrelationId、UserId、TenantId、ClientIp、ForwardedHeaders 是否准确且可用于复盘
- 脱敏、截断、采样、保留策略、日志轮转、集中式收集与告警规则是否导致安全事件被遗漏
- 调试级别日志、开发环境日志接管、第三方库 verbose 输出与 PII 泄露问题

## 枚举规则（强制细化）

- 枚举所有 ILogger、Serilog、NLog、Audit.NET、Application Insights、OpenTelemetry、ETW、自定义审计服务的写日志点
- 优先搜索 LogInformation、LogWarning、LogError、Audit、TrackEvent、TrackException、BeginScope、Enrich、Destructure 等关键调用
- 枚举认证失败、权限拒绝、配置变更、管理操作、导入导出、支付、租户切换、异常回退等高价值事件是否有日志
- 对每个日志链路记录：事件类型、敏感字段、脱敏策略、保留周期、告警联动、是否可被日志注入污染

关键证据：EVID_LOG_EVENT_COVERAGE_AND_SENSITIVE_DATA。

## 输出

- vuln_audit/log_{timestamp}.md
- 可选：vuln_poc/log_{timestamp}.md（如能构造复现路径、审计缺口验证步骤或环境依赖验证链）
