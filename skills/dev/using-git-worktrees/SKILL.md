---
name: using-git-worktrees
description: 开始需要隔离工作区的新功能、大改动或实现计划前使用。先检测当前是否已经隔离，再优先使用平台原生 worktree 工具；没有原生工具时，才回退到 git worktree。用户说"开新分支""隔离环境""建 worktree""切工作区""创建隔离目录"时也可触发。
disable-model-invocation: true
---

# 使用 Git Worktree

## 总览

确保实际修改发生在隔离工作区中。优先使用当前平台提供的原生 worktree 工具；只有在没有原生工具时，才手动使用 `git worktree` 回退方案。

**核心原则：先检测现有隔离，再用原生工具，最后才回退到 git。不要和运行环境对抗。**

**开始时说明：**“我正在使用 `using-git-worktrees` skill 来设置隔离工作区。”

## 第 0 步：检测现有隔离

**创建任何目录或 worktree 之前，先检查当前是否已经在隔离工作区中。**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**子模块保护：**在 git submodule 中，`GIT_DIR != GIT_COMMON` 也可能成立。得出“已经在 worktree 中”的结论前，必须确认当前不是子模块：

```bash
# 如果返回路径，说明当前在子模块中，不是 linked worktree；按普通仓库处理
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**如果 `GIT_DIR != GIT_COMMON`，且不是子模块：**当前已经在 linked worktree 中。跳到第 2 步（项目设置）。不要再创建另一个 worktree。

按分支状态报告：

- 在分支上：`已在隔离工作区 <path>，当前分支为 <name>。`
- Detached HEAD：`已在隔离工作区 <path>（detached HEAD，由外部管理）。收尾时需要创建分支。`

**如果 `GIT_DIR == GIT_COMMON`，或当前在子模块中：**当前是普通仓库 checkout。

检查用户是否已经在指令中声明 worktree 偏好。如果没有，创建 worktree 前先征求同意：

> “要我设置一个隔离 worktree 吗？它可以保护你当前分支不被本次修改污染。”

如果已有明确偏好，直接遵守，不重复询问。如果用户不同意创建隔离 worktree，就在当前目录工作，并跳到第 2 步。

## 第 1 步：创建隔离工作区

**有两种机制，按顺序尝试。**

### 1a. 原生 Worktree 工具（优先）

用户已同意创建隔离工作区后，先判断当前平台是否已经提供创建 worktree 的能力。它可能叫 `EnterWorktree`、`WorktreeCreate`、`/worktree` 命令，或某个 `--worktree` flag。若存在，使用原生工具，然后跳到第 2 步。

原生工具通常会自动处理目录位置、分支创建和清理。平台已经提供原生工具时再手动运行 `git worktree add`，可能制造运行环境看不见、也无法管理的幽灵状态。

只有在没有原生 worktree 工具时，才进入 1b。

### 1b. Git Worktree 回退方案

**只有 1a 不适用时才用这一节。** 也就是当前没有可用的原生 worktree 工具，才手动使用 git 创建 worktree。

#### 目录选择

按下面优先级选择目录。用户显式偏好永远优先于文件系统中观察到的状态。

1. **检查指令中是否声明了 worktree 目录偏好。** 如果用户已经指定目录，直接使用，不再询问。

2. **检查项目内是否已有 worktree 目录：**

   ```bash
   ls -d .worktrees 2>/dev/null     # 优先使用隐藏目录
   ls -d worktrees 2>/dev/null      # 备选目录
   ```

   如果找到就使用它。如果两者都存在，优先使用 `.worktrees`。

3. **检查是否已有全局目录：**

   ```bash
   project=$(basename "$(git rev-parse --show-toplevel)")
   ls -d ~/.config/superpowers/worktrees/$project 2>/dev/null
   ```

   如果存在就使用它。这是为了兼容旧版全局路径。

4. **如果没有其它指引，**默认使用项目根目录下的 `.worktrees/`。

#### 安全验证（仅项目内目录）

**在创建 worktree 前，必须确认目标目录会被 git 忽略：**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：**将对应目录加入 `.gitignore`，先让用户确认这项修改，再提交，然后再继续。

**为什么关键：**避免把 worktree 内容误提交到当前仓库。

全局目录（`~/.config/superpowers/worktrees/`）不需要这项验证。

#### 创建 Worktree

```bash
project=$(basename "$(git rev-parse --show-toplevel)")

# 根据选定位置确定路径
# 项目内目录：path="$LOCATION/$BRANCH_NAME"
# 全局目录：path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**沙箱回退：**如果 `git worktree add` 因权限错误失败，告诉用户沙箱阻止了 worktree 创建，本次将改为在当前目录工作。然后在当前目录运行项目设置和基线测试。

## 第 2 步：项目设置

自动检测项目类型，并运行合适的设置命令：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

中国网络环境下，如果安装依赖因为 registry、代理或认证问题失败，只报告失败原因并提示用户按团队规范配置。不要自动修改全局 registry、代理、认证令牌或 shell 配置。

## 第 3 步：验证干净基线

运行项目测试，确认工作区起点是干净的：

```bash
# 使用项目合适的命令
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**报告失败内容，询问用户是继续实现还是先调查失败。

**如果测试通过：**报告工作区已准备好。

### 报告格式

```text
隔离工作区已准备好：<full-path>
测试通过：<N> 个测试，0 个失败
可以开始实现：<feature-name>
```

如果未能创建 worktree，应明确报告真实状态：

```text
未创建隔离 worktree：<原因>
当前将在原目录继续，已完成/未完成基线验证：<证据>
```

## 快速参考

| 场景 | 动作 |
|---|---|
| 已在 linked worktree 中 | 跳过创建（第 0 步） |
| 在子模块中 | 按普通仓库处理（第 0 步保护） |
| 有原生 worktree 工具 | 使用原生工具（1a） |
| 没有原生工具 | 使用 `git worktree` 回退方案（1b） |
| `.worktrees/` 存在 | 使用它，并验证已被忽略 |
| `worktrees/` 存在 | 使用它，并验证已被忽略 |
| 两者都存在 | 使用 `.worktrees/` |
| 两者都不存在 | 先看指令，再默认 `.worktrees/` |
| 全局路径存在 | 使用它，兼容旧路径 |
| 目录未被忽略 | 加入 `.gitignore`，经用户确认后再提交 |
| 创建时权限错误 | 沙箱回退，在当前目录工作 |
| 基线测试失败 | 报告失败并询问 |
| 没有 `package.json` / `Cargo.toml` | 跳过对应依赖安装 |

## 常见错误

### 和运行环境对抗

- **问题：**平台已经提供隔离能力，却仍手动运行 `git worktree add`。
- **修正：**第 0 步先检测现有隔离；1a 优先使用原生工具。

### 跳过检测

- **问题：**在已有 worktree 内又创建嵌套 worktree。
- **修正：**创建任何东西前，始终先运行第 0 步。

### 跳过 ignore 验证

- **问题：**worktree 内容被 git 追踪，污染 `git status`。
- **修正：**创建项目内 worktree 前，始终使用 `git check-ignore` 验证。

### 擅自假定目录位置

- **问题：**制造不一致，违背项目约定。
- **修正：**遵循优先级：用户指令 > 已有项目目录 > 旧全局目录 > 默认目录。

### 测试失败还继续推进

- **问题：**后续无法区分新 bug 和已有失败。
- **修正：**报告失败，获得用户明确许可后再继续。

### 自动改全局配置

- **问题：**依赖安装失败后，擅自修改 npm、pip、git、代理或 shell 全局配置。
- **修正：**只报告失败和可能原因，提示用户按团队规范处理；除非用户明确要求，否则不改全局配置。

## 红色警报

**绝不要：**

- 第 0 步检测到已有隔离时还创建 worktree。
- 已有原生 worktree 工具（如 `EnterWorktree`）时仍手动使用 `git worktree add`。这是最常见错误；有原生工具就用原生工具。
- 跳过 1a，直接运行 1b 的 git 命令。
- 创建项目内 worktree 前不验证目录是否被忽略。
- 跳过基线测试验证。
- 基线测试失败时，不询问就继续推进。
- 在没有用户明确要求时修改全局 registry、代理、认证令牌或 shell 配置。

**始终要：**

- 先运行第 0 步检测。
- 优先使用原生工具，而不是 git 回退方案。
- 按目录优先级选择：用户指令 > 已有项目目录 > 旧全局目录 > 默认目录。
- 对项目内目录验证是否已被忽略。
- 自动检测并运行项目设置。
- 验证干净测试基线。
