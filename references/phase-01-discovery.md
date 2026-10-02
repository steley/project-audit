# Phase 1 — Discovery

> 执行前置：重读 `audit/` 中已有产物（若有；首次运行时该目录可能为空）。
> 输出：`audit/01-discovery.md`

任务：主动扫描整个项目，而不是只看当前打开的文件。

识别：

- 语言
- Framework
- Runtime
- Package manager
- Build system
- Entry points
- Source directories
- Tests
- API / Server / Worker
- Frontend
- CLI
- Database
- File system
- Network
- External services
- Configuration
- Environment variables
- CI/CD
- Deployment
- Static assets
- Third-party dependencies

## 扫描面声明（必须输出，作为分档依据）

**默认排除（硬规则）**：`node_modules/`、`.venv/`、`venv/`、`site-packages/`、`dist/`、`build/`、`target/`、`__pycache__/`、`vendor/`、generated 代码。这些目录不作为项目代码审计、不计入规模分级。

- 在产物中**点名**实际排除的目录及各自文件数——让用户能发现误判（例如把真实源码误当 vendor）。
- **真实规模用固定命令判定，不靠估算**（在仓库根执行）：

  ```bash
  EX='\.(ts|tsx|js|jsx|mjs|cjs|py|go|rs|rb|php|java|kt|swift|astro|vue|svelte|css|scss|sql|sh)$'
  VX='/(node_modules|dist|build|vendor|\.venv|venv|target)/'
  git ls-files | grep -vE "$VX" | grep -cE "$EX"                          # ① 代码文件数
  git ls-files | grep -vE "$VX" | grep -E "$EX" | xargs wc -l | tail -1   # ② 源码 LOC
  # 无 git 时把 git ls-files 换成：find . -type f -not -path '*/.git/*'
  ```

- 按结果对照 SKILL.md §1.3 判定档位（small / medium / large），写明将采用的执行方式与证据上限；入口点数量随扫描得出，一并写入产物。
- 排除 ≠ 禁读：Phase 9 核实框架/运行时行为时允许读这些目录。

其它注意：

- lockfile 不需要逐行人工审计，但必须检查依赖版本、脚本和异常依赖风险。
- 如果项目很大，使用搜索工具、静态分析、测试和分阶段读取，而不是一次性读取所有文件。

输出内容：技术栈清单、入口点清单、目录结构、外部依赖面、配置与环境变量清单、**扫描面声明（排除名单 + 档位判定）**，以及你未能覆盖的部分。
