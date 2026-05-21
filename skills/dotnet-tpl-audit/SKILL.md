---
name: dotnet-tpl-audit
description: .NET 模板注入审计工具。识别 Razor Runtime Compilation、自定义模板引擎、邮件模板、动态视图解析中的模板名或模板内容可控风险。
---

# .NET 模板注入审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与模板来源、运行时编译、局部模板、邮件模板和预览渲染相关的共享抓手
- 若项目存在 TemplateRenderer、MailComposer、PreviewEngine、ViewCompiler 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐模板入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充 Razor、邮件模板或运行时编译控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- Razor Runtime Compilation、自定义 IViewLocationExpander、动态视图选择器、邮件模板引擎和 CMS 模板仓库
- 模板名、模板路径、主题名、租户模板、局部视图名、布局页和自定义 TagHelper 参数是否可控
- 模板内容来自数据库、文件上传、富文本、Markdown、管理后台编辑器或远程同步时的信任边界
- 模板编译缓存、运行时编译、程序集生成、Roslyn 集成和自定义 helper 是否引入额外执行面
- 模板中是否允许 C# 片段、Raw 输出、include/import、自定义指令、函数调用或访问服务容器
- 模板执行结果是否进入邮件、PDF、静态页面、后台通知、导出文件或管理员预览界面

## 枚举规则（强制细化）

- 枚举 Razor Runtime Compilation、自定义 ViewEngine、邮件模板、CMS 模板、导出模板、通知模板和低代码模板引擎
- 优先搜索 template、view、layout、partial、render、compile、razor、liquid、scriban、mustache 等关键名称
- 枚举模板名、模板路径、模板内容、helper、include/import、模型对象和服务容器访问能力
- 对每个模板入口记录：模板来源、编译方式、运行时权限、输出落点、是否存在用户可控模板或表达式

证据引用：

- EVID_TPL_ENGINE_RENDER_OR_PARSE_ENTRY
- EVID_TPL_TEMPLATE_OR_EXPR_CONTROL
- EVID_TPL_EXEC_CHAIN_ENTRY

## 输出

- vuln_audit/tpl_{timestamp}.md
- 可选：vuln_poc/tpl_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
