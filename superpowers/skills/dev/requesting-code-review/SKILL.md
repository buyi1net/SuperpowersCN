---
name: requesting-code-review
description: 完成任务、实现重大功能、或合并前验证工作是否符合需求时使用。用户说“帮我 review 一下”“合并前审查”“代码写完了看一下”“提交前检查一下”时也可触发。
disable-model-invocation: true
---

# 请求代码审查

派一个代码审查子 agent，在问题继续扩散到后续工作前把它们拦下来。审查者只接收精确整理过的审查上下文，不接收你当前会话的完整历史；这样可以让审查聚焦于工作产物，而不是你的思考过程，同时保留当前会话上下文用于继续推进。

**核心原则：** 早审查，经常审查。

## 什么时候请求审查

**必须审查：**
- subagent-driven development 中每完成一个任务后
- 完成重大功能后
- 合并到主分支前

**可选但很有价值：**
- 卡住时，用新视角排查
- 重构前，先做基线检查
- 修复复杂 bug 后
- 涉及安全、权限、数据迁移、支付、用户隐私或生产配置时

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 也可以使用 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

如果用户已经给出提交范围，优先使用用户提供的 `{BASE_SHA}` 和 `{HEAD_SHA}`。如果工作区有未提交改动，先说明审查范围只覆盖提交范围；除非用户明确要求，也需要审查未提交 diff。

**2. 派发代码审查子 agent：**

使用 Task tool，类型为 `general-purpose`，填写 `code-reviewer.md` 模板。

**占位符：**
- `{DESCRIPTION}` - 简要说明你完成了什么
- `{PLAN_OR_REQUIREMENTS}` - 它应该满足的计划、需求或验收标准
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交

**3. 处理反馈：**
- Critical 问题必须立即修复
- Important 问题必须在继续推进前修复
- Minor 问题可以记录到后续任务
- 如果审查者判断错误，可以用技术理由反驳

不要把审查当作形式流程。只有当 Critical 和 Important 反馈已经处理或被充分反驳后，才继续合并或进入下一个任务。

## 示例

```text
[刚完成任务 2：添加验证函数]

你：我先请求代码审查，再继续。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发代码审查子 agent]
  DESCRIPTION: 添加 verifyIndex() 和 repairIndex()，覆盖 4 类问题
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661

[子 agent 返回]：
  优点：架构清晰，测试覆盖真实行为
  问题：
    Important：缺少进度提示
    Minor：报告间隔使用了魔法数字 100
  评估：修复后可以继续

你：[修复进度提示]
[继续任务 3]
```

## 与工作流集成

**Subagent-Driven Development：**
- 每个任务后都审查
- 在问题叠加前拦截
- 修复后再进入下一个任务

**Executing Plans：**
- 每个任务或自然检查点后审查
- 获取反馈、应用反馈、继续执行

**临时开发：**
- 合并前审查
- 卡住时审查

## 中国本土化注意事项

- 如果远程仓库在 GitHub、GitLab、Gitee、Coding.net 或自托管平台上，审查仍以本地 `git diff` 为准，不要求访问远端平台。
- 如果需要使用 `gh`、`glab` 或内部平台 CLI 拉取 PR/MR 信息，只提示用户按团队规范配置认证令牌、registry 或代理；不要自动修改全局配置。
- 输出给用户的审查结论默认使用中文；文件路径、命令、提交 SHA、标签常量和平台名保持原样。

## 红线

**绝不要：**
- 因为“很简单”就跳过审查
- 忽略 Critical 问题
- 带着未修复的 Important 问题继续推进
- 和有效的技术反馈争辩
- 在没有实际读取 diff 的情况下说“看起来没问题”

**如果审查者错了：**
- 用技术理由反驳
- 展示能证明实现正确的代码或测试
- 请求澄清

模板见：requesting-code-review/code-reviewer.md
