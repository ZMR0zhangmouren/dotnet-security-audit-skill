---
name: dotnet-audit-pipeline
description: 用于 .NET Web 白盒安全审计的总编排 skill。适用于需要统一执行路由映射、参数枚举、鉴权映射、route trace、框架专项审计、漏洞专项审计、PoC 落盘与最终报告汇总的场景；按 Sink 类型联动 dotnet-route-mapper、dotnet-route-tracer、dotnet-auth-audit 与各类 dotnet-*-audit skills，并复用统一分级、证据与输出目录标准。
---

# .NET 全链路代码审计流水线

目标：对 .NET 语言平台项目输出一份可复核、可修复、可量化的白盒审计报告，而不是零散的危险点列表。

## 运行依赖说明（避免误导）
本 skill 在当前安装形态下是“文档编排/方法论”版本，通常不会在本地仓库中提供可执行脚本（例如 `scripts/*.py`）。
当环境中不存在对应执行器时，应直接使用文档里描述的阶段化流程（route-mapper / auth-audit / vuln-scanner / route-tracer 等）进行源码分析与报告生成；如果看到“正在寻找执行器脚本”的提示，可将其视为“未找到执行器，改走阶段化降级流程”，不要把它当作路径错误。

## 关键约束（强制）

### 完整性约束

1. **禁止省略**：不得出现任何省略占位符；所有清单必须逐项完整输出。
2. **禁止模板未替换**：所有请求/PoC 模板必须用真实路由与真实参数完成替换，不得保留 `${var}`。对于请求头/Body 中的主机与会话字段，统一使用 `{host}`/`{cookie}`/`{token}` 等占位符，不得出现未替换的 `{{...}}`。
3. **禁止只给结论**：每个高危漏洞必须包含“位置证据 + 数据流链 + 可利用性分析 + 验证 PoC + 修复建议”。
4. **非漏洞必须有解释**：对于可疑模式，必须给出“被硬编码覆盖/被校验拦截/绕过无效/仅 Blind 无法验证”等结论与证据。

### 输出完整性校验（强制）

完成后必须执行以下检查清单：

- [ ] 路由清单与参数清单“非空且数量一致”（若拆分到多个文件，分别检查各自完整性）
- [ ] 鉴权映射表覆盖所有接口（或标明“此接口为静态资源/非入口/无路由绑定”但同样要落表）
- [ ] 漏洞清单中每条漏洞均有：编号、等级、位置、数据流链、可利用性、验证 PoC、修复建议
- [ ] 报告内没有未替换的模板占位符（例如 `${var}` 或未替换的 `{{...}}`）

## 输入

用户提供：

- `source_path`：.NET 项目源码路径（可为解决方案目录、仓库根目录或单项目目录）
- `output_path`：输出目录路径（默认 `{source_path}_audit`）

派生变量（用于保证各子 skill 输出可对齐）：

- `project_name`：从 `source_path` 目录名提取（basename）
- `timestamp`：生成时间戳，建议格式为 `YYYY_MM_DD_HH_mm`（用于文件名示例：`2026_03_20_18_00`）

## 输出目录结构（建议强制）

```
{output_path}/
├── route_mapping/
├── auth_audit/
├── route_tracer/
├── vuln_audit/
├── vuln_poc/
├── framework_audit/
├── cross_analysis/
├── vuln_report/
├── exploit_chain/
├── global_sink_fallback/
├── quality/
└── {project_name}_代码审计_{timestamp}.md
```

说明：

- 后文出现的 `routes_{timestamp}.md`、`auth_mapping_{timestamp}.md`、`vuln_audit/sql_{timestamp}.md` 等“文件路径”，默认指对应阶段目录中的真实中间产物。
- 最终汇总阶段应消费这些中间产物并生成总报告，而不是以内联章节替代阶段文件。
- **与子 skill 的路径统一**：各子 skill 中出现的 `route_mapping/routes_*.md` 等与上表**逻辑等价**，详见 `shared/IO_PATH_CONVENTION.md`（流水线合并 vs 独立落盘）。



## 编排原则

- dotnet-audit-pipeline 是唯一总入口，负责统一调度 route-mapper、route-tracer、framework_audit、vuln_audit、exploit-chain 与最终汇总
- 所有子 Skill 的输入输出都要按逻辑章节和物理落盘路径双重对齐，不得把“报告章节存在”误当成“结构化产物完整”
- dotnet-security-audit-skill 默认采用 强制落盘 模式；执行 pipeline 时，必须优先真实创建 route_mapping / auth_audit / route_tracer / vuln_audit / vuln_poc / framework_audit / cross_analysis / vuln_report / exploit_chain / global_sink_fallback / quality 等目录，再汇总成总报告
- route_tracer 产物是大多数高强度结论的主证据来源，framework_audit 产物是框架边界、运行时配置与前置控制面的主证据来源，二者都必须被显式消费
- 所有漏洞专项默认继承 shared/DOTNET_AUDIT_GRABBER_INDEX.md 的基础抓手，并在各自输出中补充项目内二次封装
- vuln_poc 是 vuln_audit 的配套产物，默认应被纳入输出链；即使只能给出待验证或环境依赖 PoC，也不得省略 vuln_poc 目录
- pipeline 必须保留 未追踪入口、未执行专项、trace 未闭合、环境依赖、框架边界不确定项，不得静默跳过


## 漏洞专项执行状态模型（强制）

所有 dotnet-*-audit 漏洞专项在本次 pipeline 中都必须被记录为以下状态之一：

| 状态             | 含义                                                       |
| ---------------- | ---------------------------------------------------------- |
| NOT_RUN          | 本次未执行；必须说明未执行原因                             |
| INITIAL_SCREENED | 已完成基础枚举/关键字抓取/入口粗筛，但尚未进入完整专项分析 |
| PARTIAL          | 已进入专项分析，已有部分证据或局部 trace，但结论未完全闭合 |
| COMPLETED        | 已完成专项分析并形成正式输出                               |
| NOT_APPLICABLE   | 对当前项目、入口或技术栈不适用                             |

约束：

- 默认执行中，非重点漏洞类型允许停留在 INITIAL_SCREENED 或 PARTIAL
- 即使没有漏洞发现，也必须记录为 COMPLETED + NO_FINDING，或 INITIAL_SCREENED / PARTIAL + 当前阻断原因
- 不允许某个 dotnet 漏洞专项 skill 在最终报告中完全消失

## framework_audit 与 route_tracer 的显式消费规则

- 若 route_tracer 已给出 COMPLETE/PARTIAL 的结构化 trace，vuln_audit 必须优先消费其中的 参数可控性矩阵、Sink Summary、Trace 完整性声明、Sink Evidence Checklist
- 若 framework_audit 已指出 中间件顺序、Filter 覆盖、序列化器选型、授权策略绑定、运行时开关 问题，相关 vuln_audit 必须显式引用这些框架结论，而不能重复从零判断
- 对同一路由，framework_audit 与 route_tracer 出现冲突时，pipeline 必须把冲突保留下来，并在最终报告中列入“证据冲突与待复核项”
- 没有 trace 证据时，允许 framework_audit 先给出 框架级风险提示；但落到高强度漏洞结论时，仍需受 trace gate 约束

## vuln_poc 的显式消费规则

- vuln_poc 必须引用对应的 vuln_audit 结论编号和 route_id，不能脱离漏洞报告单独存在
- 若 route_tracer 已给出完整请求模板、参数可控性矩阵和 Sink Summary，vuln_poc 应优先复用这些信息构造最小可复现请求
- 若 framework_audit 指出关键环境前提，如代理、鉴权、中间件顺序、文件访问面、序列化器开关，vuln_poc 必须把这些前提写入触发条件
- 若漏洞结论仅为 待验证 / 环境依赖，vuln_poc 也必须输出对应状态，而不是缺失

## 调度与分发规则

- Controller / MVC / Web API 入口：至少联动 dotnet-aspnet-core-audit、dotnet-mvc-audit 或 dotnet-webapi-audit、相关 vuln_audit、必要 route_tracer
- Minimal API 入口：至少联动 dotnet-aspnet-core-audit、route_tracer、相关 vuln_audit
- Blazor 入口：至少联动 dotnet-blazor-audit、相关后端 API 的 route_tracer，以及 AUTH/XSS/LOGIC/SESS 等相关专项
- SignalR 入口：至少联动 dotnet-signalr-audit、route_tracer、AUTH/LOGIC/XSS 等相关专项
- GraphQL 入口：至少联动 dotnet-graphql-audit、route_tracer、AUTH/LOGIC/SSRF/XSS 等相关专项
- WCF 入口：至少联动 dotnet-wcf-audit、route_tracer、AUTH/DESER/XXE/CFG 等相关专项
- 文件、模板、表达式、序列化等跨入口风险：必须从 route_tracer 反向映射到所有命中的入口，而不能只看单一 Controller 或 Service

## 强制约束

- 不得省略任何清单或矩阵行
- 所有 trace-gate 类结论都必须引用共享 EVID_* 证据点
- trace_status=PARTIAL/UNRESOLVED 时，不得直接标记为 ✅已确认可利用
- 对未闭合 trace、静态兜底命中和未执行专项，必须保留理由与后续动作
- 所有 vuln_audit 必须显式说明其是否消费了 framework_audit 与 route_tracer 产物，若未消费必须解释原因
- 所有 framework_audit 与 vuln_audit 必须默认继承 shared/DOTNET_AUDIT_GRABBER_INDEX.md 的基础抓手，不得各写各的最小搜索口径
- 默认必须真实创建 route_mapping / auth_audit / route_tracer / vuln_audit / vuln_poc / framework_audit / cross_analysis / vuln_report / exploit_chain / quality 目录，不得仅以内联逻辑章节替代
- vuln_poc 不得被视为可选附加项；若专项无法形成已确认 PoC，也必须输出待验证或环境依赖 PoC
- 每个阶段目录必须有 index_{timestamp}.md；空结果阶段必须有空结果说明
- 所有已纳入本次执行范围的 dotnet-* skills 都必须在最终报告中有执行状态记录，不允许遗漏未执行或仅初筛的专项







## 执行流程（阶段化流水线：更高真阳性）

### 漏报抑制（强制，贯穿各阶段）

- **禁止静默丢弃**：`trace_status = UNRESOLVED`、未进入 `SINK_SUSPECT` 启发式列表、或 trace 契约未通过时，**不得**在报告中消失。须在阶段 5「质量报告章节」的 **「Trace 未闭合 / 待补证风险池」**（或等价标题）中逐条列出：`route_id`（或 `SINK_ONLY:…`）、已知代码位置/片段、缺失证据类型、建议补做动作（重跑 trace、扩大 `high_risk_routes`、或对该文件走对应 `dotnet-*-audit` 静态深审）。
- **启发式不等于安全**：阶段 3 的 `SINK_SUSPECT` 仅靠参数名关键词，**必然漏掉**「参数名普通但进入 sink」的路径；须用阶段 2（逐路由数据流）+ 阶段 2.5（全局 sink 兜底）+ 各子 skill 的 sink 扫描互相补齐。
- **阶段 2.5 与阶段 4 的关系**：2.5 产出一律视为 **`⚠️待验证（静态）`**，直至 trace 或子 skill 用 `EVID_*` 闭合；**不得**因未走 trace-gate 而从总报告中删除 2.5 条目。
- **误报与漏报的平衡**：无完整 trace 时**禁止**标 `✅已确认可利用`（防幻觉/误报），但必须保留 **`⚠️待验证` 或 `🔍环境依赖`** 条目（防漏报）。

### 阶段 1：侦察（Recon）

目标：识别全部“入口→路由→参数→认证链”的基础数据。

1. **识别技术栈/入口形态（必做）**
   - ASP.NET Core / 通用宿主：`Program.cs`、`Startup.cs`、`WebApplicationBuilder`、中间件注册顺序、`MapControllers`、`MapGroup`、`MapFallback`
   - MVC / Web API：Controller、Action、Attribute Routing、Conventional Routing、Area、API Versioning
   - Minimal API：`MapGet/MapPost/MapMethods`、Route Group、Endpoint Filter
   - Blazor：`.razor` 页面、组件事件、`AuthorizeView`、`IJSRuntime`、Blazor Server / WebAssembly / 交互式 SSR
   - SignalR / GraphQL / WCF：Hub 方法、Query/Mutation/Subscription、`ServiceContract/OperationContract`
   - 非 HTTP 合成入口：`Main`、`BackgroundService`、`IHostedService`、定时任务、消息消费者、队列处理器、Webhook 回调处理器
2. **枚举路由/入口（必做，禁止省略）**
   - 从路由配置、Endpoint 注册或框架约定中提取：入口类型、HTTP 方法或操作名、路径/规则、对应 handler（文件/类/方法/委托）
   - 对动态路由（如 `{id}`、路由约束、Area、版本路由、MapGroup 前缀、Hub 方法、GraphQL 操作、WCF Operation）必须写出“解析规则与可控参数列表”
3. **枚举参数（必做）**
   - 识别 Source：`[FromRoute]`、`[FromQuery]`、`[FromBody]`、`[FromForm]`、`[FromHeader]`、`Request.Cookies`、`ClaimsPrincipal`、`Session`、`IFormFile`
   - 识别 JSON / XML / GraphQL 变量 / SOAP Body：模型绑定、Input Formatter、JsonConverter、XmlSerializer、GraphQL variables、WCF 消息体
   - 识别文件：`IFormFile`、`IFormFileCollection`、流式上传对象、下载名与 `Content-Disposition` 相关参数来源
4. **生成统一入口底图（必做）**
   - 调用：`dotnet-route-mapper`
   - 目标：统一产出 `routes_{timestamp}.md`、`params_{timestamp}.md`，并为后续 `auth_mapping`、`route_tracer`、`vuln_poc` 与最终汇总提供统一 `route_id`

5. **识别认证链与会话机制（必做）**
   - Cookie / Identity / Session：`AddAuthentication`、`AddCookie`、`AddIdentity`、`UseAuthentication`、`UseSession`
   - JWT / API Key / 自定义票据：`JwtBearer`、`TokenValidationParameters`、`Authorization: Bearer`、签名头或租户头解析
   - 授权链：`[Authorize]`、`AllowAnonymous`、`RequireAuthorization`、Policy、Role、Claim、资源归属校验、Hub/Resolver/WCF 授权
   - 自定义鉴权：`PermissionChecker`、`AuthorizationHandler`、`TenantResolver`、`OwnershipValidator` 等二次封装

6. **生成鉴权映射（静态映射，先不依赖 trace）**
   - 调用：`dotnet-auth-audit`
   - 模式：`STATIC_MAPPING`（仅生成 `auth_mapping_{timestamp}.md` 用于 P0/P1 路由分桶）
   - 约束：当尚未生成 `route_tracer/` trace 时，允许“缺少 trace 契约证据点”，但必须输出“鉴权状态分级 + P0/P1 分桶依据”

7. **供应链漏洞扫描（组件已知 CVE/Advisory）**
   - 调用：`dotnet-vuln-scanner`
   - 输出：`vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md`

8. **框架特效审计（可选：检测到对应框架才调用）**
   - 检测原则（必做）：依据 `.sln`、`*.csproj`、`Program.cs` / `Startup.cs`、`appsettings*.json`、`.razor`、GraphQL/SignalR/WCF 注册代码与典型目录特征识别实际框架面
   - ASP.NET Core 中间件与宿主：调用 `dotnet-aspnet-core-audit`，输出 `framework_audit/aspnet_core_{timestamp}.md`
   - MVC 视图型控制器：调用 `dotnet-mvc-audit`，输出 `framework_audit/mvc_{timestamp}.md`
   - Web API / REST API：调用 `dotnet-webapi-audit`，输出 `framework_audit/webapi_{timestamp}.md`
   - Blazor：调用 `dotnet-blazor-audit`，输出 `framework_audit/blazor_{timestamp}.md`
   - GraphQL：调用 `dotnet-graphql-audit`，输出 `framework_audit/graphql_{timestamp}.md`
   - SignalR：调用 `dotnet-signalr-audit`，输出 `framework_audit/signalr_{timestamp}.md`
   - WCF：调用 `dotnet-wcf-audit`，输出 `framework_audit/wcf_{timestamp}.md`

阶段 1 输出：

- `routes_{timestamp}.md`：完整路由清单（逐条）
- `params_{timestamp}.md`：每条路由参数来源与类型/结构（逐条）
- `auth_audit/auth_mapping_{timestamp}.md`：路由鉴权映射（含 P0/P1 分桶依据）
- `quality/route_mapper_validation_{timestamp}.md`：路由映射质量校验结果（用于核对遗漏入口、参数绑定异常、枚举失败原因）
- `vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md`：组件漏洞清单（用于推断受影响路由）
- `framework_audit/*_{timestamp}.md`：框架特效审计报告（仅在阶段 1 检测到对应框架时产出）

### 阶段 2：建模（Modeling）

目标：建立 Source → Sink 数据流图的“可追踪版本”。

对每个路由，建立至少一条“从入口参数到敏感 Sink 的最短路径”，并记录：

- Source 参数名/数据结构字段路径（如 `request.Sort.OrderBy`、`input.Filters[0].Field`、GraphQL variables、Hub payload 字段路径）
- 中间变量名变化（assignment/concat/transform）
- 分支条件（if/switch/try/catch）与提前返回（return/throw），以及 Filter / Middleware / Endpoint Filter / HubFilter 的短路影响
- Sink 类型与调用点（文件/行号/函数名）

阶段 2 输出：

- 路由级数据流链证据（建议嵌入到 `vuln_audit/*` 或单独建 `dataflow_tracer/`，这里保持简洁：内联到漏洞报告中也可）

### 阶段 2.5：Sink-only 全局扫描兜底（提升召回，覆盖 RCE 及扩展高危）

目标：在路由枚举/trace-gate 失败或证据不足时，仍能通过“静态 Sink 命中 + 本地可控性回溯 + 执行点确认”找到高危点，并一律降级为 `⚠️待验证（静态证据）`，**直至**阶段 3/4 补证。

> **与阶段 3/4 的差异**：阶段 2.5 面向全部 .NET 源文件与模板文件做“宽谱 Sink 初筛”，不依赖路由完整性与 trace 完整性，产出进入降级候选池；阶段 3/4 则在路由+trace 完成后，针对 trace-gate 通过的 Sink 做“高置信度精确审计”。两者扫描范围可能重叠，但执行时机、目标精度和输出流向不同，阶段 2.5 的产出不会替代阶段 4 的确认结果，仅作为补漏兜底。

1. 扫描范围（不限定目录）
   - 从 `source_path` 根目录开始递归遍历，覆盖项目内所有相关源码与模板文件（如 `*.cs`、`*.cshtml`、`*.razor`、`*.config`、`*.xaml`，必要时包含 `*.vb`）。
   - 为减少噪音可默认排除 `bin/`、`obj/`、`.vs/`、发布产物、测试快照与缓存目录（如需最大召回可把排除策略关闭）。

2. **必覆盖 Sink 集合（最低集）**（防 RCE/对象注入漏报）
   - CMD：`Process.Start`、shell 包装器、脚本启动器、`cmd` / `powershell` 参数拼接
   - EXPR：`CSharpScript.EvaluateAsync`、Dynamic LINQ、`DataTable.Compute`、NCalc、自定义规则引擎
   - TPL：Razor Runtime Compilation、邮件模板引擎、可控模板名/模板内容渲染链
   - DESER：`BinaryFormatter`、`NetDataContractSerializer`、启用 `TypeNameHandling` 的 Newtonsoft.Json、多态类型反序列化链
3. **扩展扫描（强烈推荐，防 SQL/SSRF/任意读等漏报）**：在业务代码目录上，按 `shared/DOTNET_SINK_REFERENCE.md` 再扫一轮 **SQL 执行族**（`FromSqlRaw`、`ExecuteSqlRaw`、Dapper、`SqlCommand` 动态 SQL）、**SSRF 族**（`HttpClient`、`WebRequest.Create`、Webhook/回调 URL）、**任意读/写/上传/解压族**（`File.ReadAllText`、`File.WriteAllText`、`IFormFile.CopyTo`、`ZipArchive.ExtractToDirectory`）。命中写入同一 `sink_static_index_{timestamp}.md`（或扩展 `rce_sinks` 附录），条目标注「扩展扫描」；结论仍为 `⚠️待验证`，直至 trace 或对应 `dotnet-sql-audit` / `dotnet-ssrf-audit` / `dotnet-file-read-audit` / `dotnet-file-write-audit` / `dotnet-file-upload-audit` / `dotnet-archive-extract-audit` 闭合。

4. 每个命中点的本地分析（必须输出）
   - 位置：`file:line`、函数/语句、sink 参数表达式（原样摘录关键片段）
   - 可控性回溯：在同一文件/相邻调用链内追踪 sink 参数是否来自请求输入（`Request.Query`、`RouteValues`、`Request.Form`、JSON Body 模型绑定、`Request.Headers`、`Request.Cookies`、`IFormFile`、GraphQL variables、Hub payload、SOAP Body 等）
   - 执行点确认：若无法证明危险上下文真的会执行（例如分支提前 return/异常跳过/被过滤），结论必须降级为 `⚠️待验证`

5. 输出与汇总规则（不依赖 route_id）
   - 输出到 `{output_path}/global_sink_fallback/`：
     - `rce_sinks_{timestamp}.md`：RCE/高危执行点静态命中清单（每条包含位置、可控性回溯证据、执行点确认与结论降级理由）
     - `sink_static_index_{timestamp}.md`：静态命中索引（用于质量统计与后续手工补全 trace）
   - 阶段 5 汇总时：在“质量报告章节”的“漏洞列表总览”中额外引用 `global_sink_fallback/rce_sinks_{timestamp}.md`，并将这些项标注为“静态证据（非 trace-gate）”。

### 阶段 3：分批追踪（Only When Trace Proves）

目标：通过skills  `dotnet-route-tracer` 的“实际执行证据”，把漏洞挖掘从“关键字扫描”升级为“可控性与分支证据驱动”，最大化真阳性。

阶段 3 必须按以下顺序执行：

1. **生成高危路由集合**：
   - 读取 `auth_audit/auth_mapping_{timestamp}.md`，提取 P0（❌无鉴权）与 P1（⚠️仅认证 或 🔓可绕过/越权/IDOR 的风险路由）
   - 读取 `vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md`，把与解析入口/敏感功能相关的组件漏洞触发点“推断映射”到可能受影响路由，并标注“推断理由”
   - 基于静态参数线索补充 `SINK_SUSPECT` 路由（不依赖 trace 证据点，trace-gate 在阶段 4 再做硬门槛确认）
     - LDAP：参数名/字段名包含 `dn/baseDn/filter/searchFilter/ldap/filterString` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - EXPR：参数名/字段名包含 `expr/expression/formula/rule/script/condition/where/sortExpression` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - SQL：参数名/字段名包含 `sql/query/filter/orderBy/sort/limit/search` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - CMD：参数名/字段名包含 `cmd/command/process/fileName/arguments/executable/shell/run` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - SSRF：参数名/字段名包含 `url/uri/target/host/port/protocol/path/webhook/callback/baseAddress` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - FILE：参数名/字段名包含 `file/path/filename/download/view/document/template/physicalPath` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - UPLOAD：参数名/字段名包含 `upload/file/formFile/filename/contentType` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - WRITE：参数名/字段名包含 `write/save/output/path/destination/export/logPath/tempPath` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - XSS：参数名/字段名包含 `content/body/comment/title/description/html/markup` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - TPL：参数名/字段名包含 `template/view/razor/render/layout/partial/markup` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - XXE：参数名/字段名包含 `xml/doctype/dtd/xslt/document` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - DESER：参数名/字段名包含 `payload/serialized/typeName/type/discriminator/object` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - NOSQL：参数名/字段名包含 `filter/query/pipeline/operator/regex/bson` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - REDIR：参数名/字段名包含 `next/returnUrl/redirect/target/url/callbackUrl/postLogoutRedirectUri` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - CRLF：参数名/字段名包含 `header/cookie/location/set-cookie/fileName/contentDisposition` 等关键词（或其在路由 params 中的 JSON 字段路径）
     - ARCHIVE：参数名/字段名包含 `zip/tar/archive/entry/extract/unzip` 等关键词（或其在路由 params 中的 JSON 字段路径）
   - **合成 `route_id`（强制）**：条目来自 CLI、后台任务、定时作业、消息队列消费者、事件处理器、阶段 2.5 静态 sink 等**且无 HTTP 路由**时，须使用稳定 ID（禁止留空），并在同行写明 `entry_file`（相对 `source_path`）、`entry_symbol`（函数/方法或 `Main` / 顶级语句）、`trace_from`（从何处开始向 sink 追踪）。推荐：`ENTRY_CLI:{脚本标识}`、`ENTRY_WORKER:{服务或作业名}`、`ENTRY_CRON:{任务名}`、`ENTRY_QUEUE:{消费者类或文件}`、`ENTRY_MESSAGE:{消息处理器}`、`SINK_ONLY:{序号}:{基名}:{行号}`（可与 `sink_static_index` 交叉引用）。
   - 输出：`cross_analysis/high_risk_routes_{timestamp}.md`（含 P0/P1/SINK_SUSPECT；**每条须有 `route_id`**（含合成 ID）。skills `dotnet-route-tracer` 以 `route_id` 标识任务，追踪起点以 `entry_file` + `entry_symbol` + `trace_from` 为准，不要求 ID 形如 URI。）
2. **生成分批追踪计划**：
   - 读取 `high_risk_routes_{timestamp}.md`
   - **分批参数化（强制）**：顺序 **P0 → P1 → SINK_SUSPECT**。`trace_batch_max_size` **默认 10**，可在 **5–30** 间按项目调整；须在计划中写**选型理由**（例：同模块可合并加大批次；调用链深、跨文件多、参数面宽或上下文预算紧则减小批次）。
   - `trace_batch_plan_{timestamp}.md` **每一批**须含：`batch_id`、本批采用的 `trace_batch_max_size`、**选型理由**（1–3 句）、本批 `route_id` 有序列表。
   - 输出：`cross_analysis/trace_batch_plan_{timestamp}.md`
3. **批量调用追踪器**（必须）：
   - 对每批路由调用skills :  `dotnet-route-tracer` 
   - 输出：`route_tracer/{route_id}/trace_{timestamp}.md`
4. **触发条件（硬门槛）**：
   - 只有当某类 sink 在 trace 中出现“真实 sink 执行点”，且 trace 契约校验通过，且对应参数在 trace 的可控性矩阵里不是“不可控/无实际使用（被硬编码覆盖）”，才允许进入阶段 4 触发对应 `dotnet-*-audit`

5. **trace 契约校验（必须）**：
   - 每个 trace 文件必须包含以下段落/表头（字段名必须一致，不得重命名）：
     - `## 7) Sink Summary`
     - `## 5) 参数可控性矩阵`
     - `## 6) Sink 定位`
     - `## 8) Trace 完整性声明`
     - `## 9) Sink Evidence Type Checklist`
   - 如果 trace 契约校验失败：
     - 必须标记该 route 的 trace 为 `PARTIAL` 或 `UNRESOLVED`
     - 对所有由该 trace 支持的漏洞，状态必须降级为 `⚠️待验证`，且不得直接给出 `✅已确认可利用`。

Sink 类型抓手、危险模式与强制验证的完整条文见本仓库 **`shared/DOTNET_SINK_REFERENCE.md`**。阶段 3/4 据 trace 筛选是否触发对应 `dotnet-*-audit` 时，以该文件为准。

阶段 3 输出：

- `cross_analysis/high_risk_routes_{timestamp}.md`
- `cross_analysis/trace_batch_plan_{timestamp}.md`
- `route_tracer/` 下每条追踪路由的 `trace_{timestamp}.md`（含参数可控性表与分支路径证据）
- `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可选）：单路由追踪摘要，便于阶段 4/5 汇总引用

### 阶段 4：Sink 审计执行（Only When Trace Proves）

步骤：

1. 从 `route_tracer/` 的 trace 中筛选需要“trace-gate”的漏洞类别与“参数实际使用状态”（必须以 trace 可控性矩阵为准）。
   - trace-gate 类别：SQL/NOSQL/CMD/SSRF/FILE/UPLOAD/WRITE/ARCHIVE/XSS/REDIR/CRLF/XXE/DESER/TPL/LDAP/EXPR/AUTH/CSRF/SESS
   - 非 trace-gate 类别：CFG/CRYPTO/LOGIC/LOG（可基于静态扫描与代码配置证据直接调用对应 skill，**不以** trace `## 9) … CFG` **作为**唯一门槛）。说明：`dotnet-route-tracer` 的检查表中虽含 CFG 行，仅作**增强证据**；`dotnet-config-audit` 等仍须在阶段 4 **无条件按计划执行**（除非覆盖矩阵中已用否定证据标为不适用）。
2. 仅对命中的类别调用对应子审计 skill；并满足以下“证据来源”规则（禁止仅凭关键字推断）：
   - 若类别属于 trace-gate：必须逐项引用 `dotnet-route-tracer` 的 `## 9) Sink Evidence Type Checklist` 对应行（FILE/UPLOAD/WRITE/ARCHIVE/SSRF/SQL/NOSQL/CMD/XSS/REDIR/CRLF/XXE/DESER/TPL/LDAP/EXPR/AUTH/CSRF/SESS）的证据要点作为审计依据（要求证据点 ID 原样一致）
   - 若类别为 `AUTH`：调用 `dotnet-auth-audit` 的 `TRACE_AUDIT` 模式（必须引用 trace 证据点 ID），输出 `auth_audit/auth_audit_report_{timestamp}.md`
3. 对每条疑似漏洞，必须输出“可利用性前置条件”：

- 鉴权要求：✅无需 / ⚠️需登录 / ❌不可利用门槛
- 输入可控性：✅完全可控 / ⚠️条件可控 / ❌不可控（硬编码覆盖）
- 触发条件：分支条件/异常路径/环境依赖（数据库类型、网络可达性）

4. **trace 完整性回退规则（必须）**：
   - 若 trace_status = PARTIAL：只能进入 `⚠️待验证` 模式（不得直接给出 `✅已确认可利用`）
   - 若 trace_status = UNRESOLVED：**禁止静默漏报**——不得写成「无风险」或从报告中消失。必须在总报告 **「Trace 未闭合 / 待补证风险池」** 中为该 `route_id` 单列一条：`已知片段`（来自 trace 的可达部分）、`缺失证据`、`建议下一步`（补 trace / 纳入 `SINK_ONLY` 静态深审 / 调用对应 `dotnet-*-audit`）。若该路由与阶段 2.5 静态命中可关联，须写交叉引用。

5. **文件系统操作审计（非 trace-gate，静态证据驱动）**：
   - 调用：`dotnet-filesystem-audit`
   - 输出：`vuln_audit/fs_{timestamp}.md`
   - 判定规则：允许基于源代码定位的“文件系统操作证据链 + 路径控制来源 + TOCTOU/链接绕过解释”直接输出 `✅/⚠️/❌/🔍`，不以 `dotnet-route-tracer` 的证据契约字段为前置硬门槛

并最终给出：

- 状态：✅已确认 / ⚠️待验证 / ❌不可利用 / 🔍环境依赖
- 严重等级与 CVSS 映射（按上方公式）

阶段 4 输出：

- `vuln_audit/*_{timestamp}.md`：各 trace-gate 或静态证据驱动专项的审计结果
- `vuln_poc/*_{timestamp}.md`：与专项结果配套的验证请求、待验证 PoC 或环境依赖 PoC
- `auth_audit/auth_audit_report_{timestamp}.md`：`dotnet-auth-audit` 在 `TRACE_AUDIT` 模式下的鉴权链审计报告
- `vuln_audit/fs_{timestamp}.md`：`dotnet-filesystem-audit` 的文件系统操作审计结果

### 阶段 5：汇总报告（Report）

在总报告中生成“质量报告章节”，包含：

- **子 skill / 审计面覆盖矩阵**：与上文「子审计覆盖矩阵（强制）」同结构，置于质量报告靠前位置；须与阶段 4 实际执行情况一致，不得矛盾。

- 风险统计表（🔴/🟠/🟡/🔵 数量与 CVSS 区间）

- 漏洞列表总览（逐条引用本报告中对应的漏洞详情段落）

- **「Trace 未闭合 / 待补证风险池」**（强制）：汇总 `trace_status ∈ {PARTIAL, UNRESOLVED}` 的 `route_id`、阶段 2.5 `sink_static_index` 中尚未被 trace 闭合的条目、以及「扩展扫描」命中项；与「漏洞列表总览」并列，**禁止**因证据不足而删除记录（未闭合 ≠ 无风险）。

- 覆盖率：路由覆盖、鉴权覆盖、Sink 覆盖、关键分支覆盖（用“已完成/待补充”而非省略；建议阈值：路由覆盖与 Sink 覆盖 >= 90%）

- 框架特效审计汇总：如果存在 `framework_audit/*.md`，需要将其风险映射到统一类型码（AUTH/CSRF/TPL/XSS/SQL/LOGIC/CFG/SESS/SSRF/…），统计数量与最高严重等级，并在汇总中注明“框架风险来源于哪些 framework_audit 报告文件”

   > **框架审计豁免说明**：框架特效审计（ASP.NET Core、MVC、Web API、Blazor、GraphQL、SignalR、WCF）属于“静态配置/模式匹配”型审计，其发现的风险不需要经过 `dotnet-route-tracer` 的 trace-gate 或 `## 9) Sink Evidence Type Checklist` 的证据校验。框架审计结果直接按其自身规则分级并映射到统一类型码即可。

- 证据完整度评分（Evidence Completeness）：对所有 trace-gate sink 类型（FILE/UPLOAD/WRITE/ARCHIVE/SSRF/SQL/NOSQL/CMD/XSS/REDIR/CRLF/XXE/DESER/TPL/LDAP/EXPR/AUTH/CSRF/SESS），按 `shared/EVIDENCE_POINT_IDS.md` 中定义的证据点 ID 集合进行计数：`覆盖率% = 已满足证据点数 / 该类型所需证据点总数 × 100%`；并在表格中列出缺失的证据点 ID（必须原样一致）与缺失原因（如 trace_status=PARTIAL/UNRESOLVED、证据点定位失败、参数不可控等）。

阶段 5.1：利用链聚合（强制）

- 调用：`dotnet-exploit-chain-audit`
- 输出：`exploit_chain/exploit_chains_{timestamp}.md`
- 在“质量报告章节”的“漏洞列表总览”之后增加一段“利用链聚合章节”：引用利用链聚合报告的链路总览表与最短利用链（只需引用、不重复内容）。