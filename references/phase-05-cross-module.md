# Phase 5 — Cross-Module Audit

> 执行前置：重读 `audit/01-discovery.md`、`audit/02-architecture.md` 及 `audit/` 中其它已有产物。
> 输出：`audit/05-cross-module.md`

不要只逐文件检查。

建立并追踪：

A
→ B
→ C
→ D

重点检查：

- Validation assumptions
- Trust boundary
- Authorization decisions
- State transitions
- Error propagation
- Data transformation
- Resource ownership
- Security checks
- Fallback paths
- Cache behavior

特别寻找：

- Validation gap
- Validation bypass
- State confusion
- TOCTOU
- Race condition
- Error-path vulnerability
- Parser differential
- Encoding mismatch
- Type confusion
- Security check after dangerous operation

每个问题给出完整跨模块调用链，例如：`src/a.ts:23 → src/b.ts:87 → src/c.ts:142`。
