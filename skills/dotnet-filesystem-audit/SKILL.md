---
name: dotnet-filesystem-audit
description: .NET 文件系统操作审计工具。聚焦目录创建、权限、链接、删除、重命名、TOCTOU 与宿主文件系统差异，为 FILE、UPLOAD、WRITE 等链路提供可利用性增强证据。
---

# .NET 文件系统操作审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与目录遍历、链接跟随、TOCTOU、权限继承和删除/移动/复制相关的共享抓手
- 若项目存在 StorageManager、PathHelper、ArchiveManager、SyncService 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于关联触发入口、路径来源与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于关联后台操作、管理入口与权限边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可作为文件系统副作用与入口之间的交叉证据，不是前置硬依赖）

重点检查：

- Directory.CreateDirectory、CreateSymbolicLink、File.Move、File.Copy、File.Delete、File.Replace、Directory.Delete 的危险使用方式
- 符号链接、junction、hardlink、网络共享、容器挂载卷、宿主 bind mount 和只读文件系统的行为差异
- 检查与使用分离导致的 TOCTOU，例如 Exists/ACL 检查后再打开、先校验后重命名、先解析后写入
- GetFullPath/realpath 等规范化后的落点是否在后续步骤被重新拼接、重定向或替换
- 临时目录、缓存目录、日志目录、导出目录、上传目录之间的权限继承和跨目录移动
- Windows ACL、Linux chmod/chown、IIS/Kestrel/容器服务账户权限以及 impersonation 行为
- 删除、覆盖、替换和清理逻辑是否可作为链式利用前置条件，例如删除锁文件、替换配置、覆盖计划任务输入

## 枚举规则（强制细化）

- 枚举所有 Directory.CreateDirectory、File.Move、File.Copy、File.Delete、File.Replace、Directory.Delete、符号链接创建与解析相关 API
- 优先搜索 temp、cache、upload、export、log、backup、staging、working、quarantine 等目录语义
- 枚举 ACL 变更、权限继承、容器卷挂载、网络共享、junction、symlink、hardlink 的创建和使用位置
- 对每个文件系统操作记录：触发入口、路径来源、竞态窗口、后续链式利用价值

关键证据：EVID_FS_TOCTOU_OR_LINK_BEHAVIOR。

## 输出

- vuln_audit/fs_{timestamp}.md
- 可选：vuln_poc/fs_{timestamp}.md（如能构造链接、竞态、覆盖或跨目录写入验证步骤）
