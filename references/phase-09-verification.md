# Phase 9 — Verification（必须由独立验证者执行）

> 本阶段与发现 Findings 的上下文相互独立。如果你就是执行 Phase 3–8 的同一个上下文，按 SKILL.md §2 优先改用独立子代理/新会话执行本阶段；确实无法隔离时，按下方"同会话降级"执行并在产物中声明。
> 前置输入：`audit/08-findings.md`、`audit/02-architecture.md` 末尾的 Canonical System Model（仅用于 9B 对账）和目标 repository 源码。**不要阅读** 03–07 的产物或任何发现阶段的推理。
> 输出：`audit/09-verification.md`

## 角色

你是独立验证者。你的任务是**推翻**这些 Findings，而不是确认它们。

- `audit/08-findings.md` 中的每条 Finding 只是**待检验的 claim**。
- Finding 中的 File / Line / Function 只是线索，不是事实；必须自己重新定位代码。
- Finding 中的 Data Flow / Evidence / Trigger Condition 是待复核的声明；必须从源码重新推导。
- 目标 repository 的源码与配置是唯一证据来源。
- vendor/build 排除目录（`.venv/`、`node_modules/` 等）不作为审计对象，但**允许核读**以确认框架/运行时默认行为。

## 逐条验证清单

1. 文件是否存在？
2. 行号是否合理？
3. 代码是否仍存在？
4. 原判断是否正确理解代码？
5. 输入是否真的可控？
6. 数据是否真的到达 Sink？
7. 中间是否存在验证？
8. 是否有 Framework / Runtime / Browser 防护？（先从 lockfile 确认框架实际版本及其默认防护行为，必要时读框架源码）
9. 是否依赖特定配置？
10. 实际影响是否成立？

对安全问题重新构建：

Attacker Input
→ Validation
→ Processing
→ Sink
→ Security Impact

对每条 Finding 显式记录：你尝试过哪些能推翻它的反例（遗漏的 validation、框架默认防护、调用方约束、配置前提、编码/解析差异等），结果如何。

## 运行时实验的档位门槛（SKILL.md §1.3）

- **small 档：默认禁止运行时实验。** 仅当 HIGH/Critical 候选无法用静态证据确认时，在产物中标注"建议对该 ID 做运行时实验（E4），需用户同意后执行"，不得擅自执行。
- **medium 档：** 仅 HIGH/Critical 允许 E4 实验。
- **large 档：** E4/E5 允许。
- 任何实验：只读探针、不修改源码、临时文件必须清理、在产物中记录实验清单（手段 + 结果摘要）。

## Verdict 规则

- 完整重建执行路径且无法推翻 → `CONFIRMED`，写出最小触发条件与实际影响。
- 方向正确但个别环节无法从源码证实 → `LIKELY`。
- 只有理论可能性 → `POTENTIAL`。
- 源码/框架/配置证明不成立 → `FALSE_POSITIVE`，写明反例。
- 与其它 Finding 同根因 → `DUPLICATE`，指向主 ID。
- 仅提供背景 → `INFORMATIONAL`。

最终 verdict 必须与重新核定的 Evidence Level 一致（SKILL.md §3 绑定规则）：验证只能维持或下调 verdict，除非你做出了新的 E4/E5 证据（且档位允许）才可上调。

## 9B — Coverage Re-scan（漏报对账，必做）

不以已有 Findings 为前提，重问一次："**如果这些 Findings 全是假的，这个项目还有什么问题？**"——这是有界对账，不是重跑审计：

1. 重读 Canonical System Model 的 `entry_points` 与 `sensitive_sinks` 清单。
2. 对照已报 Findings，列出**未被任何 Finding 覆盖**的入口点与 sink。
3. 对每个未覆盖项做一次快速静态追踪（到 E1/E2 即可），有可疑的登记为新 Finding（按证据等级定 verdict）。
4. 对账完成但确实无问题的项，记"已查无异常"。

输出一节 **Missed Issue Search**：对账结果 + 新增候选 + 未覆盖原因（若有）。

## 同会话降级（仅当无法隔离时）

只依据重新读到的源码下 verdict；CONFIRMED 从严：无法 100% 从源码重建的，一律降为 LIKELY / POTENTIAL；并在产物开头声明"验证未隔离"。

## 输出内容

每条 Finding：ID / 原 verdict 与 E-level / 重定位结果（file:line）/ 重建的数据流 / 反例尝试 / 最终 verdict 与 E-level / 理由。外加 9B 的 Missed Issue Search 一节。
