# project-audit

通用型 **Agent 项目全量审计 Skill**——装进支持 Agent Skills 标准的 Agent（ZCode / Claude Code 等），用一句自然语言就能对一个代码仓库做系统性、证据驱动、独立验证的全量审计。

核心原则：**Find fewer, prove more.** 审计只输出真实、可复现、可解释的结论，不做 Findings 数量最大化；全程只读，不修改被审计仓库的任何源码。

## 为什么不是"让 AI 看看这个项目有没有漏洞"

裸问 AI 审计代码有三大失败模式：幻觉漏洞（理论上可能就上报）、同一问题反复上报、自己验证自己（自我确证）。本 skill 针对性地内置了对应机制：

| 失败模式 | 对应机制 |
|---|---|
| 幻觉漏洞 | **Evidence Ladder**：每个 Finding 标注证据等级 E0–E5，verdict 与证据等级硬绑定（CONFIRMED 需要端到端可达路径 E3 以上）；E0"用了危险 API"不得单独成 Finding |
| 重复上报 | 归一化阶段合并同根因；验证阶段标 DUPLICATE 指向主 ID |
| 自我确证 | **独立验证**：Phase 9 由独立子代理/新会话执行，只拿到 Finding 的 claim、不拿到发现阶段的推理，任务是推翻而非确认 |
| 漏报无人发现 | **9B 漏报对账**：把入口点/sink 清单与已报 Findings 做集合差，专抓"发现阶段漏掉的问题" |
| 成本失控 | **分档预算**：small 档单会话、静态证据上限、目标 ~1M tokens；覆盖下限（入口点/sink 100% 追踪）不随预算打折 |

## 安装

本目录即 skill 包（目录名 `project-audit` 与 frontmatter 中的 name 一致）：

- ZCode（用户级，所有项目可用）：`~/.agents/skills/project-audit/`
- ZCode（项目级，仅当前项目）：`<project>/.agents/skills/project-audit/`
- Claude Code：`~/.claude/skills/project-audit/`
- 其它 Agent：把目录放入可读取位置，让它阅读 `SKILL.md` 并按其执行

## 使用

安装后自然语言即可触发（也可用 `/skill project-audit` 强制加载）：

| 你说 | 执行阶段 | 产出 |
|---|---|---|
| "帮我审计一下这个项目" | 1 → 10 | `AUDIT-REPORT.md` |
| "只做安全审计" | 1 → 2 → 4 → 8 → 9 | `audit/security-report.md` |
| "跨模块逻辑审一遍" | 1 → 2 → 3 → 5 → 8 → 9 | `audit/logic-report.md` |
| "从攻击者角度审计" | 1 → 2 → 6 → 8 → 9 | `audit/adversarial-report.md` |
| "验证一下这份审计报告" | 仅 9（独立验证者） | `audit/verification-report.md` |
| "把报告里的高危问题做最终确认" | 仅 9，限 High/Critical | `audit/high-critical-final.md` |

## 流程（10 阶段）

1. **Discovery** —— 扫描面声明：vendor/build 硬排除，git-tracked 代码文件数与 LOC 用固定命令判定档位
2. **Architecture** —— 输出 Canonical System Model（入口点 / 信任边界 / sink 的结构化 YAML），是后续所有阶段的共同参照
3. **Logic** · 4. **Security** · 5. **Cross-Module** · 6. **Adversarial** —— 五个透镜；medium/large 档可并行 fan-out，small 档单会话
7. **Quality** —— 只报实际影响明显的问题
8. **Normalization** —— 去重合并、E-level 标注、LOW 封顶（最多 5 条展开，其余一行入附录）
9. **Verification** —— 独立验证者逐条推翻 + 9B 漏报对账
10. **Final Report** —— 声明档位与证据天花板、分区域 Audit Confidence

所有中间产物落在被审计仓库的 `audit/` 目录，支持断点续跑（已存在的阶段产物不重跑）。

## 实测成本

| 被审计项目 | 版本 | 耗时 | tokens |
|---|---|---|---|
| ~750 文件（vendor 干扰） | v1.0 | 116 min | ~30M |
| ~44 文件 | v1.0 | 35 min | ~10M |
| ~15 文件 | v1.1（small 档） | 29.5 min | ~3M |

v1.1 的成本控制来自：分档预算、扫描面排除、工具卫生（grep 排除 vendor、二进制不读、输出截断）、LOW 封顶、E-level 绑定让模式级观察早期丢弃。质量未回退：v1.1 首跑中独立验证推翻了发现者的一条错误结论并补录一条真缺陷。

## 目前存在的问题与已知限制

1. **medium / large 档未经实测**。三次实测全部是 small 档；fan-out 预算、E4 限 HIGH 这些规则已写入但未被真实项目检验，首次使用建议盯一下 token 消耗。
2. **small 档默认纯静态（证据上限 E3）**。时序/竞态类问题最高只能给到 LIKELY；运行时实验（E4）默认禁止、需用户逐项同意。需要运行时实证的结论请用 medium/large 档或逐项申请。
3. **依赖漏洞扫描不在范围内**。不会自动跑 npm audit / osv-scanner，只人工核对 lockfile 版本与异常依赖——这是明确的范围裁剪，有重依赖的项目请自行补扫。
4. **Phase 9 独立验证占全程约 50–60% 时间**。这是设计代价：实测中它推翻过发现者的错误结论并补录真缺陷。极小项目如需提速，可用 SKILL.md §2.3 的同会话降级路径（CONFIRMED 从严、报告声明未隔离）——用独立性换速度。
5. **部署态相关结论带前提**。哪些配置在生产实际生效、仓库内不可判定时，相关 Finding 的 severity 会按前提条件化——报告是带前提的结论，不是最终判决。
6. **验证独立性依赖 Agent 的子代理能力**。没有子代理能力的 Agent 走同会话降级，并在报告中声明"验证未隔离"。
7. **只审计不修复**。不改代码、不自动修复、不更新依赖；Recommendation 只到"下一步做什么"。
8. **触发依赖 Agent 的 skill 支持**。旧版 `/audit security` 等 slash command 文件已移除（功能并入模式裁剪）；不支持 skill 机制的 Agent 需手动让它阅读 `SKILL.md` 执行。

## 目录结构

```
project-audit/
├── SKILL.md            # 入口：硬规则、分档预算、证据阶梯、验证独立性、模式裁剪
└── references/         # Phase 1–10 各阶段执行细节（进入某阶段前按需读取）
```

## 设计沿革

- **v1.0**：10 阶段 SOP + 产物接力 + 独立验证
- **v1.1**：分档预算、Evidence Ladder、9B 漏报对账、Canonical System Model、工具卫生、LOW 封顶；移除独立 slash commands（功能并入模式裁剪）

## 原则

- 不根据 README 猜测实现，以实际源码和配置为准
- 不把代码风格当漏洞，不为了数量制造 Findings
- 安全问题必须尽可能建立完整攻击路径
- 无法证明的内容标 POTENTIAL
- 同一根因的问题合并
- 不修改代码、不自动修复、不更新依赖
