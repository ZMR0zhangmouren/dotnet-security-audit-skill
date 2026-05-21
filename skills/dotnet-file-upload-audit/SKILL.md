---
name: dotnet-file-upload-audit
description: .NET 文件上传审计工具。识别 IFormFile、Multipart 处理、保存路径与文件名校验逻辑中的任意上传和可执行上传风险。
---

# .NET 文件上传审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 IFormFile、multipart、内容类型、文件名、落盘路径和后续消费相关的共享抓手
- 若项目存在 UploadService、StorageProvider、MediaManager、TempFileHelper 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐上传入口、文件参数与 `route_id`）
- 可选：`framework_audit/*_{timestamp}.md`（用于补充宿主、静态资源目录和后处理链控制面）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

## 重点检查

- IFormFile.CopyTo/CopyToAsync、MultipartReader、自定义 Stream 写入、第三方上传组件与对象存储 SDK 的保存点
- 文件名、扩展名、双扩展名、大小写绕过、尾部点/空格、Unicode 同形异义、路径分隔符和目录穿越片段处理
- MIME、Content-Type、文件签名、魔数检测、图像重编码、压缩包、Office/PDF 宏与脚本载荷识别逻辑
- 保存目录是否位于 web root、静态文件目录、临时目录、容器共享目录、CDN 同步目录或可被后台任务二次处理的目录
- 上传后是否可通过 URL 直接访问、经下载接口访问、被图像处理/模板渲染/杀毒/转码组件二次消费
- 重命名策略、唯一文件名生成、租户隔离目录、元数据记录与回滚/删除逻辑是否可靠
- Form 绑定、GraphQL Upload、SignalR 流式上传、大文件分片上传与 resumable upload 场景的鉴权与校验一致性

## 枚举规则（强制细化）

- 枚举所有 IFormFile、IFormFileCollection、MultipartReader、分片上传、对象存储直传回调、自定义上传服务入口
- 优先搜索 upload、file、attachment、avatar、image、document、import、chunk、resume、multipart 等参数名
- 枚举文件名生成、扩展名校验、魔数校验、图像重编码、压缩包检查、Office/PDF 安全检查链
- 对每个保存点记录：保存目录、访问方式、是否进入 web root、是否被后台任务或模板/转码服务再次消费

## 证据引用

- EVID_UPLOAD_DESTPATH
- EVID_UPLOAD_FILENAME_EXTENSION_PARSING_SANITIZE
- EVID_UPLOAD_ACCESSIBILITY_PROOF

## 输出

- vuln_audit/upload_{timestamp}.md
- 可选：vuln_poc/upload_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
