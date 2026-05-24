---
name: finishing-a-development-branch
description: 实现已完成、测试已通过，并且需要决定如何合入、创建 PR、保留分支或清理工作区时使用。通过结构化选项完成开发分支收尾，在任何合并、删除或清理前保持验证和人工确认；用户说"收尾""完成分支""帮我合并""代码写完了"时也可触发。
disable-model-invocation: true
---

# 收尾开发分支

## 总览

用清晰选项引导开发工作收尾，并按用户选择执行对应流程。

**核心原则：验证测试 -> 检测环境 -> 展示选项 -> 执行选择 -> 清理工作区。**

**开始时说明：**“我正在使用 `finishing-a-development-branch` skill 来完成这项工作。”

## 流程

### 第 1 步：验证测试

**展示收尾选项之前，先验证测试通过：**

```bash
# 运行项目测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**

```text
测试失败（<N> 个失败）。完成收尾前必须先修复：

[展示失败内容]

测试通过前，不能继续合并或创建 PR。
```

停止。不要进入第 2 步。

**如果测试通过：**继续第 2 步。

### 第 2 步：检测环境

**展示选项之前，先确定当前工作区状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这会决定展示哪个菜单，以及如何清理：

| 状态 | 菜单 | 清理方式 |
|---|---|---|
| `GIT_DIR == GIT_COMMON`（普通仓库） | 标准 4 个选项 | 没有 worktree 需要清理 |
| `GIT_DIR != GIT_COMMON`，具名分支 | 标准 4 个选项 | 按来源判断（见第 6 步） |
| `GIT_DIR != GIT_COMMON`，detached HEAD | 精简 3 个选项（不能直接合并） | 不清理（由外部管理） |

### 第 3 步：确定基准分支

```bash
# 尝试常见基准分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

也可以询问：`这个分支是从 main 分出来的，对吗？`

### 第 4 步：展示选项

**普通仓库和具名分支 worktree：必须准确展示这 4 个选项：**

```text
实现已完成。你想如何处理？

1. 本地合并回 <base-branch>
2. 推送并创建 Pull Request
3. 保留当前分支（我稍后自己处理）
4. 丢弃这项工作

请选择一个选项。
```

**Detached HEAD：必须准确展示这 3 个选项：**

```text
实现已完成。当前处于 detached HEAD（外部管理的工作区）。

1. 作为新分支推送并创建 Pull Request
2. 保持现状（我稍后自己处理）
3. 丢弃这项工作

请选择一个选项。
```

**不要额外解释。** 保持选项简洁。

### 第 5 步：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根目录，确保 CWD 安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并；确认成功后，才允许删除任何东西
git checkout <base-branch>
git pull
git merge <feature-branch>

# 在合并结果上验证测试
<test command>

# 只有合并成功且验证通过后：清理 worktree（第 6 步），再删除分支
```

然后：清理 worktree（第 6 步），再删除分支：

```bash
git branch -d <feature-branch>
```

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <feature-branch>

# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## 摘要
<用 2-3 个要点说明改动>

## 测试计划
- [ ] <验证步骤>
EOF
)"
```

如果 GitHub 访问、`gh` 登录、认证令牌或网络代理不可用，只报告失败原因，并提示用户按团队规范配置。不要替用户登录、修改全局认证或代理配置。

**不要清理 worktree。** 用户还需要它来处理 PR 反馈。

#### 选项 3：保持现状

报告：`保留分支 <name>。worktree 保留在 <path>。`

**不要清理 worktree。**

#### 选项 4：丢弃

**必须先确认：**

```text
这会永久删除：
- 分支 <name>
- 所有提交：<commit-list>
- worktree：<path>

请输入 `discard` 确认。
```

等待用户输入完全匹配的确认文本。

如果已确认：

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理 worktree（第 6 步），再强制删除分支：

```bash
git branch -D <feature-branch>
```

### 第 6 步：清理工作区

**只对选项 1 和选项 4 执行。** 选项 2 和选项 3 始终保留 worktree。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：**普通仓库，没有 worktree 需要清理。结束。

**如果 worktree 路径位于 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下：**这是 Superpowers 创建的 worktree，可以由本流程清理（仅限选项 1 和选项 4）。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自修复：清理陈旧注册项
```

**否则：**这个工作区由宿主环境（harness）管理。不要删除它。如果平台提供退出工作区的工具，使用该工具；否则保留工作区。

## 快速参考

| 选项 | 合并 | 推送 | 保留 worktree | 清理分支 |
|---|---|---|---|---|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持现状 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**

- **问题：**合并损坏代码，或创建失败 PR。
- **修正：**展示选项前始终验证测试。

**开放式提问**

- **问题：**“接下来做什么？”含糊不清。
- **修正：**准确展示 4 个结构化选项；detached HEAD 时展示 3 个。

**选项 2 后清理 worktree**

- **问题：**删除了用户处理 PR 反馈所需的 worktree。
- **修正：**只在选项 1 和选项 4 后清理。

**删除分支早于移除 worktree**

- **问题：**`git branch -d` 失败，因为 worktree 仍引用该分支。
- **修正：**先合并，再移除 worktree，最后删除分支。

**在 worktree 内运行 git worktree remove**

- **问题：**当前工作目录在要移除的 worktree 内，命令可能失败或行为异常。
- **修正：**运行 `git worktree remove` 前，始终 `cd` 到主仓库根目录。

**清理宿主环境创建的 worktree**

- **问题：**删除宿主环境创建的 worktree 会制造幽灵状态。
- **修正：**只清理 `.worktrees/`、`worktrees/` 或 `~/.config/superpowers/worktrees/` 下的 worktree。

**丢弃前没有确认**

- **问题：**误删工作成果。
- **修正：**要求用户输入 `discard` 才能确认。

**替用户处理平台认证**

- **问题：**`gh`、GitHub 网络或认证失败后，擅自登录、改全局 token、改代理或修改远端配置。
- **修正：**只报告失败原因，提示用户按团队规范处理；除非用户明确要求，否则不修改认证、代理或远端配置。

## 红色警报

**绝不要：**

- 测试失败时继续收尾。
- 合并后不验证测试结果。
- 未确认就删除工作。
- 没有明确请求就 force-push。
- 确认合并成功前移除 worktree。
- 清理不是本流程创建的 worktree（必须检查来源）。
- 在 worktree 内运行 `git worktree remove`。
- 替用户登录 GitHub、修改全局认证、代理或远端配置。

**始终要：**

- 展示选项前验证测试。
- 展示菜单前检测环境。
- 准确展示 4 个选项；detached HEAD 时展示 3 个。
- 选项 4 必须获得 `discard` 文本确认。
- 只为选项 1 和选项 4 清理 worktree。
- 移除 worktree 前先 `cd` 到主仓库根目录。
- 移除后运行 `git worktree prune`。
