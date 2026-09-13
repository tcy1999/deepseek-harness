# Cordis 插件模型

## Context 决定插件能访问哪些服务和注册项

Cordis `Context` 提供 Service 访问、事件分发，以及插件安装和卸载。子 Context 默认能访问父 Context 的服务，也可以通过 isolate realm 或 Harness 的 agent scope 得到另一组可见内容。`Agent.ctx` 因此决定一个 Agent 能使用哪些工具、prompt section、policy listener 和 preset 插件。Harness 入口见 [`packages/core/agent/src/index.ts`](../../packages/core/agent/src/index.ts)，作用域实现见 [`packages/core/scope/src/index.ts`](../../packages/core/scope/src/index.ts)。

Service Definition 通常是 `Service` 子类并占用一个 `ctx` key，例如 `ctx.llm`、`ctx.tools`、`ctx.sessions`。可选依赖使用 `ctx.get(name)`；声明注入的依赖才用属性代理，因为属性可见性受当前拓扑影响。

## 插件卸载时必须删除自己的注册项

`register()` 必须返回删除函数，事件通过 `ctx.on()` 注册，其他资源通过 `ctx.effect()` 绑定。插件卸载时，Cordis 调用这些清理函数。HMR、preset 卸载、Agent 销毁和测试结束都使用这套规则，不需要各自实现一套 cleanup。

常见错误是把对象加入全局 `Map` 却不返回删除函数，或启动异步任务后只发送取消信号、不等待任务停止。前者会让热重载累积重复注册；后者会让旧插件在新配置生效后继续发布状态。仓库用注册项卸载测试和异步资源释放规则检查这两类问题。

## 事件模式

| 模式 | 语义 | 例子 | 扩展责任 |
|---|---|---|---|
| emit | 已发生事实的同步通知 | `agent/status`、`session/event` | 不改变操作结果 |
| serial | 按序运行所有 listener | `agent/turn-stopping` | 不存在 `next()`，适合停机前补充工作 |
| waterfall | listener 包裹下游调用 | `agent/pre-step`、`agent/request`、`llm/stream`、`tools/execute` | 必须显式 `next()` 才委托 |

Waterfall listener 可以决定何时调用后续操作：retry 插件可以再次调用，replay 插件可以直接提供 chunk，审批插件可以在工具 body 前暂停。Listener 的安装顺序会影响结果，因此 bundle 顺序和事件携带的 Agent scope 必须稳定。事件生产者和消费者可查[生成索引](../../docs/event-producer-consumer.md)。

## Agent scope 与 isolate realm

Cordis isolate 解决“同一 Service key 需要多个独立实例”的组合问题；Harness scope 解决“一个 registry 内的贡献只对某个 Agent 可见”的问题。两者不可互换。Preset 中一个 service row 往往需要 isolate，否则多个 Agent 会共享 provider 实例；而工具和 prompt registry 使用 `scopeTarget`/`scopeOf` 在同一服务中筛选 scoped contribution。

`Agent.ctx` 还保留 Agent 关联，供事件 carrier 和开发体验使用；核心层选择注册可见性时依赖明确的 scope resolver，而不是只读 `ctx.agent`，避免嵌套 scope 误判。

## Agent 配置完成后才对外可见

Factory 先创建外部不可见的 scope，执行并等待 `setup`，可选执行同步 `commit()`，然后才把 Session 和 Agent 加入 registry、发送 created/session-start 通知并启动 driver。setup、commit 或创建者提前释放任一步失败，都会删除这个 scope，外部不会看到只完成一半配置的 Agent。接口约定见 [`CreateAgentOptions`](../../packages/core/agent/src/index.ts)，实现见 [`agent-loop/src/index.ts`](../../packages/core/agent-loop/src/index.ts)。

这不是数据库事务：已经发给 listener 的通知不能收回。如果发布过程中失败，实现会删除 registry 中的对象，并再发送配对的 `disposed` 通知。Registry 不会留下半配置对象，但 listener 必须根据 `disposed` 撤销自己对早期通知的处理。
