---
name: dotnet-crypto-audit
description: .NET 加密与密钥安全审计工具。识别弱算法、硬编码密钥、不安全模式、随机数与签名校验缺陷。
---

# .NET 加密与密钥安全审计

## 共享抓手索引引用

- 默认以 shared/DOTNET_AUDIT_GRABBER_INDEX.md 作为本 Skill 的基础检索索引
- 枚举时至少覆盖与算法选择、密钥生成、密钥存储、DataProtection、证书和令牌签名相关的共享抓手
- 若项目存在 CryptoHelper、KeyStore、TokenSigner、PasswordHasherWrapper 等二次封装，必须补充到输出中的项目内抓手章节
- 当本 Skill 的专项枚举规则比共享索引更细时，以本 Skill 为准，但不得缩减共享索引的基础覆盖范围

## 输入依赖

- 必需：`source_path`
- 可选：`route_mapping/routes_{timestamp}.md` 与 `route_mapping/params_{timestamp}.md`（用于关联登录、签名、加解密调用入口与 `route_id`）
- 可选：`auth_audit/auth_mapping_{timestamp}.md`（用于关联认证、票据与权限边界）
- 可选：`route_tracer/{route_id}/trace_{timestamp}.md` 与 `route_tracer/{route_id}/trace_summary_{timestamp}.md`（可作为算法/密钥缺陷影响到具体入口的交叉证据，不是前置硬依赖）

重点检查：

- MD5、SHA1、DES、3DES、RC2、ECB、CBC 固定 IV、弱 PBKDF 参数、不安全 HMAC 和自定义加密封装
- Random、Guid、时间戳、可预测 nonce、RNGCryptoServiceProvider / RandomNumberGenerator 的使用边界与替换错误
- 密钥、IV、salt、pepper、JWT signing key、DataProtection key、加密证书、API secret 的来源、轮换和存储位置
- ASP.NET Core DataProtection、DPAPI、Azure Key Vault、证书存储、HSM、User Secrets 与环境变量中的密钥管理模式
- 加密结果是否缺少完整性校验、AEAD、重放保护、版本号、上下文绑定或租户隔离
- 密码哈希、令牌签名、文件加密、cookie 保护、remember-me token 和 magic link 生成逻辑
- 日志、异常、调试输出、配置快照和前端返回中是否泄露敏感密钥材料

## 枚举规则（强制细化）

- 枚举所有 HashAlgorithm、Aes、RSA、ECDsa、ProtectedData、DataProtection、JwtSecurityTokenHandler、自定义 CryptoHelper 的使用点
- 优先搜索 MD5、SHA1、ECB、CBC、IV、salt、key、secret、signing、encrypt、decrypt、Protect、Unprotect 等关键字
- 枚举密码哈希、票据签名、文件加密、字段加密、魔术链接、remember-me token、API 签名、Webhook 签名场景
- 对每个加密点记录：算法、模式、随机性来源、密钥来源、轮换策略、输出使用场景和完整性保护情况

证据引用：

- EVID_CRYPTO_ALGO_SELECTION
- EVID_CRYPTO_KEY_SOURCE
- EVID_CRYPTO_MODE_PADDING_RANDOMNESS

## 输出

- vuln_audit/crypto_{timestamp}.md
- 可选：vuln_poc/crypto_{timestamp}.md（如能构造复现请求、弱配置验证步骤或环境依赖验证链）
