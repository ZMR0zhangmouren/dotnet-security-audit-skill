# .NET 审计抓手索引

本文件用于统一各 Skill 的搜索关键字、优先枚举对象和常见落点，降低不同 Skill 在枚举阶段的偏差。

## 使用规则

- 先按 入口定义、绑定转换、前置控制、业务执行、敏感操作 五层使用本索引检索
- 优先搜索 精确 API / 特性 / 类型名，其次搜索 项目内封装命名、基础设施目录、扩展方法、注册代码
- 每个 Skill 的 枚举规则（强制细化） 都应至少覆盖与本索引对应的关键词、对象类型和记录字段
- 如果项目存在二次封装，必须把封装名追加回本 Skill 输出，形成项目内私有抓手表

## 入口定义抓手

| 类型 | 优先关键字 | 重点记录 |
| ---- | ---- | ---- |
| MVC / Web API | Controller, ControllerBase, ActionResult, Route, HttpGet, HttpPost, AcceptVerbs, Area, ApiVersion | 路由模板、HTTP 方法、授权元数据、返回类型 |
| Minimal API | MapGet, MapPost, MapGroup, RouteGroupBuilder, EndpointFilter, Results | Endpoint 委托、组前缀、过滤器、元数据 |
| SignalR | Hub, HubFilter, OnConnectedAsync, SendAsync, Groups | 连接入口、Hub 方法、广播范围 |
| GraphQL | AddGraphQL, QueryType, MutationType, SubscriptionType, Resolver, DataLoader | 字段入口、调用类型、授权和批处理 |
| WCF | ServiceContract, OperationContract, BasicHttpBinding, WSHttpBinding, ServiceHost | 服务契约、绑定、安全模式 |

## 绑定与转换抓手

| 类型 | 优先关键字 | 重点记录 |
| ---- | ---- | ---- |
| 参数来源 | FromRoute, FromQuery, FromBody, FromForm, FromHeader, BindProperty, SupplyParameterFromQuery | 绑定源、名称映射、默认值 |
| 复杂绑定 | ModelBinder, IModelBinder, BinderProvider, ValueProvider, TryUpdateModel, UpdateModel | 复杂对象展开字段、可覆盖字段 |
| 类型转换 | JsonConverter, TypeConverter, BindAsync, TryParse, IParsable, JsonPatchDocument | 转换逻辑、异常吞并、回退行为 |
| 文件输入 | IFormFile, IFormFileCollection, MultipartReader, OpenReadStream, CopyToAsync | 文件名、内容类型、落盘点 |

## 前置控制抓手

| 类型 | 优先关键字 | 重点记录 |
| ---- | ---- | ---- |
| Middleware | UseRouting, UseAuthentication, UseAuthorization, UseCors, UseSession, UseWhen, MapWhen, ForwardedHeaders | 顺序、短路条件、上下文修改 |
| MVC / API 过滤器 | Authorize, AllowAnonymous, ActionFilter, ResultFilter, ExceptionFilter, ValidateAntiForgeryToken | 执行顺序、短路、参数/结果修改 |
| Endpoint / Hub / GraphQL | EndpointFilter, HubFilter, AuthorizeView, GraphQL middleware | 作用范围、授权覆盖、字段级限制 |
| WCF 行为 | MessageInspector, ServiceAuthorizationManager, BehaviorExtension | 消息修改、身份校验、异常输出 |

## 敏感操作抓手

| 类型 | 优先关键字 | 重点记录 |
| ---- | ---- | ---- |
| SQL | SqlCommand, DbCommand, FromSqlRaw, ExecuteSqlRaw, Query, Execute | SQL 来源、参数对象、动态片段 |
| NoSQL | BsonDocument, FilterDefinition, Builders, Cosmos query | 查询对象来源、拼接片段 |
| CMD | Process.Start, cmd.exe, powershell, ArgumentList | 命令构造、shell 参与度 |
| SSRF | HttpClient, WebRequest.Create, HttpWebRequest, RestClient | URL 来源、协议/主机限制 |
| 文件 | File.ReadAllText, File.WriteAllText, PhysicalFile, CopyToAsync, ExtractToDirectory | 路径来源、规范化、落盘或读取位置 |
| XML / XXE | XmlReader, XmlDocument, XDocument, XslCompiledTransform, DtdProcessing | 解析器配置、外部实体开关 |
| 反序列化 | BinaryFormatter, NetDataContractSerializer, TypeNameHandling, LosFormatter | 类型信息、输入来源、白名单 |
| 模板 / 表达式 | Razor, MarkupString, CSharpScript, Dynamic LINQ, DataTable.Compute | 模板源、上下文、执行权限 |

## 安全配置抓手

| 类型 | 优先关键字 | 重点记录 |
| ---- | ---- | ---- |
| 认证授权 | AddAuthentication, AddAuthorization, Authorize, Policy, Role, Claim | 方案选择、策略绑定、匿名例外 |
| 会话与 Cookie | AddSession, CookieOptions, SameSite, HttpOnly, SecurePolicy | 属性、作用域、跨站行为 |
| CSRF | AddAntiforgery, ValidateAntiForgeryToken, AutoValidateAntiforgeryToken, IgnoreAntiforgeryToken | token 生成与校验、例外路径 |
| 配置 | appsettings, web.config, Configure(T), BindConfiguration, Environment | 安全开关、环境覆盖、运行时改写 |
| 日志 | ILogger, LogInformation, LogError, BeginScope, Serilog, NLog | 敏感字段、异常、审计留痕 |

## 项目内封装补充要求

- 发现 Repository、Client、Gateway、Manager、Executor、Dispatcher、Helper、Extension 等二次封装时，必须把封装名加入当前 Skill 输出中的“项目内抓手”章节
- 发现统一异常处理中间件、统一认证封装、统一 HttpClient 工厂、统一模板渲染器时，必须作为跨 Skill 公共抓手重复引用
- 如果同一项目同时存在 MVC、Minimal API、SignalR、GraphQL、WCF，必须分别建立抓手子集，不得混成单一入口模型
