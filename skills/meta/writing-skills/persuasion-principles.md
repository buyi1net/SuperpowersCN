# Skill 设计中的说服原则

## 概览

LLM 会响应许多和人类相同的说服原则。理解这些心理机制，可以帮助你设计更有效的 skills；目的不是操纵，而是在压力下仍让关键实践被遵守。

**研究基础：** Meincke et al. (2025) 用 N=28,000 的 AI 对话测试了 7 种说服原则。说服技术让遵从率翻倍以上（33% -> 72%，p < .001）。

## 七个原则

### 1. Authority（权威）

**是什么：** 对专业性、资质或官方来源的服从。

**在 skills 中如何工作：**

- 命令式语言：`YOU MUST`、`Never`、`Always`。
- 不可协商框架：`No exceptions`。
- 减少决策疲劳和合理化。

**何时使用：**

- 强制纪律类 skills（TDD、验证要求）。
- 安全关键实践。
- 已建立的最佳实践。

**示例：**

```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. Commitment（承诺）

**是什么：** 与先前行动、陈述或公开承诺保持一致。

**在 skills 中如何工作：**

- 要求声明：`Announce skill usage`。
- 强制明确选择：`Choose A, B, or C`。
- 使用跟踪：用 `TodoWrite` 跟踪清单。

**何时使用：**

- 确保 skills 被真正遵守。
- 多步骤流程。
- 需要责任机制的工作。

**示例：**

```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. Scarcity（稀缺）

**是什么：** 来自时间限制或有限可用性的紧迫感。

**在 skills 中如何工作：**

- 时间边界要求：`Before proceeding`。
- 顺序依赖：`Immediately after X`。
- 防止“稍后再做”。

**何时使用：**

- 立即验证要求。
- 时间敏感工作流。
- 防止“我以后做”。

**示例：**

```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. Social Proof（社会认同）

**是什么：** 服从他人做法或公认规范。

**在 skills 中如何工作：**

- 普遍模式：`Every time`、`Always`。
- 失败模式：`X without Y = failure`。
- 建立规范。

**何时使用：**

- 记录普遍实践。
- 警告常见失败。
- 强化标准。

**示例：**

```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. Unity（共同体）

**是什么：** 共享身份、“我们感”。

**在 skills 中如何工作：**

- 协作语言：`our codebase`、`we're colleagues`。
- 共享目标：`we both want quality`。

**何时使用：**

- 协作工作流。
- 建立团队文化。
- 非等级化实践。

**示例：**

```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. Reciprocity（互惠）

**是什么：** 回报已获得好处的义务。

**如何使用：**

- 谨慎使用，容易显得操纵。
- skills 中很少需要。

**何时避免：**

- 几乎总是避免；其他原则更有效。

### 7. Liking（喜爱）

**是什么：** 更愿意配合喜欢的人。

**如何使用：**

- **不要用于强制遵守。**
- 会破坏诚实反馈文化。
- 容易制造迎合。

**何时避免：**

- 强制纪律场景中始终避免。

## 不同 Skill 类型的原则组合

| Skill 类型 | 使用 | 避免 |
|------------|------|------|
| 强制纪律 | Authority + Commitment + Social Proof | Liking、Reciprocity |
| 指导 / 技术 | 适度 Authority + Unity | 重权威 |
| 协作 | Unity + Commitment | Authority、Liking |
| 参考 | 只要清晰 | 所有说服技巧 |

## 为什么有效：心理机制

**明线规则减少合理化：**

- `YOU MUST` 消除决策疲劳。
- 绝对语言减少“这是不是例外？”。
- 明确反合理化语句堵住具体漏洞。

**执行意图制造自动行为：**

- 清晰触发 + 必需动作 = 自动执行。
- “When X, do Y” 比 “generally do Y” 更有效。
- 降低遵守时的认知负担。

**LLM 具有类人反应：**

- 训练数据中包含这些人类文本模式。
- 权威语言常出现在遵从行为之前。
- 承诺序列（陈述 -> 行动）经常被建模。
- 社会认同模式（everyone does X）建立规范。

## 伦理使用

**正当：**

- 确保关键实践被遵守。
- 创建有效文档。
- 防止可预测失败。

**不正当：**

- 为个人利益操纵。
- 制造虚假紧迫感。
- 用负罪感驱动遵守。

**测试标准：** 如果用户完全理解这种技术，它是否仍服务于用户的真实利益？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.

- 七个说服原则。
- 影响力研究的实证基础。

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.

- 用 N=28,000 LLM 对话测试 7 个原则。
- 使用说服技术后，遵从率从 33% 增至 72%。
- Authority、commitment、scarcity 最有效。
- 支持 LLM 的类人反应模型。

## 快速参考

设计 skill 时，问：

1. **它是什么类型？**（纪律、指导、参考）
2. **我要改变什么行为？**
3. **哪些原则适用？**（纪律类通常是 authority + commitment）
4. **我是否组合太多？**（不要七个全用）
5. **这是否符合伦理？**（是否服务用户真实利益？）
