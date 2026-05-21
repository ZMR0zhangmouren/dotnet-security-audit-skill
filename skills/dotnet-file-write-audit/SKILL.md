---
name: dotnet-file-write-audit
description: .NET 任意文件写入审计工具。识别用户可控数据进入写入 Sink 的路径，并评估写后可访问性、可包含性与可执行性。
---

# .NET 文件写入审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与路径拼接、目录创建、覆盖写入、临时文件、权限和原子替换相关的共享抓手
- 若项目存在 FileStore、ConfigWriter、ExportService、DocumentWriter 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐写入入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、可访问目录和运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- File.WriteAllText、WriteAllBytes、FileStream、StreamWriter、AppendAllText、Save/Export/Generate 报表等写入点
- 用户输入或外部数据如何进入目标路径、文件名、目录段、扩展名或 blob/object key
- Path.Combine/GetFullPath 后是否做 base directory 前缀校验，校验后是否又重新拼接导致绕过
- 写入内容是否来自请求体、模板内容、日志伪造、压缩包解压结果、反序列化数据或远程拉取内容
- 写后文件是否可通过静态资源、下载接口、模板引擎、程序集加载、计划任务或后台作业被进一步消费
- 覆盖、追加、截断、原子替换、临时文件再 rename、权限继承和 ACL 设置是否扩大影响
- 本地磁盘、网络共享、对象存储、容器卷与宿主挂载目录的访问面差异

## 枚举规则（强制细化）

- 枚举所有 WriteAllText、WriteAllBytes、AppendAllText、StreamWriter、Save、Export、Generate、template save、config save 操作
- 优先搜索 path、dest、target、filename、output、export、report、template、config 等目标路径字段
- 枚举路径拼接、临时文件落盘、rename/move、对象存储 key 生成、容器卷挂载落点选择逻辑
- 对每个写入点记录：路径来源、内容来源、覆盖模式、写后访问面、是否可能进入模板/配置/静态资源/程序集消费链

## 证据引用

- EVID_WRITE_CALLSITE
- EVID_WRITE_DESTPATH_JOIN_AND_NORMALIZATION
- EVID_WRITE_CONTENT_SOURCE_INTO_WRITE
- EVID_WRITE_EXECUTION_ACCESSIBILITY_PROOF

## 输出

- vuln_audit/write_{timestamp}.md
- 可选：vuln_poc/write_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
