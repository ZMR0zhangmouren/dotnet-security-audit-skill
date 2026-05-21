---
name: dotnet-nosql-audit
description: .NET NoSQL 注入审计工具。识别 MongoDB、Cosmos DB 等查询构造中的结构注入、操作符注入与动态过滤风险。
---

# .NET NoSQL 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 NoSQL 查询对象、过滤器构造、聚合管道、文档更新相关的共享抓手
- 若项目存在 Repository、DocumentStore、CustomFilterBuilder、SearchGateway 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、序列化配置和运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- MongoDB Driver 中 BsonDocument、Builders.Filter、Builders.Update、Aggregation Pipeline 的动态构造路径
- JSON、Dictionary<string, object>、ExpandoObject、匿名对象或前端透传结构直接映射为查询条件的场景
- $where、$regex、$ne、$gt、$lt、$or、$expr 等操作符是否可由用户输入直接控制
- Cosmos DB、Elastic、LiteDB 或自定义文档查询语法中的字符串查询拼接与占位边界
- 排序字段、投影字段、分页条件、聚合阶段、lookup/join 子句是否使用白名单或枚举约束
- 反序列化后的对象、GraphQL 参数、Hub 消息体是否会被原样送入查询构造器
- 查询结果是否被用作认证、授权、租户边界或高价值业务判断，从而放大影响

## 枚举规则（强制细化）

- 枚举 MongoDB Driver、Cosmos DB、Elastic、LiteDB、自定义文档查询封装中的查询入口
- 优先搜索 BsonDocument、FilterDefinition、Builders.Filter、Builders.Update、Aggregate、PipelineDefinition、QueryDefinition
- 枚举来自 JSON、Dictionary、ExpandoObject、GraphQL Input、Hub 消息、动态表单的数据直接映射到查询结构的代码
- 对每个查询点记录：filter 来源、operator 是否可控、排序和投影字段是否可控、分页片段是否可控
- 单独枚举 $where、$expr、regex、script query、文本搜索和聚合 lookup / facet / union 等高危能力

## 证据引用

- EVID_NOSQL_QUERY_CONSTRUCTION
- EVID_NOSQL_USER_INPUT_INTO_QUERY_STRUCTURE
- EVID_NOSQL_OPERATOR_INJECTION_FIELDS

## 输出

- vuln_audit/nosql_{timestamp}.md
- 可选：vuln_poc/nosql_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
