# 可视化伴随工具指南

这是一个基于浏览器的可视化 brainstorming 辅助工具，用于展示 mockup、图表和选项。

## 何时使用

按每个问题单独判断，而不是按整个会话判断。判断标准是：**用户看见它是否会比阅读它更容易理解？**

**内容本身是视觉内容时，使用浏览器：**

- **UI mockup**：线框图、布局、导航结构、组件设计。
- **架构图**：系统组件、数据流、关系图。
- **并排视觉对比**：比较两种布局、两套配色、两个设计方向。
- **设计打磨**：当问题涉及观感、间距、视觉层级时。
- **空间关系**：状态机、流程图、实体关系图。

**内容是文本或表格时，使用终端：**

- **需求和范围问题**：例如“X 是什么意思？”、“哪些功能在范围内？”。
- **概念性 A/B/C 选择**：在用文字描述的方案之间选择。
- **取舍列表**：优缺点、对比表。
- **技术决策**：API 设计、数据建模、架构方案选择。
- **澄清问题**：任何答案主要是文字，而不是视觉偏好的问题。

关于 UI 的问题不一定就是视觉问题。“你想要哪类向导？”是概念问题，使用终端。“这几种向导布局哪个感觉更合适？”是视觉问题，使用浏览器。

## 工作方式

服务器会监听一个目录中的 HTML 文件，并把最新文件提供给浏览器。你把 HTML 内容写入 `screen_dir`，用户会在浏览器中看到它，并且可以点击选择选项。选择会记录到 `state_dir/events`，你在下一轮读取。

**内容片段 vs 完整文档：** 如果 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器会原样提供它（只注入辅助脚本）。否则，服务器会自动用框架模板包裹你的内容，添加页眉、CSS 主题、选择状态提示和所有交互基础设施。**默认写内容片段。** 只有在需要完全控制页面时，才写完整文档。

## 启动会话

```bash
# Start server with persistence (mockups saved to project)
scripts/start-server.sh --project-dir /path/to/project

# Returns: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

保存返回结果中的 `screen_dir` 和 `state_dir`。告诉用户打开该 URL。

**查找连接信息：** 服务器会把启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动了服务器但没有捕获 stdout，读取这个文件即可获得 URL 和端口。使用 `--project-dir` 时，到 `<project>/.superpowers/brainstorm/` 中查找会话目录。

**注意：** 把项目根目录作为 `--project-dir` 传入，这样 mockup 会持久保存到 `.superpowers/brainstorm/`，服务器重启后仍保留。不传时，文件会写到 `/tmp` 并在清理时删除。如果项目还没有忽略 `.superpowers/`，提醒用户把它加入 `.gitignore`。

**按平台启动服务器：**

**Claude Code（macOS / Linux）：**
```bash
# Default mode works — the script backgrounds the server itself
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code（Windows）：**
```bash
# Windows auto-detects and uses foreground mode, which blocks the tool call.
# Use run_in_background: true on the Bash tool call so the server survives
# across conversation turns.
scripts/start-server.sh --project-dir /path/to/project
```
通过 Bash 工具调用时，设置 `run_in_background: true`。然后在下一轮读取 `$STATE_DIR/server-info` 获取 URL 和端口。

**Codex：**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# switches to foreground mode. Run it normally — no extra flags needed.
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# Use --foreground and set is_background: true on your shell tool call
# so the process survives across turns
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他环境：** 服务器必须在对话轮次之间持续在后台运行。如果你的环境会回收分离的进程，使用 `--foreground`，并配合所在平台的后台执行机制启动命令。

如果浏览器无法访问 URL（远程或容器环境中很常见），绑定非 loopback 主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 控制返回的 URL JSON 中打印的主机名。

## 循环流程

1. **检查服务器仍在运行**，然后把 **HTML 写入** `screen_dir` 中的新文件：
   - 每次写入前，检查 `$STATE_DIR/server-info` 是否存在。如果不存在（或 `$STATE_DIR/server-stopped` 存在），说明服务器已关闭；继续前先用 `start-server.sh` 重启。服务器会在 30 分钟无活动后自动退出。
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`。
   - **绝不复用文件名**：每个屏幕都使用新文件。
   - 使用 Write 工具，**不要使用 cat/heredoc**（会把噪声输出到终端）。
   - 服务器会自动提供最新文件。

2. **告诉用户会看到什么，然后结束当前轮次：**
   - 每一步都提醒用户 URL，不要只在第一次提醒。
   - 简短概括屏幕内容，例如“正在展示首页的 3 种布局选项”。
   - 请用户在终端回复：“看一下，然后告诉我你的想法。也可以点击选择一个选项。”

3. **下一轮中**，用户在终端回复后：
   - 如果 `$STATE_DIR/events` 存在，读取它；其中包含用户在浏览器中的交互（点击、选择），格式是 JSON lines。
   - 把浏览器事件和用户的终端文字合并理解。
   - 终端消息是主要反馈；`state_dir/events` 提供结构化交互数据。

4. **迭代或前进**：如果反馈改变当前屏幕，写入新文件，例如 `layout-v2.html`。只有当前步骤确认后，才进入下一个问题。

5. **回到终端时卸载视觉内容**：当下一步不需要浏览器时（例如澄清问题、取舍讨论），推送等待屏幕，清掉过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">继续在终端中讨论...</p>
   </div>
   ```

   这样可以避免用户继续盯着已经解决的选择，而对话已经进入下一步。下一个视觉问题出现时，再像往常一样推送新的内容文件。

6. 重复直到完成。

## 编写内容片段

只写页面内部内容。服务器会自动用框架模板包裹它（页眉、主题 CSS、选择状态提示和所有交互基础设施）。

**最小示例：**

```html
<h2>哪种布局更合适？</h2>
<p class="subtitle">请重点考虑可读性和视觉层级</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>单列</h3>
      <p>干净、聚焦的阅读体验</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>双列</h3>
      <p>侧边栏导航搭配主内容区</p>
    </div>
  </div>
</div>
```

就这些。不需要 `<html>`、CSS 或 `<script>` 标签；服务器会提供。

## 可用 CSS 类

框架模板为你的内容提供以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>标题</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

**多选：** 在容器上添加 `data-multiselect`，让用户可以选择多个选项。每次点击都会切换该项。提示条会显示已选数量。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>名称</h3>
      <p>说明</p>
    </div>
  </div>
</div>
```

### Mockup 容器

```html
<div class="mockup">
  <div class="mockup-header">预览：仪表盘布局</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分栏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 优缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>优点</h4><ul><li>收益</li></ul></div>
  <div class="cons"><h4>缺点</h4><ul><li>代价</li></ul></div>
</div>
```

### Mock 元素（线框图构建块）

```html
<div class="mock-nav">Logo | 首页 | 关于 | 联系</div>
<div style="display: flex;">
  <div class="mock-sidebar">导航</div>
  <div class="mock-content">主内容区域</div>
</div>
<button class="mock-button">操作按钮</button>
<input class="mock-input" placeholder="输入字段">
<div class="placeholder">占位区域</div>
```

### 排版和小节

- `h2`：页面标题。
- `h3`：小节标题。
- `.subtitle`：标题下方的次级文字。
- `.section`：带底部间距的内容块。
- `.label`：小号大写标签文本。

## 浏览器事件格式

用户在浏览器中点击选项时，交互会记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。推送新屏幕后，该文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整事件流能展示用户的探索路径；他们可能会先后点击多个选项再做决定。最后一个 `choice` 事件通常是最终选择，但点击模式也可能暴露犹豫或偏好，值得继续追问。

如果 `$STATE_DIR/events` 不存在，说明用户没有在浏览器中交互，只使用他们的终端文字。

## 设计建议

- **按问题调整保真度**：布局问题用线框图；视觉打磨问题再提高精细度。
- **每页都说明问题**：写“哪种布局更专业？”而不是只写“选一个”。
- **先迭代再前进**：如果反馈改变当前屏幕，写一个新版本。
- **每屏最多 2-4 个选项**。
- **需要真实内容时使用真实内容**：例如摄影作品集应使用真实图片（Unsplash）。占位内容会掩盖设计问题。若网络受限，提示用户提供本地素材或按团队网络规范配置访问方式，不要自动修改全局配置。
- **保持 mockup 简洁**：聚焦布局和结构，不追求像素级设计。

## 文件命名

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`。
- 不要复用文件名：每个屏幕都必须是新文件。
- 迭代时追加版本后缀，例如 `layout-v2.html`、`layout-v3.html`。
- 服务器按修改时间提供最新文件。

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果会话使用了 `--project-dir`，mockup 文件会保留在 `.superpowers/brainstorm/` 中，供后续参考。只有 `/tmp` 会话会在停止时删除。

## 参考

- 框架模板（CSS 参考）：`scripts/frame-template.html`
- 辅助脚本（客户端）：`scripts/helper.js`
