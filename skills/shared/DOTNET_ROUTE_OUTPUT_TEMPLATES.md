# .NET Route / Trace 输出模板

本文件统一 dotnet-route-mapper 与 dotnet-route-tracer 的产物格式，用于减少不同 Skill、不同批次分析之间的格式漂移。

## 使用规则

- route-mapper 与 route-tracer 的输出结构以本文件为准
- 具体字段可以按专项补充，但不得删除本文件定义的核心章节与关键字段
- 若项目需要额外字段，应在保持兼容的前提下附加，不要修改既有主键语义
- routes_{timestamp}.md 中的 route_id、params_{timestamp}.md 中的路由条目、trace_{timestamp}.md 中的 route_id 必须一致

## 一、route-mapper 输出模板

### 1. routes_{timestamp}.md

最小章节：

- 标题
- 生成时间
- 分析路径
- 路由清单（全部）

每条路由最少字段：

- route_id
- 入口类型
- HTTP 方法或操作名
- 路由路径规则
- 路由声明方式
- Controller / Action 或等价处理器标识
- 处理器
- 位置
- 版本
- Area
- 参数概览
- 返回模板类型

参考骨架：

```md
# {project_name} - 路由与入口索引（.NET）
生成时间: {timestamp}
分析路径: {source_path}

## 路由清单（全部）
=== [1] GET /api/v{version}/users/{id} ===
入口类型: HTTP
HTTP 方法: GET
路由路径规则: /api/v{version}/users/{id}
路由声明方式: Attribute Routing
Controller: UsersController
Action: GetById
处理器: UsersController.GetById
位置: Controllers/UsersController.cs:line
版本: v{version}
Area: 无
参数概览:
  Path: id, version
  Query: includeRoles
  Header: X-Tenant-Id
返回模板类型: JsonResult
```

### 2. params_{timestamp}.md

最小章节：

- 标题
- 生成时间
- 按 route_id 或路由标题分段的参数详细

每条入口最少字段：

- Path 参数表
- Query 参数表
- Body / Form / Header / Cookie / File / Service / Custom Binder 参数表
- 完整请求模板

参考骨架：

````md
# {project_name} - 入口参数结构（逐条）
生成时间: {timestamp}

=== 路由: GET /api/v{version}/users/{id} ===
1) Path 参数
| 参数 | 类型/约束 | 来源 | 必填 | 说明 |
|------|-----------|------|------|------|
| id | int | [FromRoute] / route template | 是 | 用户 ID |

2) Query 参数
| 参数 | 类型 | 来源 | 必填 | 说明 |
|------|------|------|------|------|
| includeRoles | bool | [FromQuery] | 否 | 是否包含角色 |

5) 完整请求模板
```http
GET /api/v1/users/123?includeRoles=true HTTP/1.1
Host: {host}
Authorization: Bearer {token}
X-Tenant-Id: {tenant}
Cookie: {cookie}
```
````

### 3. quality/route_mapper_validation_{timestamp}.md

最小章节：

- 覆盖统计
- 未确定方法或路径组合
- conventional / attribute 路由冲突提示
- 参数绑定不确定项
- 需要 route-tracer 进一步确认的入口清单

## 二、route-tracer 输出模板

### 1. route_tracer/{route_id}/trace_{timestamp}.md

最小章节：

- 请求模板
- 入口与元数据
- 参数进入点
- 绑定链
- 过滤器与中间件
- 业务与 DI 链
- 分支路径追踪
- 参数可控性矩阵
- Sink 定位
- Sink Summary
- Trace 完整性声明
- Sink Evidence Type Checklist

参数可控性矩阵最少字段：

- 参数名
- 绑定源
- 转换器 / 绑定器
- 首次校验点
- 最终使用点
- 可控等级
- 受影响 Sink

Sink Summary 最少字段：

- Sink 类型
- 具体 API
- 代码位置
- 上游参数
- 是否经过编码 / 白名单 / 授权 / 配置保护
- 证据强度

参考骨架：

````md
# Route Trace - {route_id}
生成时间: {timestamp}
入口: GET /api/v1/users/123

## 请求模板
```http
GET /api/v1/users/123?includeRoles=true HTTP/1.1
Host: {host}
Authorization: Bearer {token}
```

## 入口与元数据
- Controller: UsersController
- Action: GetById
- Authorize: Policy=UserRead

## 参数进入点
- id <- Route
- includeRoles <- Query

## 绑定链
- [FromRoute] id -> int id

## 过滤器与中间件
- AuthorizationFilter: UserReadPolicy
- Middleware: UseAuthentication -> UseAuthorization

## 业务与 DI 链
- UsersController.GetById
- IUserService.GetById
- UserRepository.Load

## 参数可控性矩阵
| 参数名 | 绑定源 | 转换器/绑定器 | 首次校验点 | 最终使用点 | 可控等级 | 受影响 Sink |
|------|------|------|------|------|------|------|
| id | Route | 默认绑定 | GetById guard | SqlCommand parameter | 高 | SQL |

## Sink Summary
| Sink 类型 | API | 代码位置 | 上游参数 | 保护措施 | 证据强度 |
|------|------|------|------|------|------|
| SQL | SqlCommand.ExecuteReader | Repository/UserRepository.cs:line | id | 参数化 | COMPLETE |

## Trace 完整性声明
- trace_status: COMPLETE
- 说明: Controller / Binder / Filter / Middleware / DI / Sink 六层均已闭合

## Sink Evidence Type Checklist
- 调用点证据: 有
- 参数来源证据: 有
- 约束证据: 有
- 分支证据: 有
- 环境证据: 有
````

### 2. route_tracer/{route_id}/trace_summary_{timestamp}.md

适用于批量分析时的轻量索引，最小字段：

- route_id
- 入口标识
- trace_status
- 关键 Sink 类型
- 关键未闭合点
- 建议交给的漏洞专项

## 三、字段一致性要求

- route_id 在 routes、params、trace、trace_summary、最终报告中必须保持同一标识
- 返回模板类型、入口类型、处理器标识在 route-mapper 与 route-tracer 中必须语义一致
- trace_status 仅允许 COMPLETE、PARTIAL、UNRESOLVED 三种值
- Sink 类型命名应与 shared/DOTNET_SINK_REFERENCE.md 和具体 vuln_audit 使用的分类保持一致
