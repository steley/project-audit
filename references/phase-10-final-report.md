# Phase 10 — Final Report

> 执行前置：整读 `audit/09-verification.md` 与 `audit/08-findings.md`；`audit/01-discovery.md` 只提取扫描面声明（档位与证据上限）、`audit/02-architecture.md` 只提取 Canonical System Model（局部读，不整读；报告需要更多架构叙述时再整读）。单会话模式下产物已在上下文中时不必重读。
> 输出：repository 根目录的 `AUDIT-REPORT.md`

生成 `AUDIT-REPORT.md`。报告开头必须声明本次审计的**档位与证据上限**（例："small 档：单会话、静态证据上限 E3、未做运行时实验"），让读者知道结论的证据天花板。

结构：

## 1. Executive Summary

## 2. Project Architecture

## 3. Attack Surface

## 4. Findings Summary

按照 severity 排列：

CRITICAL
HIGH
MEDIUM
LOW
INFO

| ID | Verdict | E-level | Severity | Category | Title |
|---|---|---|---|---|---|

## 5. Detailed Findings

每项包含完整证据和调用链。**以 Phase 9 重验证后的 verdict、E-level 和数据流为准**，不要沿用未经验证的原始结论。证据按三段式呈现：Static（file:line + 代码引用）/ Dynamic（运行时实验，若有）/ Reproduction（复现命令 + 期望/实际，若可得）。

## 6. False Positives

列出验证阶段推翻的 Finding 及反例。

## 7. Duplicate Findings

## 8. Coverage 与 Audit Confidence

**Coverage**：

- 检查了什么
- 未检查什么
- 哪些结论无法确认
- 哪些结论依赖环境
- Phase 9 是否由独立上下文执行（未隔离时必须声明）
- 9B 漏报对账的结果摘要

**Audit Confidence**（不做单一分数）：

- 分区域标注覆盖信心：High / Medium / Low（例：Authentication—High；CI/CD—Medium；生产配置—未审计）
- 未审计清单：仓库之外的东西（外部 SaaS 配置、基础设施、运行时环境）默认不在结论内
- 一句话原则写进报告："**审计没发现问题"不等于"没有问题"**，读者应能在 Coverage 中看到信心边界。

## 9. Recommended Next Steps

只针对真实 Findings 给出下一步。

---

报告完成后，过一遍 SKILL.md 的 Final Quality Gate，再交付。
