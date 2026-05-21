---
name: dotnet-route-mapper
description: .NET Web 入口与参数映射分析工具。从 ASP.NET Core、MVC、Web API、Minimal API、SignalR、GraphQL、WCF 源码中提取完整入口、参数结构与请求模板。
---

# .NET Route & Endpoint Mapper

## 目标

分析 .NET Web 项目源码，提取**完整**的 HTTP 路由入口与请求参数结构，生成可用于安全测试的请求模板（Burp/Repeater 风格），并自动保存为 Markdown。

## CRITICAL：完整输出（强制）

此 Skill 必须输出所有发现的入口与接口，不允许省略：

- 禁止出现省略占位符；必须输出完整的路由、参数和请求模板
- 禁止只输出“关键接口”或“高危接口”
- 每条入口必须包含：入口类型、HTTP 方法或操作名、路径规则、处理器位置、参数结构、完整请求模板或调用模板
- 对 Controller 场景，必须把 Controller 级与 Action 级路由、版本、Area、绑定来源和返回类型差异完整展开

## 核心输入

用户提供：

- source_path：.NET 项目、解决方案或仓库根目录

可选：

- context_path：若存在反向代理、网关前缀、PathBase、服务前缀，可用于拼接完整外部路径，但必须有配置或代码证据来源

## 输出目录结构（建议）

```text
{output_path}/
├── route_mapping/
│   ├── routes_{timestamp}.md
│   └── params_{timestamp}.md
└── quality/
	 └── route_mapper_validation_{timestamp}.md
```

路径与流水线合并模式下的对应关系见 shared/IO_PATH_CONVENTION.md。
统一的 route mapping 产物格式模板见 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md。

## 覆盖范围（必做）

1. ASP.NET Core / MVC / Web API 的 Controller 路由（必做）
	- [Route]、[HttpGet]、[HttpPost]、[HttpPut]、[HttpDelete]、[HttpPatch]、[HttpHead]、[HttpOptions]
	- [AcceptVerbs] 多方法路由
	- Controller 级 [Route] 与 Action 级 [Route]/[Http*] 的组合
	- Conventional Routing：MapControllerRoute、MapDefaultControllerRoute、Area 路由、legacy 路由模板
	- Attribute Routing 与 Conventional Routing 并存时的匹配优先级与最终路径组合
	- API Versioning：URL segment、query、header、media type 版本传递方式
	- Area：AreaAttribute、MapAreaControllerRoute、模块前缀与管理后台路由
	- ControllerBase 与 Controller 的差异：View/页面型 Controller 与纯 API Controller 必须区分
2. ASP.NET Core Minimal API
	- MapGet/MapPost/MapPut/MapDelete/MapPatch/MapMethods
	- MapGroup、MapFallback、RouteGroupBuilder 前缀与 Filter
3. SignalR Hub
	- Hub Method、OnConnectedAsync/OnDisconnectedAsync 引发的服务端入口
4. GraphQL
	- Query / Mutation / Subscription 的公开入口与操作名
5. WCF
	- ServiceContract / OperationContract

## Controller 场景枚举规则（强制细化）

对每个 Controller / Action，必须显式识别以下信息：

1. 路由声明来源
	- Controller 级特性
	- Action 级特性
	- Program.cs / Startup.cs 中的 conventional route
	- Area / versioning / gateway prefix / PathBase
2. 最终路径拼接规则
	- Controller 模板 + Action 模板 + Area + 版本前缀 + 路由约束
3. HTTP 方法
	- 来自 [HttpGet]/[HttpPost]/[AcceptVerbs] 等显式声明
	- 若为 conventional route 且未显式限制，需要标注方法限制不确定或由 action 名约定推断
4. 处理器类型
	- Controller / ControllerBase
	- MVC 页面控制器 / API 控制器 / 管理后台控制器
5. 返回类型差异
	- IActionResult、ActionResult<T>、JsonResult、FileResult、RedirectResult、ViewResult、ContentResult
	- 这些返回类型会影响后续输出模板、文件下载、重定向、JSON 响应和页面渲染判断

## 参数结构解析（必做）

对每条入口输出以下参数来源与类型信息：

1. Path 参数
	- 来自路由模板占位，如 {id}、{slug}、{version:apiVersion}
	- 必须输出参数名、约束、可空性、默认值
2. Query 参数
	- [FromQuery]、默认简单类型绑定、分页/排序/过滤参数
3. Body 参数
	- [FromBody]、JSON / XML / protobuf / 自定义 input formatter
	- 对复杂对象必须展开字段结构，而不是只写 DTO 类型名
4. Form 参数
	- [FromForm]、表单字段、multipart 表单字段
5. Header 参数
	- [FromHeader]、认证头、版本头、签名头、租户头、自定义上下文头
6. Service 参数
	- [FromServices] 不是用户可控输入，但必须标注为服务注入，避免与用户输入混淆
7. File 参数
	- IFormFile、IFormFileCollection、文件列表、流式上传对象
8. 自定义绑定
	- 自定义 ModelBinder、TypeConverter、BindAsync、TryParse、自定义 input formatter
	- 若参数由这些机制绑定，必须记录绑定器来源和转换规则

## 需要特别识别的 Controller 细节（必做）

1. Model Binding 规则
	- [ApiController] 自动 400、复杂对象默认绑定源、简单类型默认 query/route 绑定
	- BindRequired、BindNever、ValidateNever、BindProperty、TryUpdateModel
2. 上传参数
	- IFormFile / IFormFileCollection / List<IFormFile> / 嵌套 DTO 中的文件属性
3. 版本化
	- [ApiVersion]、MapToApiVersion、URL segment、query/header/media type 版本来源
4. 区域与后台入口
	- [Area]、Admin/Manage/BackOffice 等后台模块路径
5. 自定义动作选择
	- ActionName、NonAction、RemoteService、自定义约定或框架封装生成的 action
6. 输出模板差异
	- FileResult：需要给出下载型模板
	- RedirectResult：需要给出跳转型模板
	- JsonResult / ActionResult<T>：需要给出 JSON 型模板
	- ViewResult：需要给出页面请求模板并标注模板渲染输出面

## 每条入口必须输出

- route_id
- 入口类型（HTTP / HUB / GRAPHQL / WCF）
- 方法与路径或操作名
- 处理器位置
- 参数来源（Route / Query / Body / Header / Cookie / File / Message）
- 完整请求模板或调用模板

对于 HTTP Controller / API 入口，还必须额外输出：

- Controller 名称
- Action 名称
- 路由声明方式（Attribute / Conventional / Mixed）
- API 版本信息（若有）
- Area（若有）
- 参数绑定来源明细（FromRoute / FromQuery / FromBody / FromForm / FromHeader / FromServices / CustomBinder）
- 返回类型模板类别（JSON / File / Redirect / View / Content）

## 处理器位置（必做）

每条入口必须给出处理器来源证据：

- 文件路径
- 类名、方法名、签名
- 若是 Controller，则必须给出 ControllerClass.ActionMethod 形式
- 若路径来自 conventional route，必须给出 Program.cs / Startup.cs 中对应 route 模板位置
- 若由框架约定生成，应说明约定来源而不是只给最终路径

## 输出格式（强制）

- routes_{timestamp}.md、params_{timestamp}.md、quality/route_mapper_validation_{timestamp}.md 的标准骨架以 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 为准
- 本 Skill 仅为 HTTP / Controller 类入口分配 route_id：使用 routes_{timestamp}.md 中 === [N] ... === 的 N 作为 route_id，并要求其与 params_{timestamp}.md、route_tracer/{route_id}/trace_{timestamp}.md 和最终汇总报告保持一致
- CLI / hook / sink-only 等非 HTTP 合成入口不占用本文件的 [N] 序号；这类合成 route_id 由 dotnet-audit-pipeline 消费的 cross_analysis/high_risk_routes_{timestamp}.md 统一承载，供后续 route-tracer 分桶使用
- 如项目需要额外字段，可附加在标准骨架后，但不得删除共享模板定义的核心章节与关键字段

### 返回模板差异（强制）

对于 Controller / API Action，必须根据返回类型选择模板：

1. JsonResult / ActionResult<T>
	- 输出标准 HTTP/JSON 请求模板
2. FileResult / PhysicalFileResult / VirtualFileResult
	- 输出下载型请求模板，并标注下载文件名、内容类型、路径来源
3. RedirectResult / LocalRedirectResult / RedirectToActionResult
	- 输出触发重定向的请求模板，并标注可能影响的 ReturnUrl / next 参数
4. ViewResult / PartialViewResult
	- 输出页面请求模板，并标注该 Action 会进入视图渲染链
5. ContentResult
	- 输出普通文本或自定义内容类型请求模板

## 输出前自检（强制）

- [ ] routes 与 params 中每条 Controller / Action 入口的 route_id 一致
- [ ] 没有出现省略占位符
- [ ] Controller 级与 Action 级路由组合已展开
- [ ] Conventional Routing 与 Attribute Routing 并存场景已写出最终路径和来源证据
- [ ] Area、API Versioning、ControllerBase vs Controller 已区分
- [ ] 参数绑定来源已细化到 FromRoute / FromQuery / FromBody / FromForm / FromHeader / FromServices / CustomBinder
- [ ] 文件上传参数 IFormFile / IFormFileCollection 已单独列出
- [ ] 返回类型模板差异已体现到请求模板说明中