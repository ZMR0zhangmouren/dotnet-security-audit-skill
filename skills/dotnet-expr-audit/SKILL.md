---
name: dotnet-expr-audit
description: .NET 表达式注入审计工具。识别 Roslyn Scripting、Dynamic LINQ、DataTable.Compute、自定义规则引擎中的表达式求值风险。
---

# .NET 表达式注入审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与脚本引擎、表达式解释器、规则引擎、动态 LINQ 和计算表达式相关的共享抓手
- 若项目存在 RuleEngine、FormulaService、ScriptHost、QueryExpressionBuilder 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐表达式入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、规则引擎注册与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- Roslyn CSharpScript、ScriptOptions、自定义规则引擎、DataTable.Compute、Dynamic LINQ、NCalc 等表达式入口
- 表达式字符串、参数上下文、可访问类型、可访问程序集和 helper 函数的来源与约束
- compile/evaluate 前是否有白名单、沙箱、黑名单、长度限制、语法限制或类型限制
- 表达式求值结果是否进一步参与权限判断、数据库查询、文件路径、模板渲染、命令参数或业务状态机
- 管理后台规则、营销表达式、工作流条件、报表公式、搜索排序公式和低代码组件等业务场景
- 运行时编译缓存、异常回显、日志输出和脚本宿主权限是否扩大攻击面

## 枚举规则（强制细化）

- 枚举 Roslyn CSharpScript、DataTable.Compute、Dynamic LINQ、NCalc、自定义规则引擎、工作流表达式引擎
- 优先搜索 evaluate、compile、script、expression、formula、rule、condition、where、sort expression 等关键字段
- 枚举管理后台规则配置、报表公式、营销规则、流程引擎、搜索排序、自定义过滤器中的表达式入口
- 对每个表达式入口记录：表达式来源、允许访问的类型/函数、沙箱策略、求值结果流向的敏感语义

证据引用：

- EVID_EXPR_EVAL_ENTRY
- EVID_EXPR_EXPR_CONTROL
- EVID_EXPR_EXEC_CHAIN_ENTRY

## 输出

- vuln_audit/expr_{timestamp}.md
- 可选：vuln_poc/expr_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
