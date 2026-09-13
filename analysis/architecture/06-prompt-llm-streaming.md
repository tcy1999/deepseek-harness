# Prompt、LLM 与流式输出

## Prompt 由已注册内容按固定顺序生成

[`SystemPrompt`](../../packages/core/system-prompt/src/index.ts)收集 section、动态 context、变量和 tool schema。Section 和 context 先按名称与 scope 解析，再严格插值；未知或 undefined 变量直接失败。工具默认按 code-unit 字典序排列，也可用含 `<unlisted-tools>` 的显式顺序，确保不同机器生成相同 prompt。

Persona 分为 `deployment:persona-prefix` 和 `deployment:persona-suffix` 两个片段，顺序由 `systemPrompt.getSectionOrder()` 的固定位置定义。Agent preset 在自己的 scope 注册同名片段，覆盖该会话的部署人格；其他 Agent 的片段不受影响。实现见 [`persona`](../../packages/preset/persona/src/index.ts)。

动态 runtime context 会先生成完整快照。Agent Loop 通过 `RuntimeContextProjection` 比较当前内容与最后一个仍在 surface 中的快照，只在变化时追加 `user/message`。新消息的固定前缀声明它在语义上取代更早的 runtime-context 快照，但旧消息仍保留在追加式日志和当前模型历史中，直到 compaction 用 surface replacement 移除；恢复时 projection 扫描当前 surface，重新找到最后一个仍保留的快照，而不是改写旧事件。

## LLM Runtime 是路由注册表

[`LlmRuntime`](../../packages/llm/llm/src/index.ts)按 provider route 注册 `LlmAdapter`。Adapter 只必须实现 `stream()`，还可提供 model discovery、exact model metadata、context、adapter defaults 和 retry policy。Route replacement 是同步原子操作：候选全集验证后一次交换，调用不会观察到空窗。

模型目录是 advisory，未列出的 model id 不因此被拒绝；真正调用在 exact provider/model resolve。这个选择允许 provider 新模型无需先升级 catalog，但把拼写错误推迟到 provider 响应。

## Prepared Call

请求前先解析 provider route、model metadata、adapter defaults 和 retry policy，形成一次性的 `PreparedLlmCall`。它捕获同一 adapter registration，防止配置解析后 provider 被热替换导致 metadata 与实际执行来自不同实现。Prepared call 只能使用一次，并校验请求 config 与准备结果一致。

Loop-built request 使用进程内标记并 deep-freeze；`llm/stream` listener 可观察、重放或路由，但不能就地修改消息。因为这些消息必须是 Session Log 的纯函数，修改只能发生在更早的 `agent/pre-step`（通过写入可记录输入）或 `agent/request` 配置阶段。

```text
assemble → agent/pre-step → step/start → agent/request → prepareCall
  → 根据实际路由协调 system/message → 提交 user/message 与请求元数据
  → deriveMessages + freeze → llm/stream → 绑定的 adapter
```

系统提示词由 `system/message` 派生到历史中，不能通过请求上的独立 `system` 参数绕过日志。动态 context 则通过接纳后的用户消息进入历史，两者采用不同事件和更新规则。

## 不同 Provider 使用同一种流式输出格式

Adapter 输出统一 `StreamChunk`，`BlockAssembler` 合并 reasoning、text、tool calls、usage、finish 和 replay state。Loop 将 chunk 放入进程内的紧凑流记录，并同步发送实时帧；尝试结算时将完整 stream 写入 `assistant/message` 或 `assistant/attempt`。错误尝试可经 `agent/request-error` 决定重试，取消仍服从当前 turn 的信号；失败尝试不会成为成功模型历史。

Provider 特有的 HTTP、SDK 和错误解析留在 `llm-deepseek`、`llm-pi-ai` 等包中。`LlmError` 只保留所有 Provider 都能表达的 code、status、retry-after 和 request id；API key 诊断不会回显 secret。重试由 `llm-retry` 插件实现，因此部署可以替换重试规则，无需修改 LLM Runtime。

## Token 与缓存影响

Prompt section、tool schema 和 runtime context 都进入每次请求；稳定顺序有助于 KV cache。Session history 由日志派生，compaction 或 tool-result pruning 必须通过新事件改变派生视图，不能在发送前临时删数组。`token-meter` 读取统一 usage，避免每个 adapter 重复统计。
