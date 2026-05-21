---
name: dotnet-open-redirect-audit
description: .NET 开放重定向审计工具。识别 Redirect、LocalRedirect、ReturnUrl、Challenge 回跳和登录后跳转中的目标地址控制风险。
---

# .NET 开放重定向审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与 ReturnUrl、Redirect、导航目标、外部地址白名单和代理前缀处理相关的共享抓手
- 若项目存在 RedirectHelper、LoginFlowService、NavigationGuard、CallbackResolver 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于对齐跳转入口、ReturnUrl 参数与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于补充登录/登出/挑战入口的鉴权边界）
- trace-gate 场景下优先消费：`route_tracer/{route_id}/trace_{timestamp}.md`；如存在 `route_tracer/{route_id}/trace_summary_{timestamp}.md`，可作为摘要索引
- 如通过 `dotnet-audit-pipeline` 执行，当前专项引用的 `route_id` 应与 `cross_analysis/high_risk_routes_{timestamp}.md` 保持一致

重点检查：

- ReturnUrl、redirect_uri、post_logout_redirect_uri、continue、next、returnTo 等参数来源及其在登录、登出、错误回退、Challenge 回跳中的流转
- Redirect、LocalRedirect、RedirectToAction、Results.Redirect、Challenge/Forbid/SignOut 的目标地址构造方式
- IsLocalUrl、allowlist、host allowlist、scheme 限制、端口限制以及多租户子域校验逻辑
- 编码、双斜杠、协议相对地址、反斜杠、大小写、Unicode 点分隔、嵌套 URL 与双重解码绕过
- 代理头、X-Forwarded-Host、PathBase、反向代理重写以及外部登录提供方返回地址校验
- 跳转目标是否承接敏感票据、授权码、一次性 token 或身份恢复流程，从而放大危害

## 枚举规则（强制细化）

- 枚举所有 Redirect、LocalRedirect、RedirectToAction、Challenge、SignOut、returnUrl / next / continue / callback 参数处理逻辑
- 优先搜索 IsLocalUrl、redirect_uri、post_logout_redirect_uri、ReturnUrl、redirectTo、callbackUrl、PathBase 相关代码
- 枚举登录、登出、认证回调、支付回跳、邀请链接、深链跳转、错误回跳等业务场景
- 对每个跳转点记录：目标来源、校验逻辑、外部身份提供方约束、代理头影响、是否携带敏感票据

证据建议：复用 XSS/CFG 的路径归一化思路，并在报告中说明目标校验分支。

## 输出

- vuln_audit/redir_{timestamp}.md
- 可选：vuln_poc/redir_{timestamp}.md（已确认 / 待验证 / 环境依赖 PoC；如不适合直接构造请求，至少给出验证步骤）
