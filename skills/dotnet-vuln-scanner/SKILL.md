---
name: dotnet-vuln-scanner
description: .NET 依赖漏洞扫描器。扫描 *.csproj、Directory.Packages.props、packages.config 与 NuGet 锁定文件，匹配已知安全公告并推断受影响入口。
---

# NuGet 供应链漏洞扫描

## 目标

识别项目依赖中的已知漏洞，并将组件漏洞与可能受影响的入口或功能域对齐。

## 输入

- `source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于把受影响组件映射到统一入口与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需把受影响组件和匿名/P0/P1 入口一起排序）

## 必做项

- 解析依赖包名与版本
- 识别影响版本区间
- 输出代码中直接引用该组件的位置
- 推断可能受影响的 route_id 或入口类型

## 输出

- vuln_report/{project_name}_nuget_vuln_report_{timestamp}.md
