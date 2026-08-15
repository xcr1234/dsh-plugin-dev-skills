# 事件系统

事件是 Cordis 插件间通信的核心机制。Harness 大量使用事件来实现松耦合的扩展点。事件监听器也是效果：通过 ctx.on() 注册的监听器在插件卸载时自动移除。

## 基本用法

```ts
ctx.on('event-name', (payload) => { /* 处理事件 */ })
ctx.emit('event-name', payload)
```

## 四种分发模式

| 模式 | 语义 | 何时用 |
| --- | --- | --- |
| emit | 广播：所有监听器同步执行，返回值被忽略 | 通知型事件（状态变更、日志） |
| bail | 短路：按顺序运行，第一个非 null/false/undefined 的返回值成为最终结果 | 拦截检查（权限、校验） |
| serial | 顺序执行并等待异步结果；第一个非空返回值终止后续执行 | 有序初始化/阶段流程 |
| waterfall | 流水线：每个监听器可包装下游返回值形成处理链；**必须调用 next()** | 转换、组装、可插拔策略 |

### emit — 广播

```ts
ctx.emit('my-plugin/ready', { id: 'worker-1' })
ctx.on('my-plugin/ready', ({ id }) => console.log(id + ' is ready'))
```

### bail — 短路

```ts
const result = ctx.bail('some-check', input)
ctx.on('some-check', (input) => {
  if (shouldBlock(input)) return 'blocked'   // 非空返回 → 短路并成为结果
  // 返回 null / false / undefined → 继续下一个监听器
})
```

### serial — 顺序执行

```ts
await ctx.serial('setup-phase', context)
```

### waterfall — 流水线（关键规则）

```ts
const output = await ctx.waterfall('my-plugin/transform', input, async () => input)

ctx.on('my-plugin/transform', async (_input, next) => {
  const downstream = await next()   // next() 是强制的
  return downstream.trim()
})
```

**waterfall 监听器必须调用 next()。** 不调用会短路整个流水线——这是故意的设计，用于拦截/网关逻辑；否则必须调用并把结果返回。

## 类型安全的事件

用 TypeScript 声明合并给事件提供类型：

```ts
declare module '@deepseek-ai/cordis' {
  interface Events {
    'my-plugin/ready': (payload: { id: string }) => void
    'my-plugin/check': (input: string) => boolean | undefined
    'my-plugin/transform': (input: string, next: () => Promise<string>) => Promise<string>
  }
}
// 之后 ctx.on / ctx.emit / ctx.bail / ctx.waterfall 都能正确推导
```

## 事件命名

Harness 的 Cordis 事件遵循 namespace/action 命名，例如 agent/step、agent/request、agent/request-error、tools/result、session/event。完整签名与触发模式见子系统页面生成的 cordis-surface 区块（以生成为准）。

## Cordis 事件 vs 持久会话事件

- turn/*、step/*、tool/call、tool/result、compaction/* 是**持久化的会话事件**类型，不是同名 Cordis 事件。
- 需要观察它们时，监听 session/event 并检查 event.type，而不是 ctx.on('tool/result')。

```ts
ctx.on('session/event', (event) => {
  if (event.type === 'tool/result') { /* ... */ }
})
```

## 示例：日志插件

```ts
import type { Context } from '@deepseek-ai/cordis'
import '@deepseek-ai/dsh-tools'

export const name = 'tool-logger'
export function apply(ctx: Context) {
  ctx.on('tools/result', (exec, result) => {
    console.log('[tool] ' + exec.name + ' ' + JSON.stringify(exec.arguments))
    const text = result.content
      .map(block => block.type === 'text' ? block.text : '')
      .join('')
    console.log('[tool result] ' + text.slice(0, 100))
  })
}
```

## 监听器是效果

ctx.on() 注册的监听器随插件卸载自动移除；无需手动 off。除 ctx.on 外，相关 API 还有 ctx.once、ctx.parallel、ctx.waterfall、ctx.bail、ctx.serial、ctx.emit（细节以 Inherited Cordis API / 上下文页的生成区块为准）。
