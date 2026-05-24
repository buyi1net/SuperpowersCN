# Superpowers 非 Skill 文档汉化计划

> 当前状态：86 个文件，仅 README.md（130 行）为中文。其余 85 个全部英文。

---

## 一、分类与决策

| 分类 | 数量 | 决策 | 理由 |
|------|:---:|:--:|------|
| **文档 .md**（可译） | 11 | **汉化** | 面向使用者，影响上手体验 |
| **文档 .md**（不译） | 19 | **跳过** | 历史开发计划/规格，低价值 |
| **代码/配置** | 54 | **不译** | JSON / sh / js / py / cmd / txt prompt |
| **图片** | 2 | 不动 | 二进制资源 |

---

## 二、汉化清单（11 个文件，约 1,635 行）

按优先级排列：

| 优先级 | 文件 | 行数 | 内容 | 策略 |
|:---:|------|:---:|------|:--:|
| P0 | `README.md` | 130 | 项目首页说明 | ✅ 已汉化 |
| P0 | `CLAUDE.md` | 60 | Contributor 指南 | 全译 |
| P0 | `docs/README.opencode.md` | 104 | OpenCode 集成指南 | 全译 |
| P1 | `AGENTS.md` | 1 | 入口指针 | 全译（内容仅 `CLAUDE.md`） |
| P1 | `GEMINI.md` | 2 | 入口指针 | 全译（内容仅 `@./`） |
| P1 | `docs/testing.md` | 224 | 测试方法说明 | 全译 |
| P1 | `docs/windows/polyglot-hooks.md` | 170 | Windows Hook 说明 | 全译 |
| P2 | `CODE_OF_CONDUCT.md` | 93 | 行为准则 | 全译 |
| P2 | `RELEASE-NOTES.md` | 773 | 版本发布记录 | **仅译最新版**，历史版本保留英文 |

---

## 三、跳过清单（19 个文档）

| 路径 | 数量 | 原因 |
|------|:---:|------|
| `docs/plans/` | 4 | 历史开发计划，已执行完毕 |
| `docs/superpowers/plans/` | 5 | 同上 |
| `docs/superpowers/specs/` | 5 | 历史设计规范，已定稿 |
| `tests/claude-code/README.md` | 1 | 测试套件说明，开发者参考 |
| `tests/subagent-driven-dev/go-fractals/` | 2 | 测试用示例项目 |
| `tests/subagent-driven-dev/svelte-todo/` | 2 | 测试用示例项目 |

---

## 四、不译清单（54 个代码/配置）

| 分类 | 数量 | 文件类型 |
|------|:---:|------|
| 根级配置 | 2 | `package.json`、`gemini-extension.json` |
| 许可证 | 1 | `LICENSE`（MIT，法律文本不翻译） |
| hooks/ | 4 | `.json` + `.cmd` + `session-start`（bash） |
| scripts/ | 2 | `bump-version.sh`、`sync-to-codex-plugin.sh` |
| tests/ | 45 | `.sh` `.js` `.mjs` `.py` `.txt` `.json` |

---

## 五、执行步骤

| 步 | 内容 | 行数 |
|:--:|------|:---:|
| 1 | `CLAUDE.md` 出草稿 + 审核 + 定稿 | 60 |
| 2 | `docs/README.opencode.md` 出草稿 + 审核 + 定稿 | 104 |
| 3 | `AGENTS.md` + `GEMINI.md`（体量极小） | 3 |
| 4 | `docs/testing.md` 出草稿 + 审核 | 224 |
| 5 | `docs/windows/polyglot-hooks.md` 出草稿 + 审核 | 170 |
| 6 | `CODE_OF_CONDUCT.md` 出草稿 + 审核 | 93 |
| 7 | `RELEASE-NOTES.md` 仅译最新版 | ~50 |
| 总计 | 7 份草稿 | ~704 行 |

---

## 六、注意事项

- 翻译流程与 skill 相同：出 `_zh-CN.enhanced.md` → 审核 → 定稿（删原稿，重命名）
- `CLAUDE.md` 和 `AGENTS.md` 是 agent 直接读取的配置文件，翻译需注意不破坏其中的文件引用路径和命令
- `CODE_OF_CONDUCT.md` 保留法律条款号
- `RELEASE-NOTES.md` 仅译最新版本（约 50 行），历史 changelog 不动
- `LICENSE` 不译（法律文本）
