# Changelog And Decisions

## 时间线与关键决策

### 1. 从 PHP 项目理解开始，而不是直接生成 .NET 项目

决策：先完整读取原始 PHP Skill 项目，再生成 .NET 项目。

原因：

- 用户明确要求“按当前项目架构思路来”
- 若不先理解 PHP 项目的方法论，生成的 .NET 项目会变成表面命名相似、但内部协作关系不一致的集合

结果：

- 先完成 output.md 和 project-improvement-plan.md
- 再基于理解生成 .NET 项目

### 2. .NET 项目采用“同构迁移”而不是“最小可用集合”

决策：按 PHP 项目模式生成较完整的 .NET Skill 集，而不是只做几个示例。

原因：

- 用户要求“全面，不遗漏”
- .NET 的漏洞面与框架面都需要系统化覆盖

结果：

- 生成基础设施 Skill
- 生成漏洞专项 Skill
- 生成 framework 专项 Skill
- 生成 shared 规则文件

### 3. Controller 友好的 route-mapper 是关键补丁

决策：把 dotnet-route-mapper 升级为 Controller 友好版。

原因：

- 用户指出 .NET 中 Controller 定义路由极为常见
- 如果 route-mapper 只会泛泛识别入口，后续 route-tracer 和漏洞专项都不稳定

结果：

- 引入 Attribute / Conventional 路由细化
- 引入 Area、Versioning、Binder、返回类型模板差异

### 4. 漏洞专项必须从“重点检查”升级到“重点检查 + 枚举规则”

决策：把各漏洞专项的执行层规则补齐。

原因：

- 光有重点检查不足以保障执行一致性
- 用户明确要求结合 .NET / C# 语言特性做更具体的枚举规则

结果：

- 24 个左右漏洞专项加入 .NET 平台语义化枚举规则

### 5. framework 专项也要和漏洞专项一样三层化

决策：framework 专项补成“重点检查 + 枚举规则 + 输出要求”。

原因：

- 只写框架要点不足以支撑 pipeline 和漏洞专项稳定消费

结果：

- 7 个 framework 专项统一为三层结构

### 6. route-tracer 升级为执行级规范

决策：route-tracer 不再只是“入口到 Sink”的简要描述，而要有 Controller / Binder / Filter / Middleware / DI 六层追踪要求。

原因：

- 用户明确点名这几层需要细化
- 这是高强度结论的主要证据来源

结果：

- route-tracer 输出结构被强制标准化

### 7. shared 必须承载统一抓手与模板，不再把规则散落在各 Skill

决策：逐步把扩展规则和输出骨架抽到 shared。

原因：

- 用户非常重视后续扩展能力和前后统一
- 模板与骨架散落在 README 和个别 Skill 中，会造成双份维护

结果：

- 新增 shared/DOTNET_AUDIT_GRABBER_INDEX.md
- 新增 shared/DOTNET_VULN_SKILL_TEMPLATE.md
- 新增 shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- 新增 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md

### 8. dotnet-audit-pipeline 要从“说明文档”变成“协调入口”

决策：显式定义 pipeline 对 framework_audit 与 route_tracer 的消费关系。

原因：

- 用户要求 pipeline 能统一完整地协调调用各入口 Skill
- 如果不显式定义消费关系，最终还是会退化成各专项分散输出

结果：

- pipeline 明确阶段输入输出契约
- 明确覆盖矩阵、待补证风险池、证据冲突项等要求

## 已创建或重点修改的关键文件

- README.md
- dotnet-route-mapper/SKILL.md
- dotnet-route-tracer/SKILL.md
- dotnet-audit-pipeline/SKILL.md
- 多个 dotnet-*-audit/SKILL.md
- shared/EVIDENCE_POINT_IDS.md
- shared/IO_PATH_CONVENTION.md
- shared/SEVERITY_RATING.md
- shared/DOTNET_SINK_REFERENCE.md
- shared/DOTNET_AUDIT_GRABBER_INDEX.md
- shared/DOTNET_VULN_SKILL_TEMPLATE.md
- shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md

## 验证策略

每轮大改后都进行了错误检查，关键结果是：

- 多次修复 Markdown 表格、代码块、重复标题、换行等问题
- 当前 shared 模板、README、route-mapper、route-tracer 等关键文件均已校验通过

## 后续恢复时的判断原则

- 若要扩新漏洞专项，优先用 DOTNET_VULN_SKILL_TEMPLATE.md
- 若要扩新 framework 专项，优先用 DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- 若要修改 route-mapper / route-tracer 产物格式，优先改 DOTNET_ROUTE_OUTPUT_TEMPLATES.md，再同步消费方
- 若要扩共享检索规则，优先改 DOTNET_AUDIT_GRABBER_INDEX.md
- 若要扩最终汇总格式，优先考虑再抽 shared 的 pipeline / report 模板
