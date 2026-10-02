# project-audit

通用型 **Agent 项目全量审计 Skill**——装进支持 Agent Skills 标准的 Agent（ZCode / Claude Code 等），一句自然语言即可对一个代码仓库做系统性、证据驱动、独立验证的全量审计。

核心原则：**Find fewer, prove more.** 全程只读，不修改被审计仓库的任何源码。

## 核心机制

- **Evidence Ladder**：每个 Finding 标注证据等级 E0–E5，verdict 硬绑定（CONFIRMED 需端到端可达路径 E3+）；"用了危险 API"这类模式级观察不得单独成 Finding
- **独立验证**：Phase 9 由独立子代理/新会话执行，只拿 claim、不拿推理，任务是推翻而非确认；附 9B 漏报对账（入口点/sink 清单与 Findings 做集合差，专抓漏报）
- **分档预算**：small / medium / large 三档；small 单会话、静态证据上限 E3、目标 ~1M tokens；覆盖下限（入口点/sink 100% 追踪）不随预算打折
- **去重合并**：同根因合并、DUPLICATE 指向主 ID；LOW 封顶（最多 5 条展开，其余一行入附录）

## 安装

- ZCode（用户级）：`~/.agents/skills/project-audit/`
- ZCode（项目级）：`<project>/.agents/skills/project-audit/`
- Claude Code：`~/.claude/skills/project-audit/`
- 其它 Agent：放入可读取位置，让它阅读 `SKILL.md` 并按其执行

## 使用

自然语言触发，或 `/skill project-audit` 强制加载：

| 你说 | 执行阶段 | 产出 |
|---|---|---|
| "帮我审计一下这个项目" | 1 → 10 | `AUDIT-REPORT.md` |
| "只做安全审计" | 1→2→4→8→9 | `audit/security-report.md` |
| "跨模块逻辑审一遍" | 1→2→3→5→8→9 | `audit/logic-report.md` |
| "从攻击者角度审计" | 1→2→6→8→9 | `audit/adversarial-report.md` |
| "验证一下这份审计报告" | 仅 9（独立验证者） | `audit/verification-report.md` |
| "把报告里的高危问题做最终确认" | 仅 9，限 High/Critical | `audit/high-critical-final.md` |

中间产物落在被审计仓库的 `audit/` 目录，支持断点续跑（已存在的阶段产物不重跑）。

## 实测成本

| 被审计项目 | 版本 | 耗时 | tokens |
|---|---|---|---|
| ~750 文件（vendor 干扰） | v1.0 | 116 min | ~30M |
| ~44 文件 | v1.0 | 35 min | ~10M |
| ~15 文件 | v1.1（small 档） | 29.5 min | ~3M |

## 目前存在的问题与已知限制

1. medium / large 档未经实测（三次实测均为 small 档），fan-out 预算规则未检验
2. small 档纯静态（证据上限 E3）：时序/竞态类最高 LIKELY；运行时实验（E4）默认禁止、需逐项同意
3. 不自动跑 npm audit / osv-scanner，重依赖项目的漏洞扫描需自行补
4. Phase 9 独立验证占全程 50–60% 时间（设计代价：实测中推翻过错误结论、补录过真缺陷）；极小项目可用 SKILL.md §2.3 同会话降级换速度
5. 部署态仓库内不可判定时，severity 按前提条件化——报告是带前提的结论，不是最终判决
6. 无子代理能力的 Agent 走同会话降级，报告声明"验证未隔离"
7. 只审计不修复、不更新依赖；触发依赖 Agent 的 skill 支持（不支持时可手动让它读 SKILL.md 执行）

## 目录结构

```
project-audit/
├── SKILL.md            # 硬规则、分档预算、证据阶梯、验证独立性、模式裁剪
└── references/         # Phase 1–10 各阶段执行细节（按需读取）
```

## 原则

以实际源码为准，不猜实现；不把风格当漏洞，不制造 Findings；攻击路径必须证明，无法证明标 POTENTIAL；同根因合并；不改代码、不修复、不更新依赖。
