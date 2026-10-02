# Phase 3 — Functional & Logic Audit

> 执行前置：重读 `audit/01-discovery.md`、`audit/02-architecture.md` 及 `audit/` 中其它已有产物。
> 输出：`audit/03-logic.md`

检查：

- Incorrect assumptions
- Incorrect conditions
- Missing conditions
- Boundary conditions
- Empty/null/undefined
- Invalid types
- Unexpected values
- Error paths
- Retry
- Timeout
- Partial failure
- State transition
- Race condition
- Resource lifecycle
- Transaction consistency
- Cache consistency
- Idempotency
- Duplicate operations
- Client/server disagreement
- Parser differences
- Encoding / Unicode
- URL encoding
- Time/date
- Numeric precision/overflow
- Fallback behavior

重点寻找：

> 正常路径正确，但异常路径错误。

以及：

> 单个模块正确，但组合后产生错误。

每个真实问题记录证据，给出完整调用链（`file:line → file:line → file:line`）。
