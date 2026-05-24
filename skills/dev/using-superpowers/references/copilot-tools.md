# Copilot CLI 工具映射

Skills 使用 Claude Code 工具名。当你在 skill 中遇到这些工具名时，使用当前平台的等价工具：

| Skill 中的引用 | Copilot CLI 等价工具 |
|----------------|----------------------|
| `Read`（读取文件） | `view` |
| `Write`（创建文件） | `create` |
| `Edit`（编辑文件） | `edit` |
| `Bash`（运行命令） | `bash` |
| `Grep`（搜索文件内容） | `grep` |
| `Glob`（按名称搜索文件） | `glob` |
| `Skill` tool（调用 skill） | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` tool（派发子 agent） | `task`，设置 `agent_type: "general-purpose"` 或 `"explore"` |
| 多个 `Task` 调用（并行） | 多个 `task` 调用 |
| Task 状态 / 输出 | `read_agent`、`list_agents` |
| `TodoWrite`（任务跟踪） | `sql`，使用内置 `todos` 表 |
| `WebSearch` | 无等价工具；用 `web_fetch` 搭配搜索引擎 URL |
| `EnterPlanMode` / `ExitPlanMode` | 无等价工具；留在主会话中 |

## 异步 shell 会话

Copilot CLI 支持持久异步 shell 会话，Claude Code 没有直接等价能力：

| Tool | 用途 |
|------|------|
| `bash` with `async: true` | 在后台启动长时间运行的命令 |
| `write_bash` | 向正在运行的异步会话发送输入 |
| `read_bash` | 读取异步会话输出 |
| `stop_bash` | 终止异步会话 |
| `list_bash` | 列出所有活跃 shell 会话 |

## 其他 Copilot CLI 工具

| Tool | 用途 |
|------|------|
| `store_memory` | 持久保存代码库事实，供后续会话使用 |
| `report_intent` | 更新 UI 状态栏中的当前意图 |
| `sql` | 查询会话 SQLite 数据库（todos、metadata） |
| `fetch_copilot_cli_documentation` | 查询 Copilot CLI 文档 |
| GitHub MCP tools (`github-mcp-server-*`) | 原生 GitHub API 访问（issues、PRs、code search） |
