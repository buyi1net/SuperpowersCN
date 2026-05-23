# opencode-skills-chinese 合并可行性分析

> 分析 [linxuan-sys/opencode-skills-chinese](https://github.com/linxuan-sys/opencode-skills-chinese) 仓库中 17 个已中文化 skill 与 `D:\skills\superpowers` 合成版的重合度、互补性和合并可行性。

---

## 总览

opencode-skills-chinese（以下简称 OC）是基于 [Anthropic 官方技能包](https://github.com/anthropics/skills) 的汉化移植，共 17 个 skill。本项目 superpowers（以下简称 SP）是 skills-main + superpowers-main 的合成版，共 27 个 skill。双方合并后新增能力按以下维度分类：

```text
A. 已重叠领域（OC 弱于 SP，不合并）
  -> B. 已重叠领域（OC 有可借鉴模式）
  -> C. 互补领域（高价值，建议直接合并）
  -> D. 互补领域（中等价值，按需合并）
  -> E. 互补领域（低价值或场景窄）
  -> X. 不可合并

当前评估：合并 7~8 个 OC skill 后，SP 总数预计 34~35 个。
```

---

## 来源对比

| 维度 | superpowers (SP) | opencode-skills-chinese (OC) |
|------|------------------|------------------------------|
| 总数 | 27 | 17 |
| 语言 | 12 个中文 + 15 个英文（汉化进行中） | 全部中文 |
| 来源 | skills-main + superpowers-main 合成 | Anthropic 官方技能包汉化 |
| 定位 | AI 编程全流程工作流 | 通用终端 AI 工具集 |
| 组织方式 | 按工作流阶段（A→K） | 按功能类型平铺 |
| 平台绑定 | OpenCode / Claude Code / Codex / Cursor | 无特定平台绑定 |

---

## A. 已重叠——OC 弱于 SP，不合并（3 个）

这三个 skill 在双方仓库中都存在，但 SP 版本功能更完善。

| OC skill | 对应 SP skill | OC 行数 | SP 行数 | 差距 |
|----------|-------------|:--:|:--:|------|
| `code-reviewer` | `requesting-code-review` + `receiving-code-review` | ~20 | 318 | OC 极简模板仅列安全/性能/最佳实践检查项，无工作流、无子 agent 派发、无分级报告 |
| `systematic-debugging` | `systematic-debugging` | ~200 | 297 + 3 子文件 | OC 结构完整（4 阶段），但 SP 含 root-cause-tracing、defense-in-depth、condition-based-waiting 等子文件，生态更全 |
| `skill-creator` | `writing-skills` + `write-a-skill` | ~560 | 653 + 187 | OC 来自 Anthropic 官方，含 eval/benchmark/描述优化循环，流程极长；SP 的两个版本各有所长，暂不替换 |

**结论：3 个均不正式合并。** 但 OC 版的内容可作为 SP 版汉化时的对照参考或模式借鉴。

---

## B. 重叠但有可借鉴模式（1 个）

| OC skill | SP 对应 | 可借鉴内容 |
|----------|---------|-----------|
| `systematic-debugging` | `systematic-debugging`（待审，英文） | OC 已全文汉化，结构化防错模式值得学习：**"铁律"**（未调查根本原因不得提修复方案）、**"借口表"**（自合理化防御）、**"红旗信号"**（发现自己在猜就停止）。SP 原版有类似内容但分散在各处，OC 的模式组织更紧凑。 |

**建议：** 在 SP 的 `systematic-debugging` 汉化时，借鉴 OC 的"铁律→四阶段→红旗→借口表→快速参考"结构进行中文增强。不直接替换。

---

## C. 互补——高价值，建议直接合并（4 个）

| OC skill | 建议 SP 归类 | 行数 | 价值 |
|----------|-------------|:--:|------|
| `doc-coauthoring` | **B. 需求与设计** | ~300 | 结构化文档协作工作流：分上下文收集→优化→读者测试三阶段。填补 SP 中"从想法到正式文档"之间的空缺。当前 SP 有 brainstorming（宽泛设计）和 to-prd（结构化需求），但缺少"逐节雕琢文档"的协作模式 |
| `opencode` | **A. 开工准备**（或独立 L 类） | ~200 | OpenCode CLI 完整操作手册：一次性 `run` 任务、交互式 TUI、PR 审查工作流、并行工作模式。SP 当前有 `using-superpowers` 定义使用规则，但缺少 OpenCode 具体操作方法。本项目就在用 OpenCode，此 skill 直接实用 |
| `playwright-cli` | **D. 实现**（测试工具） | ~280 | 浏览器自动化完整命令参考：页面交互、多标签页、Cookie/LocalStorage、网络 mock、开发者工具。对前端测试和 E2E 场景是刚需 |
| `docx` + `pdf` | **B. 需求与设计**（文档产出） | 各~150 | 文档格式生成工具——OC 十六个技能中最实用的两个文档处理。docx 创建/编辑 Word 文档，pdf 处理 PDF。补 SP 在文档生成上的空白 |

### 工作流说明

```text
doc-coauthoring 在 brainstorming 和 to-prd 之间提供逐节协作：
  想法已大致清晰
    -> grill-me 追问
    -> brainstorming 输出设计方向
    -> doc-coauthoring 逐节雕琢正式文档 ← 新增
    -> 最后用 to-prd 沉淀为需求文档

opencode 作为开工准备补充：
  安装好 OpenCode
    -> opencode skill 提供 CLI 操作参考 ← 新增
    -> using-superpowers 定义使用规则
    -> setup 配置项目级设置

playwright-cli 作为实现阶段的测试工具：
  功能实现中
    -> test-driven-development 写测试
    -> playwright-cli 在浏览器中验证 E2E ← 新增
    -> 回归通过后继续
```

### 小白理解

- `doc-coauthoring`：像请了个编辑帮你一起写文章，先聊想法、再逐段打磨、最后找第三个人看看读不读得懂。
- `opencode`：像你的 OpenCode 遥控器说明书——一次性命令怎么用、交互模式怎么进、快捷键是什么。
- `playwright-cli`：给浏览器装了个遥控器，不用手点鼠标就能自动操作网页。
- `docx / pdf`：像文档格式的万能转换头——需要 Word 就生成 .docx，需要 PDF 就生成 .pdf。

---

## D. 互补——中等价值，按需合并（3 个）

| OC skill | 建议 SP 归类 | 价值评估 |
|----------|-------------|------|
| `pptx` | B. 需求与设计（文档产出） | PowerPoint 生成，汇报场景需要但频率低于 docx/pdf |
| `xlsx` | B. 需求与设计（文档产出） | Excel 生成，数据导出场景需要 |
| `mcp-builder` | K. 元技能 | MCP 服务器构建指南。MCP 生态正在增长，后续可能成为高频需求；当前场景有限，可暂缓 |

**建议：** `pptx` + `xlsx` 与 `docx` + `pdf` 组成 L 类"文档生成"打包引入。`mcp-builder` 在需要时再引入。

---

## E. 互补——低价值或场景窄，暂不合并（5 个）

| OC skill | 内容 | 暂不合并原因 |
|----------|------|-------------|
| `algorithmic-art` | p5.js 生成艺术 | 场景极窄，与主要编程工作流无关 |
| `canvas-design` | 画布设计（海报/视觉） | 同上，纯设计类 |
| `chart-generator` | 图表生成 | 有参考价值但 chart.js 等图表库 agent 已可直接使用 |
| `frontend-design` | 前端设计（组件/页面） | 有参考价值但 prototype 已覆盖 UI 原型场景 |
| `theme-factory` | 主题/配色方案生成 | 纯视觉类，与编程核心工作流脱节 |

**建议：** 暂不合并。如果后续用户有设计类高频需求，再重新评估 `chart-generator` 和 `frontend-design`。

---

## X. 不可合并（1 个）

| OC skill | 原因 |
|----------|------|
| `arch-system-troubleshooter` | 硬编码个人路径 `/home/LinXuan/Desktop/data/Shorin-Niri系统配置详解.html`，绑定 Arch + niri + kitty + fish 特定环境，完全不可移植。作为 skill 设计参考（"如何为特定系统写排查 skill"）有价值，但作为正式 skill 不可合并 |

---

## 合并优先级建议

| 优先级 | 数量 | 列表 | 预计新增行数 |
|:--:|:--:|------|:--:|
| P1 高 | 4 | `doc-coauthoring`、`opencode`、`playwright-cli`、`docx`+`pdf` | ~1000 |
| P2 中 | 3 | `pptx`、`xlsx`、`mcp-builder` | ~500 |
| P3 低 | 5 | 创意/设计类 5 个 | ~1000 |
| — | (3) | 重叠类不合并 | — |

---

## 合并后预计结构

假设 P1 和 P2 全部合并，SP 从 27 个扩展到 34 个：

```text
A. 开工准备 (2 → 3)
  setup, using-superpowers
  + opencode（OpenCode CLI 操作参考）

B. 需求与设计 (4 → 7)
  grill-me, brainstorming, prototype, to-prd
  + doc-coauthoring（结构化文档协作）
  + docx, pdf（文档格式生成）

C. 计划与拆解 (3)
  writing-plans, to-issues, triage

D. 实现 (5 → 6)
  using-git-worktrees, test-driven-development, executing-plans,
  subagent-driven-development, dispatching-parallel-agents
  + playwright-cli（浏览器自动化测试）

E. 调试 (1)
  systematic-debugging

F. 质量与验证 (3)
  verification-before-completion, setup-pre-commit, git-guardrails-claude-code

G. 代码审查 (2)
  requesting-code-review, receiving-code-review

H. 交付收尾 (1)
  finishing-a-development-branch

I. 持续改进 (2)
  improve-codebase-architecture, zoom-out

J. 沟通辅助 (2)
  caveman, handoff

K. 元技能 (2 → 3)
  write-a-skill, writing-skills
  + mcp-builder（MCP 服务器构建）

L. 文档生成（新增类别）
  + pptx, xlsx（按需从 D 类移入）
```

---

## 合并流程建议

1. **审阅阶段**：逐个读取 OC 原版 skill，对照 CLAUDE.md 规范审核
   - 检查是否已有中文、是否需要再汉化增强
   - 补齐触发词（按 §2.7 + A09 新规）
   - 添加中国本土化注意事项
2. **适配阶段**：将 OC 的 `superpowers:xxx` 引用改为 SP 实际路径
3. **定稿阶段**：按 CLAUDE.md §9 定稿流程走

建议从 `doc-coauthoring` 开始（体量大但价值最高），再逐步推进 `opencode` 和 `playwright-cli`。

---

## 参考

- OC 仓库：https://github.com/linxuan-sys/opencode-skills-chinese（127 stars，18 forks）
- SP 工作流结构：`D:\skills\其它资料\AI编程Skill合成版工作流结构.md`
- 汉化规范：`D:\skills\CLAUDE.md`
