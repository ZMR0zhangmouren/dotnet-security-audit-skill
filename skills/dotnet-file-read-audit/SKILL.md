---
name: dotnet-file-read-audit
description: .NET 文件读取与路径穿越审计工具。识别 File、FileStream、PhysicalFileResult 等读取点中的路径控制与目录逃逸风险。
---

# .NET 文件读取审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与路径来源、规范化、虚拟路径映射、下载接口和文件流读取相关的共享抓手
- 若项目存在 FileProvider、AttachmentService、ExportReader、StorageGateway 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐下载/读取入口、参数来源与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、静态文件/模板目录和运行时控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- Path.Combine、Path.Join、GetFullPath、GetRelativePath、PhysicalFileProvider、IFileProvider 的路径规范化边界
- File.ReadAllText、OpenRead、FileStream、MemoryMappedFile、PhysicalFileResult、VirtualFileResult、StaticFileMiddleware 读取点
- 用户输入控制文件名、相对路径、扩展名、目录段、租户目录或 blob key 时的路径逃逸可能性
- UNC 路径、网络共享、符号链接、junction、容器挂载、大小写差异和 Windows 保留设备名处理
- 读取目标是否包含配置文件、证书、源码、视图文件、日志、上传目录、NuGet 包缓存或 Secrets 文件
- 读取和执行边界，例如读取 Razor/View/程序集/配置是否会进一步影响模板执行、程序集加载或配置注入
- 下载接口的 content-type、content-disposition、缓存头和鉴权条件是否造成额外信息泄露

## 枚举规则（强制细化）

- 枚举 File.ReadAllText、ReadAllBytes、OpenRead、FileStream、PhysicalFileResult、VirtualFileResult、StaticFile 映射、自定义下载器
- 优先搜索 path、file、filename、template、view、config、download、export、doc、attachment 等参数名
- 枚举所有 Path.Combine、GetFullPath、IFileProvider、PhysicalFileProvider、ContentRoot/WebRoot 相关路径拼接逻辑
- 对每个读取点记录：目标根目录、规范化逻辑、鉴权条件、输出方式、是否可能读到配置/源码/证书/日志/上传文件

## 证据引用

- EVID_FILE_RESOLVED_TARGET
- EVID_FILE_PATH_NORMALIZATION
- EVID_FILE_EXECUTION_BOUNDARY

## 输出

- vuln_audit/file_{timestamp}.md
- 可选：vuln_poc/file_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
