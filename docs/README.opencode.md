# 在 OpenCode 中使用 Superpowers Skills

与 [OpenCode.ai](https://opencode.ai) 配合使用的简要指南。

## 安装

在 `opencode.json`（全局或项目级）中添加本仓库的 `skills` 路径：

```json
{
  "skills": {
    "paths": ["/path/to/skills"]
  }
}
```

重启 OpenCode。OpenCode 会扫描该目录下的 `**/SKILL.md`。

验证安装：让 OpenCode 使用 `skill` 工具列出 skills，或请求它加载某个具体 skill。

如果你同时使用 Claude Code、Codex、Gemini CLI 或其他 harness，需要为每个环境单独配置对应入口。

## 使用

### 查找 Skills

使用 OpenCode 原生的 `skill` 工具列出所有可用 skills：

```
use skill tool to list skills
```

### 加载 Skill

```
use skill tool to load brainstorming
```

### 个人 Skills

在 `~/.config/opencode/skills/` 中创建你自己的 skills：

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

创建 `~/.config/opencode/skills/my-skill/SKILL.md`：

```markdown
---
name: my-skill
description: 用于[适用场景]。[说明它做什么]。用户说"[口语触发词]"或"[另一种说法]"时使用。
---

# My Skill

[在这里填写你的 skill 内容]
```

### 项目 Skills

在项目的 `.opencode/skills/` 中创建项目专属 skills。

**Skill 优先级：** 项目 skills > 个人 skills > Superpowers skills

## 更新

更新本地仓库后，重启 OpenCode。运行中的会话不会热加载新增或修改后的 skill。

## 工作原理

OpenCode 的 skill loader 会扫描 `skills.paths` 下的 `**/SKILL.md` 文件。每个 skill 的 `description` 用于判断何时加载该 skill。

### 工具映射

为 Claude Code 编写的 skills 会自动适配 OpenCode：

- `TodoWrite` → `todowrite`
- `Task`（子 agent）→ OpenCode 的 `@mention` 系统
- `Skill` 工具 → OpenCode 原生的 `skill` 工具
- 文件操作 → OpenCode 原生文件工具

## 排错

### 插件未加载

1. 检查 `opencode.json` 中的 `skills.paths` 是否指向本仓库的 `skills/` 目录。
2. 确认路径内存在多个 `SKILL.md` 文件。
3. 修改配置后重启 OpenCode。

### Skills 未找到

1. 使用 OpenCode 的 `skill` 工具列出可用 skills
2. 检查插件是否正在加载（见上文）
3. 每个 skill 都需要一个包含有效 YAML frontmatter 的 `SKILL.md` 文件

### 引导未出现

本仓库当前只声明 skills 集合，不提供 OpenCode 插件级自动引导。如果需要会话启动时自动注入引导上下文，需要另行实现 OpenCode 插件。

## 获取帮助

- OpenCode 文档：https://opencode.ai/docs/
