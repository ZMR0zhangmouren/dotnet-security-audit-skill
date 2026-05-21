---
name: dotnet-xxe-audit
description: .NET XXE 审计工具。识别 XmlDocument、XmlReader、XDocument、XPath 和 XSLT 相关解析场景中的外部实体与 DTD 风险。
---

# .NET XXE 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 XML 解析器、DTD、XmlResolver、XSLT、schema 导入和外部资源访问相关的共享抓手
- 若项目存在 XmlHelper、ConfigImporter、FeedParser、SoapBridge 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐 XML/SOAP 入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充 XML 解析配置、宿主与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- XmlDocument、XmlReader、XDocument、XPathDocument、XslCompiledTransform、SoapFormatter 等 XML 解析入口
- DtdProcessing、XmlResolver、XmlSecureResolver、MaxCharactersFromEntities、ProhibitDtd 等安全选项配置
- XML 输入来源：HTTP Body、SOAP 消息、文件上传、消息总线、配置导入、SAML/metadata、Office/OpenXML 二次解析
- 外部实体、参数实体、外部 DTD、XInclude、XSLT document()、schemaLocation、DTD 缓存等扩展能力是否可用
- 解析结果是否回显到响应、写入日志、拼入错误信息、参与下游请求、写入文件或进入模板
- 解析前后是否有白名单 schema、签名验证、内容长度限制和安全解析器封装

## 枚举规则（强制细化）

- 枚举 XmlDocument、XmlReader、XDocument、XPathDocument、XslCompiledTransform、DataSet.ReadXml、SOAP 解析入口
- 优先搜索 LoadXml、CreateReader、DtdProcessing、XmlResolver、Transform、ReadXml、schema、metadata import 等关键调用
- 枚举 HTTP Body、上传文件、消息总线、配置导入、WCF/SOAP、SAML/metadata、Office/OpenXML 二次解析入口
- 对每个解析点记录：安全选项、输入来源、DOCTYPE/Entity 是否允许、结果回显或后续使用路径

证据引用：

- EVID_XXE_PARSER_CALL
- EVID_XXE_INPUT_SOURCE
- EVID_XXE_ENTITY_DOCTYPE_SAFETY_AND_ECHO

## 输出

- vuln_audit/xxe_{timestamp}.md
- 可选：vuln_poc/xxe_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
