# 测试 Superpowers Skills

本文档说明如何测试 Superpowers skills，尤其是 `subagent-driven-development` 这类复杂 skill 的集成测试。

## 概览

测试涉及子 agent、工作流和复杂交互的 skills，需要在 headless 模式下运行真实的 Claude Code 会话，并通过会话记录来验证行为。

## 测试结构

```
tests/
├── claude-code/
│   ├── test-helpers.sh                    # 共享测试工具
│   ├── test-subagent-driven-development-integration.sh
│   ├── analyze-token-usage.py             # Token 分析工具
│   └── run-skill-tests.sh                 # 测试运行器（如有）
```

## 运行测试

### 集成测试

集成测试会用真实 skills 执行实际的 Claude Code 会话：

```bash
# Run the subagent-driven-development integration test
cd tests/claude-code
./test-subagent-driven-development-integration.sh
```

**注意：** 集成测试可能耗时 10-30 分钟，因为它会执行包含多个子 agent 的真实实现计划。

### 运行要求

- 必须从 **superpowers 插件目录**运行（不能从临时目录运行）
- 必须安装 Claude Code，且 `claude` 命令可用
- 必须启用本地开发者 marketplace：在 `~/.claude/settings.json` 的 `enabledPlugins` 中添加 `"superpowers@superpowers-dev": true`

## 集成测试：subagent-driven-development

### 测试内容

集成测试验证 `subagent-driven-development` skill 是否正确做到：

1. **计划加载**：在开始时读取一次计划
2. **完整任务文本**：向子 agent 提供完整的任务描述（不让它们自己读文件）
3. **自我审查**：确保子 agent 在报告前执行自我审查
4. **审查顺序**：在代码质量审查之前运行规格合规审查
5. **审查循环**：发现问题时使用审查循环
6. **独立验证**：规格审查者独立阅读代码，不信任实现者的报告

### 工作方式

1. **准备**：创建一个临时 Node.js 项目，包含最小实现计划
2. **执行**：在 headless 模式下运行带该 skill 的 Claude Code
3. **验证**：解析会话记录（`.jsonl` 文件），验证：
   - Skill 工具被调用
   - 子 agent 被派发（Task 工具）
   - TodoWrite 被用于跟踪
   - 实现文件被创建
   - 测试通过
   - Git 提交记录符合正确的工作流
4. **Token 分析**：按子 agent 拆解展示 token 用量

### 测试输出

```
========================================
 Integration Test: subagent-driven-development
========================================

Test project: /tmp/tmp.xyz123

=== Verification Tests ===

Test 1: Skill tool invoked...
  [PASS] subagent-driven-development skill was invoked

Test 2: Subagents dispatched...
  [PASS] 7 subagents dispatched

Test 3: Task tracking...
  [PASS] TodoWrite used 5 time(s)

Test 6: Implementation verification...
  [PASS] src/math.js created
  [PASS] add function exists
  [PASS] multiply function exists
  [PASS] test/math.test.js created
  [PASS] Tests pass

Test 7: Git commit history...
  [PASS] Multiple commits created (3 total)

Test 8: No extra features added...
  [PASS] No extra features added

=========================================
 Token Usage Analysis
=========================================

Usage Breakdown:
----------------------------------------------------------------------------------------------------
Agent           Description                          Msgs      Input     Output      Cache     Cost
----------------------------------------------------------------------------------------------------
main            Main session (coordinator)             34         27      3,996  1,213,703 $   4.09
3380c209        implementing Task 1: Create Add Function     1          2        787     24,989 $   0.09
34b00fde        implementing Task 2: Create Multiply Function     1          4        644     25,114 $   0.09
3801a732        reviewing whether an implementation matches...   1          5        703     25,742 $   0.09
4c142934        doing a final code review...                    1          6        854     25,319 $   0.09
5f017a42        a code reviewer. Review Task 2...               1          6        504     22,949 $   0.08
a6b7fbe4        a code reviewer. Review Task 1...               1          6        515     22,534 $   0.08
f15837c0        reviewing whether an implementation matches...   1          6        416     22,485 $   0.07
----------------------------------------------------------------------------------------------------

TOTALS:
  Total messages:         41
  Input tokens:           62
  Output tokens:          8,419
  Cache creation tokens:  132,742
  Cache read tokens:      1,382,835

  Total input (incl cache): 1,515,639
  Total tokens:             1,524,058

  Estimated cost: $4.67
  (at $3/$15 per M tokens for input/output)

========================================
 Test Summary
========================================

STATUS: PASSED
```

## Token 分析工具

### 使用方式

分析任何 Claude Code 会话的 token 用量：

```bash
python3 tests/claude-code/analyze-token-usage.py ~/.claude/projects/<project-dir>/<session-id>.jsonl
```

### 查找会话文件

会话记录存储在 `~/.claude/projects/` 中，工作目录路径会经过编码：

```bash
# Example for /Users/yourname/Documents/GitHub/superpowers/superpowers
SESSION_DIR="$HOME/.claude/projects/-Users-yourname-Documents-GitHub-superpowers-superpowers"

# Find recent sessions
ls -lt "$SESSION_DIR"/*.jsonl | head -5
```

### 展示内容

- **主会话用量**：协调者（你或主 Claude 实例）的 token 用量
- **按子 agent 拆解**：每次 Task 调用包含：
  - Agent ID
  - 描述（从 prompt 提取）
  - 消息数量
  - 输入/输出 token 数
  - 缓存用量
  - 预估费用
- **汇总**：整体 token 用量和费用估算

### 理解输出

- **高缓存读取**：好 —— 说明 prompt 缓存正在生效
- **主会话高输入 token**：符合预期 —— 协调者拥有完整上下文
- **各子 agent 成本相近**：符合预期 —— 每个收到的任务复杂度相似
- **每任务成本**：每个子 agent 通常 $0.05-$0.15，取决于任务量

## 排错

### Skills 未加载

**问题**：headless 测试中找不到 skill

**解决方法**：
1. 确保你从 superpowers 目录运行：`cd /path/to/superpowers && tests/...`
2. 检查 `~/.claude/settings.json` 的 `enabledPlugins` 中是否有 `"superpowers@superpowers-dev": true`
3. 确认 skill 存在于 `skills/` 目录

### 权限错误

**问题**：Claude 被阻止写入文件或访问目录

**解决方法**：
1. 使用 `--permission-mode bypassPermissions` 标志
2. 使用 `--add-dir /path/to/temp/dir` 授予对测试目录的访问
3. 检查测试目录的文件权限

### 测试超时

**问题**：测试耗时太长并超时

**解决方法**：
1. 增加超时：`timeout 1800 claude ...`（30 分钟）
2. 检查 skill 逻辑中是否存在无限循环
3. 审查子 agent 任务的复杂度

### 找不到会话文件

**问题**：测试运行后找不到会话记录

**解决方法**：
1. 检查 `~/.claude/projects/` 中正确的项目目录
2. 使用 `find ~/.claude/projects -name "*.jsonl" -mmin -60` 查找最近会话
3. 确认测试确实运行了（检查测试输出中是否有错误）

## 编写新的集成测试

### 模板

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

# Create test project
TEST_PROJECT=$(create_test_project)
trap "cleanup_test_project $TEST_PROJECT" EXIT

# Set up test files...
cd "$TEST_PROJECT"

# Run Claude with skill
PROMPT="Your test prompt here"
cd "$SCRIPT_DIR/../.." && timeout 1800 claude -p "$PROMPT" \
  --allowed-tools=all \
  --add-dir "$TEST_PROJECT" \
  --permission-mode bypassPermissions \
  2>&1 | tee output.txt

# Find and analyze session
WORKING_DIR_ESCAPED=$(echo "$SCRIPT_DIR/../.." | sed 's/\\//-/g' | sed 's/^-//')
SESSION_DIR="$HOME/.claude/projects/$WORKING_DIR_ESCAPED"
SESSION_FILE=$(find "$SESSION_DIR" -name "*.jsonl" -type f -mmin -60 | sort -r | head -1)

# Verify behavior by parsing session transcript
if grep -q '"name":"Skill".*"skill":"your-skill-name"' "$SESSION_FILE"; then
    echo "[PASS] Skill was invoked"
fi

# Show token analysis
python3 "$SCRIPT_DIR/analyze-token-usage.py" "$SESSION_FILE"
```

### 最佳实践

1. **始终清理**：使用 trap 清理临时目录
2. **解析记录**：不要 grep 面向用户的输出，解析 `.jsonl` 会话文件
3. **授予权限**：使用 `--permission-mode bypassPermissions` 和 `--add-dir`
4. **从插件目录运行**：只有从 superpowers 目录运行时 skills 才会加载
5. **展示 token 用量**：始终包含 token 分析以了解成本
6. **测试真实行为**：验证实际创建的文件、通过的测试、完成的提交

## 会话记录格式

会话记录是 JSONL（JSON Lines）文件，每行是一个 JSON 对象，代表一条消息或工具结果。

### 关键字段

```json
{
  "type": "assistant",
  "message": {
    "content": [...],
    "usage": {
      "input_tokens": 27,
      "output_tokens": 3996,
      "cache_read_input_tokens": 1213703
    }
  }
}
```

### 工具结果

```json
{
  "type": "user",
  "toolUseResult": {
    "agentId": "3380c209",
    "usage": {
      "input_tokens": 2,
      "output_tokens": 787,
      "cache_read_input_tokens": 24989
    },
    "prompt": "You are implementing Task 1...",
    "content": [{"type": "text", "text": "..."}]
  }
}
```

`agentId` 字段关联到子 agent 会话，`usage` 字段包含该子 agent 调用的 token 用量。
