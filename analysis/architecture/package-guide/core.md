# 核心、LLM、Typert 与诊断

## core

- [`core/agent`](../../../packages/core/agent/README.md)（D）：实时 Agent 接口、Registry、Inbox、initiator AsyncLocalStorage 与 `agent/*` 事件；具体驱动不在此包。入口 `src/index.ts`、`runtime-types.ts`、`inbox.ts`、`dispatch.ts`。
- [`core/agent-default-model`](../../../packages/core/agent-default-model/README.md)（X）：在 Agent request 配置缺省时应用部署默认 provider/model；通过扩展点工作，不修改 Loop。
- [`core/agent-loop`](../../../packages/core/agent-loop/README.md)（P）：默认 `ReactLoopAgent`、Agent factory、turn/step 状态机、LLM stream 和 tool scheduler。入口 `src/agent.ts`、`tool-calls.ts`、`index.ts`。
- [`core/agent-tool-presentation`](../../../packages/core/agent-tool-presentation/README.md)（X）：按 Agent/preset 解析 native/code 工具展示模式，保持 schema 与 executor 的 scope 决策一致。
- [`core/scope`](../../../packages/core/scope/README.md)（S）：Agent scoped contribution 的 target、解析与可见性原语；不同于 Cordis isolate。
- [`core/session`](../../../packages/core/session/README.md)（D）：追加式 Session Event、live store、派生模型消息、fork、repair、JSON 和 runtime invariant。入口 `src/index.ts`、`types.ts`、`surface.ts`。
- [`core/system-prompt`](../../../packages/core/system-prompt/README.md)（D/X）：section/context/variable/tool schema 注册与确定性装配、严格插值和 scoped persona replacement。
- [`core/tools`](../../../packages/core/tools/README.md)（D/X）：工具定义与 scoped registry、schema、审批/guard/执行 waterfall、调度、取消和 presentation。入口 `src/index.ts`、`schema.ts`、`code-mode.ts`。

## llm

- [`llm/llm`](../../../packages/llm/llm/README.md)（D）：消息/内容/chunk vocabulary、Adapter registry、prepared call、统一错误和 `llm/stream` waterfall。
- [`llm/llm-deepseek`](../../../packages/llm/llm-deepseek/README.md)（P）：DeepSeek HTTP Provider、请求/流转换、认证与 provider error 规范化。
- [`llm/llm-pi-ai`](../../../packages/llm/llm-pi-ai/README.md)（P）：基于 pi-ai 的多模型 Provider；库级类型和 stream 映射留在 Provider 内。
- [`llm/llm-retry`](../../../packages/llm/llm-retry/README.md)（X）：在 request-error/stream 扩展点执行 retry policy，不把重试写死到 LLM Runtime。
- [`llm/token-meter`](../../../packages/llm/token-meter/README.md)（X）：从统一 usage 记录 token 消耗，供投影/遥测使用。

## typert

- [`typert/generator`](../../../packages/typert/generator/README.md)（S）：从公开 TypeScript API 生成可传输 type graph。
- [`typert/loader`](../../../packages/typert/loader/README.md)（P）：装载生成 artifact 并接入 runtime。
- [`typert/protocol`](../../../packages/typert/protocol/README.md)（D）：lookup/context 的 host 与 wire 类型协议。
- [`typert/registry`](../../../packages/typert/registry/README.md)（D/P）：运行时类型、lookup 和 context registry；把 branded wire id 解析为 live object。

## runtime diagnostics

- [`runtime-diagnostics/invariants`](../../../packages/runtime-diagnostics/invariants/README.md)（X）：发现并安装各 package 的 `./invariant`，集中报告跨事件/数据关系破坏。
