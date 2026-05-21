---
name: dotnet-sql-audit
description: .NET SQL 注入审计工具。识别 ADO.NET、Dapper、EF Core RawSQL 和动态查询构造中的注入风险，输出分级、PoC 与修复建议。
---

# .NET SQL 注入审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 SQL 相关的入口定义、绑定与转换、前置控制、敏感操作、安全配置抓手
- 若项目存在 Repository、QueryBuilder、SQL Helper、DbExecutor 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充 ORM、序列化配置和运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- SqlCommand、DbCommand、SqlDataAdapter、ExecuteReader/ExecuteScalar/ExecuteNonQuery 的调用点与命令文本来源
- 通过字符串拼接、插值字符串、StringBuilder、FormattableString、AppendFormat、Join 生成 SQL 的路径
- Dapper Query/Execute、多结果集、匿名对象参数与 DynamicParameters 的参数化边界
- EF Core FromSqlRaw、ExecuteSqlRaw、Database.SqlQueryRaw、迁移脚本与仓储层封装的 RawSQL 执行点
- 排序字段、列名、表名、分页片段、条件片段、IN 列表等“无法参数化”的动态片段是否有白名单
- LINQ 最终是否退化为原始 SQL、表达式树转 SQL 时是否混入未校验字符串
- 多租户、软删除、行级权限过滤器是否被 RawSQL、IgnoreQueryFilters 或自定义仓储绕过
- 异常处理、日志和调试输出中是否暴露完整 SQL、参数值和连接信息

## 枚举规则（强制细化）

- 枚举所有继承 Controller / ControllerBase、Repository、Service、Background Job、GraphQL Resolver、Hub 中的数据库调用点
- 优先搜索 SqlCommand、DbCommand、FromSqlRaw、ExecuteSqlRaw、Query、Execute、SqlQueryRaw、CreateCommand、CommandText
- 枚举字符串拼接、插值字符串、StringBuilder、FormattableString、AppendLine、Join、Format、Raw SQL helper 的使用位置
- 单独枚举“不可参数化片段”：排序字段、列名、表名、where 片段、group by、having、offset/fetch、TOP、IN 列表
- 枚举仓储封装、QueryBuilder、自定义 Specification、Dynamic LINQ 到 RawSQL 的退化链路
- 对每个调用点记录：SQL 来源、参数对象来源、调用栈上游入口、是否在事务/多租户过滤/软删除过滤器下执行

## 证据引用（强制）

- EVID_SQL_EXEC_POINT
- EVID_SQL_STRING_CONSTRUCTION
- EVID_SQL_USER_PARAM_TO_SQL_FRAGMENT

## 输出

- vuln_audit/sql_{timestamp}.md
