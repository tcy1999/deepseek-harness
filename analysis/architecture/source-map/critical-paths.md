# 关键源码路径

## 启动与组合

| 问题 | 入口 | 后续路径 |
|---|---|---|
| CLI 如何选择运行模式？ | [`apps/cli/src/bin.ts`](../../../apps/cli/src/bin.ts) | `run` / `dump-config` / plugin dispatch |
| Profile 在哪里解析？ | [`boot/app-boot/src/profile.ts`](../../../packages/boot/app-boot/src/profile.ts) | `resolveProfileDir` → `loadProfile` → patch composition |
| 实际产品装了什么？ | [`bundle/base/cordis.patch.yml`](../../../packages/bundle/base/cordis.patch.yml) | 再叠加 web/headless/profile overlays |
| Loader 如何启动并 settle？ | [`boot/app-boot/src/index.ts`](../../../packages/boot/app-boot/src/index.ts) | Context → Loader → entry tree → patch watcher |

## Agent 生命周期

| 问题 | 入口 | 后续路径 |
|---|---|---|
| 谁定义 Agent 公共接口？ | [`core/agent/src/runtime-types.ts`](../../../packages/core/agent/src/runtime-types.ts) | Registry 在 `src/index.ts` |
| Inbox 如何区分 followup/steer/inject？ | [`core/agent/src/inbox.ts`](../../../packages/core/agent/src/inbox.ts) | `ReactLoopAgent.send()` 分类与 wake latch |
| Agent 如何在配置完整后发布？ | [`core/agent-loop/src/index.ts`](../../../packages/core/agent-loop/src/index.ts) | prepare → setup/commit → enter/announce → start |
| Turn/Step 在哪里运行？ | [`core/agent-loop/src/agent.ts`](../../../packages/core/agent-loop/src/agent.ts) | `kick()` → `turn()` → `preStep()` → `step()` |
| 工具调用如何调度？ | [`core/agent-loop/src/tool-calls.ts`](../../../packages/core/agent-loop/src/tool-calls.ts) | executionMode → scheduler → Session results |

## 模型请求与日志

| 问题 | 入口 | 后续路径 |
|---|---|---|
| 模型历史如何生成？ | [`core/session/src/index.ts`](../../../packages/core/session/src/index.ts) `deriveMessages()` | [`surface.ts`](../../../packages/core/session/src/surface.ts) per-node fold |
| Prompt 如何装配？ | [`core/system-prompt/src/index.ts`](../../../packages/core/system-prompt/src/index.ts) | scoped sections/context/tools → render |
| Provider/model 如何绑定一次调用？ | [`llm/llm/src/index.ts`](../../../packages/llm/llm/src/index.ts) | prepare → captured adapter registration → stream |
| Chunk 如何成为 message？ | [`core/agent-loop/src/agent.ts`](../../../packages/core/agent-loop/src/agent.ts) `step()` | `BlockAssembler` → chunk events → assistant message |
| 请求是否可重建由谁检查？ | [`core/agent-loop/src/invariant.ts`](../../../packages/core/agent-loop/src/invariant.ts) | 比较 loop request 与 Session 派生结果 |

## 工具与政策

| 问题 | 入口 | 后续路径 |
|---|---|---|
| 工具在哪里注册/按 scope 可见？ | [`core/tools/src/index.ts`](../../../packages/core/tools/src/index.ts) | definition map → scoped view → schema provider |
| 参数/返回如何定义？ | [`core/tools/src/schema.ts`](../../../packages/core/tools/src/schema.ts) | JSON schema → typed execute → content blocks |
| 审批/guard/body/post 顺序在哪？ | [`core/tools/src/index.ts`](../../../packages/core/tools/src/index.ts) `prepareExecution()` | pre-execute → ask → guard → execute → post |
| Code Mode 如何阻止直接调用？ | 同文件 `collapses()` / `createExecution()` | model-direct denial，nested dispatch 例外 |
| 工具事件关系由谁检查？ | [`core/tools/src/invariant.ts`](../../../packages/core/tools/src/invariant.ts) | execution stage 与 session call/result 关系 |

## 保存日志与生成读取结果

| 问题 | 入口 | 后续路径 |
|---|---|---|
| Session Store 与磁盘如何分离？ | [`core/session/src/index.ts`](../../../packages/core/session/src/index.ts) `SessionStore` | persistence listener 订阅事件 |
| Write-behind 谁协调？ | [`session-persistence/src/coordinator.ts`](../../../packages/session/session-persistence/src/coordinator.ts) | queue/revision/checkpoint/dispose |
| 外部行为前谁保证落盘？ | [`session-checkpoint-policy/src/index.ts`](../../../packages/session/session-checkpoint-policy/src/index.ts) | pre-step、LLM stream、顶层工具 listener |
| Projection 如何定义？ | [`session-projection/src/index.ts`](../../../packages/session/session-projection/src/index.ts) | pure fold registry → cache |
| Query 如何保持 backend-neutral？ | [`session-query/src/index.ts`](../../../packages/session-query/session-query/src/index.ts) | corpus contract → SQLite Provider/tool |

## 集成与 UI

| 问题 | 入口 | 后续路径 |
|---|---|---|
| Host object 如何变成 wire id？ | [`typert/protocol/src/index.ts`](../../../packages/typert/protocol/src/index.ts) | registry lookup/context → gateway |
| JSON-RPC server 如何接 runtime？ | [`sdk/server/src/server.ts`](../../../packages/sdk/server/src/server.ts) | fixed request dispatch → owned Agent → unfiltered notifications |
| ACP 如何创建并拥有 Agent？ | [`acp/acp/src/index.ts`](../../../packages/acp/acp/src/index.ts) | connection/session → AgentHandle → dispose |
| Web API 如何进入浏览器？ | [`host/apiproxy/src/index.ts`](../../../packages/host/apiproxy/src/index.ts) | gateway → client connection/runtime |
| 新 conversation node 如何显示？ | [`client/ui-conversation/src/index.ts`](../../../packages/client/ui-conversation/src/index.ts) | node definition → keyed renderer/slot |
