---
name: dotnet-cmd-audit
description: .NET 命令执行审计工具。识别 Process.Start、shell 参数拼接与脚本执行风险，输出可利用性分级、PoC 与修复建议。
---

# .NET 命令执行审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与命令构造、shell 选择、参数传递、工作目录、环境变量相关的共享抓手
- 若项目存在 ShellHelper、JobRunner、ScriptExecutor、DeployManager 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主平台、外部命令运行环境与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- Process.Start、ProcessStartInfo.FileName、Arguments、ArgumentList、UseShellExecute、Verb 等调用点
- cmd.exe、powershell.exe、pwsh、bash、sh、wscript、cscript、msbuild、dotnet tool 等间接执行链
- 用户输入进入命令名、参数、工作目录、环境变量、脚本文件路径或标准输入的路径
- 是否使用 ProcessStartInfo.ArgumentList 安全分隔参数，还是退回到整条命令行字符串拼接
- 文件上传、导出、压缩、图像处理、备份、git/svn/hg 操作、ffmpeg、wkhtmltopdf 等常见外部工具调用
- Windows 与 Linux 宿主差异、引号规则、转义规则、空格与 shell 元字符处理差异
- 低权限服务账户、容器、沙箱、AppLocker、ConstrainedLanguageMode 等缓解条件是否真实存在

## 枚举规则（强制细化）

- 枚举所有 Process.Start、ProcessStartInfo、cmd/powershell/bash 包装器、自定义脚本执行服务
- 优先搜索 FileName、Arguments、ArgumentList、UseShellExecute、RedirectStandardInput、WorkingDirectory、EnvironmentVariables
- 枚举文件转换、备份恢复、git/svn、压缩解压、图像处理、PDF 生成、诊断脚本、CI/CD helper 这类常见外部命令调用业务
- 对每个执行点记录：命令构造方式、参数来源、是否经过白名单、宿主平台、服务账户权限、输出是否回显

## 证据引用

- EVID_CMD_EXEC_POINT
- EVID_CMD_COMMAND_STRING_CONSTRUCTION
- EVID_CMD_USER_PARAM_TO_CMD_FRAGMENT

## 输出

- vuln_audit/cmd_{timestamp}.md
- 可选：vuln_poc/cmd_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
