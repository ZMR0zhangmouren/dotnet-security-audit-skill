# Current State And Resume

## 当前状态

temp/dotnet-security-audit-skill 当前已经从“生成的第一版 .NET Skill 项目”演进成“具备 shared 统一规范、扩展模板、route/trace 统一产物格式、pipeline 显式编排关系”的较完整骨架。

## 已完成事项

### 项目级

- 根 README 已不只是简介，而是统一入口文档
- shared 目录已有规则文件、模板文件、输出骨架文件
- memory 目录已加入，用于保存会话持久化上下文

### route 侧

- dotnet-route-mapper 已支持 Controller 友好规则
- dotnet-route-tracer 已支持执行级追踪规范
- 二者的输出骨架已统一到 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md

### framework 侧

- framework 专项已具备三层结构
- framework 模板已抽到 shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md

### vuln 侧

- 漏洞专项已有 shared 抓手索引引用章节
- 漏洞专项模板已抽到 shared/DOTNET_VULN_SKILL_TEMPLATE.md

### pipeline 侧

- dotnet-audit-pipeline 已显式定义 structure-aware 消费关系
- 明确消费 routes、params、auth mapping、trace、framework_audit、vuln_audit、vuln_report 等产物

## 当前重要 shared 文件

- shared/DOTNET_AUDIT_GRABBER_INDEX.md
- shared/DOTNET_VULN_SKILL_TEMPLATE.md
- shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
- shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md
- shared/DOTNET_SINK_REFERENCE.md
- shared/EVIDENCE_POINT_IDS.md
- shared/IO_PATH_CONVENTION.md
- shared/SEVERITY_RATING.md

## 当前最推荐的恢复入口

如果后续重新接手，建议按以下顺序读文件：

1. README.md
2. shared/DOTNET_AUDIT_GRABBER_INDEX.md
3. shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md
4. shared/DOTNET_VULN_SKILL_TEMPLATE.md
5. shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
6. dotnet-audit-pipeline/SKILL.md
7. dotnet-route-mapper/SKILL.md
8. dotnet-route-tracer/SKILL.md

## 当前最自然的后续工作

如果继续迭代，这几个方向最自然：

1. 抽 dotnet-audit-pipeline 最终汇总报告模板
2. 抽 dotnet-exploit-chain-audit 的统一聚合模板
3. 如果要扩新漏洞专项，先从 DOTNET_VULN_SKILL_TEMPLATE.md 起稿
4. 如果要扩新 framework 专项，先从 DOTNET_FRAMEWORK_SKILL_TEMPLATE.md 起稿
5. 如果要变更 route / trace 输出格式，先改 DOTNET_ROUTE_OUTPUT_TEMPLATES.md

## 当前协作要求

- 用户非常重视“全面、不遗漏”
- 用户要求重要改动同步整合到 README
- 用户偏好显式规则、显式模板、显式共享化，而不是仅靠上下文记忆
- 用户希望断线后可以快速恢复到之前的工作进度和设计意图

## 说明

本文件用于快速恢复当前项目的状态与下一步入口；更完整的历史与背景请结合 SESSION_FULL_CONTEXT.md 和 CHANGELOG_AND_DECISIONS.md 一起阅读。
