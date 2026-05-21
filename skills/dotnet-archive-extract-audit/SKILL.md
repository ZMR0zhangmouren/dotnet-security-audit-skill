---
name: dotnet-archive-extract-audit
description: .NET 归档解压路径穿越审计工具。识别 ZipArchive、Tar、SharpCompress 等解压逻辑中的 entry name 逃逸与最终目标路径越界风险。
---

# .NET 归档解压审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与压缩包格式、条目名、解压目录、覆盖策略、软链接和后续消费相关的共享抓手
- 若项目存在 ArchiveService、PackageImporter、PluginInstaller、BackupRestore 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐入口、参数绑定与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、静态文件目录、上传/插件目录与运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- ZipArchive、Tar、SharpCompress、第三方解压组件的 extract 调用点与目标目录来源
- ZipArchiveEntry.FullName、entry.Key、tar path 等归档条目名是否包含 ../、绝对路径、盘符、UNC、保留设备名等逃逸片段
- 条目名在规范化、替换分隔符、去前导斜杠、大小写归一化后的最终解析目标是否仍越出 base directory
- 解压目录是否位于 web root、插件目录、模板目录、任务输入目录、可下载目录或后台扫描目录
- 压缩包中的符号链接、硬链接、重复文件名、目录覆盖、文件替换和压缩炸弹防护是否处理
- 解压结果是否会被后续模板渲染、程序集加载、脚本执行、配置读取或静态资源发布链消费

## 枚举规则（强制细化）

- 枚举 ZipArchive、Tar、SharpCompress、SevenZip 封装、自定义解压服务、导入功能中的 extract 调用点
- 优先搜索 extract、unzip、untar、archive、entry、restore、import package、theme/plugin import 等业务语义
- 枚举 entry name 规范化、目标目录拼接、重复文件覆盖、软链接/硬链接条目处理逻辑
- 对每个解压点记录：archive 来源、目标目录、最终解析路径、解压后文件是否进入可访问或可执行区域

证据引用：

- EVID_ARCHIVE_EXTRACT_CALLSITE
- EVID_ARCHIVE_ENTRY_NAME_SOURCE
- EVID_ARCHIVE_FINAL_TARGET

## 输出

- vuln_audit/archive_{timestamp}.md
- 可选：vuln_poc/archive_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
