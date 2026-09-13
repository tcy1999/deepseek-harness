# Web Host 与 Client 架构

## Host/Browser 分离

Host 侧提供 webserver、API proxy、静态资源、目录选择和 plugin inventory；browser 侧通过 connection/runtime 接入 RPC，再由大量 `ui-*` 插件贡献领域 UI。`client/web` 与 `web-react` 是装配外壳，不拥有 Agent 业务。组合由 `dsh-web-app` bundle 决定。

这样 CLI/headless 不需要浏览器代码，UI 模块也能随插件安装和卸载。代价是一个完整功能通常要同时提供 host 服务、远程接口、客户端对象和 renderer。

## Client Runtime 与 Modules

`client/runtime` 提供浏览器中的 Cordis context，并跟踪远程对象何时创建和失效；`connection` 管理传输、重连和未完成请求；`modules` 注册和查找客户端模块；`hmr` 更新浏览器插件。`locale`、`theme`、`primitives` 和 `slots` 提供多个页面共用的基础功能。

UI 插件向 slot 注册 renderer 或 action，并在 fiber dispose 时撤销。页面不是一个中央 switch 列出所有领域组件；conversation、tool、trajectory、settings 等插件按 capability 自注册。这样第三方功能可以增加 UI，而无需修改单体前端入口。

## Session 驱动的 UI

Conversation 和 trajectory 根据 Session Event 或服务端计算结果，显示 user、assistant、tool、plan、goal、subagent 等节点。原始 chunk 用于流式显示；提交后的 message/result 用于刷新和恢复。UI 可以暂时显示“正在发送”等本地状态，但服务端事件到达后必须以服务端记录为准，否则刷新页面会看到不同历史。

`ConversationNodeDefinition` 与 keyed renderer 把新 durable node 类型接入客户端。节点 identity 来自事件 seq/关系，而不是数组 index，支持增量更新和 fork。Tool call presentation 使用 `generic`、`terminal`、`diff` card；完成结果还可使用 `read`、`search`、`web` card。Client 按服务端定义的 tagged intent 选择 renderer，未知或畸形 intent 才退回扁平文本，不解析自由文本猜类型。

## Settings 与 Host 能力

Settings UI 分为 general、models、plugin inventory/plugins 等模块；写入必须调用 settings service 或 host API，不能直接修改内存中的 Cordis tree。Directory picker 有 auto、browse、native Provider，UI 只依赖共同接口。Plugin inventory 只显示 Loader 已生效的插件和配置，不能反过来保存另一份配置。

## UI 插件族

领域插件包括 agent preset、attachment、commands、deliverables、goal、jobs、message feedback、model selection、permission presets、plan、skill、subagent、tool、user questions、workflow 和 workspace。这些包大多只负责把服务端数据交给页面中的指定位置和 renderer，不负责保存或改变业务状态。详见[Web 与 Client package 指南](package-guide/web-and-client.md)。

## 风险与边界

插件式 UI 避免所有页面都堆进一个中央模块，但插件注册顺序、slot 接口、远程类型和 host/client 版本必须匹配。Client 使用独立 TypeScript 配置，不能引入只在 host 运行的 Node 模块；仓库用依赖图和编译检查强制这一点。重连后还要恢复订阅，因此必须区分只能实时看到的 Agent event 和可以从日志重放的 Session event。
