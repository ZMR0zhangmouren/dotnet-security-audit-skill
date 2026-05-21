# .NET 漏洞专项 Skill 模板

本模板用于新增 dotnet-{type}-audit 类漏洞专项 Skill。

## 使用说明

- 将 {type} 替换为具体漏洞类型标识
- 根据专项需要补充重点检查、枚举规则、证据引用和输出要求
- 默认接入 shared/DOTNET_AUDIT_GRABBER_INDEX.md、shared/EVIDENCE_POINT_IDS.md、shared/IO_PATH_CONVENTION.md、shared/SEVERITY_RATING.md、shared/DOTNET_SINK_REFERENCE.md
- 必须显式写出 `## 输入依赖`，不要把 `source_path`、`route_mapping/*`、`route_tracer/*`、`auth_audit/*`、`framework_audit/*` 的关系留成隐含约定
- 若该专项属于 trace-gate 类型，应在 `## 输入依赖` 中写明优先消费 `route_tracer/{route_id}/trace_{timestamp}.md`，并可选消费 `route_tracer/{route_id}/trace_summary_{timestamp}.md`
- 若该专项属于静态证据驱动类型，应在 `## 输入依赖` 中写明 `route_tracer/*` 仅作为交叉证据而不是硬前置条件
- 如通过 `dotnet-audit-pipeline` 执行，应在 `## 输入依赖` 中写明当前专项引用的 `route_id` 必须与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致
- 若专项与鉴权、匿名入口或 P0/P1 排序有关，应把 `auth_audit/auth_mapping_{timestamp}.md` 写为可选输入
- 若专项会受宿主、框架控制面、序列化配置、静态目录或中间件顺序影响，应把 `framework_audit/*_{timestamp}.md` 写为可选输入
- 如具备可复现条件或至少可构造待验证请求，应同时生成 vuln_poc/{type}_{timestamp}.md，并参考 shared/DOTNET_VULN_POC_TEMPLATE.md
- 若该专项依赖 route_tracer 或 framework_audit，必须在正文中明确其消费关系
- 若该专项需要新增 EVID_* 或新的 shared 抓手，应先更新 shared 再落 Skill

## 模板

```md
---
name: dotnet-{type}-audit
description: .NET {type} 审计工具。识别 ... 风险，输出分级、证据和修复建议。
---

# .NET {type} 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 {type} 相关的共享抓手
- 若项目存在二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、参数绑定与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于补充匿名入口、P0/P1 分桶、鉴权边界或业务前置条件）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、框架控制面、序列化配置、静态目录或运行时边界）
- 若属于 trace-gate 专项：优先消费 `route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 若属于静态证据驱动专项：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md` 仅作为交叉证据，不是前置硬依赖
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- ...

## 枚举规则（强制细化）

- ...

## 证据引用

- EVID_...

## 输出

- vuln_audit/{type}_{timestamp}.md
- vuln_poc/{type}_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC）
```

## 最低验收要求

- 名称、description、职责边界清晰
- 已引用 shared 抓手索引和其他必要 shared 规则
- 已具备四层结构：输入依赖 + 重点检查 + 枚举规则 + 输出要求
- 已说明证据引用口径
- 已说明与 route_tracer / framework_audit / pipeline 的关系
- 已写明该专项是 trace-gate 还是静态证据驱动，以及 `route_tracer/*` 属于硬前置还是交叉证据
- 输出路径符合 shared/IO_PATH_CONVENTION.md
- 如存在可复现或可构造请求模板，应同时给出 vuln_poc 产物
- 能与现有专项区分边界，不产生大面积重复职责
