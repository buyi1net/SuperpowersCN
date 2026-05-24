# 测试反模式

**加载此参考的时机：**正在编写或修改测试、添加 mock，或想给生产代码添加只供测试使用的方法。

## 总览

测试必须验证真实行为，而不是验证 mock 行为。mock 是隔离手段，不是被测试对象。

**核心原则：测试代码做了什么，而不是 mock 做了什么。**

**严格遵循 TDD 可以避免这些反模式。**

## 铁律

```text
1. 永远不要测试 mock 行为
2. 永远不要给生产类添加只供测试使用的方法
3. 永远不要在不理解依赖的情况下 mock
```

## 反模式 1：测试 mock 行为

**违规示例：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么错：**

- 你验证的是 mock 能工作，不是组件能工作
- 有 mock 时测试通过，没有 mock 时测试失败
- 它不能说明真实行为是否正确

**你的 human partner 的纠正：**“我们是在测试 mock 的行为吗？”

**修复方式：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 闸门函数

```text
在对任何 mock 元素写断言之前：
  先问：“我是在测试真实组件行为，还是只是在测试 mock 是否存在？”

  如果是在测试 mock 是否存在：
    停下 - 删除这个断言，或取消 mock

  改测真实行为
```

## 反模式 2：生产代码中的测试专用方法

**违规示例：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**为什么错：**

- 生产类被测试专用代码污染
- 如果在生产环境中被误调用，会有危险
- 违反 YAGNI 和关注点分离
- 混淆对象生命周期和实体生命周期

**修复方式：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 闸门函数

```text
在给生产类添加任何方法之前：
  先问：“这个方法是否只会被测试使用？”

  如果是：
    停下 - 不要添加
    把它放进测试工具中

  再问：“这个类是否拥有该资源的生命周期？”

  如果不是：
    停下 - 这个方法不属于这个类
```

## 反模式 3：不理解依赖就 mock

**违规示例：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**为什么错：**

- 被 mock 的方法有测试依赖的副作用（写入配置）
- 为了“保险”而过度 mock，破坏了真实行为
- 测试可能因错误原因通过，或莫名其妙失败

**修复方式：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 闸门函数

```text
在 mock 任何方法之前：
  停下 - 先不要 mock

  1. 问：“真实方法有哪些副作用？”
  2. 问：“这个测试是否依赖其中任何副作用？”
  3. 问：“我是否完全理解这个测试需要什么？”

  如果依赖副作用：
    在更低层级 mock（真正缓慢或外部的操作）
    或使用保留必要行为的测试替身
    不要 mock 测试本身依赖的高层方法

  如果不确定测试依赖什么：
    先用真实实现运行测试
    观察真正需要发生什么
    然后只在正确层级添加最小 mock

  红色警报：
    - “我先 mock 掉，保险一点”
    - “这个可能慢，最好 mock”
    - 没理解依赖链就 mock
```

## 反模式 4：不完整 mock

**违规示例：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**为什么错：**

- **局部 mock 会隐藏结构假设** - 你只 mock 了自己知道的字段
- **下游代码可能依赖你没包含的字段** - 静默失败
- **测试通过但集成失败** - mock 不完整，真实 API 是完整的
- **虚假信心** - 测试无法证明真实行为

**铁律：**mock 真实存在的完整数据结构，而不是只 mock 当前测试直接用到的字段。

**修复方式：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 闸门函数

```text
在创建 mock 响应之前：
  检查：“真实 API 响应包含哪些字段？”

  操作：
    1. 从文档/示例中查看真实 API 响应
    2. 包含系统下游可能消费的所有字段
    3. 验证 mock 与真实响应 schema 完整匹配

  关键点：
    如果你在创建 mock，就必须理解完整结构
    当代码依赖遗漏字段时，局部 mock 会静默失败

  如果不确定：包含所有已文档化字段
```

## 反模式 5：把集成测试当成事后补充

**违规示例：**
```text
✅ 实现完成
❌ 没有写测试
“准备好测试了”
```

**为什么错：**

- 测试是实现的一部分，不是可选后续工作
- TDD 本该提前抓住这个问题
- 没有测试就不能声称完成

**修复方式：**
```text
TDD 循环：
1. 编写失败测试
2. 实现到测试通过
3. 重构
4. 然后才声称完成
```

## 当 mock 变得过于复杂

**警告信号：**

- mock 准备代码比测试逻辑还长
- 为了让测试通过而 mock 一切
- mock 缺少真实组件拥有的方法
- mock 改动会导致测试崩掉

**你的 human partner 的问题：**“这里真的需要用 mock 吗？”

**考虑：**使用真实组件的集成测试通常比复杂 mock 更简单。

## TDD 如何避免这些反模式

**TDD 有帮助的原因：**

1. **先写测试** -> 迫使你思考自己到底在测试什么
2. **看着它失败** -> 确认测试测的是真实行为，而不是 mock
3. **最小实现** -> 不会混入测试专用方法
4. **真实依赖** -> mock 之前先看见测试实际需要什么

**如果你正在测试 mock 行为，就已经违反了 TDD** - 你没有先看着测试针对真实代码失败，就添加了 mock。

## 快速参考

| 反模式 | 修复 |
|---|---|
| 对 mock 元素断言 | 测试真实组件，或取消 mock |
| 生产代码中的测试专用方法 | 移到测试工具中 |
| 不理解依赖就 mock | 先理解依赖，最小化 mock |
| 不完整 mock | 完整镜像真实 API |
| 事后补测试 | TDD - 先写测试 |
| mock 过于复杂 | 考虑集成测试 |

## 红色警报

- 断言检查的是 `*-mock` 测试 ID
- 方法只在测试文件中调用
- mock 准备代码超过测试的一半
- 移除 mock 后测试失败
- 说不清为什么需要 mock
- “保险起见”而 mock

## 底线

**mock 是隔离工具，不是测试对象。**

如果 TDD 暴露出你正在测试 mock 行为，你已经走偏了。

修复方式：测试真实行为，或重新追问为什么要 mock。
