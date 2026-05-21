# Session Full Context

## 一、项目背景

本项目最初不是 .NET 项目，而是用户要求先完整理解当前仓库中的 PHP 白盒代码审计 Skill 体系，再基于该 PHP 项目的方法论生成一个对应的 .NET 白盒代码审计 Skill 项目。

原始工作区项目是 PHP-Code-Audit-Skill-main，包含大量 php-*-audit、php-route-mapper、php-route-tracer、php-audit-pipeline 等 Skill。后续在根目录 temp/ 下新建了 dotnet-security-audit-skill，作为与 PHP 项目同构的 .NET 版审计 Skill 库。

## 二、用户需求演进全过程

### 阶段 1：理解和总结原始 PHP 项目

用户最开始的目标是：

- 读取 README 和每个 Skill
- 理解整个项目的设计和原理
- 通过流程图等方式输出 output.md
- 帮助后续学习和改进该项目

围绕这个目标完成了：

- 读取原项目 README.md
- 读取 shared/EVIDENCE_POINT_IDS.md、IO_PATH_CONVENTION.md、SEVERITY_RATING.md 等共享规范
- 读取 php-audit-pipeline、php-route-mapper、php-route-tracer 等核心 Skill
- 抽样读取 php-sql-audit、php-auth-audit、php-vuln-scanner、php-filesystem-audit、php-laravel-audit、php-exploit-chain-audit 等代表性 Skill
- 创建 output.md

随后用户又要求：

- 继续做 1 和 2

这里的 1 和 2 对应：

- 把 output.md 扩展为更适合初学者理解的版本
- 再给出一个项目改进计划

因此又完成了：

- 扩展 output.md，增加误区、术语表、最小工作示例、学习路径等内容
- 创建 project-improvement-plan.md，梳理项目改进方向、阶段性路线图和结构治理建议

### 阶段 2：生成 .NET 版审计 Skill 项目

用户随后提出关键需求：

- 根据当前项目和理解，生成审计 .NET 语言平台的项目
- 新项目保存到根目录 temp 文件夹中
- 项目架构思路按当前 PHP 项目来
- 要全面，不遗漏

因此创建了：

- temp/dotnet-security-audit-skill/

并在其中生成：

- README.md
- shared/IO_PATH_CONVENTION.md
- shared/SEVERITY_RATING.md
- shared/DOTNET_SINK_REFERENCE.md
- shared/EVIDENCE_POINT_IDS.md
- 36 个左右的 dotnet-* Skill 目录及其 SKILL.md

其中包括：

- 基础设施：dotnet-audit-pipeline、dotnet-route-mapper、dotnet-route-tracer、dotnet-vuln-scanner、dotnet-exploit-chain-audit
- 漏洞专项：SQL、NoSQL、CMD、SSRF、XSS、FILE READ、FILE UPLOAD、FILE WRITE、FILESYSTEM、ARCHIVE、XXE、DESER、TPL、LDAP、EXPR、AUTH、CSRF、OPEN REDIRECT、CRLF、SESSION/COOKIE、CONFIG、CRYPTO、LOGIC、LOGGING
- 框架专项：ASP.NET Core、MVC、Web API、Blazor、SignalR、GraphQL、WCF

### 阶段 3：解释概念与查漏补缺

生成项目后，用户又提出一些具体问题和增强要求。

#### 3.1 解释 dotnet-config-audit 中的“证据引用”

用户询问证据引用是什么意思。对此整理并说明了：

- 证据引用不是随手备注，而是结论成立的最小证明契约
- EVID_CFG_CONFIG_LOCATION、EVID_CFG_RUNTIME_SETTING_CODE、EVID_CFG_IMPACT_ASSOCIATION、EVID_CFG_SECURITY_SWITCH_EVIDENCE 等证据点如何约束结论强度
- 当证据不闭合时，只能降级为 待验证 或 环境依赖

#### 3.2 补 Controller 友好的 route-mapper

用户指出：

- .NET 里很常见的是 Controller 路由
- route-mapper 需要更详细支持 Controller 场景

随后将 dotnet-route-mapper 强化为 Controller 友好版本，明确要求覆盖：

- [Route]、[HttpGet]、[HttpPost]、[HttpPut]、[HttpDelete]、[HttpPatch]、[HttpHead]、[HttpOptions]
- [AcceptVerbs]
- Attribute Routing 与 Conventional Routing
- Area、API Versioning
- ControllerBase vs Controller
- FromRoute、FromQuery、FromBody、FromForm、FromHeader、FromServices
- 文件上传参数
- 自定义 ModelBinder、TypeConverter、BindAsync、TryParse 等绑定机制
- IActionResult / JsonResult / FileResult / RedirectResult / ViewResult / ContentResult 的差异化输出模板

#### 3.3 扩充漏洞专项中的枚举规则

用户要求：

- 根据 C# / .NET 语言特性，把各漏洞专项进一步细化
- 不只写重点检查，还要写更具体的枚举规则

因此对大量漏洞专项补了“枚举规则（强制细化）”，明确：

- 应搜索哪些 API / 方法 / 类 / 组件
- 应枚举哪些入口、服务、包装层
- 对每个命中点需要记录哪些字段
- 如何结合参数来源、保护措施、最终 Sink 形成判断

这一步把 .NET 技术栈的真实语义显式写进了各专项，例如：

- ADO.NET、EF Core Raw SQL、Dapper
- HttpClient、WebRequest、Webhook / Callback
- Razor Raw、MarkupString
- XmlReader、XmlDocument、XslCompiledTransform
- BinaryFormatter、NetDataContractSerializer、TypeNameHandling
- CSharpScript、Dynamic LINQ、DataTable.Compute
- AddAuthentication、AddAuthorization、Antiforgery、Cookie、Session、Options 绑定

### 阶段 4：补 framework 专项、route-tracer、shared 抓手索引

用户继续要求：

- framework 专项也补成和漏洞专项一样的三层结构
- dotnet-route-tracer 细化成执行级规范
- 再给 shared 增一份 .NET 专用的搜索关键字字典 / 审计抓手索引

因此完成了：

- 将 7 个 framework 专项统一补成 重点检查 + 枚举规则（强制细化） + 输出要求
- 将 dotnet-route-tracer 从提纲级文档改造成执行级链路追踪规范
- 新增 shared/DOTNET_AUDIT_GRABBER_INDEX.md
- README 补入“本次增强内容”与相关说明

dotnet-route-tracer 的细化重点包括：

- Controller / ModelBinder / Filter / Middleware / DI 六层追踪
- Trace 状态 COMPLETE / PARTIAL / UNRESOLVED
- 参数可控性矩阵、Sink Summary、Trace 完整性声明、Sink Evidence Type Checklist
- 对异步链路、过滤器、中间件、运行时实现、装饰器、工厂等情况的追踪要求

### 阶段 5：统一漏洞专项引用 shared 抓手索引，并补强 pipeline

用户进一步要求：

- 各漏洞专项统一引用 shared 抓手索引
- dotnet-audit-pipeline 显式消费 framework_audit 与 route_tracer 的结构化产物
- README 一并完善

因此：

- 24 个漏洞专项都加上了“共享抓手索引引用”章节
- 强制说明它们默认继承 shared/DOTNET_AUDIT_GRABBER_INDEX.md 的基础抓手
- 明确要求项目内二次封装要写入“项目内抓手”

同时 dotnet-audit-pipeline 被补强为真正的统一编排入口，明确：

- 要读取哪些 shared 规范
- 各阶段的输入 / 输出契约
- route_tracer 和 framework_audit 如何被后续漏洞专项显式消费
- 最终报告应至少包含哪些覆盖矩阵、冲突保留项和待补证风险池

### 阶段 6：为后续扩展抽取 shared 模板

用户随后要求继续提升扩展能力：

- 在 README 中增加后续扩展漏洞类型的说明，声明如何扩展 Skill
- 再把“新增 Skill 模板”抽成 shared 独立文件
- 再抽 framework 专项模板和 route-tracer / route-mapper 配套输出模板，并同步更新 README

因此最终形成了以下 shared 模板体系：

- shared/DOTNET_VULN_SKILL_TEMPLATE.md
- shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md

同时 README 被同步改造成统一入口文档，负责：

- 描述 shared 中有哪些规范与模板
- 描述 route-mapper / route-tracer 的结构化产物如何被消费
- 描述后续如何扩展新 Skill 与产物模板
- 描述 dotnet-audit-pipeline 的编排方式

## 三、当前 shared 体系的职责分工

当前 shared 目录中的主要文件职责如下：

- EVIDENCE_POINT_IDS.md：证据点字典
- IO_PATH_CONVENTION.md：逻辑章节名与物理落盘路径约定
- SEVERITY_RATING.md：风险分级规范
- DOTNET_SINK_REFERENCE.md：.NET 技术栈常见敏感操作 / Sink 参考
- DOTNET_AUDIT_GRABBER_INDEX.md：统一搜索关键字与审计抓手索引
- DOTNET_VULN_SKILL_TEMPLATE.md：新增漏洞专项模板
- DOTNET_FRAMEWORK_SKILL_TEMPLATE.md：新增 framework 专项模板
- DOTNET_ROUTE_OUTPUT_TEMPLATES.md：route-mapper / route-tracer 统一输出模板

## 四、当前 route-mapper / route-tracer / pipeline 的关系

### route-mapper

职责：

- 建立所有入口、参数、返回类型、绑定来源、请求模板的统一底图

标准产物：

- route_mapping/routes_{timestamp}.md
- route_mapping/params_{timestamp}.md
- quality/route_mapper_validation_{timestamp}.md

其标准输出骨架由 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 统一定义。

### route-tracer

职责：

- 针对 route_id，从入口一直追到最终 Sink，输出结构化证据链

标准产物：

- route_tracer/{route_id}/trace_{timestamp}.md
- route_tracer/{route_id}/trace_summary_{timestamp}.md

其标准输出骨架由 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 统一定义。

### dotnet-audit-pipeline

职责：

- 统一编排 route-mapper、auth/static mapping、vuln scanner、route-tracer、framework 专项、漏洞专项、exploit-chain 和最终汇总

其显式消费的关键结构化产物包括：

- routes_{timestamp}.md
- params_{timestamp}.md
- auth_mapping_{timestamp}.md
- high_risk_routes_{timestamp}.md
- route_tracer/{route_id}/trace_{timestamp}.md
- framework_audit/{name}_{timestamp}.md
- vuln_audit/{type}_{timestamp}.md
- vuln_report/{project}_nuget_{timestamp}.md

## 五、关键设计原则

1. 证据契约驱动
   - 高强度结论必须引用对应 EVID_* 证据点
   - 证据不闭合就降级为 待验证 / 环境依赖

2. 入口完整建模
   - route-mapper 不允许只列高危接口或关键接口
   - 必须输出完整路由、参数、处理器、请求模板

3. route_tracer 是高强度漏洞结论的重要前置证据
   - 参数可控性矩阵
   - Sink Summary
   - Trace 完整性声明
   - Sink Evidence Type Checklist

4. framework_audit 是框架边界和运行时控制面的重要前置证据
   - Middleware 顺序
   - Filter / EndpointFilter / HubFilter 覆盖
   - 鉴权策略绑定
   - 宿主 / 代理 / 配置 / 序列化差异

5. shared 统一口径优先
   - 搜索抓手、输出格式、模板骨架、命名规则、路径约定、证据点命名都应尽量在 shared 统一，而不是散落在单个 Skill 内各自维护

## 六、当前项目已完成的主要能力清单

- .NET 项目根 README 已系统化
- shared 规范齐全
- 漏洞专项已有 24 个左右
- framework 专项已有 7 个
- route-mapper 已支持 Controller 友好细化规则
- route-tracer 已支持执行级追踪规范
- 所有漏洞专项已显式引用 shared 抓手索引
- framework 模板 / 漏洞模板 / route 输出模板 已全部抽到 shared
- dotnet-audit-pipeline 已显式定义结构化产物消费关系

## 七、当前还未继续展开但最自然的后续方向

如果后续继续推进，这些方向最自然：

- 抽取 dotnet-audit-pipeline 最终汇总报告模板
- 抽取 dotnet-exploit-chain-audit 的聚合输出模板
- 在 shared 中进一步定义统一的最终报告章节模板和矩阵模板
- 如果新增新的漏洞专项或 framework 专项，优先按 shared 模板起稿

## 八、用户偏好与协作风格（来自本轮互动）

- 用户强调“全面，不遗漏”
- 用户要求把每轮重要改动同步整合到 README
- 用户更偏好把规则显式化，而不是只保留在上下文中
- 用户倾向于把后续扩展能力提前模板化、共享化
- 用户对结构化、体系化整理非常重视

## 九、恢复会话时应优先关注的入口文件

恢复对话时，优先查看：

- README.md
- shared/DOTNET_AUDIT_GRABBER_INDEX.md
- shared/DOTNET_VULN_SKILL_TEMPLATE.md
- shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md
- dotnet-audit-pipeline/SKILL.md
- dotnet-route-mapper/SKILL.md
- dotnet-route-tracer/SKILL.md

## 十、说明

本文件保存的是可用于恢复工作的完整上下文、需求演进、技术决策与项目状态说明，目的是在后续对话中断后，能够快速从零重建当前工作的背景与进度一致性。
