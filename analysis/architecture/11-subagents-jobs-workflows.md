# 子 Agent、后台任务与工作流

## 三类后台工作

Subagent 表示把自然语言任务交给另一个 Agent runtime；Job 表示当前 runtime 管理的后台生命周期；Workflow 表示按已知执行协议运行的任务引擎。三者都异步，但不应合并：subagent 有会话和消息语义，job 只承诺状态/结果/停止，workflow 有 worker 与步骤协议。

## Subagent 的共同接口和不同实现

[`subagent`](../../packages/subagent/subagent/src/index.ts)定义 provider registry、任务、控制和报告事件。Provider 包覆盖多种隔离级别：

- `subagent-fork-in-process` 从当前 Session 的 closed-turn 前缀 fork；
- `subagent-spawn-in-process` 创建新 Agent；
- `subagent-in-process-driver` 复用当前运行时能力驱动；
- `subagent-dsh-sdk`、`subagent-acp` 连接另一个 Harness/ACP；
- `subagent-claude-code`、`subagent-codex` 对接外部产品。

`tool-subagent` 发起任务，control/report tools 管理已有任务。Consumer 不暴露 provider 进程细节，统一使用 opaque branded id 和结构化状态。Delegation depth 写入 Session metadata，防止无界递归不是只靠 prompt 提醒。

## Fork 会继承历史，Spawn 不会

Fork 继承父 Session 的已完成历史，因此 child 能理解上下文，但复制更多 token 并建立 lineage；spawn 只接收明确任务与 preset，隔离更强。二者都必须创建完整 Agent scope，setup 成功后才发布。In-process 不等于共享全部注册：preset 和 scope 可以给 child 不同工具与 persona。

外部 Provider 不能提供与 in-process 完全相同的实时事件，因此公共接口只保留所有 Provider 可兑现的控制和报告。将某一 Provider 的 streaming detail 加入 Definition 会迫使其他 Provider 模拟不存在的语义。

## Jobs

`jobs` 定义 job registry，`jobs-local` 管理 Promise/AbortController、状态和结果，`tool-jobs` 提供 `job_output`、`job_list` 和 `job_kill`。Job owner 负责在插件或 Agent 卸载时取消并等待；`job_output` 是读取已发布结果，不成为结果事实的第二 owner。

Job 适合让长工具执行脱离当前 turn，但 `jobs-local` 的 registry、状态、输出 cursor 和 Promise 都只存在于当前进程，不写专用 Session Event，重启或 resume 不会恢复任务。Durable history 只记录几个模型可见接点：启动后台工作的原始工具结果包含 job id；未被读取的完成通知通过 `inject` 或 `followup` 排入 inbox，并在被 Agent claim 后成为普通 `user/message`；`job_output`、`job_list`、`job_kill` 的响应作为普通 tool result 保存。完整后台输出只有在模型调用 `job_output` 后才进入历史。

后台任务不能继续使用已经卸载的 Agent 专属能力；它要么使用不依赖 Agent 的独立服务，要么必须在 Agent 销毁前停止。Owner disposal 会取消并等待仍在运行的任务，然后移除它们的进程内快照。

## Schedule

[`schedule`](../../packages/schedule/schedule/src/index.ts)与 Job 的持久性相反：提醒状态由 Session-local `schedule/change` 的 create/delete/dispatch 事件折叠得到，timer 只是可丢弃的 live projection。`after` 和 `at` 是一次性提醒；`every` 是至少五分钟的 creation-anchor-aligned 固定间隔，错过多个周期时只选择最新一期，不枚举积压。Fork 只折叠 seed 后的事件，因此不继承父 Session 的提醒。

每次读取或决定提醒前都要通过 Session persistence flush；create/delete 成功后也等待 post-append barrier。提醒只在原 Session live 时运行，冷 Session 的到期项会在 resume 后成为 overdue。Runtime 在 Agent idle maintenance 阶段先同步排入 followup，再追加 dispatch，释放阶段后 followup 才开始普通 turn；这提供可重放状态，但仍存在“已排入 followup、dispatch 尚未 checkpoint 就崩溃”导致重复提醒的窄窗口，不承诺 exactly-once 或用户已读。

## Workflow

`workflow` 定义运行协议，`workflow-worker-thread` 用 worker thread 隔离引擎，`tool-workflow` 和 `tool-ralph` 是两种模型入口。主线程必须验证 worker 消息；取消时双方要确认；释放时要等待线程退出。Worker 不是 Cordis child context，不能直接访问主线程服务，必须通过明确的消息接口请求。

## 谁负责取消和等待后台工作

一次异步操作只由一个 controller 负责启动完成、取消、释放和最终结果。只有某个 reservation 或 callback 确实有独立完成时刻时，才由另一个对象负责。用多个 boolean、Promise 和 sentinel 分散表示同一操作的状态，容易造成“已经取消却仍发布结果”或释放一直无法完成。

## 风险

困难不在于启动并发，而在于把权限、workspace、credential 和 Session 父子关系正确传给任务，并在取消或卸载时等任务停止。In-process Provider 隔离较弱但能提供完整事件；外部 Provider 隔离较强但只能提供协议规定的事件和控制。公共接口只包含两类 Provider 都能做到的操作，因此可能无法表达更复杂的实时协作。
