# .NET 漏洞 PoC 输出模板

本文件用于统一 vuln_poc 目录下各漏洞专项的 PoC 产物格式，使其能够被 dotnet-audit-pipeline、人工复核和 exploit-chain 聚合稳定消费。

## 适用范围

- 适用于已完成 vuln_audit 且具备可复现条件的漏洞专项
- 适用于 SQL、CMD、SSRF、XSS、文件上传、开放重定向、XXE、反序列化、表达式执行等可以构造请求或触发链路的场景
- 不适用于完全无法构造触发条件、仅能输出框架级风险提示的场景；这类情况应在 PoC 文件中明确标注为“待验证”或“环境依赖”而不是省略

## 使用规则

- 默认输出文件为 vuln_poc/{type}_{timestamp}.md
- 每个 vuln_poc 文件必须与对应的 vuln_audit/{type}_{timestamp}.md 建立引用关系
- 每个 PoC 条目必须写明关联的 route_id、漏洞编号、触发前提、请求模板和预期结果
- 当不满足直接复现条件时，也必须输出“待验证 PoC”或“环境依赖 PoC”，不能因为无法完全复现而不生成 vuln_poc 目录
- 请求模板中的真实路由、参数、Header、Cookie、Body、文件名应尽可能来自 route-mapper / route-tracer 产物，不应凭空捏造

## 模板

````md
# {project_name} - {type} PoC
生成时间: {timestamp}
关联漏洞报告: vuln_audit/{type}_{timestamp}.md

## PoC 列表

### POC-001 - {finding_id}
- route_id: {route_id}
- 漏洞编号: {finding_id}
- 风险等级: {severity}
- 可利用性: 已确认 / 待验证 / 环境依赖
- 触发前提:
  - {condition_1}
  - {condition_2}
- 关联证据:
  - {route_tracer_evidence}
  - {framework_audit_evidence}
  - {vuln_audit_evidence}

### 请求模板

```http
POST /example HTTP/1.1
Host: {host}
Authorization: Bearer {token}
Content-Type: application/json

{"key":"value"}
```

### 预期结果

- {expected_behavior}

### 验证说明

- {notes}
````

## 最低验收要求

- 每个 PoC 条目都能对应到 vuln_audit 中的具体漏洞编号或发现编号
- 每个 PoC 条目都能对应到 route_id 或明确的非 HTTP 入口标识
- 每个 PoC 条目都要写明 已确认 / 待验证 / 环境依赖
- 请求模板与 route-mapper / route-tracer 产物在入口、方法、参数名上保持一致
- 不允许只写“可自行构造请求”而不给出最小请求模板
