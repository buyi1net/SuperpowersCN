# Skill 触发矩阵验收

本文档用于回归验证本仓库 skills 的 `description` 触发边界，避免相邻生命周期 skill 互相抢触发。

## 前置条件

- OpenCode 的 `skills.paths` 指向 `D:/SuperpowersCN/skills`。
- 修改 skill 或配置后，先重启 OpenCode。
- 建议逐条新开会话或清空上下文测试，避免上一条提示影响下一条触发判断。

## 通过标准

- 每条提示词应优先命中“预期触发”列。
- 如果同时加载多个 skill，只要主 skill 正确且辅助 skill 合理，可以接受。
- 如果命中“不应触发”列，记录提示词、实际触发 skill、当前会话上下文，再收窄对应 `description`。
- 只改触发边界时，优先修改 `SKILL.md` frontmatter 的 `description`，不要重写正文 workflow。

## 验收矩阵

| 测试提示词 | 预期触发 | 不应触发 |
|---|---|---|
| 汉化这个 skill，顺便检查触发边界是否清楚 | `write-a-skill` | `writing-skills` |
| 写一个新的 skill，用 TDD 验证它是否生效 | `writing-skills` | `write-a-skill` |
| 把我们刚才讨论的方案整理成 PRD | `to-prd` | `doc-coauthoring` |
| 起草一份技术规范，先帮我润色结构 | `doc-coauthoring` | `to-prd` |
| PR 前验证一下，确认测试真的通过了 | `verification-before-completion` | `requesting-code-review`, `finishing-a-development-branch` |
| 代码写完了，帮我 review 一下 | `requesting-code-review` | `receiving-code-review`, `finishing-a-development-branch` |
| review 意见怎么处理？这个反馈对吗 | `receiving-code-review` | `requesting-code-review` |
| 这个分支怎么合？帮我做收尾 | `finishing-a-development-branch` | `verification-before-completion`, `requesting-code-review` |
| 执行这个实现计划，在当前会话里按步骤做 | `executing-plans` | `subagent-driven-development` |
| 每个任务开一个 agent，按计划分别实现 | `subagent-driven-development` | `executing-plans`, `dispatching-parallel-agents` |
| 这几个模块互不相关，并行查一下 | `dispatching-parallel-agents` | `executing-plans`, `subagent-driven-development` |
| 把这个 PRD 拆成可执行 issue | `to-issues` | `triage` |
| triage 这个 bug，看看信息是否足够交给 AFK agent | `triage` | `to-issues` |
| 我要做个功能，先聊方案 | `brainstorming` | `prototype`, `test-driven-development` |
| 帮我压力测试这个设计方案，追问我 | `grill-me` | `brainstorming`, `writing-plans` |
| 做个可丢弃原型，让我先试一下状态机 | `prototype` | `brainstorming`, `test-driven-development` |
| 测试挂了，先别改，帮我定位根因 | `systematic-debugging` | `test-driven-development` |
| 根因已经明确，修这个 bug，先写测试再实现 | `test-driven-development` | `systematic-debugging` |
| 这段代码在系统里是什么位置？先讲全局 | `zoom-out` | `improve-codebase-architecture` |
| 帮我找架构重构机会，让代码更易测试 | `improve-codebase-architecture` | `zoom-out` |
| 交给 opencode 跑一下这个重构任务 | `opencode` | `test-driven-development`, `requesting-code-review` |
| 简短点，少废话，省流模式 | `caveman` | 其他任务型 skill |
| 处理这个 PDF，提取表格并做 OCR | `pdf` | `docx`, `doc-coauthoring` |
| 改一下这个 Word 文档，保留修订和评论 | `docx` | `pdf`, `doc-coauthoring` |
| 打开网页测一下按钮交互，用浏览器点一下 | `playwright-cli` | `test-driven-development` |
| 做个交接，把当前工作整理给下一个 agent | `handoff` | `doc-coauthoring` |
| 用中文回复我，以后代码注释和文档都用中文写 | `zh-cn-mode` | 其他任务型 skill |

## 当前验收记录

- 日期：2026年05月24日
- 分支：`develop`
- 结论：上述矩阵已人工验收通过。
- 备注：验收前已将 OpenCode 全局 `skills.paths` 指向 `D:/SuperpowersCN/skills`，并重启后测试。

## 维护规则

- 新增或改写 skill 的 `description` 后，应补充或更新本矩阵。
- 如果两个 skill 生命周期相邻，优先用“勿触发于……”写清反向边界。
- 对宽泛能力类 skill，必须写明硬触发门槛，例如显式提到工具名、文件类型或阶段状态。
- 矩阵中的提示词应尽量使用用户真实口语，不要只写技术术语。
