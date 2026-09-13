# Agent 能力与协作 Packages

## Context 与 Skill

- [`context/agent-instructions`](../../../packages/context/agent-instructions/README.md)（X）：发现 workspace 指令并贡献 runtime context。
- [`context/session-reference`](../../../packages/context/session-reference/README.md)（X）：把其他 Session 的受控引用加入当前上下文。
- [`context/time-context`](../../../packages/context/time-context/README.md)（X）：贡献可记录的当前时间上下文。
- [`context/tmux-context`](../../../packages/context/tmux-context/README.md)（X）：贡献 tmux/终端环境信息。
- [`skill/skill`](../../../packages/skill/skill/README.md)（D）：Skill provider registry 与 metadata。
- [`skill/skill-filesystem`](../../../packages/skill/skill-filesystem/README.md)（P）：从文件系统 catalog 发现、验证和加载 Skill。
- [`skill/tool-skill`](../../../packages/skill/tool-skill/README.md)（C）：按需列出/加载 Skill 的模型工具。
- [`skill/skill-badge`](../../../packages/skill/skill-badge/README.md)（X）：Skill 载入状态与展示 badge。

## Compaction

- [`compaction/compaction`](../../../packages/compaction/compaction/README.md)（D）：日志压缩/摘要能力。
- [`compaction/compaction-basic`](../../../packages/compaction/compaction-basic/README.md)（P）：基础 LLM summary Provider。
- [`compaction/command-compact`](../../../packages/compaction/command-compact/README.md)（C）：人类直接触发 compact 的命令。
- [`compaction/compaction-tool-result-pruner`](../../../packages/compaction/compaction-tool-result-pruner/README.md)（X）：针对历史大工具结果的投影/压缩政策。

## Goal、Plan、Todo、Schedule 与 Feedback

- [`goal/goal`](../../../packages/goal/goal/README.md)（D）：同 Session objective、状态和 durable events。
- [`goal/goal-round-driver`](../../../packages/goal/goal-round-driver/README.md)（X）：通过 Agent lifecycle 继续未完成 goal。
- [`goal/command-goal`](../../../packages/goal/command-goal/README.md)（C）：人类 goal 命令。
- [`goal/tool-goal`](../../../packages/goal/tool-goal/README.md)（C）：模型 goal 工具。
- [`plan/plan-mode`](../../../packages/plan/plan-mode/README.md)（D/C）：日志化 plan collaboration state、直接入口和审阅退出。
- [`todo/tool-todo`](../../../packages/todo/tool-todo/README.md)（C）：轻量 `todo_write` 模型工具。
- [`schedule/schedule`](../../../packages/schedule/schedule/README.md)（D/P）：Session-local 定时 follow-up 与唤醒。
- [`feedback/message-feedback`](../../../packages/feedback/message-feedback/README.md)（D）：消息反馈事实与服务。
- [`feedback/command-feedback`](../../../packages/feedback/command-feedback/README.md)（C）：直接反馈命令。

## Jobs 与 Workflow

- [`jobs/jobs`](../../../packages/jobs/jobs/README.md)（D）：后台 job registry、状态和控制协议。
- [`jobs/jobs-local`](../../../packages/jobs/jobs-local/README.md)（P）：本地 Promise/AbortController job runtime。
- [`jobs/tool-jobs`](../../../packages/jobs/tool-jobs/README.md)（C）：list、collect、stop 模型工具。
- [`workflow/workflow`](../../../packages/workflow/workflow/README.md)（D）：workflow run/control/report 协议。
- [`workflow/workflow-worker-thread`](../../../packages/workflow/workflow-worker-thread/README.md)（P）：worker thread workflow 引擎。
- [`workflow/tool-workflow`](../../../packages/workflow/tool-workflow/README.md)（C）：通用 workflow 模型工具。
- [`workflow/tool-ralph`](../../../packages/workflow/tool-ralph/README.md)（C）：Ralph 风格迭代工作流入口。

## Interaction 与 Guard

- [`interaction/commands`](../../../packages/interaction/commands/README.md)（D）：不经过模型 turn 的人类命令 registry。
- [`interaction/user-approval`](../../../packages/interaction/user-approval/README.md)（D）：异步审批能力及 cancellation。
- [`interaction/user-questions`](../../../packages/interaction/user-questions/README.md)（D）：结构化用户问题与回答生命周期。
- [`interaction/permission-presets`](../../../packages/interaction/permission-presets/README.md)（X）：将用户 preset 解析为执行政策。
- [`interaction/tool-ask-user`](../../../packages/interaction/tool-ask-user/README.md)（C）：模型主动提问工具。
- [`guard/repeat-tool-reminder`](../../../packages/guard/repeat-tool-reminder/README.md)（X）：重复工具调用的提示性 loop hygiene。
- [`guard/timeout-policy`](../../../packages/guard/timeout-policy/README.md)（X）：在 `tools/execute` 层实施 deadline。

## Preset 与 Persona

- [`preset/agent-presets`](../../../packages/preset/agent-presets/README.md)（X）：发现、验证并为单 Agent mount preset `cordis.yml`；setup 完成后才发布 Agent。
- [`preset/persona`](../../../packages/preset/persona/README.md)（X）：persona 配置和 prompt section replacement。

## Runtime Self-Extension

- [`extensions/cordis-host-runner`](../../../packages/extensions/cordis-host-runner/README.md)（P/X）：在 host runtime mount/unmount 模型生成的 Cordis 插件。
- [`extensions/cordis-client-runner`](../../../packages/extensions/cordis-client-runner/README.md)（P/X）：浏览器侧对应 runner。
- [`extensions/tool-cordis`](../../../packages/extensions/tool-cordis/README.md)（C）：模型检查和修改自身 host 插件树的工具。
- [`extensions/ui-cordis`](../../../packages/extensions/ui-cordis/README.md)（C）：自扩展状态的 UI。

## Identity

- [`identity/anonymous-user-id`](../../../packages/identity/anonymous-user-id/README.md)（D/P）：生成并提供匿名稳定用户标识，不承担认证。
