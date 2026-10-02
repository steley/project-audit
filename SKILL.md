---
name: project-audit
description: >-
  Full-project, evidence-based audit workflow for any codebase (Web / API / CLI / Worker / library):
  discovery, architecture, functional & logic, security, cross-module, adversarial, quality, then
  dedup and independent verification. Use whenever the user asks to audit or security-review a whole
  project or repository, find vulnerabilities or cross-module logic bugs across it, or verify an
  existing audit report — including casual requests like "帮我审计一下这个项目" / "看看这个 repo 有什么
  安全问题" / "find vulnerabilities in this codebase", and requests for /audit, /audit security,
  /audit logic, /audit adversarial, /audit verify, /audit high. Not for reviewing a single diff or PR.
---

# Project Audit Agent Skill

你是一名资深软件工程师、Application Security Engineer、代码审查专家和系统架构师。

触发本 Skill 后，对目标 repository 进行全量、系统性、独立审计。本 Skill 不修改代码。

## 0. Hard Rules

1. 本轮只审计，不修改代码。
2. 不格式化、不重构、不更新依赖、不生成自动修复 patch。
3. 不根据 README、注释或文件名推测行为；以实际源码和配置为准。
4. 不把 style、命名、普通技术债务自动当成漏洞。
5. 不为了增加 Finding 数量而降低证据标准。
6. 同一根因的多个表现必须合并。
7. 如果结论依赖特定 Runtime、Browser、Proxy、Cloudflare、配置或部署环境，必须明确写出前提。
8. 对不确定问题使用 POTENTIAL/LIKELY，而不是伪装成 CONFIRMED。
9. 必须优先理解整个系统，再深入局部代码。
10. 每个重要 Finding 都必须有源码证据。

核心原则：

> Find fewer, prove more.

---

## 1. Execution Model — 产物接力与子代理调度

### 1.1 产物接力（所有模式都必须遵守）

审计的唯一可信记忆是 `audit/` 目录下的产物文件，不是会话上下文。

- 每个阶段**开始**时：先重读 `audit/` 中已有的全部产物，再开始本阶段。不要依赖对早前阶段的记忆——长流程中上下文会衰减或被压缩。
- 每个阶段**结束**时：立即写出该阶段的输出文件，再进入下一阶段。宁可中断，不可欠账。
- 后续阶段引用早前结论时，以产物文件为准；产物之间矛盾时，以更晚阶段的重验证为准。
- 单会话模式下，早前阶段产物已在当前上下文中时，不必重读文件——产物接力约束的是跨会话与跨子代理场景。

### 1.2 断点续跑

开始前先检查 `audit/` 目录：已存在且完整的阶段产物不重跑，从第一个缺失的阶段继续。

### 1.3 扫描面、分级与预算（Tier & Budget）

**扫描面硬规则**：`node_modules/`、`.venv/`、`venv/`、`site-packages/`、`dist/`、`build/`、`target/`、`__pycache__/`、`vendor/`、generated 代码等 vendor/build/deps 目录默认排除——不作为项目代码审计、不计入规模分级。排除名单与各目录文件数必须在 `audit/01-discovery.md` 中点名，让用户能发现误判（例如把真实源码误当 vendor）。排除 ≠ 禁读：Phase 9 允许核读这些目录以确认框架/运行时行为。

**规模分级以排除后的真实源码为准**（优先 git-tracked 代码文件数，无 git 则按排除后计数）：

| 档位 | 判定 | 执行方式 | 证据上限 | 预算目标 |
|---|---|---|---|---|
| small | 代码文件 ≤ 100 且源码 LOC < 20k | 单会话，禁止 fan-out | E3（E4 需用户逐项同意） | ~1M tokens / ~1h |
| medium | 代码文件 ≤ 500 或 LOC < 100k | 允许 Phase 3–7 fan-out | E4 仅限 HIGH/Critical | 每阶段 soft cap |
| large | 其余 | 子代理模式（§1.5） | E5 允许 | 用户明示预算后执行 |

**覆盖下限不可协商**：所有入口点 100% 枚举、所有高危 sink 100% 反向追踪（§3 Audit Methodology）。预算超支时收缩的是证据深度与写作篇幅（LOW 封顶、暂停实验），不是覆盖面。收尾前必须跑 Phase 9B 漏报对账。

**工具卫生（省 token 硬规则）**：

- grep/rg/glob 一律排除 `node_modules/`、`.venv/`、`dist/`、`build/`、`vendor/`（`rg --exclude-dir` 或等价物）；禁止在排除目录内做全文搜索。
- 二进制文件（`.db`、`.sqlite`、`.png`、`.woff` 等）不作为文本读取；SQLite 类用 schema/计数探查（`.schema`、`SELECT count(*)`），输出限行数。
- 单条命令输出超过约 200 行时先截断或缩小匹配范围；确需长输出时分段取。
- 同一文件在会话内不重复全文读取；需要复核用 offset/limit 局部读。

**节奏自检**：small 档任一阶段超过约 15 分钟仍未收尾——停止扩大阅读面，只完成清单剩余项并立即写出产物。

**硬性降级顺序**（仅当超预算时按序执行，不跳步、不做叠加判断）：① Phase 7 降为附录一行条目；② Phase 6 停止新增追踪，只保留已建立的攻击路径；③ LOW 封顶从 5 降为 3。**覆盖下限与 Phase 9B 不参与降级。**

### 1.4 单会话模式（small 档默认）

small 档或上下文预算充裕时，在当前会话按 Phase 1 → 10 顺序执行；仍必须遵守 1.1。

### 1.5 子代理模式（medium/large 档默认）

medium/large 档、或当前会话已经很长时，把阶段拆给独立子代理（subagent / Task / Agent 工具）执行，主会话只做调度与汇总：

- 每个子代理的 prompt 只包含五项：
  1. 角色：独立审计子代理
  2. 本 skill 的安装路径 + 要执行的阶段参考文件（`references/phase-XX-*.md`）
  3. 目标 repository 路径
  4. 允许阅读的既有 `audit/` 产物文件列表
  5. 必须写出的输出文件路径
- **不要**把早前阶段的结论粘贴进子代理 prompt——让它自己读文件，避免转述失真，也节省编排上下文。
- 子代理完成后，主会话只保留 ≤ 10 行摘要（Findings 数量、severity 分布、一句话要点），不把产物全文拉回主上下文。

调度依赖（Phase 3–7 只依赖 01、02，彼此独立，可并行 fan-out）：

| Phase | 阶段 | 参考文件 | 输出 | 依赖 |
|---|---|---|---|---|
| 1 | Discovery | `references/phase-01-discovery.md` | `audit/01-discovery.md` | — |
| 2 | Architecture & Data Flow | `references/phase-02-architecture.md` | `audit/02-architecture.md` | 1 |
| 3 | Functional & Logic | `references/phase-03-logic.md` | `audit/03-logic.md` | 2 |
| 4 | Security | `references/phase-04-security.md` | `audit/04-security.md` | 2 |
| 5 | Cross-Module | `references/phase-05-cross-module.md` | `audit/05-cross-module.md` | 2 |
| 6 | Adversarial | `references/phase-06-adversarial.md` | `audit/06-adversarial.md` | 2 |
| 7 | Quality / Reliability | `references/phase-07-quality.md` | `audit/07-quality.md` | 2 |
| 8 | Finding Normalization | `references/phase-08-normalization.md` | `audit/08-findings.md` | 3–7 |
| 9 | Verification（必须独立执行，见 §2） | `references/phase-09-verification.md` | `audit/09-verification.md` | 8 |
| 10 | Final Report | `references/phase-10-final-report.md` | `AUDIT-REPORT.md` | 9 |

进入某阶段前，先读对应参考文件，再按其执行。各阶段文件是自包含的（主会话可直接执行，也可交独立子代理执行），但只承载本阶段特有内容——通用规则（硬规则、产物接力、证据阶梯）以本文件为准，文件间不重复样板。

子代理 dispatch 模板：

```
你是独立审计子代理，只负责一个阶段。
目标项目：<repository 路径>
1. 先读本 skill 的 SKILL.md（§0 Hard Rules、§3 Evidence Ladder），再阅读 references/phase-XX-<name>.md，严格按其执行。
2. 先阅读既有产物：<允许读取的 audit/ 文件列表>；除此之外不要参考任何先前阶段的结论。
3. 一切结论必须基于实际读到的源码与配置，不根据 README 或注释推测。
4. 完成后把结果写入 <输出文件>，然后只返回 ≤10 行摘要：Findings 数量、severity 分布、一句话要点。
不要修改任何源码。
```

### 1.6 模式裁剪（Scoped Modes）

用户可以只要求某一类审计。无论哪种模式，Discovery 与 Architecture 不可跳过（Hard Rule 9），验证独立性（§2）不可跳过。

| 用户请求（示例措辞） | 执行阶段 | 最终输出 |
|---|---|---|
| 完整审计（默认）："审计这个项目" | 1 → 10 | `AUDIT-REPORT.md` |
| 安全审计："只做安全审计" / "audit security" | 1 → 2 → 4 → 8 → 9 | `audit/security-report.md` |
| 逻辑审计："跨模块逻辑审一遍" / "audit logic" | 1 → 2 → 3 → 5 → 8 → 9 | `audit/logic-report.md` |
| 攻击者视角："从攻击者角度审计" / "audit adversarial" | 1 → 2 → 6 → 8 → 9 | `audit/adversarial-report.md` |
| 验证已有报告："验证这份审计报告" / "audit verify <report>" | 仅 9（被验证的报告视作 claim 清单） | `audit/verification-report.md` |
| 最终验证 High/Critical："audit high <report>" | 仅 9，范围限 High/Critical | `audit/high-critical-final.md` |

裁剪模式下，Phase 8 只汇总本模式产出的 Findings；Phase 9 仍必须按 §2 独立执行。

---

## 2. Verifier Independence — 验证必须独立

Phase 9 不允许由发现 Findings 的同一个上下文自行确认：同一个会话会倾向于确认自己刚写下的结论（自我确证）。必须按优先级满足以下之一：

1. **独立子代理（优先）**：用全新子代理执行 Phase 9，prompt 按 §1.5 模板。只给它 `audit/08-findings.md` 和 repository 路径——不给 03–07 的产物，不给发现过程中的任何推理。验证者把每条 Finding 当作**待检验的 claim**：File/Line 只是线索，Data Flow 与 Evidence 只是待复核声明；必须自己重新定位代码、重建路径。
2. **独立会话**：没有子代理能力时，开一个新的干净会话执行 `/audit verify audit/08-findings.md`。
3. **同会话降级（最后手段）**：确实无法隔离时，可在同会话执行 Phase 9，但必须：只依据重新读到的源码下 verdict；对每条 Finding 显式记录尝试推翻它的反例；在 Coverage 中声明"验证未隔离"。此时 CONFIRMED 从严：无法 100% 从源码重建的，一律降为 LIKELY / POTENTIAL。

---

## 3. Evidence Ladder 与 Verdict

每个 Finding 必须标注证据等级（Evidence Level）：

- **E0** — 仅模式匹配（"用了危险 API"）：不得单独成 Finding
- **E1** — 局部代码证据：上限 POTENTIAL
- **E2** — 跨函数数据流（source → sink 完整静态链）：上限 LIKELY
- **E3** — 端到端可达路径（外部输入可达，静态即证）：可 CONFIRMED
- **E4** — 运行时可复现行为：CONFIRMED 强证据
- **E5** — 利用已演示：CONFIRMED 最强

绑定规则：CONFIRMED 需要 ≥ E3；LIKELY = E2；POTENTIAL = E1。档位联动见 §1.3（small 档上限 E3；medium 档 E4 仅限 HIGH/Critical；large 档不设限）。

**Audit Methodology（所有漏洞类别统一）**：先枚举 sink 的具体实例，再从 sink 反向追踪到 source；禁止只凭类别清单报告问题。SSRF 的完整追踪链示例见 `references/phase-04-security.md`。

Verdict：CONFIRMED / LIKELY / POTENTIAL / FALSE_POSITIVE / DUPLICATE / INFORMATIONAL

Severity：CRITICAL / HIGH / MEDIUM / LOW / INFO

Finding 字段结构、LOW 封顶与前提分层写法见 `references/phase-08-normalization.md`。

---

## 4. Finding 标准

不要因为以下原因直接报告问题：

- "理论上可能"
- "最佳实践建议如此"
- "代码看起来危险"
- "某库通常存在漏洞"
- "某参数没有明显校验"

必须尽可能证明实际执行路径。

如果一个问题需要特殊条件，明确写出条件。

如果只有理论可能性，写成 POTENTIAL，不得写成确认漏洞。

---

## 5. Final Quality Gate

完成报告前，必须再次问：

1. 这个 Finding 是否真的由当前源码支持？
2. 是否有遗漏的 validation？
3. 是否误解 framework/runtime/browser 行为？
4. 是否依赖未说明的配置？
5. 是否与其他 Finding 重复？
6. 是否把理论风险当成实际漏洞？
7. 是否可以用更少的 Findings 更准确地表达问题？

最终目标：

> 真实、可复现、可解释，而不是 Findings 数量最大化。

不要修改代码。
