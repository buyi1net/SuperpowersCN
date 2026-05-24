# Skill 编写最佳实践（官方参考中文导读）

> 本文件采用 **P1：英文原版 + 中文导读** 策略。原因：这是 Anthropic 官方文档的长参考，包含外部链接、HTML 组件、图片标签、代码块和官方表述；全文翻译会增加上游同步成本，也可能误改官方语义。使用时以官方原文为准，本导读帮助中文团队快速定位和应用要点。

> 原文标题：Skill authoring best practices  
> 原文用途：Learn how to write effective Skills that Claude can discover and use successfully.

## 适用方式

- 需要官方 skill authoring 规则时，先读本导读。
- 涉及字段含义、平台能力、Claude 官方约束时，回到原文相应小节核对。
- 外部链接、图片、`<Note>`、`<Warning>`、`<Tip>`、`<CardGroup>` 等官方文档结构不要随意改写。
- 示例代码、路径、命令和 YAML 字段名保持原样。

## 核心原则

### 1. 简洁优先

上下文窗口是公共资源。`SKILL.md` 一旦被加载，它的每个 token 都会和对话历史、系统提示、其他 skills 竞争上下文。

默认假设 Claude 已经很聪明。只补充 Claude 不知道、但完成任务必须知道的信息。写每段内容前问：

- Claude 真的需要这个解释吗？
- 能不能假设 Claude 已经知道？
- 这段话值得它占用的 token 吗？

好 skill 应删掉通用背景，只保留可执行做法。

### 2. 匹配自由度

根据任务脆弱度选择指令粒度：

- **高自由度**：多种方案都可行，依赖上下文判断。用文字启发式说明。
- **中自由度**：有推荐模式，但允许配置变化。用伪代码或带参数脚本。
- **低自由度**：操作脆弱、顺序关键、一致性重要。给精确命令和明确禁止项。

数据库迁移、批量删除、权限变更等场景应低自由度；代码审查、结构分析等可高自由度。

### 3. 用目标模型测试

Skills 是模型能力的补充。不同模型对同一 skill 的响应不同。

- Haiku：是否给了足够指导？
- Sonnet：是否清晰且高效？
- Opus：是否避免过度解释？

跨模型使用时，写成所有目标模型都能稳定执行的说明。

## Skill 结构要点

### Frontmatter

`SKILL.md` frontmatter 需要：

- `name`
- `description`

完整结构以官方 Skills overview 为准。

### 命名

官方建议用 gerund form（动词 + `-ing`），例如：

- `Processing PDFs`
- `Analyzing spreadsheets`
- `Managing databases`

也可使用名词短语或动作式名称。避免 `Helper`、`Utils`、`Tools` 这类泛名。

中文本地化项目中，`name` 仍保持 ASCII slug，不翻译。

### Description

`description` 影响 skill 发现。它应具体说明 skill 做什么、何时使用，并包含关键触发词。

注意：

- 第三人称。
- 不用第一人称。
- 避免 “Helps with documents” 这类空泛描述。
- 中文本地化时按本项目规范写成两句，并加入“用户说...”触发短语。

## 渐进披露（Progressive Disclosure）

`SKILL.md` 应像目录和入口指南：只放概览和导航，细节按需拆文件。

建议：

- `SKILL.md` 主体少于 500 行。
- 接近上限时拆出参考文件。
- 按任务、领域或功能拆分内容。
- 引用文件一层即可，避免层层跳转。
- 超过 100 行的参考文件顶部放目录。

### 常见组织模式

**高层指南 + 参考文件：**

```text
pdf/
├── SKILL.md
├── FORMS.md
├── reference.md
├── examples.md
└── scripts/
```

**按领域组织：**

```text
bigquery-skill/
├── SKILL.md
└── reference/
    ├── finance.md
    ├── sales.md
    ├── product.md
    └── marketing.md
```

**条件详情：**

主文件只说明基本工作流；需要 tracked changes、OOXML 等高级内容时，再读对应文件。

## 工作流与反馈循环

复杂任务应拆成清晰步骤。必要时提供可跟踪 checklist。

常见循环：

1. 生成草稿或计划。
2. 运行验证器或按清单检查。
3. 修复错误。
4. 重新验证。
5. 全部通过后继续。

对无代码 skill，验证器可以是 `STYLE_GUIDE.md` 或审查清单；对有代码 skill，验证器通常是脚本。

## 内容准则

### 避免时间敏感信息

不要写将很快过期的规则，例如“2025 年 8 月前用旧 API”。更好的方式是：

- 主体写当前方法。
- 用 “Old patterns” 或 `<details>` 保存历史方法。

### 术语一致

全文只选一个术语：

- 始终写 `API endpoint`，不要混用 `URL`、`API route`、`path`。
- 始终写 `field`，不要混用 `box`、`element`、`control`。

中文本地化时同理，按项目术语表统一。

## 常见模式

### 模板模式

严格输出要求用固定模板；灵活分析任务给默认结构并允许调整。

### 示例模式

质量依赖风格时，给输入 / 输出对。一个好示例通常比抽象描述更有用。

### 条件工作流模式

先判断任务类型，再进入对应流程。例如创建新文档和编辑已有文档分流。

## 评估与迭代

### 先建评估

**在写大量文档前先创建评估。** 这能确保 skill 解决真实问题，而不是记录想象中的需求。

流程：

1. 不带 skill 跑代表性任务，找出失败或缺口。
2. 创建 3 个测试场景。
3. 建立无 skill 基线。
4. 写最小说明。
5. 运行评估，对比基线并迭代。

### 与 Claude 协作开发 Skills

有效模式：

- Claude A：帮助创建或改进 skill 的“专家”。
- Claude B：加载 skill 后执行真实任务的测试 agent。
- 用户：提供领域知识，观察 Claude B 行为。

观察 Claude B 是否找得到信息、是否正确应用规则、是否遗漏关键约束。把具体失败带回 Claude A 改进 skill。

## 观察 Claude 如何导航 Skill

迭代时观察：

- Claude 是否按预期顺序读文件？
- 是否漏读关键引用？
- 是否反复读某个文件，说明该内容应进 `SKILL.md`？
- 是否从不访问某个文件，说明它可能没必要或入口不清楚？

`name` 和 `description` 对发现尤其关键。

## 反模式

### 避免 Windows 风格路径

即使在 Windows 上，也用 `/`：

- ✅ `scripts/helper.py`
- ❌ `scripts\helper.py`

### 避免给太多选项

不要无必要地列出多个库或方案。给默认方案，再给 escape hatch。

## 高级：带可执行代码的 Skills

如果 skill 只有 markdown 指令，可跳到清单；如果包含脚本，注意以下事项。

### 解决问题，不要甩锅给 Claude

脚本应处理错误，而不是直接失败后让 Claude 猜。

配置参数要有理由，避免 “voodoo constants”。如果你不知道为什么是这个值，Claude 也不知道。

### 提供工具脚本

即使 Claude 可以临时写脚本，预置脚本仍有优势：

- 更可靠。
- 节省 token。
- 节省时间。
- 保证一致性。

明确告诉 Claude 是执行脚本还是阅读脚本：

- 执行：`Run analyze_form.py to extract fields`
- 阅读：`See analyze_form.py for the extraction algorithm`

多数工具脚本应执行而非读取。

### 使用视觉分析

当输入能渲染成图片时，让 Claude 分析图片，例如表单布局、页面结构、视觉标注。

### 创建可验证中间产物

复杂开放任务应使用 plan-validate-execute：

1. 先生成结构化计划文件。
2. 用脚本验证计划。
3. 验证通过后执行。
4. 再验证输出。

适用于批量操作、破坏性变更、复杂验证规则、高风险任务。

### 依赖包

不同运行环境限制不同：

- `claude.ai`：可从 npm / PyPI 安装包，也可从 GitHub 拉取。
- Anthropic API：无网络访问，也不能运行时安装包。

在 `SKILL.md` 中列出所需包，并确认目标环境可用。

中国本土化注意：如果 npm / PyPI / GitHub 访问受网络影响，只提示用户按团队规范配置 registry、代理或认证；不要自动修改全局配置。

## Runtime environment

Skills 在有文件系统、bash 命令和代码执行能力的环境中运行。

关键点：

1. 启动时只预加载所有 skills 的 `name` 和 `description`。
2. `SKILL.md` 和附属文件按需读取。
3. 工具脚本可直接执行，不必把全文读进上下文。
4. 大参考文件只有被读取时才消耗上下文。

作者应：

- 使用 `/` 路径。
- 文件名要描述内容。
- 按领域或功能组织。
- 偏好脚本处理确定性操作。
- 明确脚本是执行还是阅读。
- 用真实请求测试文件访问模式。

## MCP 工具引用

如果 skill 使用 MCP 工具，始终写完整工具名，避免 “tool not found”。

格式：`ServerName:tool_name`

示例：

```markdown
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中 `BigQuery` 和 `GitHub` 是 MCP server 名称。

## 不要假设工具已安装

明确依赖：

````markdown
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 技术说明

### YAML frontmatter

`SKILL.md` frontmatter 需要 `name` 和 `description`。完整结构见官方 Skills overview。

### Token budgets

`SKILL.md` 主体建议少于 500 行。超过时用渐进披露拆文件。

## 有效 Skills 清单

### 核心质量

- [ ] Description 具体，包含关键词。
- [ ] Description 同时说明做什么和何时使用。
- [ ] `SKILL.md` 主体少于 500 行。
- [ ] 需要时把细节拆到单独文件。
- [ ] 无时间敏感信息；或放到 old patterns。
- [ ] 术语一致。
- [ ] 示例具体，不抽象。
- [ ] 文件引用只深一层。
- [ ] 正确使用渐进披露。
- [ ] 工作流步骤清晰。

### 代码和脚本

- [ ] 脚本解决问题，不甩锅给 Claude。
- [ ] 错误处理明确有用。
- [ ] 没有无法解释的 magic number。
- [ ] 依赖包已列出并确认可用。
- [ ] 脚本文档清楚。
- [ ] 不用 Windows 风格路径。
- [ ] 关键操作有验证步骤。
- [ ] 高质量任务有反馈循环。

### 测试

- [ ] 至少创建 3 个评估。
- [ ] 用 Haiku、Sonnet、Opus 测试。
- [ ] 用真实使用场景测试。
- [ ] 如适用，已纳入团队反馈。

## 后续链接

原文末尾包含官方卡片链接，指向：

- Agent Skills quickstart
- Claude Code Skills
- API Skills guide

这些链接属于官方文档导航，定稿时建议保留原链接或以团队文档替代，但不要误写成项目本地路径。
