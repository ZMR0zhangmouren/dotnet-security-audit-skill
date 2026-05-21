---
name: dotnet-deser-audit
description: .NET 反序列化审计工具。识别 BinaryFormatter、NetDataContractSerializer、Newtonsoft.Json TypeNameHandling、LosFormatter 等危险反序列化路径。
---

# .NET 反序列化审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与类型信息、多态解析、序列化器选择、KnownType 白名单和输入源相关的共享抓手
- 若项目存在 SerializerHelper、MessageCodec、CacheSerializer、StateFormatter 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、反序列化输入来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、序列化配置和运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- BinaryFormatter、NetDataContractSerializer、SoapFormatter、LosFormatter、ObjectStateFormatter、Newtonsoft.Json TypeNameHandling 等入口
- 用户输入如何影响类型信息、Binder、SerializationBinder、SurrogateSelector 或 ContractResolver 的选择
- ViewState、Session、Cookie、缓存、消息队列、SignalR、GraphQL、自定义 RPC 和文件导入中的反序列化边界
- 反序列化后对象是否触发 OnDeserialized、ISerializable、ObjectDataProvider、属性 setter、副作用方法或依赖注入钩子
- 允许类型白名单、AssemblyQualifiedName、KnownTypes、Polymorphic serialization 配置是否安全
- 反序列化数据是否进入后台任务、模板引擎、命令执行、文件写入或数据库更新，形成更长利用链

## 枚举规则（强制细化）

- 枚举 BinaryFormatter、NetDataContractSerializer、SoapFormatter、LosFormatter、ObjectStateFormatter、Json.NET TypeNameHandling 相关入口
- 优先搜索 Deserialize、TypeNameHandling、SerializationBinder、KnownTypes、AssemblyQualifiedName、ViewState、Session、message payload 等关键字
- 枚举 Cookie、Session、缓存、消息队列、上传文件、API Body、GraphQL/Hub 消息、自定义 RPC 的反序列化链路
- 对每个反序列化点记录：类型信息来源、白名单/黑名单策略、后续副作用链、宿主框架与运行环境

证据引用：

- EVID_DESER_CALLSITE
- EVID_DESER_INPUT_SOURCE
- EVID_DESER_OBJECT_TYPE_MAGIC_TRIGGER_CHAIN

## 输出

- vuln_audit/deser_{timestamp}.md
- 可选：vuln_poc/deser_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
