---
name: dotnet-blazor-audit
description: Blazor 框架专项审计工具。针对 Blazor Server 与 Blazor WebAssembly 的状态同步、组件输出、JSInterop、认证状态和 API 边界进行白盒审计。
---

# Blazor 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐页面/API 入口、参数绑定与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐组件级授权、页面级授权与后端 API 授权边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需把组件事件、后端 API 调用和服务端副作用链做交叉引用）

## 重点检查

- Blazor Server 事件处理、组件生命周期、Circuit 重连、状态同步与服务端副作用入口
- MarkupString、RenderFragment、动态组件、参数级联、模板组件和第三方 UI 组件的输出边界
- JSInterop 输入输出信任边界、浏览器本地存储、剪贴板、文件访问、导航与前端脚本互操作
- Blazor WebAssembly 客户端代码暴露、令牌持有、API 调用链、离线缓存和 source map 泄露
- 身份认证状态提供器、租户上下文、组件级授权、页面级授权与后端 API 授权是否一致
- 预渲染、静态 SSR、交互式渲染混用时的状态泄露与双重执行问题

## 枚举规则（强制细化）

- 枚举所有 .razor 组件、页面路由、事件处理器、生命周期方法、AuthorizeView、CascadingParameter、JSInvokable 与 JSRuntime 调用点
- 优先搜索 @page、@onclick、NavigationManager、MarkupString、RenderFragment、AuthorizeView、AuthenticationStateProvider、IJSRuntime、ProtectedLocalStorage、ProtectedSessionStorage
- 区分 Blazor Server、Blazor WebAssembly、交互式 SSR、预渲染混合模式，记录状态所在端、令牌所在端、是否存在双重执行或状态漂移
- 枚举文件上传、导航重定向、客户端缓存、离线存储、剪贴板和浏览器 API 互操作，记录哪些输入会进入服务端副作用或输出上下文
- 对每个关键组件记录：路由、授权方式、状态依赖、JS 互操作、后端 API 调用、可触发的副作用类型

## 输出要求

- 输出文件：framework_audit/blazor_{timestamp}.md
- 必须按 组件入口、状态与授权、输出与 JS 互操作、客户端暴露面 四个章节组织结果
- 每个问题点必须写明 组件或页面、触发事件、状态来源、渲染上下文、后端调用链或本地存储依赖
- 对 Server 与 WebAssembly 必须分别下结论，不得把服务端状态风险与纯前端暴露风险混为一类

## 证据引用

- EVID_XSS_OUTPUT_POINT
- EVID_AUTH_PATH_PROTECTED_MATCH
- EVID_AUTH_PERMISSION_CHECK_EXEC
- EVID_SESS_COOKIE_FLAGS

输出类型映射：XSS、AUTH、LOGIC、CFG、SESS。
