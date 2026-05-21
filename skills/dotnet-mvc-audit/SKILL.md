---
name: dotnet-mvc-audit
description: ASP.NET MVC 框架专项审计工具。针对 Controller、ModelBinder、Razor View、HtmlHelper、ActionFilter 和传统认证授权模式进行白盒审计。
---

# ASP.NET MVC 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐 Action 入口、绑定源与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐匿名入口、P0/P1 分桶与过滤器授权边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需把 View/Redirect/File 等返回链与具体副作用路径做交叉引用）

## 重点检查

- Controller / Action 级鉴权、全局过滤器、Area、ChildAction、FilterAttribute 与自定义 ActionFilter 的覆盖关系
- Model Binding、TryUpdateModel、Bind/BindNever、FromForm、JSON Patch、上传模型与 over-posting 风险
- Razor Raw / HtmlString、Layout、Partial、EditorTemplate、DisplayTemplate、TagHelper、ViewComponent，以及 Controller Content / Json / 手工拼接 HTML / JS 的输出边界
- TempData、ViewBag、ViewData、ModelState、ReturnUrl、局部视图组合、错误页与重定向链中的信任边界
- Anti-forgery、OutputCache、FileResult、Content-Disposition 下载名、内容协商与异常过滤器的安全影响
- 传统 ASP.NET MVC 与 ASP.NET Core MVC 混合项目中的配置漂移和兼容层问题

## 枚举规则（强制细化）

- 枚举所有 Controller、Area、Action、ChildAction、Partial 渲染点、ViewComponent 以及自定义 FilterAttribute，记录其路由来源、鉴权元数据和视图渲染目标
- 优先搜索 Controller、ActionResult、TryUpdateModel、UpdateModel、Bind、BindNever、ValidateAntiForgeryToken、Html.Raw、HtmlString、Redirect、LocalRedirect、RedirectToAction、Content、Json、File、PartialView
- 枚举 ModelBinder、ValueProvider、ModelState 修改点、上传模型、复杂对象绑定、集合绑定和 JSON Patch 入口，记录哪些字段可由客户端直接覆盖
- 枚举 Razor View、Layout、Partial、EditorTemplate、DisplayTemplate、HtmlHelper、自定义 TagHelper 与 ViewBag/ViewData/TempData 数据注入点，记录输出编码与信任边界
- 对每个 Action 记录：入口路由、绑定来源、前置过滤器、视图/返回类型、状态变更能力、是否依赖传统配置或兼容层

## 输出要求

- 输出文件：framework_audit/mvc_{timestamp}.md
- 必须按 入口与过滤器、模型绑定与字段覆盖、视图输出与编码边界、重定向/下载/缓存 四个章节组织结果
- 每个条目必须写明 Action 标识、视图或返回类型、关键绑定字段、前置过滤器、潜在影响类别
- 对兼容层或传统 MVC 专有风险必须单列 版本/宿主依赖，避免与 ASP.NET Core MVC 混淆

## 证据引用

- EVID_AUTH_TOKEN_DECODE_JUDGMENT
- EVID_AUTH_PERMISSION_CHECK_EXEC
- EVID_XSS_OUTPUT_POINT
- EVID_CSRF_TOKEN_VERIFY

输出类型映射：AUTH、XSS、LOGIC、CFG、CSRF。
