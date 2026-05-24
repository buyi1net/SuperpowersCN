# Gemini CLI 工具映射

Skills 使用 Claude Code 工具名。当你在 skill 中遇到这些工具名时，使用当前平台的等价工具：

| Skill 中的引用 | Gemini CLI 等价工具 |
|----------------|----------------------|
| `Read`（读取文件） | `read_file` |
| `Write`（创建文件） | `write_file` |
| `Edit`（编辑文件） | `replace` |
| `Bash`（运行命令） | `run_shell_command` |
| `Grep`（搜索文件内容） | `grep_search` |
| `Glob`（按名称搜索文件） | `glob` |
| `TodoWrite`（任务跟踪） | `write_todos` |
| `Skill` tool（调用 skill） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` tool（派发子 agent） | `@agent-name`（见[子 agent 支持](#子-agent-支持)） |

## 子 agent 支持

Gemini CLI 通过 `@` 语法原生支持子 agent。使用内置 `@generalist` agent 派发任意任务；它可以访问所有工具，并遵循你提供的 prompt。

当 skill 要求派发某个命名 agent 类型时，使用 `@generalist`，并填入该 skill 的完整 prompt 模板：

| Skill 指令 | Gemini CLI 等价方式 |
|------------|---------------------|
| `Task tool (superpowers:implementer)` | 使用 `@generalist`，并填入 `implementer-prompt.md` 模板 |
| `Task tool (superpowers:spec-reviewer)` | 使用 `@generalist`，并填入 `spec-reviewer-prompt.md` 模板 |
| `Task tool (superpowers:code-reviewer)` | 使用 `@code-reviewer`（内置 agent）或 `@generalist`，并填入 review prompt |
| `Task tool (superpowers:code-quality-reviewer)` | 使用 `@generalist`，并填入 `code-quality-reviewer-prompt.md` 模板 |
| `Task tool (general-purpose)` with inline prompt | 使用 `@generalist`，并传入 inline prompt |

### 填充 Prompt

Skills 提供的 prompt 模板会包含 `{WHAT_WAS_IMPLEMENTED}` 或 `[FULL TEXT of task]` 这类占位符。填好所有占位符后，把完整 prompt 作为消息传给 `@generalist`。prompt 模板本身包含 agent 角色、审查标准和预期输出格式；`@generalist` 会遵循它。

### 并行派发

Gemini CLI 支持并行派发子 agent。当 skill 要求并行派发多个相互独立的子 agent 任务时，在同一条 prompt 中一起请求所有这些 `@generalist` 或命名子 agent 任务。依赖任务仍保持顺序执行，但不要为了让历史更简单而串行执行独立子 agent 任务。

## 其他 Gemini CLI 工具

这些工具在 Gemini CLI 中可用，但没有 Claude Code 等价工具：

| Tool | 用途 |
|------|------|
| `list_directory` | 列出文件和子目录 |
| `save_memory` | 将事实持久保存到 `GEMINI.md`，供后续会话使用 |
| `ask_user` | 向用户请求结构化输入 |
| `tracker_create_task` | 丰富任务管理（创建、更新、列出、可视化） |
| `enter_plan_mode` / `exit_plan_mode` | 在修改前切换到只读研究模式 |
