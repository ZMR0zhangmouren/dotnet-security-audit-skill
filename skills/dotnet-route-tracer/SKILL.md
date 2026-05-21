---
name: dotnet-route-tracer
description: .NET 入口到 Sink 的多层数据流追踪工具。根据 route_id 追踪从 Controller、Minimal API、Hub、GraphQL Resolver、WCF Service 到最终敏感操作点的证据链，不输出漏洞结论。
---

# .NET Route Tracer

## 目标

根据 `dotnet-route-mapper` 的路由（或 `dotnet-audit-pipeline` 给出的合成入口），追踪从 **入口处理器 / CLI 入口 / 指定符号** 到最终敏感操作点（SQL/命令/文件/SSRF/XSS/模板渲染/反序列化等）的位置，输出完整数据流链、参数变量名变化与可控性分析。

## CRITICAL 规则

- 只输出追踪，不输出漏洞建议
- 禁止给出“漏洞结论/风险等级/CVSS/修复建议”
- 只输出：数据流链证据、分支执行路径、Sink 定位、参数可控性判定表（可控性不等于漏洞结论）
- 必须优先覆盖 Controller / Minimal API / Hub / GraphQL Resolver / WCF Operation 等外部入口；没有确认入口归属前不得直接追 Sink
- 每条追踪必须至少回答六个问题：谁接收请求、谁完成绑定、谁做前置校验、谁执行业务、谁发起敏感操作、谁控制了关键参数
- Controller 场景必须显式追踪 Route Metadata、ModelBinder、Filter、Controller/Action、Service/Repository、Sink 六层；任一层缺失都要在 Trace 完整性声明中说明
- Middleware / Endpoint Filter / Delegating Handler / Message Inspector / HubFilter 等框架层短路点，必须作为独立分支记录，不能只写“可能经过中间件”
- DI 注入对象若承接关键参数或决定 Sink 类型，必须继续向实现类、工厂、装饰器、代理层下钻，直到确定最终执行点或声明无法解析

## 输入依赖

- `routes_{timestamp}.md`：路由定义（独立落盘时常为 `route_mapping/routes_{timestamp}.md`，见 `shared/IO_PATH_CONVENTION.md`）
- `params_{timestamp}.md`：参数结构（独立落盘时常为 `route_mapping/params_{timestamp}.md`）
- `source_path`：源码根目录
- `route_id`：来自 `cross_analysis/high_risk_routes_{timestamp}.md`；可为 HTTP 路由序号，或由 dotnet-audit-pipeline 汇总写入该文件的合成 ID（`ENTRY_CLI:` / `ENTRY_CRON:` / `ENTRY_QUEUE:` / `ENTRY_HOOK:` / `ENTRY_INCLUDE:` / `SINK_ONLY:` 等）。
- **合成入口（当非 HTTP 路由时必填）**：`entry_file`（相对 `source_path`）、`entry_symbol`（函数/方法或 `Main` / top-level statements）、`trace_from`（从何处开始向敏感操作点追踪的说明）。

## 核心追踪范围(必做)

- 入口定义：Controller Action、Minimal API Delegate、Hub 方法、GraphQL Resolver、WCF Operation
- 参数绑定：FromRoute、FromQuery、FromBody、FromForm、FromHeader、IFormFile、复杂对象绑定、自定义 ModelBinder、ValueProvider、JSON Patch
- 前置控制：Middleware、Endpoint Filter、ActionFilter、AuthorizationFilter、ResourceFilter、ExceptionFilter、HubFilter、GraphQL 中间件、WCF Message Inspector
- 业务与依赖：Controller / 入口处理器 / Service / Domain Service / Repository / Client / Background Dispatcher / Template Engine
- 敏感操作：数据库、文件、命令、网络、模板、反序列化、表达式、认证授权、配置切换、日志输出
- .NET 入口处理函数（如 Controller Action、Minimal API Delegate、SignalR Hub 方法、GraphQL Resolver、WCF Operation；非 HTTP 场景还包括 CLI/Main、BackgroundService/HostedService、定时任务回调、消息消费函数等）；**非 HTTP 任务**时以输入中的 `entry_file` + `entry_symbol` 为追踪起点（见「输入依赖」）
- 外部输入进入入口处理函数的位置（如 Route/Query/Form/Header/Body/Cookie/Claims/Session/Hub payload/GraphQL variables/SOAP Body 的绑定、反序列化、类型转换与对象赋值）
- 参数在 Controller/Endpoint/Hub/Resolver/Service/Repository/Client 等方法之间的传递与变形（局部变量赋值、对象属性写入、集合元素访问、DTO 映射、匿名对象封装、异步任务载荷传递）
- 影响执行链的分支与短路点（如 if/switch/try/catch/return/throw，以及 Filter、Middleware、Endpoint Filter、授权/租户判断、特性开关、空值回退、异常吞并）
- 最终 Sink 的实际调用点与关键实参位置（具体 API/方法、参数名/参数序号、实例来源，以及调用前是否经过拼接、编码、白名单、配置或权限约束）





## 枚举规则（强制细化）

- 先从 route_id 回到路由定义，确认 Endpoint 类型、路由模板、HTTP 方法、Area、版本、授权元数据、返回类型，再开始追踪
- 枚举 Controller / 入口处理器签名中的全部参数，区分 原始标量、复杂对象、集合、文件、服务注入参数；标记绑定来源与默认值
- 若存在自定义 ModelBinder、TypeConverter、JsonConverter、BindAsync、TryParse、IParsable、ValueProvider，必须追到字段赋值与转换逻辑
- 若存在 Filter 或 Middleware，必须区分 只读观察、拦截短路、修改参数、注入上下文、覆盖返回值 五种行为，并记录分支条件
- 对 DI 链路，必须定位接口到实现类、命名/条件注册、Decorator、Factory、Proxy、HttpClient typed client、Repository 封装，直到找到实际执行点
- 对异步链路，必须继续追踪 await 之后的调用、队列投递、后台任务、事件发布、SignalR 广播、通知发送，标记同步触发还是异步延后触发
- 若数据进入 View、Razor、JsonResult、FileResult、RedirectResult、ContentResult、Hub 广播、GraphQL 响应字段，必须明确输出上下文而不是只写“返回前端”

## Controller / Binder / Filter / Middleware / DI 追踪细则

### Controller 追踪

- 记录 Controller、Action、路由模板、HTTP 方法、Area、ApiVersion、Authorize/AllowAnonymous、Consumes/Produces、返回类型
- 追踪 Action 内部对 Request、RouteData、HttpContext、User、TempData、ViewData、ModelState、Session 的直接读取与分支使用
- 若 Action 只做转发，继续追踪到 Service、Repository、Domain Service、Client；不得停在“调用 service”这一层

### ModelBinder 追踪

- 记录每个参数的绑定源、绑定名称、前缀、复杂对象展开字段、是否支持空值、默认值回退、模型验证结果
- 自定义 ModelBinder / BinderProvider 必须记录 绑定条件、支持类型、数据来源、字段赋值规则、异常吞并与回退行为
- 对文件上传参数记录 文件名、内容类型、长度限制、暂存位置、落盘函数、后续消费点

### Filter 追踪

- 区分 AuthorizationFilter、ResourceFilter、ActionFilter、ExceptionFilter、ResultFilter、EndpointFilter、HubFilter、GraphQL Field Middleware
- 记录其执行顺序、作用范围、是否能短路、是否修改参数/结果/上下文、是否附加授权或审计语义
- 若 Filter 影响模型、身份、租户、审计开关、返回值编码，必须在主调用链中标出其前后状态差异

### Middleware 追踪

- 记录路由命中的分支 pipeline、UseWhen/MapWhen 条件、PathBase 重写、ForwardedHeaders、鉴权、Session、CORS、异常处理中间件、静态文件短路
- 若中间件会在 HttpContext.Items、User、Request.Path、Request.Scheme、RemoteIpAddress、Header、Body 上做修改，必须进入参数可控性矩阵
- 对 Minimal API / Endpoint Routing，必须记录 Route Group 与 Endpoint Filter 叠加后的最终元数据

### DI 链路追踪

- 记录接口、实现类、生命周期、命名注册、泛型关闭类型、装饰器、工厂代理和条件分支注册
- 若最终执行点来自 HttpClientFactory、DbContext、Repository、Template Service、Message Publisher、自定义 Executor，必须继续追到具体方法
- 无法解析运行时实现时，必须写明候选实现集合、推断依据和缺失证据，trace_status 至少降为 PARTIAL

## 输出要求

- 主输出文件：route_tracer/{route_id}/trace_{timestamp}.md
- 可选索引文件：route_tracer/{route_id}/trace_summary_{timestamp}.md
- 必须按 请求模板、入口与元数据、绑定链、过滤器与中间件、业务与 DI 链、Sink 证据、Trace 完整性声明 七个章节组织
- 参数可控性矩阵至少包含 参数名、绑定源、转换器/绑定器、首次校验点、最终使用点、可控等级、受影响 Sink
- Sink Summary 至少包含 Sink 类型、具体 API、代码位置、上游参数、是否经过编码/白名单/授权/配置保护、证据强度
- Trace 完整性声明必须明确 COMPLETE / PARTIAL / UNRESOLVED，并说明卡在 路由、绑定、过滤器、DI、异步派发、反射/动态调用 的哪一层
- trace_{timestamp}.md 与 trace_summary_{timestamp}.md 的标准骨架以 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 为准

## 输出文件结构（建议）
```
{output_path}/route_tracer/
└── {route_id}/
    ├── trace_{timestamp}.md
    └── trace_summary_{timestamp}.md
```
说明：**HTTP 路由**：`route_id` 宜与 `dotnet-route-mapper` 的 `routes_{timestamp}.md` 中 `=== [N] {route_summary} ===` 的序号 `N` 一致。**CLI / hook / sink-only 等**：使用流水线合成 `route_id`，追踪起点以输入中的 `entry_file` + `entry_symbol` + `trace_from` 为准，不要求与 URI 对应。

## 单路由报告模板（强制）

```markdown
# Route Trace

追踪入口: {HTTP_METHOD} {route_rule} / {entry_symbol}
生成时间: {timestamp}
项目路径: {source_path}
```
---

## 1) 完整请求模板（来自 route-mapper）
```http
{完整请求模板}
```

---
## 2) 参数进入点
| 参数名 | HTTP输入来源 | 进入处理器的变量/属性 | 位置 |
|------|----------|------------------------|------|
|      |          |                        |      |

---
## 3) 调用链层级证据（逐层，不省略）
第 1 层: {file}:{line} {Class/Type}.{Member}({args})
关键代码:
```csharp
// 展示参数如何从当前层继续流向下一层或最终 Sink 的关键片段
```

第 2 层: {file}:{line} {Class/Type}.{Member}({args})

---
## 4) 分支路径追踪（分支执行证明，必须输出）
分支 A:
条件: {if/switch/try/catch/return/throw 的条件摘要}
执行: {在该分支下参数如何继续流转到敏感操作点的摘要}
敏感操作点：是否执行 = 是/否

分支 B:
条件: {另一条分支条件摘要}
执行: {另一条分支下参数的处理/提前退出摘要}

---
## 5) 参数可控性矩阵（强制字段名与列必须一致）
| 参数名 | 输入字段/绑定名 | 绑定源 | 预处理/转换 | 首次校验点 | 最终使用点 | 受影响 Sink | 可控等级 | 约束或阻断条件 | 实际使用(是/否) |
|------|----------------|--------|-------------|------------|------------|---------------|----------|------------------|------------------|

---
## 6) Sink 定位（参数映射，强制字段名与列必须一致）
| Sink类型 | API/方法 | 调用点(文件:行) | Sink 参数名/位置 | 参数映射（Sink 参数 <- 可控参数） | 证据（关键代码/配置片段） |
|----------|----------|----------------|-------------------|----------------------------------|----------------------------|

---

## 7) Sink Summary（强制：用于 pipeline 的硬门槛校验）
必须输出一个汇总表（不允许省略）：
| route_id | Sink类型 | Sink API/语句 | 是否执行(是/否) | 触发分支(条件摘要) | 关联参数(来自可控性矩阵) | 证据(代码/配置片段) |
|----------|---------|---------------|----------------|--------------------|--------------------------|----------------------|

---
## 8) Trace 完整性声明（强制）
| trace_status | 含义 |
|-------------|------|
| COMPLETE | 调用链与 sink 证据齐全，参数可控性可判断 |
| PARTIAL | 只有局部链路证据或分支证据不完整，但 sink 存在或参数流可证明一部分 |
| UNRESOLVED | 无法定位到明确 sink 或可控性证据不足 |

并给出：
- 缺失项列表（必须逐条列出缺失：调用链层级/分支/映射/证据）
- 回退建议（允许的回退：重新执行 route-tracer，或进入对应子 Skill 的 ⚠️待验证模式）

## 必做的“禁止项”
- 禁止用省略符省略调用链层级
- 禁止保留未替换模板变量（如 `${route}` 未替换为真实 route_id）
- 禁止将“不可控”与“无风险”混为一谈：本 Skill 只输出可控性，不输出漏洞结论

## 9) Sink Evidence Type Checklist（强制：用于 pipeline 硬门槛）
要求在每条 trace 报告中输出本检查清单（允许内容为空，但必须存在标题与表头；并对实际命中的 Sink 写出证据来源与证据片段）。

| Sink类型 | 必须包含的证据要点（来自代码/配置的可证据片段） | 证据位置 |
|----------|----------------------------------------------------|----------|
| FILE | `EVID_FILE_RESOLVED_TARGET`: 最终读取目标路径或资源定位证据（如 `File.ReadAllText`、`FileStream`、`PhysicalFileResult` 的实际目标）；`EVID_FILE_PATH_NORMALIZATION`: 路径拼接、`Path.GetFullPath`、base path 约束与规范化证据；`EVID_FILE_EXECUTION_BOUNDARY`: 读取、下载、视图渲染或动态执行之间的边界证据 | file:line |
| WRITE | `EVID_WRITE_CALLSITE`: 写入调用点证据（如 `File.WriteAllText`、`FileStream`、`CopyToAsync`、`Move/Copy`）；`EVID_WRITE_DESTPATH_JOIN_AND_NORMALIZATION`: 目的路径拼接、规范化与越界判定证据；`EVID_WRITE_CONTENT_SOURCE_INTO_WRITE`: 写入内容与用户输入映射证据；`EVID_WRITE_EXECUTION_ACCESSIBILITY_PROOF`: 落盘后是否位于 web root、静态文件目录、可下载目录或可被后续处理链消费的证据 | file:line |
| UPLOAD | `EVID_UPLOAD_DESTPATH`: 上传保存目录与最终落点证据；`EVID_UPLOAD_FILENAME_EXTENSION_PARSING_SANITIZE`: 文件名、扩展名、Content-Type 与服务端重命名/净化逻辑证据；`EVID_UPLOAD_ACCESSIBILITY_PROOF`: 写入后是否形成静态访问面、下载面或后续处理面证据 | file:line |
| ARCHIVE | `EVID_ARCHIVE_EXTRACT_CALLSITE`: 解压调用点证据（如 `ZipArchive.ExtractToDirectory`）；`EVID_ARCHIVE_ENTRY_NAME_SOURCE`: 归档条目名或待解压条目列表来源证据；`EVID_ARCHIVE_FINAL_TARGET`: `base directory + entry` 合并后的最终目标路径及目录约束判定证据 | file:line |
| SSRF | `EVID_SSRF_URL_NORMALIZATION`: URL 拼接、规范化与协议处理证据；`EVID_SSRF_FINAL_URL_HOST_PORT`: `HttpClient`、`WebRequest.Create` 等发起请求前的最终 URL / Host / Port 证据；`EVID_SSRF_DNSIP_AND_INNER_BLOCK`: DNS 解析、IP 判定与内网拦截 / allowlist 证据 | file:line |
| SQL | `EVID_SQL_EXEC_POINT`: SQL 执行点证据（如 `SqlCommand.ExecuteReader`、`FromSqlRaw`、`ExecuteSqlRaw`、Dapper `Query/Execute`）；`EVID_SQL_STRING_CONSTRUCTION`: SQL 字符串构造、拼接或 Raw SQL 入口证据；`EVID_SQL_USER_PARAM_TO_SQL_FRAGMENT`: 用户输入到 SQL 片段或参数位的映射证据 | file:line |
| NOSQL | `EVID_NOSQL_QUERY_CONSTRUCTION`: NoSQL 查询构造点证据（如 `FilterDefinition`、`BsonDocument`、Cosmos 查询字符串）；`EVID_NOSQL_USER_INPUT_INTO_QUERY_STRUCTURE`: 用户输入进入查询结构证据；`EVID_NOSQL_OPERATOR_INJECTION_FIELDS`: 操作符、表达式或特殊字段注入证据 | file:line |
| CMD | `EVID_CMD_EXEC_POINT`: 命令执行点证据（如 `Process.Start`、shell 包装器、脚本启动器）；`EVID_CMD_COMMAND_STRING_CONSTRUCTION`: 命令行字符串或关键参数构造证据；`EVID_CMD_USER_PARAM_TO_CMD_FRAGMENT`: 用户输入进入命令片段或参数数组的映射证据 | file:line |
| XSS | `EVID_XSS_OUTPUT_POINT`: 输出点证据（如 Razor `Html.Raw`、`HtmlString`、Blazor `MarkupString`、`ContentResult`、手工拼接 HTML / JS）；`EVID_XSS_USER_INPUT_INTO_OUTPUT`: 用户输入进入具体输出上下文证据；`EVID_XSS_ESCAPE_OR_RAW_CONTROL`: HTML 编码、Raw 输出、模板默认转义或关闭转义证据 | file:line |
| REDIR | `EVID_REDIR_OUTPUT_POINT`: 跳转输出点证据（如 `Redirect`、`LocalRedirect`、`Results.Redirect`、`Challenge`、`SignOut`）；`EVID_REDIR_DEST_SOURCE_MAPPING`: 目标地址来源与变量映射证据；`EVID_REDIR_DEST_VALIDATION_NORMALIZATION`: `IsLocalUrl`、allowlist、scheme / host / port 限制与归一化证据 | file:line |
| CRLF | `EVID_CRLF_OUTPUT_POINT`: Header / Cookie / Location / Content-Disposition 写出点证据；`EVID_CRLF_USER_INPUT_INTO_HEADER_COOKIE`: 用户输入进入 header 或 cookie 值的映射证据；`EVID_CRLF_CONTROL_CHAR_FILTERING_ENCODING`: `\r`、`\n`、编码、框架拦截与代理层标准化证据 | file:line |
| XXE | `EVID_XXE_PARSER_CALL`: XML 解析器调用点证据（如 `XmlDocument.LoadXml`、`XDocument.Parse`、`XmlReader.Create`）；`EVID_XXE_INPUT_SOURCE`: 输入来源证据（如 `Request.Body`、上传文件、字符串参数、消息体）；`EVID_XXE_ENTITY_DOCTYPE_SAFETY_AND_ECHO`: `DtdProcessing`、`XmlResolver`、外部实体相关配置与解析结果回显证据 | file:line |
| DESER | `EVID_DESER_CALLSITE`: 反序列化调用点证据（如 `BinaryFormatter`、`NetDataContractSerializer`、启用 `TypeNameHandling` 的 Newtonsoft.Json）；`EVID_DESER_INPUT_SOURCE`: 反序列化输入来源证据；`EVID_DESER_OBJECT_TYPE_MAGIC_TRIGGER_CHAIN`: 目标类型、后续方法调用、反射或敏感操作触发链证据 | file:line |
| TPL | `EVID_TPL_ENGINE_RENDER_OR_PARSE_ENTRY`: 模板渲染或解析入口证据（如 Razor Runtime Compilation、邮件模板渲染器）；`EVID_TPL_TEMPLATE_OR_EXPR_CONTROL`: 模板名、模板内容或表达式可控性证据；`EVID_TPL_EXEC_CHAIN_ENTRY`: 编译、求值、执行链入口及沙箱 / 限制策略证据 | file:line |
| LDAP | `EVID_LDAP_EXEC_POINT`: LDAP 查询执行点证据（如 `DirectorySearcher.FindAll`、`PrincipalContext` 相关调用）；`EVID_LDAP_FILTER_STRING_CONSTRUCTION`: Filter / DN 构造证据；`EVID_LDAP_USER_PARAM_TO_FILTER_FRAGMENT`: 用户输入进入 Filter / DN 片段映射证据 | file:line |
| EXPR | `EVID_EXPR_EVAL_ENTRY`: 表达式求值入口证据（如 Dynamic LINQ、`CSharpScript.EvaluateAsync`、`DataTable.Compute`）；`EVID_EXPR_EXPR_CONTROL`: 表达式字符串可控性证据；`EVID_EXPR_EXEC_CHAIN_ENTRY`: 求值结果进入敏感语义或后续执行链证据 | file:line |
| AUTH | `EVID_AUTH_PATH_PROTECTED_MATCH`: 路由进入受保护处理端点的匹配证据；`EVID_AUTH_TOKEN_DECODE_JUDGMENT`: Cookie / JWT / ClaimsPrincipal 解析与判断证据；`EVID_AUTH_PERMISSION_CHECK_EXEC`: `Authorize`、policy、role、claim 或自定义权限检查证据；`EVID_AUTH_IDOR_OWNERSHIP_CONDITION`: 资源归属字段、租户隔离或 owner 条件约束证据 | file:line |
| CSRF | `EVID_CSRF_STATE_CHANGE_HANDLER_EXEC`: 状态变更处理方法或处理端点执行证据；`EVID_CSRF_TOKEN_SOURCE`: Antiforgery token 或等价 token 的生成 / 存储来源证据；`EVID_CSRF_TOKEN_VERIFY`: 服务端校验、比较、失败早退与验证分支证据；`EVID_CSRF_BYPASS_BRANCH`: 仅某些 Content-Type / AJAX / 特定 endpoint 校验的绕过分支证据 | file:line |
| SESS | `EVID_SESS_COOKIE_FLAGS`: Cookie 的 `HttpOnly`、`Secure`、`SameSite`、过期策略证据；`EVID_SESS_TICKET_REGEN_ROTATION`: 登录后认证票据轮换、刷新或重新签发证据；`EVID_SESS_JWT_VERIFY_CLAIMS`: JWT 签名算法与 `exp/nbf/iss/aud` 校验证据；`EVID_SESS_LOGOUT_CLEAR`: 登出时 cookie、ticket、session 或 refresh token 清理证据 | file:line |
| CFG | `EVID_CFG_CONFIG_LOCATION`: 配置位置证据（如 `appsettings.json`、`web.config`、环境变量、容器配置）；`EVID_CFG_RUNTIME_SETTING_CODE`: 运行时设置代码证据（如 middleware 顺序、CORS、安全头、异常处理）；`EVID_CFG_IMPACT_ASSOCIATION`: 配置影响到的入口、响应或组件关联证据；`EVID_CFG_SECURITY_SWITCH_EVIDENCE`: 调试开关、错误暴露、安全头、代理信任等关键安全开关证据 | file:line |


## 输出模板约束

### 完整请求模板

- 必须使用 route-mapper 已确认的真实路由模板与参数名
- 必须区分 Query、Body、Form、Header、Cookie、File、WebSocket/Hub payload、GraphQL variables、SOAP Body
- 允许保留的占位符仅限 {host}、{cookie}、{token}、{apiKey}、{jwt}

### Sink Evidence Type Checklist

- 调用点证据：是否找到具体 API/方法
- 参数来源证据：是否能把用户输入映射到关键实参
- 约束证据：是否存在白名单、编码、权限、配置、长度/格式校验
- 分支证据：是否确认保护逻辑一定执行、条件执行还是可绕过
- 环境证据：是否依赖部署模式、代理、配置文件、注册顺序、运行时实现

## 输出模板统一规则

- route_id、入口标识、返回模板类型、处理器标识 必须与 route-mapper 产物保持一致
- 如需要附加专项字段，必须在保留共享模板章节的前提下追加，不能重排核心章节语义
- 最终被 pipeline 消费的 trace 产物格式以 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 为准

## Trace 状态

- COMPLETE：调用链、执行点和可控性证据齐全
- PARTIAL：局部链路可证实，但分支或映射不完整
- UNRESOLVED：无法定位明确执行点或关键映射缺失
