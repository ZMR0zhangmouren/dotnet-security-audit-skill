---
name: dotnet-logic-audit
description: .NET 业务逻辑审计工具。识别授权以外的流程绕过、竞态、状态机缺陷、Mass Assignment、租户隔离缺陷与业务时序漏洞。
---

# .NET 业务逻辑审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与业务状态迁移、租户边界、金额计算、审批绕过、批量操作和重放保护相关的共享抓手
- 若项目存在 DomainService、WorkflowEngine、ApprovalService、OrderManager 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于关联业务动作入口、批量接口与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于关联业务前置权限、租户边界与资源归属）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可作为业务状态流转与具体副作用入口的交叉证据，不是前置硬依赖）

重点检查：

- DTO / ModelBinder / JSON Patch / FormCollection / GraphQL Input 对象引发的 over-posting、字段污染和批量赋值问题
- 并发更新、重复提交、库存扣减、优惠券核销、审批流跳步、幂等键缺失与分布式锁缺失
- 多租户 tenantId、organizationId、resource owner、分支机构、项目空间与数据隔离对齐问题
- 业务校验与最终写入、异步消息、后台任务、缓存更新之间的不一致与竞态窗口
- 状态机合法流转、回滚、取消、退款、发货、激活、解绑等关键业务动作是否存在非法路径
- 管理后台、批处理接口、导入导出、定时任务、Webhook 回调与前台接口之间权限和约束不一致
- 审计日志、告警、补偿逻辑和失败回退是否足以防止逻辑缺陷被持续利用

## 枚举规则（强制细化）

- 枚举所有涉及状态变更、批量处理、导入导出、支付、库存、审批、租户切换、优惠券、邀请、权限授予的业务接口
- 优先搜索 create/update/batch/approve/reject/pay/refund/confirm/cancel/assign/import/export/replay/retry 等业务动作
- 枚举控制器、后台任务、消息消费者、Webhook、Hub 方法中对同一资源的多入口操作路径
- 对每个业务动作记录：前置校验、状态机约束、并发控制、幂等保护、权限边界、补偿逻辑

证据建议：重点记录状态流转、业务条件和最终敏感动作之间的链路。

## 输出

- vuln_audit/logic_{timestamp}.md
- 可选：vuln_poc/logic_{timestamp}.md（如能构造复现请求、竞态步骤或环境依赖验证链）
