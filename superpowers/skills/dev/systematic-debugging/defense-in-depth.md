# Defense-in-Depth 校验

## 总览

修复由无效数据导致的 bug 时，只在一个地方加校验看起来够了。但单点检查可能被其他代码路径、重构或 mock 绕过。

**核心原则：在数据经过的每一层校验。让 bug 在结构上不可能发生。**

## 为什么需要多层

单层校验：“我们修了 bug。”  
多层校验：“我们让 bug 不可能发生。”

不同层捕捉不同情况：

- 入口校验捕捉大多数 bug
- 业务逻辑捕捉边界情况
- 环境 guard 防止特定上下文中的危险操作
- 调试日志帮助排查其他层失败时的情况

## 四层

### 第 1 层：入口点校验

**目的：**在 API 边界拒绝明显无效输入。

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第 2 层：业务逻辑校验

**目的：**确保数据对当前操作有意义。

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第 3 层：环境 guard

**目的：**在特定上下文中阻止危险操作。

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第 4 层：调试诊断

**目的：**捕获上下文，供事后取证。

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 应用模式

找到 bug 后：

1. **追踪数据流** - 坏值从哪里来？在哪里被使用？
2. **映射所有检查点** - 列出数据经过的每个点
3. **在每层添加校验** - 入口、业务、环境、调试
4. **测试每一层** - 尝试绕过第 1 层，验证第 2 层能拦住

## 会话示例

Bug：空 `projectDir` 导致在源代码目录执行 `git init`

**数据流：**

1. 测试设置 -> 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 中运行

**添加的四层：**

- 第 1 层：`Project.create()` 校验非空/存在/可写
- 第 2 层：`WorkspaceManager` 校验 projectDir 非空
- 第 3 层：`WorktreeManager` 在测试中拒绝在 tmpdir 外执行 git init
- 第 4 层：git init 前记录堆栈

**结果：**全部 1847 个测试通过，bug 无法复现。

## 关键洞察

四层都必要。测试过程中，每层都抓住了其他层漏掉的问题：

- 不同代码路径绕过了入口校验
- mock 绕过了业务逻辑检查
- 不同平台上的边界情况需要环境 guard
- 调试日志定位了结构性误用

**不要停在一个校验点。**在每一层都加检查。
