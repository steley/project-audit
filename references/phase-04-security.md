# Phase 4 — Security Audit

> 执行前置：重读 `audit/01-discovery.md`、`audit/02-architecture.md` 及 `audit/` 中其它已有产物。
> 输出：`audit/04-security.md`

## 方法（所有漏洞类别统一，SKILL.md §3）

先枚举 sink 的具体实例，再从 sink 反向追踪到 source；禁止只凭类别清单报告问题。

以 SSRF 为例，一条完整追踪链：

1. 枚举所有出站 HTTP client
2. 识别攻击者可控的 URL 来源
3. 沿 source → sink 追踪 URL
4. 检查首次网络请求前的 validation
5. 检查 redirect 跟随策略
6. 检查 DNS 解析行为
7. 检查 IPv4 / IPv6 归一化
8. 检查 proxy 环境变量
9. 检查超时 / 响应大小限制
10. 尝试构造具体 exploit 路径

## 检查类别

根据项目实际情况选择适用类别，不要机械套清单。

检查：

- Injection
- XSS
- DOM XSS
- CSRF
- SSRF
- SQL/NoSQL injection
- Command injection
- Path traversal
- File inclusion
- Open redirect
- Header injection
- Host header issues
- Authentication bypass
- Authorization bypass
- Session/cookie issues
- CORS
- Secret leakage
- Information disclosure
- Unsafe file handling
- Deserialization
- Prototype pollution
- Cache poisoning
- Cache key problems
- Rate-limit bypass
- Resource exhaustion / DoS
- ReDoS
- Race conditions
- Dependency/supply-chain risk

对于 URL、SSRF、Proxy、IP、Redirect 尤其检查：

- URL parser differences
- hostname validation
- protocol validation
- localhost / loopback
- private IP
- IPv4 / IPv6
- DNS resolution
- redirects
- DNS rebinding
- encoded IP/hostname
- userinfo
- alternate schemes
- X-Forwarded-For
- X-Real-IP
- Forwarded
- Host
- Origin
- Referer
- User-Agent

安全问题尽可能证明：

Attacker Input
→ Validation
→ Transformation
→ Sink
→ Security Impact

下结论前，先从 lockfile 确认所用框架/库的实际版本及其默认防护行为（参数化查询、自动转义、body 大小限制等），不要按"某框架通常有漏洞"推测。
