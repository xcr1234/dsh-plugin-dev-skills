# 核心 seam 与架构映射

DSH 的每一部分都是插件：模型适配器、工具注册表、会话日志、agent loop 本身，都可以从配置替换。一个 **seam** 是一项可替换能力，包含三种角色：声明接口的 Service Definition、实现它的 Service Provider、使用它的 Consumer（通常是面向模型的工具）。seam 正是「替换一个提供方就能改变整个产品」的原因。

## 主干核心包

| 包 | 职责 | ctx 键 |
| --- | --- | --- |
| core/session | 仅追加的 SessionEvent 日志与内存存储（唯一真源） | ctx.sessions |
| core/system-prompt | 提示词片段与工具 schema 的组装 | ctx.systemPrompt |
| core/tools | 作用域化的工具注册表与带把关的执行流水线 | ctx.tools |
| core/agent | Agent 接口、活跃注册表、agent/* 事件词汇 | ctx.agents |
| core/agent-loop | 实现 Agent 接口的默认驱动器 | ctx.agentLoop |
| core/scope | 按 agent 划分作用域的注册原语 | 库，无 ctx 键 |
| llm/llm | 消息与流式词汇表 + 适配器 seam | ctx.llm |

扩展插件依赖 agent（包括需要发起 Agent 时），而绝不直接依赖 agent-loop——循环保持可替换。

## 能力 seam 与核心服务（节选，完整表格以「能力 Seams 与核心服务」页面为准）

| ctx 键 | 角色 | 所属包 | 已知实现 |
| --- | --- | --- | --- |
| ctx.llm | seam | llm | llm-deepseek, llm-pi-ai, llm-replay |
| ctx.attachments | seam | attachment | attachment-local |
| ctx.tokenMeter | core | token-meter | - |
| ctx.toolResultPruner | core | compaction-tool-result-pruner | - |
| ctx.sessions | core | session | - |
| ctx.invariants | core | invariants | - |
| ctx.typert | core | typert-registry | - |
| ctx.typertGateway | core | api-gateway | - |
| ctx.sessionPersistence | seam | session-persistence | session-persistence-jsonl, session-persistence-sqlite |
| ctx.sessionQuery | seam | session-query | session-query-sqlite |
| ctx.sessionTitle | core | session-title | session-title-first-prompt-llm 等 |
| ctx.sessionProjections | core | session-projection | session-projection-cache |
| ctx.sessionTelemetry | seam | session-telemetry | session-telemetry-otel |
| ctx.compaction | seam | compaction | compaction-basic |
| ctx.skills | seam | skill | skill-filesystem |
| ctx.settings | seam | settings | settings-file |
| ctx.credentials | seam | credentials | credentials-local |
| ctx.shell | seam | shell | bash-local, bash-sandbox |
| ctx.subprocess | seam | subprocess | subprocess-e2b |
| ctx.terminals | seam | terminal | terminal-bash |
| ctx.jobs | seam | jobs | jobs-local |
| ctx.fs | seam | fs | fs-local, fs-sandbox |
| ctx.sandbox | seam | sandbox | sandbox-local |
| ctx.lsp | seam | lsp | lsp-stdio |
| ctx.codeRuntime | seam | code-runtime | code-runtime-worker-thread |
| ctx.web | seam | web | web-fetch-http, web-search-deepseek/exa/perplexity |
| ctx.workflowEngine | seam | workflow | workflow-worker-thread |
| ctx.subagents | seam | subagent | spawn/fork in-process, acp, claude-code, codex |
| ctx.storage | seam | storage | storage-json, storage-sqlite |
| ctx.goals | core | goal | - |
| ctx.approval | core | user-approval | - |
| ctx.permissionPresets | core | permission-presets | - |
| ctx.commands | core | command | - |
| ctx.planMode | core | plan-mode | - |
| ctx.workspaceRegistry | core | workspace | - |
| ctx.userQuestions | core | user-interaction | - |
| ctx.spillStore | seam | spill | spill-local |

抽象 seam（abstract seam）的含义：接口在 harness 定义，实现由组合时选择的提供方插件提供。完整清单、消费方与配套插件见「能力 Seams 与核心服务」页面的生成表格，全部可用插件见「插件配置目录」。

## 新行为的归属位置

| 目标 | 机制 |
| --- | --- |
| 添加模型提供方 | 在 ctx.llm 注册其适配器 |
| 添加面向模型的能力 | 在 ctx.tools 注册；其 schema 加入提示词组装 |
| 让某个会话拥有不同的能力集合 | 组装 agent preset；其中的服务行需要 isolate realm |
| 添加 shell 执行 | 注册 ctx.shell 后端；本地后端经 ctx.subprocess spawn 进程 |
| 添加持久化终端执行 | 注册 ctx.terminals 后端和 dsh-tool-terminal |
| 添加用户命令 | 在 ctx.commands 注册；无需模型轮次即可分派 |
| 添加后台工作 | 在 ctx.jobs 注册；job_* 工具负责收集或停止 |
| 添加文件系统访问或策略 | 注册 ctx.fs 提供方，或监听 fs/* 事件 |
| 限制所启动的进程 | 使用 ctx.sandbox 后端；消费方在启动进程前包装 argv |
| 拦截请求、工具或轮次 | 使用相应的 agent/* 或 tools/* 事件；agent/turn-stopping 会停止轮次 |
| 添加模型可见上下文 | 调用 agent.inject()；它落到下一次获准的请求中 |
| 添加 UI 或编辑器集成 | 驱动 ctx.agents 并从 session/event 渲染 |

## 轮次流程

一个**步骤** = 一次模型请求 + 它调用的工具；一个**轮次** = 零到多个步骤（领取首条输入之前打开，不再欠任何工作时关闭）。

```
turn/start
  → claim next-step input + 一条排队消息
  → 组装 prompt 段与工具 schema → agent/pre-step
  → reject | enter(messages)（首次被拒绝或改写为空 → 关闭无步骤的轮次）
  → step/start → 追加 user/message → 从日志派生模型历史
  → agent/request → llm/stream → assistant/chunk* → assistant/message
  → tool/call* → tools/pre-execute → tools/execute → tools/post-execute → tool/result*
  → step/end
  → 工具欠另一个请求或新输入到达 → 下一步；否则
  → agent/turn-stopping
turn/end
```

- turn/*、step/*、user/message、assistant/*、tool/* 是**持久会话事件**；其余分属三个事件域（会话、agent、能力）的实时扩展点。
- agent/pre-step、agent/request、llm/stream 与三个 tools/* 事件是 waterfall（监听器必须调用 next()）；agent/turn-stopping 是 serial（无 next()）。
- **模型可见即已记录**：抵达模型请求的一切必须能从日志重建，由运行时不变式断言。新增模型可见输入 = 新增会话事件（扩展 SessionEventMap 并从日志渲染）。

## 会话事件（12 种变体）

turn/start、turn/end、step/start、step/end、user/message、assistant/chunk、assistant/message、tool/call、tool/result、steering/message、todo/write、request/header。每个条目携带单调 seq、time 与按 type 判别的 data payload；surface 变体还可引用较早事件（sourceEventSeqs）并携带 surfaceOp。LLM 消息历史由日志派生（deriveMessages()），不单独存储。

## 全仓通用类型模式（…Map → derived-union）

几乎所有可扩展和类型都用同一模式：以判别标签为键的接口（…Map），联合类型由 keyof 派生；插件通过声明合并添加变体，无需修改拥有该类型的包。

```ts
interface ThingMap { 'a': { kind: 'a' }; 'b': { kind: 'b' } }
type Thing = ThingMap[keyof ThingMap]   // 判别联合

declare module '@deepseek-ai/dsh-llm' {
  interface ThingMap { 'c': { kind: 'c' } }   // 插件扩展
}
```

规范 map 包括 ContentBlockMap、MessageSourceMap、FinishReasonMap（dsh-llm）、TurnTriggerMap（dsh-session）等；工具 schema 与事件签名以生成的目录为准。
