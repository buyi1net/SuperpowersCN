# 代码质量审查子 agent Prompt 模板

派发代码质量审查子 agent 时使用这个模板。

**目的：**验证实现是否构造良好（清晰、已测试、可维护）。

**只有规格符合性审查通过后，才派发。**

```text
Task tool (general-purpose):
  Use template at requesting-code-review/code-reviewer.md

  DESCRIPTION: [task summary, from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
```

**除了标准代码质量关注点，审查者还应该检查：**

- 每个文件是否有一个清晰职责和明确接口？
- 单元是否拆分到可以被独立理解和测试？
- 实现是否遵循计划中的文件结构？
- 这次实现是否创建了已经很大的新文件，或显著扩大了既有文件？不要标记改动前就存在的文件大小问题，重点看本次改动带来的影响。

**代码审查者返回：**优点、问题（Critical/Important/Minor）、评估。
