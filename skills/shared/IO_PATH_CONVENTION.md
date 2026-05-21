# 输入/输出路径约定（.NET 全 Skill 统一）

本文件消除“逻辑章节名”和“物理落盘路径”之间的歧义，dotnet-* Skill 与 dotnet-audit-pipeline 全部适用。

## 两种执行形态

| 形态 | 含义 |
| ---- | ---- |
| 流水线合并 | 由 dotnet-audit-pipeline 编排时，最终会形成一份总报告，但默认仍要求先真实创建各阶段结构化产物目录与文件 |
| 单 Skill 独立跑 | 仅调用某个 Skill 时，可按建议路径真实创建目录与文件 |

## 默认执行模式（强制落盘）

dotnet-security-audit-skill 默认采用 强制落盘 模式，而不是仅保留逻辑章节。

默认情况下，执行 dotnet-audit-pipeline 后应至少真实创建以下目录：

- route_mapping/
- auth_audit/
- route_tracer/
- vuln_audit/
- vuln_poc/
- framework_audit/
- cross_analysis/
- vuln_report/
- quality/

如果某一阶段没有产出可确认内容，也应创建对应目录，并在该目录的索引文件或阶段文件中写明“无结果 / 未执行 / 待验证 / 环境依赖”的原因，而不是直接缺目录。

## 阶段索引文件与空结果文件（强制）

每个阶段目录默认应至少包含一份索引文件，用于记录：

- 本阶段调用了哪些 Skill
- 实际生成了哪些文件
- 哪些结果为空
- 哪些结果仅为初筛或部分完成

推荐的阶段索引文件：

- route_mapping/index_{timestamp}.md
- auth_audit/index_{timestamp}.md
- route_tracer/index_{timestamp}.md
- vuln_audit/index_{timestamp}.md
- vuln_poc/index_{timestamp}.md
- framework_audit/index_{timestamp}.md
- cross_analysis/index_{timestamp}.md
- vuln_report/index_{timestamp}.md
- quality/pipeline_execution_{timestamp}.md

如果某目录下无明确结果，也必须至少创建以下两者之一：

- index_{timestamp}.md：记录“无结果 / 未执行 / 待验证 / 环境依赖 / 初步筛选”的状态
- empty_result_{timestamp}.md：用于说明该阶段为何没有形成正式发现或正式文件

不允许目录存在但没有索引说明文件。

## 漏洞专项状态模型（强制）

dotnet-*-audit 在执行过程中，至少应使用以下状态之一：

| 状态 | 含义 |
| ---- | ---- |
| NOT_RUN | 未执行；必须说明原因 |
| INITIAL_SCREENED | 已做初步筛选；仅进行了基础枚举或粗筛，尚未完成证据闭合 |
| PARTIAL | 部分完成；已进入专项分析，但 trace、框架前提或环境证据未完全闭合 |
| COMPLETED | 已完成；具备当前阶段要求的专项结论输出 |
| NOT_APPLICABLE | 不适用；必须说明为何该项目或该入口不适用 |

约束：

- 非重点漏洞类型在一次默认执行中，允许停留在 INITIAL_SCREENED 或 PARTIAL，但不能完全没有记录
- 即使没有发现问题，也必须记录 skill 已执行且结果为 NO_ISSUE 或 NO_FINDING
- 最终 report 必须可回溯每个 dotnet-*-audit skill 的执行状态与是否发现问题

## 规范名与常见物理路径

| 规范名 | 常见物理路径 | 说明 |
| ---- | ---- | ---- |
| routes_{timestamp}.md | route_mapping/routes_{timestamp}.md | HTTP / Hub / GraphQL / WCF 入口清单 |
| params_{timestamp}.md | route_mapping/params_{timestamp}.md | 与入口逐条对应的参数结构 |
| auth_mapping_{timestamp}.md | auth_audit/auth_mapping_{timestamp}.md | 鉴权静态分桶 |
| high_risk_routes_{timestamp}.md | cross_analysis/high_risk_routes_{timestamp}.md | 高危入口集合 |
| trace_batch_plan_{timestamp}.md | cross_analysis/trace_batch_plan_{timestamp}.md | 分批追踪计划 |
| route_tracer/{route_id}/... | route_tracer/{route_id}/... | 追踪分桶产物 |
| vuln_audit/{type}_{timestamp}.md | vuln_audit/{type}_{timestamp}.md | 漏洞专项产物 |
| vuln_poc/{type}_{timestamp}.md | vuln_poc/{type}_{timestamp}.md | 漏洞专项 PoC / 待验证 PoC 产物 |
| framework_audit/{name}_{timestamp}.md | framework_audit/{name}_{timestamp}.md | 框架专项产物 |
| vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md | vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md | 依赖漏洞报告 |
| index_{timestamp}.md | {stage}/index_{timestamp}.md | 阶段索引文件，记录本阶段执行状态与产物清单 |
| empty_result_{timestamp}.md | {stage}/empty_result_{timestamp}.md | 空结果说明文件，记录无结果原因 |
| pipeline_execution_{timestamp}.md | quality/pipeline_execution_{timestamp}.md | 总执行清单与 Skill 使用跟踪 |

## 读取规则

当某 Skill 写“读取 route_mapping/routes_*.md”时，在合并模式下应理解为“读取汇总报告中与 routes_{timestamp}.md 等价的路由章节”。

但 dotnet-security-audit-skill 的默认执行模型仍要求先真实创建物理产物，再由 pipeline 汇总，因此汇总报告不应替代原始阶段文件。

最终 report 中的执行矩阵必须能反向映射到各阶段目录下的 index_{timestamp}.md 或 empty_result_{timestamp}.md。

## PoC 占位符规则

- 真实路由与参数必须替换进请求模板
- 允许保留的会话占位符：{host}、{cookie}、{token}、{apiKey}、{jwt}
- 当条件不足以构造完整可复现请求时，也应在 vuln_poc/{type}_{timestamp}.md 中输出待验证或环境依赖的 PoC 模板，不得因无法完全复现而省略该目录
