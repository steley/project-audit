# Phase 8 — Finding Normalization

> 执行前置：重读 `audit/03-logic.md`、`audit/04-security.md`、`audit/05-cross-module.md`、`audit/06-adversarial.md`、`audit/07-quality.md` 的全部内容（单会话模式下已在上下文中时不必重读）。
> 输出：`audit/08-findings.md`

将前面各阶段的 Findings 汇总、去重、统一编号。

## 合并规则

- 多个 Finding 属于同一个根因 → 合并为一条，在 Data Flow 中列出全部表现位置。
- 明显不成立的 → 标 FALSE_POSITIVE 并写明理由，不要直接丢弃（供验证阶段复核）。

## Verdict 与证据等级绑定（SKILL.md §3）

每条 Finding 标注 Evidence Level（E0–E5），verdict 不得高于证据等级允许的上限：

- E0（仅模式匹配）→ 不得单独成 Finding，直接丢弃或并入备注
- E1（局部代码证据）→ 最高 POTENTIAL
- E2（跨函数数据流）→ 最高 LIKELY
- E3（端到端可达路径，静态即证）→ 可 CONFIRMED
- E4 / E5（运行时复现 / 利用已演示）→ CONFIRMED 强证据

同时受档位约束：small 档证据上限 E3；medium 档 E4 仅限 HIGH/Critical（SKILL.md §1.3）。

## Severity 取值

- CRITICAL
- HIGH
- MEDIUM
- LOW
- INFO

## LOW 封顶

LOW 完整展开最多 5 条（按潜在实际影响挑），其余 LOW 一行一条列入产物末尾附录（ID / 位置 / 一句话描述）。不要让 LOW 淹没报告，也不要为凑数把 LOW 写成全文。

## 编号

为每条 Finding 建立唯一 ID：`AUDIT-001`, `AUDIT-002`, ...

## 每条 Finding 必须包含

- ID
- Verdict
- Evidence Level（E0–E5）
- Severity
- Category
- Title
- File
- Line
- Function
- Description
- Root Cause
- Trigger Condition
- Data Flow
- Actual Impact
- Exploitability（安全问题）
- Evidence（分三段：**Static** file:line 与代码引用；**Dynamic** 运行时实验结果，若做过；**Reproduction** 复现命令 + 期望/实际行为，若可得）
- Environmental Assumptions（分层写：Source-level fact / Runtime behavior / Deployment configuration / Assumption，见下）
- Recommendation

## Environmental Assumptions 分层写法

前提必须分层声明，避免"理论上可以、生产上不成立"的误报：

- **Source-level fact**：代码中确实存在的（例：代码调用了 `fetch(userInput)`）
- **Runtime behavior**：该语言/框架版本的默认行为（例：Node 22 fetch 默认跟随 redirect；从 lockfile/框架源码确认）
- **Deployment configuration**：部署层的前提（例：前面有 Cloudflare、生产关闭了 debug）
- **Assumption**：纯假设（例：假设攻击者能控制 X）——必须显式标注为假设

本文件将交给独立验证者当作**待检验的 claim 清单**使用：请把"声称的位置与影响"和"推导过程"都写清楚，便于后续逐条复核与推翻。
