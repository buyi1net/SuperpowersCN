# Claude Code 跨平台多语言 Hooks

Claude Code 插件需要能在 Windows、macOS 和 Linux 上工作的 hooks。本文档说明实现这一目标的多语言包装器技术。

## 问题

Claude Code 通过系统默认 shell 运行 hook 命令：
- **Windows**：CMD.exe
- **macOS/Linux**：bash 或 sh

这带来了几个挑战：

1. **脚本执行**：Windows CMD 不能直接执行 `.sh` 文件——它会尝试用文本编辑器打开
2. **路径格式**：Windows 使用反斜杠（`C:\path`），Unix 使用正斜杠（`/path`）
3. **环境变量**：`$VAR` 语法在 CMD 中不可用
4. **`bash` 不在 PATH 中**：即使安装了 Git Bash，CMD 运行时 PATH 里也没有 `bash`

## 解决方案：多语言 `.cmd` 包装器

一段多语言脚本同时是多种语言的有效语法。我们的包装器在 CMD 和 bash 中都合法：

```cmd
: << 'CMDBLOCK'
@echo off
"C:\Program Files\Git\bin\bash.exe" -l -c "\"$(cygpath -u \"$CLAUDE_PLUGIN_ROOT\")/hooks/session-start.sh\""
exit /b
CMDBLOCK

# Unix shell runs from here
"${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"
```

### 工作原理

#### Windows 端（CMD.exe）

1. `: << 'CMDBLOCK'`——CMD 将 `:` 视为标签（类似于 `:label`），忽略 `<< 'CMDBLOCK'`
2. `@echo off`——关闭命令回显
3. bash.exe 命令运行时附带：
   - `-l`（login shell），以获取包含 Unix 工具的完整 PATH
   - `cygpath -u` 将 Windows 路径转为 Unix 格式（`C:\foo` → `/c/foo`）
4. `exit /b`——退出批处理脚本，CMD 在此停止
5. `CMDBLOCK` 之后的所有内容 CMD 永远不会到达

#### Unix 端（bash/sh）

1. `: << 'CMDBLOCK'`——`:` 是空操作，`<< 'CMDBLOCK'` 开始一个 here-doc（heredoc）
2. 到 `CMDBLOCK` 之前的所有内容都被 heredoc 吃掉（忽略）
3. `# Unix shell runs from here`——注释
4. 脚本直接使用 Unix 路径运行

## 文件结构

```
hooks/
├── hooks.json           # Points to the .cmd wrapper
├── session-start.cmd    # Polyglot wrapper (cross-platform entry point)
└── session-start.sh     # Actual hook logic (bash script)
```

### hooks.json

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/session-start.cmd\""
          }
        ]
      }
    ]
  }
}
```

注意：路径必须用引号包围，因为 `${CLAUDE_PLUGIN_ROOT}` 在 Windows 上可能包含空格（如 `C:\Program Files\...`）。

## 环境要求

### Windows
- 必须安装 **Git for Windows**（提供 `bash.exe` 和 `cygpath`）
- 默认安装路径：`C:\Program Files\Git\bin\bash.exe`
- 如果 Git 安装在其他位置，需要修改包装器中的路径

### Unix（macOS/Linux）
- 标准 bash 或 sh shell
- 在 Unix 端运行时，`.cmd` 文件需要执行权限（`chmod +x`）

## 编写跨平台 Hook 脚本

实际的 hook 逻辑写在 `.sh` 文件中。为确保它在 Windows 上（通过 Git Bash）也能运行：

### 应做：
- 优先使用纯 bash 内建功能
- 使用 `$(command)` 而非反引号
- 用引号包围所有变量展开：`"$VAR"`
- 使用 `printf` 或 here-doc 输出

### 应避免：
- PATH 中可能不存在的外部命令（sed、awk、grep）
- 如果必须使用它们，它们在 Git Bash 中可用，但需确保 PATH 设置正确（使用 `bash -l`）

### 示例：不使用 sed/awk 做 JSON 转义

不要这样：
```bash
escaped=$(echo "$content" | sed 's/\\/\\\\/g' | sed 's/"/\\"/g' | awk '{printf "%s\\n", $0}')
```

用纯 bash：
```bash
escape_for_json() {
    local input="$1"
    local output=""
    local i char
    for (( i=0; i<${#input}; i++ )); do
        char="${input:$i:1}"
        case "$char" in
            $'\\') output+='\\' ;;
            '"') output+='\"' ;;
            $'\n') output+='\n' ;;
            $'\r') output+='\r' ;;
            $'\t') output+='\t' ;;
            *) output+="$char" ;;
        esac
    done
    printf '%s' "$output"
}
```

## 可复用包装器模式

对于有多个 hooks 的插件，可以创建一个通用包装器，将脚本名作为参数传入：

### run-hook.cmd
```cmd
: << 'CMDBLOCK'
@echo off
set "SCRIPT_DIR=%~dp0"
set "SCRIPT_NAME=%~1"
"C:\Program Files\Git\bin\bash.exe" -l -c "cd \"$(cygpath -u \"%SCRIPT_DIR%\")\" && \"./%SCRIPT_NAME%\""
exit /b
CMDBLOCK

# Unix shell runs from here
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
"${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

### 使用可复用包装器的 hooks.json
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start.sh"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

## 排错

### "bash is not recognized"
CMD 找不到 bash。包装器使用了完整路径 `C:\Program Files\Git\bin\bash.exe`。如果 Git 安装在其他位置，更新此路径。

### "cygpath: command not found" 或 "dirname: command not found"
Bash 没有以 login shell 运行。确保使用了 `-l` 标志。

### 路径中出现奇怪的 `\/`
`${CLAUDE_PLUGIN_ROOT}` 展开为以反斜杠结尾的 Windows 路径，然后拼接了 `/hooks/...`。用 `cygpath` 转换整个路径。

### 脚本被文本编辑器打开而不是运行
hooks.json 直接指向了 `.sh` 文件。改为指向 `.cmd` 包装器。

### 终端中能运行，但作为 hook 不行
Claude Code 运行 hooks 的方式可能不同。模拟 hook 环境进行测试：
```powershell
$env:CLAUDE_PLUGIN_ROOT = "C:\path\to\plugin"
cmd /c "C:\path\to\plugin\hooks\session-start.cmd"
```

## 相关 Issue

- [anthropics/claude-code#9758](https://github.com/anthropics/claude-code/issues/9758)——.sh 脚本在 Windows 上被编辑器打开
- [anthropics/claude-code#3417](https://github.com/anthropics/claude-code/issues/3417)——Hooks 在 Windows 上不工作
- [anthropics/claude-code#6023](https://github.com/anthropics/claude-code/issues/6023)——CLAUDE_PROJECT_DIR 未找到
