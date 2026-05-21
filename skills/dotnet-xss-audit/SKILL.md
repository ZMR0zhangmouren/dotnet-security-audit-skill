---
name: dotnet-xss-audit
description: .NET XSS 审计工具。识别 Razor、Blazor、MVC/View 输出上下文中的未编码输出、Raw 输出和 DOM 注入风险。
---

# .NET XSS 审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与输出上下文、编码边界、模板渲染、前端组件回显和 JSON/HTML/JS 返回相关的共享抓手
- 若项目存在 HtmlHelper、TemplateRenderer、EmailRenderer、PreviewService 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐输出入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充 Razor/Blazor/GraphQL/SignalR 等输出上下文控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- Razor 的 Html.Raw、IHtmlContent、TagHelper、Partial、ViewComponent、DisplayTemplate、EditorTemplate 输出边界
- Blazor 的 MarkupString、RenderFragment、动态组件参数、JSInterop 回填和 prerender 混合渲染场景
- ViewData、ViewBag、Model、TempData、Query 参数、API 返回 JSON、数据库内容进入 HTML、属性、JS、CSS、URL 上下文的映射
- 手工拼接 script、onclick、href、style、data-* 属性以及 JSON 序列化后嵌入页面的模式
- 自定义 HtmlEncoder、JavaScriptEncoder、UrlEncoder、禁用编码或使用 UnsafeRelaxedJsonEscaping 的代码路径
- 富文本、Markdown、模板内容、WYSIWYG 编辑器输出和服务端清洗器的 allowlist/denylist 规则
- CSP、Trusted Types、Razor 默认编码、Blazor 组件隔离和前端框架二次渲染是否真实降低影响

## 枚举规则（强制细化）

- 枚举所有 Razor View、Partial、Layout、ViewComponent、TagHelper、Blazor Component、邮件模板、导出 HTML 模板
- 优先搜索 Html.Raw、MarkupString、IHtmlContent、RenderFragment、UnsafeRelaxedJsonEscaping、自定义 Encoder 配置
- 枚举用户输入、数据库字段、富文本内容、CMS 模板、GraphQL 数据、Hub 消息回显到 HTML/属性/JS/URL/CSS 的位置
- 对每个输出点记录：上下文类型、编码方式、是否经过清洗、下游前端框架是否再次解释

## 证据引用

- EVID_XSS_OUTPUT_POINT
- EVID_XSS_USER_INPUT_INTO_OUTPUT
- EVID_XSS_ESCAPE_OR_RAW_CONTROL

## 输出

- vuln_audit/xss_{timestamp}.md
- 可选：vuln_poc/xss_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
