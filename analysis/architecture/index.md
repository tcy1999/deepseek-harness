# DeepSeek Harness 架构与源码分析

本目录解释 DeepSeek Harness 怎样组装、怎样运行，以及各部分分别负责什么。它是独立分析，不替代仓库的[正式架构文档](../../docs/architecture.md)、子系统参考或 package README。

## 简介

DeepSeek Harness 不是一个固定的 Agent Loop 再外挂插件。它用 Cordis 插件组装模型适配器、Agent、循环、工具、日志、持久化、审批和 Web UI。

其中很多部分都能替换。例如，模型请求可以交给 DeepSeek Provider 或其他 Provider，文件操作可以在本机或 E2B 中执行，产品也可以带 Web UI 或只运行命令行。替换后的实现必须提供调用方需要的相同操作和错误结果，不能只做到“类型能编译”。

插件还要遵守几条具体规则：

1. 插件卸载时，要删除自己添加的工具和事件监听器，并停止自己启动的后台任务。`ctx.effect()` 用来登记“安装时做什么”和“卸载时怎样清理”。
2. 只给某个 Agent 安装的工具、Prompt 或权限规则，不能被其他 Agent 使用。这就是本文所说的“作用域可见性”。
3. 创建 Agent 时，要先安装它的工具、Prompt 和权限规则，然后才能让 UI 或 API 查到它。这样外部不会拿到一个只配置了一半的 Agent。
4. 收到取消请求后，已经开始的工具或后台任务可能还没有停止。系统要等它结束，避免已经结束的会话仍在写文件或发送事件。
5. 进程崩溃或重启后，模型必须看到与之前相同的会话历史，否则可能重复执行已经产生外部影响的工具。DSH 分三步处理：发送模型请求前，把实际消息和 Session Log 重新生成的消息逐项比较；写入 Session Event 前，检查 turn、step 和工具结果的顺序；重启加载日志时，把崩溃留下的未完成工具标为“没有开始”或“结果未知”，并明确结束未完成的 step 和 turn。在默认 Agent Loop 路径中，前两项检查失败会终止当前 turn、报告 `agent/error`，最外层 driver 会接住错误，不会使整个 DSH 进程或其他 Agent 退出；Loop 之外直接写 Session 的调用方会收到同一个 `InvariantError`，不能假定总有 Agent driver 代为收口。

Profile 决定一次实际启动使用哪些模型、工具、存储和界面。Bundle 是可复用的插件配置，例如 `base` 提供基础 Agent 功能，`web-app` 再添加 Web 服务和 UI。Profile 选择一个或多个 bundle，并覆盖其中的配置。

这些规则让 DSH 可以替换实现和恢复会话。代价是一个完整功能可能分散在接口、具体实现、调用方、启动配置、Session Log 和 UI 中，不能只看一个入口文件。

## 阅读路线

- 先看完整图：打开[系统架构与一次请求流程](visualizations/deepseek-harness-architecture-flow.html)，可在系统架构图和请求时序图之间切换。
- 建立全局模型：从[系统总览](01-system-overview.md)、[Cordis 插件模型](02-cordis-plugin-model.md)和[启动组合](03-composition-and-boot.md)开始。
- 跟踪一次请求：依次阅读 [Agent Runtime](04-agent-runtime.md)、[Session Event Log](05-session-event-log.md)、[Prompt 与 LLM](06-prompt-llm-streaming.md)和[工具执行](07-tools-and-execution.md)。
- 理解扩展：阅读[可替换能力](08-capability-seams.md)、[执行环境](09-execution-environment.md)、[上下文与记忆](10-context-and-memory.md)和[子 Agent/任务/工作流](11-subagents-jobs-workflows.md)。
- 理解数据与产品入口：阅读[持久化与投影](12-persistence-and-projections.md)、[API/SDK/ACP/Hooks](13-api-sdk-acp-hooks.md)和[Web Client](14-web-client-architecture.md)。
- 评估工程设计：阅读[安全与正确性](15-safety-and-correctness.md)、[测试与构建](16-testing-and-build-design.md)和[设计取舍](17-design-tradeoffs.md)。
- 查代码：使用 [package 指南](package-guide/index.md)和[关键源码路径](source-map/critical-paths.md)。

## 本文用词

结论优先来自当前源码、bundle 配置和测试，再用正式文档与活跃 Agent Note 解释原因。包名、Service key、事件名和类型名保留英文。

- `Context`：插件访问 Service、注册事件以及登记清理操作时使用的对象。Agent 的 Context 还决定它能使用哪些专属工具和 Prompt。
- `ctx.effect()`：执行一段安装操作，并保存它返回的清理函数。相关插件或 Agent 卸载时，Cordis 自动调用清理函数。
- 作用域：某个注册项的可见范围。例如，一个工具可以对所有 Agent 可见，也可以只对指定 Agent 可见。
- Profile：用户实际启动的一套最终配置。
- Bundle：一组可复用的插件配置，由 Profile 选择和覆盖。
- 投影：从 Session Log 计算出的读取结果，例如模型消息、会话标题、统计信息和 UI 节点。它不是另一份会话记录。
- UI：用户界面。它读取服务端状态并提交用户操作，不独立保存会话历史。
- Service Definition、Provider、Consumer：Definition 定义一组操作，Provider 实现这些操作，Consumer 调用它们。仓库把这三个角色合称为“能力接缝”。
- 实时事件：只协调当前进程中正在发生的操作。Session Event 则记录可以保存和重放的内容。
