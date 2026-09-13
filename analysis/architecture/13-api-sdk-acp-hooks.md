# API Gateway、Typert、SDK、ACP 与 Hooks

## 多入口共享同一 runtime

Web、JSON-RPC、ACP 和 hook bridge 不各自实现 Agent。它们解析外部输入，查找或创建 `ctx.agents` 中的对象，调用公共接口，再根据 Session/Agent 事件生成各自的输出。共用同一套 Agent 实现可以避免不同入口表现不一致；每种协议仍要自己处理身份、取消、序列化和连接关闭。

## Typert

`typert/generator` 从公开类型生成 type graph，`loader` 装载 artifact，`protocol` 定义 lookup/context wire 类型，`registry` 在 runtime 发布。Service 在注册 Typert lookup 时声明 host object 如何由 wire id 解析，例如 SessionId 到 live Session。这样 API 方法可保留强类型对象语义，wire 只传 branded id。

生成 artifact 是 source-derived，不应手改。Typert 把“类型可传输”与“对象可查找”分开：一个类型存在不代表任意客户端可构造 live object。业务 package 注册 lookup/context 的稳定声明与默认 resolver，Host composition 可用 `configure()`/`configureHost()` 替换具体解析和访问政策；Gateway 只执行当前 resolver，并保留 `TypertLookupFailure` 表达的政策拒绝。Type graph 本身不提供授权。

## API Gateway 与 Remotes

`api/gateway` 组装 BFF 和 RPC dispatch，`api/remotes` 保存生成或手写的远程接口。Gateway 只负责传输，不保存业务状态；它通过 Typert registry 查找服务和对象，并把内部事件转换为客户端订阅。Web host 的 `apiproxy` 再把浏览器连接接入 gateway。

## SDK

`sdk/protocol` 定义换行分隔 JSON-RPC 消息与固定 wire 类型，`sdk/server` 把 Agent runtime 发布为 stdio server，`sdk/client` 提供 TypeScript client。Server 只分派 `initialize`、`session/prompt` 和 `shutdown`，按调用方提供的 `sessionId` 获取或创建一个 owned Agent，并向连接无筛选发送当前 Context 的全部 `session.event`、`session.status` 与本地 subagent 通知；它不是 Typert object lookup 或授权层。

`session/prompt` 只在 inbox 接纳消息后返回 `{ messageId }`，不等待 assistant message 或 turn 结束，也没有 prompt-level result。Wire 没有 per-session close 或 prompt-cancel；`shutdown` 才释放全部 server-owned Agent。Client 负责 request correlation、通知过滤和断线收口；request timeout 只放弃本地 pending entry，server-side 工作会继续到 runtime 关闭。高层 `DeepSeekHarness.run()` 自己以 inbox receipt 到下一次 whole-agent idle 定义收集区间，不能把结果解释成该 prompt 的因果响应。

## ACP

`acp/acp` 是 automation-only Agent Client Protocol server。它只实现 fresh `session/new`：服务端生成 UUID，通过 `ctx.agents.create` 得到 owned handle；list/load/resume/delete/fork 都不支持。每个 Session 同时只允许一个 `session/prompt`，输入限 baseline text/resource link 和一个 workspace；连接结束时只释放本连接创建的 Agent。ACP 不依赖 `dsh-agent-loop`，因此可以搭配满足公共 Agent 接口的其他 factory。

ACP 只把已提交 `assistant/message` 的非空文本块转换为 `session/update`；调用是 fire-and-forget，发送失败只记日志，prompt response 不等待通知传输完成。正常 prompt 在 whole-agent `whenIdle()` 后返回 `end_turn`，不会把某个 `turn/end` 或 token-limit 原因归因给该 prompt；相关 model error 可以提前拒绝，显式 cancel、disposal 或未进入 turn 的 admission 返回 `cancelled`。断线 teardown 先停止接纳并结算 pending prompt，再 drain 本连接 owned roots 下仍可继续的 descendants，最后释放 handles。快照测试通过真实 Loader 组合验证协议消息，而不是只 mock Agent。

## Hooks

`hook-protocol` 定义共享消息格式，`hooks-claude-code` 和 `hooks-codex` 把外部产品 hook 转换为 Harness 事件或操作。Hook 内容来自外部进程，必须先验证。外部产品的事件顺序不一定对应 Session 的 turn/step 顺序，因此 bridge 只能记录输入中能够确认的事实。

## 设计取舍

所有入口共用同一 runtime，因此得到相同的审批、日志和工具。相应地，连接关闭时必须区分对象由谁创建：一个连接不能销毁另一个连接或插件创建的 Agent。`AgentHandle`、initiator scope、lookup registry 和连接清理函数共同记录这种创建和释放关系。
