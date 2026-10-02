# Project Audit Agent Workflow

通用型 Agent 项目全量审计工作流（标准 Skill 格式），适用于 Web、API、CLI、Worker、桌面应用、库、脚本等项目。

## 目标

核心原则：**Find fewer, prove more.**

工作流不会修改源码。它先理解项目，再进行功能、逻辑、安全、跨模块、攻击者视角审计，最后对 Findings 去重，并由独立验证者复核。

## 安装（作为可自动触发的 Skill）

本目录即 skill 包，目录名 `project-audit` 与 frontmatter 中的 name 一致。

- ZCode（用户级，所有项目可用）：`~/.agents/skills/project-audit/`
- ZCode（项目级，仅当前项目可用）：`<project>/.agents/skills/project-audit/`
- Claude Code：`~/.claude/skills/project-audit/`
- 其它 Agent：把目录放入 Agent 可读取的位置，让它阅读 `SKILL.md` 并按其执行。

安装后，以下请求会自动触发（也可先 `/skill project-audit` 强制加载，再说明要哪种模式）：

- 完整审计："帮我审计一下这个项目" / "audit this project"
- 只做安全审计："看看这个 repo 有什么安全问题" / "security review this project"
- 跨模块逻辑审计："跨模块逻辑审一遍" / "audit logic"
- 攻击者视角："从攻击者角度审计" / "audit adversarial"
- 验证已有报告："验证一下这份审计报告" / "audit verify <report>"
- 只复核 High/Critical："把报告里的高危问题做最终确认" / "audit high <report>"

各模式执行哪些阶段，见 SKILL.md §1.6 模式裁剪表。

## 三种核心执行机制

1. **产物接力（Context Relay）**：审计的唯一可信记忆是 `audit/` 目录下的产物文件。每个阶段开始时重读既有产物，结束时立即写出输出；支持断点续跑。medium/large 档把各 Phase 拆给独立子代理执行，Phase 3–7 可并行 fan-out（SKILL.md §1.1、§1.5）。
2. **分级预算与证据阶梯（Tier & Evidence Ladder）**：规模按排除 vendor/build 后的真实源码分级（small 单会话、目标 ~1M tokens；medium/large 才 fan-out）。覆盖下限不可打折：所有入口点 100% 枚举、所有高危 sink 100% 反向追踪。Findings 按证据等级 E0–E5 绑定 verdict——CONFIRMED 需要端到端可达路径（E3）以上，small 档静态证据为上限（SKILL.md §1.3、§3）。
3. **独立验证与漏报对账（Verifier Independence + Coverage Re-scan）**：Phase 9 必须由未接触原始推理的独立子代理或新会话执行，Findings 只作为待检验的 claim；9B 把入口点/sink 清单与 Findings 对账，专抓"发现阶段漏掉的问题"（SKILL.md §2）。

## 结构

```
project-audit/
├── SKILL.md            # 入口：硬规则、执行模型、模式裁剪、验证独立性、Finding 标准
└── references/         # Phase 1–10 各阶段执行细节（进入某阶段前按需读取）
```

## 阶段

1. Discovery
2. Architecture
3. Functional & Logic
4. Security
5. Cross-Module
6. Adversarial
7. Quality / Reliability
8. Finding Deduplication
9. Verification（独立执行）
10. Final Report

## 原则

- 不根据 README 猜测实现
- 不把代码风格当漏洞
- 不为了数量制造 Findings
- 安全问题必须尽可能建立完整攻击路径
- 无法证明的内容标记 POTENTIAL
- 同一根因的问题合并
- 不修改代码、不自动修复、不更新依赖
