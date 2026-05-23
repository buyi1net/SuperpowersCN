# AI 编程 Skill 合成版

这是一个面向 AI 编程工作流的 skill 合成版，当前目录为 `D:\skills\superpowers`。

当前共 32 个 skill：27 个原合成版 skill 已完成汉化定稿，5 个来自 `opencode-skills-chinese` 的 P1 高价值 skill 已导入并做了基础规范化，后续仍需按本地规范逐个复审。

## 设计原则

- **目录按能力域组织**：用 `core/`、`design/`、`planning/`、`dev/`、`quality/`、`docs/` 等目录承载长期扩展，而不是继续平铺。
- **工作流优先**：使用时仍按开工、设计、计划、实现、调试、验证、审查、交付来理解 skill。
- **谨慎合并**：重合度高的 skill 先并存，经过真实使用和复审后再合并或淘汰。
- **保留原行为**：目录重构不改变已定稿 skill 的正文行为；路径型入口已同步到新目录。
- **OC 分批导入**：先导入文档处理、OpenCode 委托和浏览器自动化等 P1 skill；视觉/图表/Office 其他格式等后续再评估。

## 当前安装状态

OpenCode 已配置为加载本地插件：

```text
D:\skills\superpowers
```

OpenCode 配置文件：

```text
C:\Users\Administrator\.config\opencode\opencode.jsonc
```

配置中的关键项：

```json
"plugin": ["D:/skills/superpowers"]
```

修改 OpenCode 配置、插件或 skill 文件后，需要重启 OpenCode。运行中的会话不会热加载这些配置。

## 目录结构

| 目录 | 数量 | 用途 |
|---|:---:|---|
| `core/` | 3 | 启动规则、基础配置、外部 agent 委托 |
| `design/` | 4 | 需求澄清、方案设计、PRD、原型 |
| `planning/` | 3 | 计划拆解、issue 化、triage |
| `dev/` | 6 | 实现、TDD、工作树、子 agent 与并行开发 |
| `quality/` | 6 | 验证、调试、代码审查、git/pre-commit 护栏 |
| `maintain/` | 2 | 架构体检、全局理解 |
| `comms/` | 2 | 压缩沟通、交接 |
| `docs/` | 3 | 文档协作、DOCX、PDF |
| `automation/` | 1 | 浏览器自动化 |
| `creative/` | 0 | 预留：视觉、图表、主题等创意生成 |
| `meta/` | 2 | skill 创建、增强、测试 |

## 推荐工作流

```text
1. core/setup
   初始化仓库级 issue tracker、triage 标签和领域文档配置。

2. design/grill-me 或 design/brainstorming
   把模糊想法问清楚，比较方案并产出设计。

3. planning/writing-plans
   把设计拆成可执行实现步骤。

4. dev/using-git-worktrees + dev/test-driven-development
   创建隔离工作区，先写失败测试，再实现。

5. dev/executing-plans 或 dev/subagent-driven-development
   按计划执行，必要时并行分派。

6. quality/systematic-debugging
   出 bug 时系统化排查。

7. quality/verification-before-completion
   完成前运行验证并读取结果。

8. quality/requesting-code-review + quality/receiving-code-review
   审查实现并处理反馈。

9. dev/finishing-a-development-branch
   决定合并、PR、保留或丢弃。
```

## Skill 清单

### core

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `setup` | skills-main | 已定稿 | 配置 issue tracker、triage 标签和领域文档布局。 |
| `using-superpowers` | superpowers-main | 已定稿 | 会话启动总规则：先判断并调用可能适用的 skill。 |
| `opencode` | OC P1 | 已导入，待复审 | 将编码任务委托给 OpenCode CLI agent。 |

### design

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `brainstorming` | superpowers-main | 已定稿 | 澄清想法、比较方案、产出设计文档。 |
| `grill-me` | skills-main | 已定稿 | 高强度追问想法、方案和风险。 |
| `prototype` | skills-main | 已定稿 | 做一次性原型验证关键逻辑、状态机或 UI。 |
| `to-prd` | skills-main | 已定稿 | 将需求整理成 PRD。 |

### planning

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `writing-plans` | superpowers-main | 已定稿 | 将设计拆成小步实现计划。 |
| `to-issues` | skills-main | 已定稿 | 将 PRD 或计划拆成 issue。 |
| `triage` | skills-main | 已定稿 | 管理 issue 状态与派活准备度。 |

### dev

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `dispatching-parallel-agents` | superpowers-main | 已定稿 | 并行分派互不依赖的任务或故障。 |
| `executing-plans` | superpowers-main | 已定稿 | 按计划逐任务执行。 |
| `finishing-a-development-branch` | superpowers-main | 已定稿 | 完成后决定合并、PR、保留或丢弃。 |
| `subagent-driven-development` | superpowers-main | 已定稿 | 每个任务交给全新子 agent 并审查。 |
| `test-driven-development` | superpowers-main | 已定稿 | 红绿重构：失败测试、最小实现、重构。 |
| `using-git-worktrees` | superpowers-main | 已定稿 | 创建隔离工作区。 |

### quality

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `git-guardrails-claude-code` | skills-main | 已定稿 | 拦截危险 git 命令。 |
| `receiving-code-review` | superpowers-main | 已定稿 | 验证并处理 review 反馈。 |
| `requesting-code-review` | superpowers-main | 已定稿 | 派子 agent 审查代码。 |
| `setup-pre-commit` | skills-main | 已定稿 | 设置 pre-commit 验证。 |
| `systematic-debugging` | superpowers-main | 已定稿 | 系统化排查 bug。 |
| `verification-before-completion` | superpowers-main | 已定稿 | 完成前必须验证。 |

### maintain

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `improve-codebase-architecture` | skills-main | 已定稿 | 架构体检和重构建议。 |
| `zoom-out` | skills-main | 已定稿 | 从整体理解陌生代码或复杂模块。 |

### comms

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `caveman` | skills-main | 已定稿 | L1/L2/L3 三级压缩沟通。 |
| `handoff` | skills-main | 已定稿 | 生成交接文档。 |

### docs

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `doc-coauthoring` | OC P1 | 已导入，待复审 | 结构化文档协作、读者测试。 |
| `docx` | OC P1 | 已导入，待复审 | 创建、读取、编辑、分析 `.docx`。 |
| `pdf` | OC P1 | 已导入，待复审 | 读取、合并、拆分、填表、OCR 等 PDF 处理。 |

### automation

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `playwright-cli` | OC P1 | 已导入，待复审 | 浏览器自动化、页面检查、Playwright 测试辅助。 |

### meta

| Skill | 来源 | 状态 | 用途 |
|---|---|---|---|
| `write-a-skill` | skills-main | 已定稿 | 创建、改造、汉化或增强 skill。 |
| `writing-skills` | superpowers-main | 已定稿 | 用 TDD 方法创建、编辑和验证 skill。 |

## 后续合并队列

| 阶段 | Skill | 当前决策 |
|---|---|---|
| P2 | `pptx`、`xlsx`、`mcp-builder` | 后续评估后导入。 |
| 暂缓 | `algorithmic-art`、`canvas-design`、`chart-generator`、`frontend-design`、`theme-factory` | 先不导入，避免创意/视觉类过早膨胀。 |
| 暂缓合并 | `code-reviewer`、`systematic-debugging`、`skill-creator` | 与现有 skill 重合，后续逐个对比取舍。 |
| 不导入 | `arch-system-troubleshooter` | 含个人路径/强上下文绑定，不适合直接进入合集。 |

## 说明

本地 checkout 中未发现 `.codex-plugin/plugin.json` 或 `.Codex-plugin/plugin.json`。当前以 OpenCode 的本地插件路径方式加载 `D:\skills\superpowers`。
