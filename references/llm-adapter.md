# LLM 适配器

为 Harness 接入新的模型提供方。LLM 适配器是继承 LlmAdapter 并实现 stream() 的类：把 Harness 的提供方无关请求转换为具体提供方的 API 调用，再把响应转换回 Harness 分片。

## 最小实现

```ts
import type { Context } from '@deepseek-ai/cordis'
import Schema from '@deepseek-ai/schemastery'
import { LlmAdapter, type GenerateOptions, type StreamChunk } from '@deepseek-ai/dsh-llm'

class MyAdapter extends LlmAdapter {
  private apiKey: string
  constructor(apiKey: string) { super(); this.apiKey = apiKey }

  async *stream(options: GenerateOptions): AsyncIterable<StreamChunk> {
    // 1. 把 options.messages 转成提供方格式
    // 2. 调用流式 API
    // 3. 把响应转换为 StreamChunk 序列
  }
}

export interface Config { apiKey: string; providers: string[] }
export const Config: Schema<Config> = Schema.object({
  apiKey: Schema.string().required(),
  providers: Schema.array(Schema.string()).required(),
})

export const name = 'my-llm-adapter'
export const inject = ['llm']
export function apply(ctx: Context, config: Config) {
  ctx.llm.registerAdapter(config.providers, new MyAdapter(config.apiKey))
}
```

## StreamChunk 协议

stream() 必须按以下协议生成分片：

```ts
import { CallId, type StreamChunk } from '@deepseek-ai/dsh-llm'

async function* exampleChunks(): AsyncIterable<StreamChunk> {
  // 1. 每个内容块以 block-start 开始
  yield { type: 'block-start', index: 0, blockType: 'text' }
  // 2. 文本经 text-delta 流式输出
  yield { type: 'text-delta', index: 0, text: 'Hello' }
  yield { type: 'text-delta', index: 0, text: ' world' }
  // 3. 每个内容块以 block-end 结束，携带完整块
  yield { type: 'block-end', index: 0, block: { type: 'text', text: 'Hello world' } }
  // 4. 工具调用块
  yield { type: 'block-start', index: 1, blockType: 'tool-call' }
  yield { type: 'tool-call-delta', index: 1, id: CallId('call-123'), name: 'bash', argumentsDelta: '{"command":"ls"}' }
  yield { type: 'block-end', index: 1, block: { type: 'tool-call', id: CallId('call-123'), name: 'bash', arguments: '{"command":"ls"}' } }
  // 5. token 用量（必须在 finish 之前）
  yield { type: 'usage', usage: { inputTokens: 100, outputTokens: 50 } }
  // 6. 结束原因（必须是最后一个分片）
  yield { type: 'finish', reason: { kind: 'stop' } }
  // 或 { kind: 'tool-calls' } 请求执行工具
}
```

关键规则：

- 每个 block-start 必须有对应的 block-end。
- index 从 0 递增，标识内容块顺序。
- tool-call-delta 的 argumentsDelta 是原始 JSON 文本增量，可一次或分多次生成。
- finish 必须是最后一个分片；usage 必须在 finish 之前。

## GenerateOptions

stream() 接收 GenerateOptions：模型、适配器拥有的推理强度 ID、对话历史、系统提示词、工具 schema、生成参数、停止序列与中止 signal。完整字段以 @deepseek-ai/dsh-llm 导出的 TypeScript 类型为准。适配器必须把支持的字段映射到具体 API；**无法支持的字段应抛出带稳定 code 的 LlmError，不得静默丢弃。**

## resolveModel 与 listModels

- 覆写 resolveModel(provider, model, signal?)，在一次查询中返回确切的提供方/模型身份以及可选的 context 与 reasoning 元数据。异步查询必须响应该 signal，使取消与资源释放完全停稳。服务会校验聚合结果，并在调用 stream() 前拒绝显式指定但不受支持的推理强度；省略 reasoning 表示该模型没有可选的推理强度能力。
- 适配器能向选择器公布模型选项时，覆写 listModels()。

## 注册适配器

```ts
ctx.llm.registerAdapter(['my-provider'], adapter)
```

第一个参数是该适配器处理的提供方路由列表。GenerateOptions.provider 选择已注册的适配器；GenerateOptions.model 传入由适配器拥有、无需在生命周期启动时注册的模型 id。

## 在 cordis.yml 中使用

```yaml
- id: my-llm
  name: './src/my-llm-adapter.ts'
  config:
    apiKey: !!js process.env.MY_API_KEY
    providers:
      - my-provider
- id: agent-loop
  name: '@deepseek-ai/dsh-agent-loop'
  config:
    agents:
      - id: main
        provider: my-provider
        model: my-model-v1
```

## 错误处理

- 适配器通过带稳定 code 的 LlmError 抛出传输与协议故障；agent loop 保留错误及其 code 用于诊断与策略处理。**不要依赖普通 Error 被自动转换。**
- 每个提供方 HTTP 请求必须合并 attributionHeaders()，并传递 options.signal。

```ts
import { attributionHeaders, LlmAdapter, LlmError, type GenerateOptions, type StreamChunk } from '@deepseek-ai/dsh-llm'

class HttpAdapter extends LlmAdapter {
  constructor(private readonly endpoint: string) { super() }
  async *stream(options: GenerateOptions): AsyncIterable<StreamChunk> {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: { 'content-type': 'application/json', ...attributionHeaders() },
      body: JSON.stringify({ model: options.model, messages: options.messages }),
      ...options.signal ? { signal: options.signal } : {},
    })
    if (!response.ok) {
      throw new LlmError('Provider API error: ' + response.status, 'PROVIDER_HTTP_ERROR')
    }
    yield { type: 'finish', reason: { kind: 'stop' } }
  }
}
```

## 实战参考

仓库中两个完整实现：

- packages/llm/llm-deepseek/ —— DeepSeek API 适配器（OpenAI 兼容格式）
- packages/llm/llm-pi-ai/ —— Pi AI 适配器（不同的 API 格式）

对比这两个已交付适配器，可以看到同一套 harness 契约如何在不同提供方 SDK 之上实现。
