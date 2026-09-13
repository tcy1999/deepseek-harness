# Package README 补充说明

本页不提出 README 修改，只汇总“仅看单包 README 容易遗漏、但当前源码或组合能确认”的跨包事实。许多 README 已详细记录本包契约，缺口主要来自事实归属于另一包或组合层。

## 核心

| Package | 容易遗漏的事实 | 依据 |
|---|---|---|
| `core/agent` | `AgentHandle.dispose()` 是创建者能力；factory provider 同时是结构 owner，Registry 查询到的裸 Agent 不能销毁自身 | [`AgentHandle`](../../../packages/core/agent/src/index.ts) |
| `core/agent-loop` | waking input 在已 abort activity 后会改投 next-turn；`whenIdle()` 防止完成回调立即启动下一 activity 时提前返回 | [`agent.ts`](../../../packages/core/agent-loop/src/agent.ts) |
| `core/session` | Store 有意不实现 persistence；Agent 需要按顺序创建和释放 Session 时使用 prepare/enter/announce，而不是让普通 fiber 直接创建 | [`SessionStore`](../../../packages/core/session/src/index.ts) |
| `core/system-prompt` | 同名 scoped `deployment:persona-prefix` / `deployment:persona-suffix` 是替换槽位；tool order 的 `<unlisted-tools>` 同时是完整性与稳定排序机制 | [`system-prompt`](../../../packages/core/system-prompt/src/index.ts) |
| `core/tools` | code-mode collapse 在 executor 再次强制，且在审批前拒绝；已启动 body 取消后仍 drain | [`ToolRuntime.execute`](../../../packages/core/tools/src/index.ts) |
| `llm/llm` | Prepared call 捕获 adapter registration 并单次使用，避免热替换造成 metadata/执行错配 | [`PreparedLlmCall`](../../../packages/llm/llm/src/index.ts) |

## 组合与执行环境

| Package | 容易遗漏的事实 | 依据 |
|---|---|---|
| `boot/app-boot` | Bundle 解析采用安装位置优先、profile 目录回退的双锚点；profile `baseUrl` 同时支持 out-of-tree plugin | [`profile.ts`](../../../packages/boot/app-boot/src/profile.ts) |
| `bundle/base` | 包的主要实现不是 `src/index.ts`，而是 patch 中完整的 Service/Provider/Consumer 闭合组合 | [`cordis.patch.yml`](../../../packages/bundle/base/cordis.patch.yml) |
| `fs/*`、`subprocess/*` | 两个 Provider 共同定义执行世界；只替换其中一个可能使 shell/LSP 观察不同 workspace | [执行环境分析](../09-execution-environment.md) |
| `sandbox/*` | Sandbox 只覆盖经过 seam 的 spawn，不能限制同进程第三方插件或直接 Node spawn | [`sandbox`](../../../packages/sandbox/sandbox/src/index.ts) |
| `terminal/*` | PTY 是实时 owner-scoped 资源，Session resume 不会复活它；只有工具观察进入日志 | [`terminal`](../../../packages/terminal/terminal/src/index.ts) |
| `spill-policy` | 限制在最终 materialized result 处决定，才能包含 wrapper/metadata 并处理多字节 | [`spill-policy`](../../../packages/spill/spill-policy/src/index.ts) |

## Session 与异步能力

| Package | 容易遗漏的事实 | 依据 |
|---|---|---|
| `session-persistence` | write-behind 允许内存领先磁盘，durability 由 checkpoint 边界而非每次 append 定义 | [`storage.ts`](../../../packages/session/session-persistence-jsonl/src/storage.ts) |
| `session-checkpoint-policy` | 覆盖模型请求、pre-step 与顶层 tool；nested dispatch 复用外层 checkpoint | [`tests`](../../../packages/session/session-checkpoint-policy/tests/session-checkpoint-policy.spec.ts) |
| `session-title` | 标题以 last-wins Session Event 表达，显式刷新、fallback 和 service disposal 竞争由 coordinator 处理 | [`service-contracts.spec.ts`](../../../packages/session/session-title/tests/service-contracts.spec.ts) |
| `subagent/*` | In-process Provider 仍通过 Agent scope/preset 隔离；delegation depth 是 durable metadata，不只是 prompt 提醒 | [`subagent`](../../../packages/subagent/subagent/src/index.ts) |
| `jobs-local` | Stop 必须等任务真正结束，不能发送取消信号后立即发布 stopped；registry 和输出不持久，resume 不会复活任务 | [`jobs-local`](../../../packages/jobs/jobs-local/src/index.ts) |
| `schedule` | `schedule/change` 是 durable state，timer/followup 是 live projection；冷 Session 只在 resume 后处理 overdue 项 | [`schedule`](../../../packages/schedule/schedule/src/runtime.ts) |
| `attachment/*` | 图片对象在消息事件之前单独持久化；Session 只记录 content-addressed ref，Provider request 时再读回并校验 | [`attachment`](../../../packages/attachment/attachment/src/index.ts) |
| `workflow-worker-thread` | Worker thread 是 wire boundary，无法直接继承 Cordis services，访问能力需显式消息 bridge | [`workflow-worker-thread`](../../../packages/workflow/workflow-worker-thread/src/index.ts) |

## Web 与协议

| Package | 容易遗漏的事实 | 依据 |
|---|---|---|
| `typert/*` | 类型可传输与 live object 可 lookup 是两项独立能力；wire id 不自动授予对象访问 | [`protocol`](../../../packages/typert/protocol/src/index.ts) |
| `acp/acp` | 连接只销毁自己持有 handle 的 Agent，不能把 lookup 到的共享 Agent 当作 owned resource | [`acp`](../../../packages/acp/acp/src/index.ts) |
| `client/ui-*` | 大多数包是 slot/renderer contribution，不拥有服务端状态；reload 后权威状态来自 durable event/projection | [Client 架构](../14-web-client-architecture.md) |
| `ui-permission-presets` | UI 只配置政策，真实拒绝必须发生在 host Tool Runtime/Provider | [安全分析](../15-safety-and-correctness.md) |
| `plugin-inventory` | Inventory 是 Loader 树的只读投影，不能作为第二配置源 | [`plugin-inventory`](../../../packages/host/plugin-inventory/src/index.ts) |

## 如何使用这些补充

阅读某个包时，先用 package README 理解公开契约，再用本页定位跨包所有权，最后沿[关键源码路径](critical-paths.md)追踪运行链。若本页与当前源码不一致，应以源码、生成参考和正式 subsystem 文档为准，并将差异视为本分析需要更新，而不是 package contract 自动改变。
