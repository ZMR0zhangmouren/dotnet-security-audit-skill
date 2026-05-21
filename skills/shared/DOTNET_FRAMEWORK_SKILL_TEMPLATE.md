# .NET Framework 专项 Skill 模板

本模板用于新增 dotnet-{framework}-audit 类框架专项 Skill。

## 使用说明

- 将 {framework} 替换为具体框架或运行时子域
- 框架专项默认用于描述 框架边界、运行时控制面、宿主差异、组件暴露面，而不是替代漏洞专项
- 默认接入 shared/DOTNET_AUDIT_GRABBER_INDEX.md、shared/EVIDENCE_POINT_IDS.md、shared/IO_PATH_CONVENTION.md、shared/SEVERITY_RATING.md、shared/DOTNET_SINK_REFERENCE.md
- 必须显式写出 `## 输入依赖`，不要把 `source_path`、`route_mapping/*`、`auth_audit/*`、`route_tracer/*` 的关系留成隐含约定
- 框架专项默认以 `source_path` 为必需输入；若已存在 `dotnet-route-mapper` 产物，应把 `route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md` 写为可选输入，用于对齐入口、参数绑定与 `route_id`
- 若框架专项涉及匿名入口、P0/P1 分桶、授权元数据或资源归属边界，应把 `auth_audit/auth_mapping_{timestamp}.md` 写为可选输入
- 若框架专项需要将中间件、组件、控制面风险与具体入口、副作用链做交叉引用，应把 `route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md` 写为可选输入
- 如果该框架专项会影响漏洞强结论，必须显式写明其如何被 route_tracer、vuln_audit 和 dotnet-audit-pipeline 消费

## 模板

```md
---
name: dotnet-{framework}-audit
description: .NET {framework} 框架专项审计工具。针对 ... 的框架边界、运行时配置与控制面进行白盒审计。
---

# {framework} 框架专项审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 {framework} 相关的入口定义、绑定转换、前置控制、敏感操作、安全配置抓手
- 若项目存在框架二次封装、统一扩展方法、平台基座或宿主包装层，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐入口、参数绑定与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐匿名入口、P0/P1 分桶、授权元数据或资源归属边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需将框架控制面与具体入口、副作用链做交叉引用）

## 重点检查

- ...

## 枚举规则（强制细化）

- ...

## 输出要求

- framework_audit/{framework}_{timestamp}.md
- 必须写明入口类型、框架边界、运行时配置、环境差异、与其他专项的联动关系

## 证据引用

- EVID_...

输出类型映射：AUTH、CFG、LOGIC、XSS、DESER、SESS ...
```

## 最低验收要求

- 名称、description、职责边界清晰
- 已引用 shared 抓手索引和其他必要 shared 规则
- 已具备四层结构：输入依赖 + 重点检查 + 枚举规则 + 输出要求
- 已说明证据引用口径
- 已说明与 route_tracer / vuln_audit / pipeline 的关系
- 已说明如何利用 `route_mapping/*`、`auth_audit/*`、`route_tracer/*` 与框架控制面做交叉引用
- 输出路径符合 shared/IO_PATH_CONVENTION.md
- 能与现有 framework 专项区分边界，不产生大面积重复职责
