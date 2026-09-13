# 组合、协议、子 Agent 与示例 Packages

## Boot 与 Bundle

- [`boot/app-boot`](../../../packages/boot/app-boot/README.md)（X）：Profile/bundle/patch、Harness home、Loader、热重载和应用 settle。关键实现 `src/profile.ts`、`src/index.ts`。
- [`boot/cmdline`](../../../packages/boot/cmdline/README.md)（S/X）：CLI 参数与 dispatch 意图解析，不负责具体 runtime。
- [`bundle/base`](../../../packages/bundle/base/README.md)（X）：所有 Profile 的共同 patch 骨架；实现主体是 `cordis.patch.yml`。
- [`bundle/headless`](../../../packages/bundle/headless/README.md)（X）：一次性无服务器 runner 组合。
- [`bundle/web-app`](../../../packages/bundle/web-app/README.md)（X）：Web host/client 启动组合。

## API 与 SDK

- [`api/gateway`](../../../packages/api/gateway/README.md)（X）：Typert RPC/BFF 组装、方法与订阅 dispatch。
- [`api/remotes`](../../../packages/api/remotes/README.md)（X）：转发明确允许的 Host 事件，并组装 Client 领域控制器。
- [`sdk/protocol`](../../../packages/sdk/protocol/README.md)（D）：JSON-RPC wire、request/response/event 和 branded identity。
- [`sdk/server`](../../../packages/sdk/server/README.md)（P/X）：将 Agent runtime 暴露为 JSON-RPC server。
- [`sdk/client`](../../../packages/sdk/client/README.md)（C）：管理连接和订阅，把响应对应到请求，并在断线时结束未完成请求。
- [`acp/acp`](../../../packages/acp/acp/README.md)（P/X）：automation-only ACP server，拥有其创建的 AgentHandle 并映射协议生命周期。

## API 领域控制器

- [`api/session-controller`](../../../packages/api/session-controller/README.md)（X）：Session 创建、恢复、命令、历史 follow 与客户端状态。
- [`api/workspace-controller`](../../../packages/api/workspace-controller/README.md)（X）：Workspace 操作、列表和目录选择。
- [`api/settings-controller`](../../../packages/api/settings-controller/README.md)（X）：设置与凭据操作。
- [`api/workspace-files`](../../../packages/api/workspace-files/README.md)（X）：工作区文件读取与变更流。

## Hooks 与 MCP

- [`hooks/hook-protocol`](../../../packages/hooks/hook-protocol/README.md)（D）：Claude Code/Codex bridge 共用 wire protocol。
- [`hooks/hooks-claude-code`](../../../packages/hooks/hooks-claude-code/README.md)（P/X）：Claude Code hook 适配器。
- [`hooks/hooks-codex`](../../../packages/hooks/hooks-codex/README.md)（P/X）：Codex hook 适配器。
- [`mcp/mcp-client`](../../../packages/mcp/mcp-client/README.md)（P/X）：MCP server 连接、工具发现和注册；MCP 数据在外部 wire 边界验证。

## Subagent

- [`subagent/subagent`](../../../packages/subagent/subagent/README.md)（D）：provider registry、task/control/report 公共协议。
- [`subagent/subagent-fork-in-process`](../../../packages/subagent/subagent-fork-in-process/README.md)（P）：从 closed-turn Session 前缀 fork child。
- [`subagent/subagent-spawn-in-process`](../../../packages/subagent/subagent-spawn-in-process/README.md)（P）：在同一 runtime 创建隔离 Agent。
- [`subagent/subagent-in-process-driver`](../../../packages/subagent/subagent-in-process-driver/README.md)（P/S）：共享的进程内 drive/lifecycle 实现。
- [`subagent/subagent-dsh-sdk`](../../../packages/subagent/subagent-dsh-sdk/README.md)（P）：经 dsh SDK 连接另一个 runtime。
- [`subagent/subagent-acp`](../../../packages/subagent/subagent-acp/README.md)（P）：经 ACP 委托。
- [`subagent/subagent-claude-code`](../../../packages/subagent/subagent-claude-code/README.md)（P）：Claude Code Provider。
- [`subagent/subagent-codex`](../../../packages/subagent/subagent-codex/README.md)（P）：Codex Provider。
- [`subagent/tool-subagent`](../../../packages/subagent/tool-subagent/README.md)（C）：创建/委托子任务。
- [`subagent/tool-subagent-control`](../../../packages/subagent/tool-subagent-control/README.md)（C）：控制运行中子任务。

## 应用与示例

- [`apps/cli`](../../../apps/cli/README.md)：`dsh` profile 启动与插件管理。
- [`apps/desktop`](../../../apps/desktop/README.md)：Electron 壳、内置运行时和 desktop profile。
- [`bundle/sdk-app`](../../../packages/bundle/sdk-app/README.md)（X）：SDK stdio 应用组合。
- [`bundle/sdk-minimal`](../../../packages/bundle/sdk-minimal/README.md)（X）：独立的最小 SDK 配置树。
- [`bundle/acp-app`](../../../packages/bundle/acp-app/README.md)（X）：ACP 应用组合。

可运行示例通过受支持的 `dsh` profile 启动，调用方式见 [TypeScript SDK](../../../packages/sdk/client/README.md)与 [Python SDK](../../../python/README.md)，记录回放见 [Session snapshot 支撑](../../../packages/test-support/session-snapshot/README.md)。
