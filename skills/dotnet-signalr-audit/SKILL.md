---
name: dotnet-signalr-audit
description: SignalR 框架专项审计工具。针对 Hub 方法授权、组管理、连接状态、参数验证与跨连接消息边界进行白盒审计。
---

# SignalR 框架专项审计

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（如已由 `dotnet-route-mapper` 生成，用于对齐 Hub 方法、参数来源与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（如需对齐连接建立、Hub 授权和租户/房间边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（如需把 Hub 方法与下游数据库、文件、模板、通知等副作用链做交叉引用）

## 重点检查

- Hub 方法上的 [Authorize] / policy / role / claim 检查、IUserIdProvider、自定义 HubFilter、连接建立时的 claims 映射与权限校验
- Group 加入/退出、广播目标、租户隔离、房间边界、Presence 状态与跨连接消息可见性
- ConnectionId、UserIdentifier、Context.Items、查询字符串令牌、Cookie flags / JWT validation parameters 与 access token 的绑定关系
- 客户端消息参数校验、模型绑定、流式上传、二进制消息和重试机制对服务端副作用的影响
- Backplane、Redis、Azure SignalR、sticky session、Scale-out 与连接迁移导致的授权和状态一致性问题
- Hub 方法是否会触发数据库、文件、命令、模板、通知等高价值操作且缺少额外约束

## 枚举规则（强制细化）

- 枚举所有 Hub、Hub 方法、HubFilter、IUserIdProvider、OnConnectedAsync、OnDisconnectedAsync、Group 管理逻辑与客户端回调方法
- 优先搜索 Hub、SendAsync、Groups.AddToGroupAsync、Clients.All、Clients.User、Clients.Group、Authorize、HubFilter、Context.UserIdentifier、access_token
- 枚举连接建立参数、查询字符串令牌、Cookie 认证、Claims 映射、Group 名构造、租户标识拼接和房间加入条件
- 枚举服务端副作用方法：数据库写入、文件操作、任务下发、通知推送、模板渲染、管理动作，记录其是否只依赖连接态而未做业务鉴权
- 对每个 Hub 方法记录：调用方向、参数来源、授权方式、目标广播范围、状态依赖、scale-out 影响与后续副作用

## 输出要求

- 输出文件：framework_audit/signalr_{timestamp}.md
- 必须按 连接建立与身份、Hub 方法与参数、组与广播边界、scale-out 与状态一致性 四个章节组织结果
- 每个问题点必须写明 Hub 方法名、授权元数据、客户端可控参数、广播目标、下游副作用
- 必须单列 查询字符串令牌、Cookie 认证、access token 回退、Backplane/Azure SignalR 差异 等环境依赖项

## 证据引用

- EVID_AUTH_PATH_PROTECTED_MATCH
- EVID_AUTH_PERMISSION_CHECK_EXEC
- EVID_LOGIC_STATE_TRANSITION
- EVID_SESS_COOKIE_FLAGS

输出类型映射：AUTH、LOGIC、XSS、CFG。
