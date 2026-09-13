# Prompt、LLM 与流式输出

## Prompt 由已注册内容按固定顺序生成

[`SystemPrompt`](../../packages/core/system-prompt/src/index.ts)收集 section、动态 context、变量和 tool schema。Section 和 context 先按名称与 scope 解析，再严格插值；未知或 undefined 变量直接失败。工具默认按 code-unit 字典序排列，也可用含 `<unlisted-tools>` 的显式顺序，确保不同机器生成相同 prompt。

Persona 使用固定名称 `deployment:persona` 和 order 0。Agent preset 可以在自身 scope 注册同名 section 覆盖部署 persona，而非在全局 prompt 后追加第二人格。这说明 scoped registry 同时承担“局部替换”语义。

动态 runtime context 会先生成完整快照。Agent Loop 通过 `RuntimeContextProjection` 比较当前内容与最后一个仍在 surface 中的快照，只在变化时追加 `user/message`。新消息的固定前缀声明它在语义上取代更早的 runtime-context 快照，但旧消息仍保留在追加式日志和当前模型历史中，直到 compaction 用 surface replacement 移除；恢复时 projection 扫描当前 surface，重新找到最后一个仍保留的快照，而不是改写旧事件。

## LLM Runtime 是路由注册表

[`LlmRuntime`](../../packages/llm/llm/src/index.ts)按 provider route 注册 `LlmAdapter`。Adapter 只必须实现 `stream()`，还可提供 model discovery、exact model metadata、context、adapter defaults 和 retry policy。Route replacement 是同步原子操作：候选全集验证后一次交换，调用不会观察到空窗。

模型目录是 advisory，未列出的 model id 不因此被拒绝；真正调用在 exact provider/model resolve。这个选择允许 provider 新模型无需先升级 catalog，但把拼写错误推迟到 provider 响应。

## Prepared Call

请求前先解析 provider route、model metadata、adapter defaults 和 retry policy，形成一次性的 `PreparedLlmCall`。它捕获同一 adapter registration，防止配置解析后 provider 被热替换导致 metadata 与实际执行来自不同实现。Prepared call 只能使用一次，并校验请求 config 与准备结果一致。

Loop-built request 使用进程内标记并 deep-freeze；`llm/stream` listener 可观察、重放或路由，但不能就地修改消息。因为这些消息必须是 Session Log 的纯函数，修改只能发生在更早的 `agent/pre-step`（通过写入可记录输入）或 `agent/request` 配置阶段。

## 不同 Provider 使用同一种流式输出格式

Adapter 输出统一 `StreamChunk`，`BlockAssembler` 合并 reasoning、text、tool calls、usage、finish 和 replay state。Loop 逐 chunk 写日志，stream 完成后才写 assistant message；finish error/aborted 先进入 `agent/request-error` waterfall，retry 不会把失败尝试伪装成成功 message。

Provider 特有的 HTTP、SDK 和错误解析留在 `llm-deepseek`、`llm-pi-ai` 等包中。`LlmError` 只保留所有 Provider 都能表达的 code、status、retry-after 和 request id；API key 诊断不会回显 secret。重试由 `llm-retry` 插件实现，因此部署可以替换重试规则，无需修改 LLM Runtime。

## Token 与缓存影响

Prompt section、tool schema 和 runtime context 都进入每次请求；稳定顺序有助于 KV cache。Session history 由日志派生，compaction 或 tool-result pruning 必须通过新事件改变派生视图，不能在发送前临时删数组。`token-meter` 读取统一 usage，避免每个 adapter 重复统计。
