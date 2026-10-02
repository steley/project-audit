# Phase 6 — Adversarial Audit

> 执行前置：重读 `audit/01-discovery.md`、`audit/02-architecture.md` 及 `audit/` 中其它已有产物。
> 输出：`audit/06-adversarial.md`

假设外部攻击者拥有普通用户权限。

先列出攻击者能控制：

- URL
- Query
- Path
- Body
- Headers
- Cookies
- Files
- External URLs
- JSON/Form data
- Redirect targets
- Browser state
- CLI arguments（若适用）

再寻找危险 Sink：

- HTML / DOM
- innerHTML
- SQL
- Shell
- File system
- HTTP requests
- Redirect
- Template
- Logs
- Database
- Cache
- Browser APIs
- Privileged APIs

尝试建立：

Attacker Control
→ Validation Bypass
→ Dangerous Sink
→ Impact

主动考虑：

- Encoding bypass
- Unicode bypass
- Parser differential
- Type confusion
- URL parsing
- IPv4/IPv6
- Redirect chains
- Error paths
- Race conditions
- Cache behavior
- Boundary conditions
- Unexpected state transitions

不能证明攻击路径时，标记 POTENTIAL。
