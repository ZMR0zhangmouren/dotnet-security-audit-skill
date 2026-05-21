---
name: dotnet-ldap-audit
description: .NET LDAP 注入审计工具。识别 DirectorySearcher、PrincipalContext 和自定义 LDAP Filter / DN 字符串拼接中的注入风险。
---

# .NET LDAP 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 LDAP 过滤器、目录查询、绑定、搜索基准、属性映射和认证旁路相关的共享抓手
- 若项目存在 DirectoryService、AuthAdapter、PrincipalHelper、SyncConnector 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐登录/目录查询入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充认证集成、宿主配置与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- DirectorySearcher.Filter、SearchRequest、PrincipalSearcher、自定义 LDAP 客户端的 Filter 和 DN 构造方式
- 用户输入进入 CN、OU、base DN、attribute name、filter condition、sort key、scope 的路径
- 特殊字符转义、RFC 4515 / RFC 4514 编码、白名单、枚举约束与规范化大小写处理
- LDAP 查询结果是否直接用于登录、组成员判断、角色映射、同步任务、用户搜索或目录浏览
- 多域、信任林、全局编录、匿名绑定、简单绑定、TLS/LDAPS 降级与 referral 跟随行为
- 查询超时、分页、范围查询和错误信息是否泄露目录结构或放大枚举风险

## 枚举规则（强制细化）

- 枚举 DirectorySearcher、PrincipalSearcher、SearchRequest、SearchResponse、自定义 LDAP 客户端与认证服务
- 优先搜索 Filter、DistinguishedName、baseDn、searchRoot、PrincipalContext、group search 等关键字段
- 枚举登录、用户搜索、组织树浏览、同步任务、角色映射、SSO 集成中的 LDAP 查询与 DN 构造位置
- 对每个 LDAP 调用点记录：filter 来源、转义策略、绑定方式、搜索范围、查询结果如何参与认证和授权

证据引用：

- EVID_LDAP_EXEC_POINT
- EVID_LDAP_FILTER_STRING_CONSTRUCTION
- EVID_LDAP_USER_PARAM_TO_FILTER_FRAGMENT

## 输出

- vuln_audit/ldap_{timestamp}.md
- 可选：vuln_poc/ldap_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
