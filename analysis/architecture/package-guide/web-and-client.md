# Host 与 Client Packages

## Host

- [`host/webserver`](../../../packages/host/webserver/README.md)（D/P）：HTTP server/route 能力与生命周期。
- [`host/apiproxy`](../../../packages/host/apiproxy/README.md)（P/X）：浏览器到 API Gateway 的 transport bridge。
- [`host/frontend-static`](../../../packages/host/frontend-static/README.md)（P）：前端静态 artifact 服务。
- [`host/plugin-inventory`](../../../packages/host/plugin-inventory/README.md)（D/X）：发布有效插件/配置的只读 inventory，不拥有 Loader 状态。
- [`host/directory-picker`](../../../packages/host/directory-picker/README.md)（D）：目录选择能力。
- [`host/directory-picker-auto`](../../../packages/host/directory-picker-auto/README.md)（P/X）：按平台/可用能力选择 Provider。
- [`host/directory-picker-browse`](../../../packages/host/directory-picker-browse/README.md)（P）：浏览式目录选择。
- [`host/directory-picker-native`](../../../packages/host/directory-picker-native/README.md)（P）：原生系统 picker。

## Client Runtime 基础

- [`client/connection`](../../../packages/client/connection/README.md)（D/P）：浏览器 transport、重连、请求 correlation 和订阅。
- [`client/runtime`](../../../packages/client/runtime/README.md)（D/X）：浏览器 Cordis runtime 与远程对象生命周期。
- [`client/modules`](../../../packages/client/modules/README.md)（D）：客户端模块注册与发现。
- [`client/hmr`](../../../packages/client/hmr/README.md)（X）：浏览器插件热重载。
- [`client/locale`](../../../packages/client/locale/README.md)（D/X）：locale service 与资源贡献。
- [`client/schema-form`](../../../packages/client/schema-form/README.md)（C/S）：配置 schema 到表单。
- [`client/web`](../../../packages/client/web/README.md)（X）：浏览器应用 composition shell。
- [`client/web-react`](../../../packages/client/web-react/README.md)（P/X）：React renderer 与根生命周期。
- [`client/ui-primitives`](../../../packages/client/ui-primitives/README.md)（S）：共享 UI primitives。
- [`client/ui-slots`](../../../packages/client/ui-slots/README.md)（D）：可撤销的 UI slot/renderer registry。
- [`client/ui-layout`](../../../packages/client/ui-layout/README.md)（X）：应用布局和 slot placement。
- [`client/ui-theme`](../../../packages/client/ui-theme/README.md)（X）：主题状态与样式应用。
- [`client/ui-sidebar`](../../../packages/client/ui-sidebar/README.md)（X）：Session/Workspace 导航侧栏。

## Conversation 与执行呈现

- [`client/ui-conversation`](../../../packages/client/ui-conversation/README.md)（C/X）：Session Event conversation nodes 与 keyed renderer。
- [`client/ui-trajectory`](../../../packages/client/ui-trajectory/README.md)（C）：turn/step/tool trajectory 视图。
- [`client/ui-tool`](../../../packages/client/ui-tool/README.md)（C）：按 presentation intent 渲染 generic/terminal/diff call，以及 generic/terminal/diff/read/search/web result card。
- [`client/ui-attachment`](../../../packages/client/ui-attachment/README.md)（C）：attachment 引用和内容 UI。
- [`client/ui-deliverables`](../../../packages/client/ui-deliverables/README.md)（C）：交付物展示。
- [`client/ui-input-trigger`](../../../packages/client/ui-input-trigger/README.md)（C/X）：输入触发与 followup/steer 交互。

## Agent 与协作 UI

- [`client/ui-agent-preset`](../../../packages/client/ui-agent-preset/README.md)（C）：选择 Agent preset/persona。
- [`client/ui-commands`](../../../packages/client/ui-commands/README.md)（C）：命令入口。
- [`client/ui-goal`](../../../packages/client/ui-goal/README.md)（C）：Goal 状态和控制。
- [`client/ui-jobs`](../../../packages/client/ui-jobs/README.md)（C）：后台 job 状态。
- [`client/ui-message-feedback`](../../../packages/client/ui-message-feedback/README.md)（C）：消息反馈。
- [`client/ui-plan`](../../../packages/client/ui-plan/README.md)（C）：Plan mode 状态和审阅。
- [`client/ui-skill`](../../../packages/client/ui-skill/README.md)（C）：Skill catalog/load/badge。
- [`client/ui-subagent`](../../../packages/client/ui-subagent/README.md)（C）：子任务生命周期和结果。
- [`client/ui-user-questions`](../../../packages/client/ui-user-questions/README.md)（C）：结构化问题回答。
- [`client/ui-workflow-run`](../../../packages/client/ui-workflow-run/README.md)（C）：Workflow run 进度与控制。
- [`client/ui-workspace`](../../../packages/client/ui-workspace/README.md)（C）：Workspace 选择与状态。

## Settings 与选择器 UI

- [`client/ui-settings`](../../../packages/client/ui-settings/README.md)（D/X）：Settings page/section registry。
- [`client/ui-settings-general`](../../../packages/client/ui-settings-general/README.md)（C）：通用设置。
- [`client/ui-settings-models`](../../../packages/client/ui-settings-models/README.md)（C）：模型 route 与 credential reference 设置。
- [`client/ui-settings-plugin-inventory`](../../../packages/client/ui-settings-plugin-inventory/README.md)（C）：只读插件 inventory。
- [`client/ui-settings-plugins`](../../../packages/client/ui-settings-plugins/README.md)（C）：插件配置 UI。
- [`client/ui-model-selection`](../../../packages/client/ui-model-selection/README.md)（C）：provider/model 选择。
- [`client/ui-permission-presets`](../../../packages/client/ui-permission-presets/README.md)（C）：permission preset 选择；最终 enforcement 在 host tool pipeline。
- [`client/ui-directory-picker-browse`](../../../packages/client/ui-directory-picker-browse/README.md)（C）：browse picker 前端。
- [`client/ui-directory-picker-native`](../../../packages/client/ui-directory-picker-native/README.md)（C）：native picker 前端桥。
