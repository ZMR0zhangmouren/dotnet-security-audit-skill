# Memory Index

本目录用于保存 temp/dotnet-security-audit-skill 项目的会话级持久化上下文，便于后续对话中断后快速恢复一致性。

## 建议恢复顺序

1. 先读 SESSION_FULL_CONTEXT.md
2. 再读 CURRENT_STATE_AND_RESUME.md
3. 最后按需要查看 CHANGELOG_AND_DECISIONS.md

## 文件说明

- SESSION_FULL_CONTEXT.md：完整背景、需求演进、架构设计、关键改动、文件清单、规则沉淀
- CURRENT_STATE_AND_RESUME.md：当前状态、已完成事项、当前目录下的重要 shared 模板、后续最自然的下一步
- CHANGELOG_AND_DECISIONS.md：按时间顺序记录用户请求、采取的改动、重要判断、验证结果

## 当前总览

- 项目位置：temp/dotnet-security-audit-skill
- 项目类型：.NET 白盒代码审计 Skill 库
- 当前状态：已完成基础 shared 规范、漏洞专项与 framework 专项模板体系、route-mapper / route-tracer 统一产物模板、pipeline 编排强化、README 统一整理
- 当前 shared 模板：
  - shared/DOTNET_AUDIT_GRABBER_INDEX.md
  - shared/DOTNET_VULN_SKILL_TEMPLATE.md
  - shared/DOTNET_FRAMEWORK_SKILL_TEMPLATE.md
  - shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md

## 使用原则

- 后续继续扩项目时，以 shared 中的模板和规范为准，不要重新在 README 或单个 Skill 内重复维护另一套结构
- route-mapper 与 route-tracer 的输出骨架以 shared/DOTNET_ROUTE_OUTPUT_TEMPLATES.md 为准
- 新漏洞专项与新 framework 专项应分别使用对应 shared 模板起稿
