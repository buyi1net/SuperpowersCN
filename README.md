# SuperpowersCN — AI 编程 Skills 简体中文合集

面向 AI 编程工作流的简体中文 skill 集合，共 32 个 skill，覆盖从开工、设计、计划、实现、调试、验证、审查到交付的完整开发流程。

本项目基于 [Superpowers](https://github.com/obra/superpowers) 与 [Anthropic 官方技能包](https://github.com/anthropics/skills) 汉化、增强并整合，适配中文团队和中国网络环境。

## 快速开始

### OpenCode

在 `opencode.json` 中添加本仓库的 `skills` 路径：

```json
{
  "skills": {
    "paths": ["/path/to/this-repo/skills"]
  }
}
```

重启 OpenCode 后生效。详细说明见 [`docs/README.opencode.md`](docs/README.opencode.md)。

### Claude Code

将本仓库设为 Claude Code 插件目录：

```bash
# 安装
claude mcp add --plugin /path/to/this-repo
```

或通过 marketplace 安装（参见上游 [Superpowers README](https://github.com/obra/superpowers)）。

### 通用方式

将 `skills/` 目录复制或链接到你的 agent 工具的 skills 目录，确保每个 skill 目录下的 `SKILL.md` 可被扫描到。

修改配置后需要重启对应工具。

## 目录结构

| 目录 | 数量 | 用途 |
|---|:---:|---|
| `dev/` | 24 | 编程全流程：初始化→头脑风暴→规划→实现→调试→审查→完成 |
| `docs/` | 3 | 文档协作、DOCX 创建/编辑、PDF 处理 |
| `automation/` | 1 | 浏览器自动化 |
| `meta/` | 4 | skill 创建、增强、压缩沟通、交接 |
| `creative/` | 0 | 预留：视觉、图表、主题等创意生成 |
| `systems/` | 0 | 预留：OS 排查、网络诊断、性能调优 |
| `ai/` | 0 | 预留：prompt 工程、RAG、模型评估 |

## 推荐工作流

```text
1. dev/setup
   初始化仓库级 issue tracker、triage 标签和领域文档配置。

2. dev/grill-me 或 dev/brainstorming
   把模糊想法问清楚，比较方案并产出设计。

3. dev/writing-plans
   把设计拆成可执行实现步骤。

4. dev/using-git-worktrees + dev/test-driven-development
   创建隔离工作区，先写失败测试，再实现。

5. dev/executing-plans 或 dev/subagent-driven-development
   按计划执行，必要时并行分派。

6. dev/systematic-debugging
   出 bug 时系统化排查。

7. dev/verification-before-completion
   完成前运行验证并读取结果。

8. dev/requesting-code-review + dev/receiving-code-review
   审查实现并处理反馈。

9. dev/finishing-a-development-branch
   决定合并、PR、保留或丢弃。
```

## Skill 清单

### dev（开发全流程，24 个）

| Skill | 用途 |
|---|---|
| `setup` | 配置 issue tracker、triage 标签和领域文档布局。 |
| `using-superpowers` | 会话启动总规则：先判断并调用可能适用的 skill。 |
| `opencode` | 将编码任务委托给 OpenCode CLI 代理。 |
| `brainstorming` | 澄清想法、比较方案、产出设计文档。 |
| `grill-me` | 高强度追问想法、方案和风险。 |
| `prototype` | 做一次性原型验证关键逻辑、状态机或 UI。 |
| `to-prd` | 将需求整理成 PRD。 |
| `writing-plans` | 将设计拆成小步实现计划。 |
| `to-issues` | 将 PRD 或计划拆成 issue。 |
| `triage` | 管理 issue 状态与派活准备度。 |
| `dispatching-parallel-agents` | 并行分派互不依赖的任务或故障。 |
| `executing-plans` | 按计划逐任务执行。 |
| `finishing-a-development-branch` | 完成后决定合并、PR、保留或丢弃。 |
| `subagent-driven-development` | 每个任务交给全新子 agent 并审查。 |
| `test-driven-development` | 红绿重构：失败测试、最小实现、重构。 |
| `using-git-worktrees` | 创建隔离工作区。 |
| `git-guardrails-claude-code` | 拦截危险 git 命令。 |
| `receiving-code-review` | 验证并处理 review 反馈。 |
| `requesting-code-review` | 派子 agent 审查代码。 |
| `setup-pre-commit` | 设置 pre-commit 验证。 |
| `systematic-debugging` | 系统化排查 bug。 |
| `verification-before-completion` | 完成前必须验证。 |
| `improve-codebase-architecture` | 架构体检和重构建议。 |
| `zoom-out` | 从整体理解陌生代码或复杂模块。 |

### docs（文档处理，3 个）

| Skill | 用途 |
|---|---|
| `doc-coauthoring` | 结构化文档协作，上下文收集→迭代组织→读者测试。 |
| `docx` | 创建、读取、编辑 .docx，含修订/评论/XML 参考和 Python 工具链。 |
| `pdf` | 读取、合并、拆分、填表、OCR，含表单填写指南和高级参考。 |

### automation（自动化，1 个）

| Skill | 用途 |
|---|---|
| `playwright-cli` | 浏览器自动化、页面交互、Playwright 测试调试与生成。 |

### meta（元工具，4 个）

| Skill | 用途 |
|---|---|
| `caveman` | L1/L2/L3 三级压缩沟通。 |
| `handoff` | 生成交接文档。 |
| `write-a-skill` | 创建、改造、汉化或增强 skill。 |
| `writing-skills` | 用 TDD 方法创建、编辑和验证 skill。 |

## 文档

- [OpenCode 安装与使用](docs/README.opencode.md)
- [测试指南](docs/testing.md)
- [Skill 触发矩阵验收](docs/skill-trigger-matrix.md)
- [Windows hook 说明](docs/windows/polyglot-hooks.md)

## 许可与致谢

本项目以 MIT 协议发布，见 [LICENSE](LICENSE)。

Skills 原始内容来自：

- [obra/superpowers](https://github.com/obra/superpowers) — 核心开发工作流 skills（TDD、调试、子 agent 调度、代码审查等）
- [anthropics/skills](https://github.com/anthropics/skills) — 官方技能包（文档处理、浏览器自动化等）

本仓库在原版基础上做了简体中文本地化、中国本土化适配和中文增强。行为准则见 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。
