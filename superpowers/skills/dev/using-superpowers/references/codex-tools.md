# Codex 工具映射

Skills 使用 Claude Code 工具名。当你在 skill 中遇到这些工具名时，使用当前平台的等价工具：

| Skill 中的引用 | Codex 等价工具 |
|----------------|----------------|
| `Task` tool（派发子 agent） | `spawn_agent`（见[子 agent 派发需要 multi-agent 支持](#子-agent-派发需要-multi-agent-支持)） |
| 多个 `Task` 调用（并行） | 多个 `spawn_agent` 调用 |
| Task 返回结果 | `wait_agent` |
| Task 自动完成 | `close_agent`，用于释放 slot |
| `TodoWrite`（任务跟踪） | `update_plan` |
| `Skill` tool（调用 skill） | Skills 原生加载；直接遵循指令 |
| `Read`、`Write`、`Edit`（文件） | 使用当前平台原生文件工具 |
| `Bash`（运行命令） | 使用当前平台原生 shell 工具 |

## 子 agent 派发需要 multi-agent 支持

在 Codex 配置（`~/.codex/config.toml`）中添加：

```toml
[features]
multi_agent = true
```

这会为 `dispatching-parallel-agents`、`subagent-driven-development` 等 skill 启用 `spawn_agent`、`wait_agent` 和 `close_agent`。

兼容性说明：`rust-v0.115.0` 之前的 Codex build 曾把等待已派发 agent 的能力暴露为 `wait`。当前 Codex 使用 `wait_agent` 获取已派发 agent 的结果。`wait` 现在属于 code-mode 的 `exec/wait`，通过 `cell_id` 恢复已 yield 的 exec cell；它不是获取子 agent 结果的工具。

## 环境检测

创建 worktree 或收尾分支的 skills，应先用只读 git 命令检测环境：

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

- `GIT_DIR != GIT_COMMON` -> 已经在 linked worktree 中（跳过创建）。
- `BRANCH` 为空 -> detached HEAD（不能从 sandbox 中创建分支、push 或 PR）。

各 skill 如何使用这些信号，见 `using-git-worktrees` 的 Step 0 和 `finishing-a-development-branch` 的 Step 1。

## Codex App 收尾

当 sandbox 阻止分支或 push 操作时（例如处于外部管理的 worktree 且 detached HEAD），agent 应提交所有工作，并告知用户使用 App 原生控件：

- **"Create branch"**：命名分支，然后通过 App UI commit / push / PR。
- **"Hand off to local"**：把工作转交到用户本地 checkout。

agent 仍可运行测试、stage 文件，并输出建议的分支名、commit message 和 PR 描述，供用户使用。
