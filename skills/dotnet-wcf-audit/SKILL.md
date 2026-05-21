---
name: dotnet-wcf-audit
description: WCF 框架专项审计工具。针对 SOAP/WCF 服务的绑定、安全模式、序列化、消息检查器与服务契约暴露进行白盒审计。
---

# WCF 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐服务操作、消息参数与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐服务契约授权、匿名操作与资源归属边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需将消息处理器、序列化链与具体副作用路径做交叉引用）

## 重点检查

- BasicHttpBinding、WSHttpBinding、NetTcpBinding、NetNamedPipeBinding、安全模式、传输保护和消息级保护配置
- NetDataContractSerializer、DataContractSerializer、XmlSerializer、XmlReader、XDocument、MessageContract、KnownTypes、BinaryMessageEncoding 和消息体解析边界
- ServiceAuthorizationManager、PrincipalPermission、自定义消息检查器、行为扩展、身份模拟与客户端证书校验
- MEX 元数据暴露、svc 调试页面、IncludeExceptionDetailInFaults、详细 fault、诊断跟踪和服务清单泄露
- SOAP Header、WS-Security、会话、可靠消息、事务、回调契约和双工通信中的状态边界
- 旧版 .NET Framework 宿主、IIS/WAS、自托管 Windows Service、端口暴露与防护策略差异

## 枚举规则（强制细化）

- 枚举所有 ServiceContract、OperationContract、Binding、BehaviorExtension、MessageInspector、ServiceAuthorizationManager、PrincipalPermission 和宿主配置项
- 优先搜索 BasicHttpBinding、WSHttpBinding、NetTcpBinding、NetNamedPipeBinding、security mode、IncludeExceptionDetailInFaults、serviceMetadata、mex、KnownType、NetDataContractSerializer、XmlReader、XDocument、DtdProcessing、XmlResolver、XmlSerializerFormat
- 枚举 SOAP Header、MessageContract、事务、可靠消息、双工回调、会话实例模式、证书验证、身份模拟，以及 XmlResolver、DtdProcessing、XslCompiledTransform 相关解析配置，记录状态保持与信任边界
- 枚举 web.config、app.config、自托管初始化代码与 IIS/WAS 宿主配置，记录调试开关、元数据暴露、绑定地址和传输保护差异
- 对每个服务操作记录：绑定方式、认证方式、序列化器、消息检查器、异常/fault 输出、宿主环境与可达性

## 输出要求

- 输出文件：framework_audit/wcf_{timestamp}.md
- 必须按 绑定与传输保护、消息与序列化、授权与行为扩展、元数据与宿主暴露 四个章节组织结果
- 每个问题点必须写明 服务契约、操作名、绑定类型、安全模式、消息检查器或行为扩展、运行宿主
- 必须单列 仅 .NET Framework / IIS / WAS / 自托管 才成立的环境差异，避免泛化结论

## 证据引用

- EVID_AUTH_TOKEN_DECODE_JUDGMENT
- EVID_DESER_OBJECT_TYPE_MAGIC_TRIGGER_CHAIN
- EVID_XXE_ENTITY_DOCTYPE_SAFETY_AND_ECHO
- EVID_CFG_SECURITY_SWITCH_EVIDENCE

输出类型映射：AUTH、DESER、XXE、CFG、SESS。
