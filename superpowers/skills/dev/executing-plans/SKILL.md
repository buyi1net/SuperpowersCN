---
name: executing-plans
description: 已有书面实现计划，并准备在当前会话中按任务执行、在检查点复核时使用。用户说“执行这个计划”、“按计划做”、“按实现计划执行”、“在当前会话里跑这个计划”时也可触发。
disable-model-invocation: true
---

# 执行计划

## 总览

加载计划，严格审查，执行全部任务，并在完成后报告。

**开始时说明：**“我正在使用 `executing-plans` skill 来实现这个计划。”

**注意：**告诉用户，Superpowers 在可使用子 agent 时效果会好很多。如果平台支持子 agent（例如 Claude Code 或 Codex），工作质量通常会明显更高。若子 agent 可用，优先使用 `superpowers:subagent-driven-development`，而不是这个 skill。

## 流程

### 第 1 步：加载并审查计划

1. 读取计划文件。
2. 严格审查计划，找出任何疑问或风险。
3. 如果有疑问或风险，开始实现前先向用户提出。
4. 如果没有疑问或风险，创建 TodoWrite 任务列表，然后继续。

### 第 2 步：执行任务

对每个任务：

1. 标记为 `in_progress`。
2. 严格按计划中的每一步执行（计划已经拆成小块步骤）。
3. 按计划要求运行验证。
4. 标记为 `completed`。

### 第 3 步：完成开发

所有任务完成并验证后：

- 说明：“我正在使用 `finishing-a-development-branch` skill 来完成这项工作。”
- **必需子 skill：**使用 `superpowers:finishing-a-development-branch`
- 按该 skill 验证测试、展示选项、执行用户选择。

## 何时停止并寻求帮助

**遇到下面情况时，必须立刻停止执行：**

- 遇到阻塞项（缺失依赖、测试失败、指令不清）。
- 计划存在关键缺口，导致无法开始。
- 不理解某条指令。
- 验证反复失败。

**优先请求澄清，不要猜。**

## 何时回到前面步骤

**遇到下面情况时，回到审查阶段（第 1 步）：**

- 用户根据你的反馈更新了计划。
- 基本实现思路需要重新考虑。

**不要硬闯阻塞项。** 停下来询问。

## 记住

- 先严格审查计划。
- 严格遵循计划步骤。
- 不要跳过验证。
- 当计划要求使用其它 skill 时，必须引用对应 skill。
- 被阻塞时停止，不要猜，更不要自行修改计划。
- 未经用户明确同意，绝不要在 `main` / `master` 分支上开始实现。

## 集成关系

**必需工作流 skill：**

- **`superpowers:using-git-worktrees`** - 确保隔离工作区存在（创建新的，或验证已有的）。
- **`superpowers:writing-plans`** - 创建本 skill 要执行的计划。
- **`superpowers:finishing-a-development-branch`** - 所有任务完成后收尾开发工作。

## 中国本土化注意事项

- 如果执行计划中的依赖安装、测试或构建命令因为 registry、代理或认证失败，报告失败原因并询问用户下一步。不要自动修改全局 registry、代理、认证令牌或 shell 配置。
- 如果计划要求操作 GitHub、GitLab、Gitee、Coding.net、Jira、Linear 或本地 markdown issue，先确认当前平台和认证是否可用；不可用时改为输出手动步骤，不要替用户登录或修改全局配置。
- 如果计划要求写入语雀、飞书等外部文档系统，而当前环境无法直接操作，只输出可复制的 markdown 草稿或文本步骤，并请用户在对应系统中处理。
