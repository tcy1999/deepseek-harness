# 上下文、技能、压缩与协作状态

## “记忆”不是一个统一存储

Harness 将模型输入相关状态拆为四类：静态或动态 prompt 内容、Session Log 中的对话记录、根据日志生成的摘要或引用，以及 goal/plan/todo 等结构化状态。它们可以使用不同 Service 和 UI，但进入模型请求的内容必须已经记录，或者能根据记录重新计算。

## Context Plugins

`agent-instructions` 从 workspace 指令文件生成模型可见内容；`time-context` 提供当前时间；`tmux-context` 提供终端环境；`session-reference` 允许引用其他会话。它们注册动态 prompt 内容。Agent Loop 在 pre-step 时生成完整快照，并在内容变化时写入 Session Log。Provider 负责读取和生成文本；Agent Loop 负责决定哪个快照进入下一次请求。

动态 context 使用完整快照而非增量 patch。内容变化时，新快照以追加消息写入，并在文本中声明更早快照不再适用；旧快照不会被这次追加物理覆盖，所以在 compaction 移除前仍占用历史 token。恢复逻辑从当前 surface 找到最后一个保留快照并与新装配结果比较，避免依赖一串可能缺失的增量 patch。

## Skill

`skill/skill` 是 provider registry，`skill-filesystem` 从文件目录发现和加载 skill，`tool-skill` 给模型目录/加载操作，`skill-badge` 提供展示元数据。Skill 指令并非启动时全部塞进 prompt：先暴露摘要目录，需要时再加载正文，降低常驻 token。

Skill 文件来自不可信 workspace 内容；加载器负责路径和格式，指令只影响模型决策，不能越过工具审批与 sandbox。把“模型被要求做什么”和“执行器允许做什么”分层是安全基础。

## Compaction

`compaction` 定义压缩能力，`compaction-basic` 生成摘要，`command-compact` 提供直接入口，`compaction-tool-result-pruner` 针对大工具结果。压缩结果写成新的 Session Event，并改变后续请求使用的历史消息。它不会覆盖旧事件，因此原始对话、审计和 fork 仍可访问。

摘要是模型生成事实而非无损编码；它减少上下文长度但可能丢信息。系统保留 source relationship，并让压缩策略成为 Provider，以便部署选择质量、成本和触发点。

## Goal、Plan 与 Todo

Goal 是同 Session 的持久 objective 与进展状态，`goal-round-driver` 通过 Agent 事件继续回合，command/tool 是人和模型入口。Plan mode 是日志化协作状态，直接命令进入并经审阅退出；`todo_write` 是更轻量的模型工具。三者没有合并成一个“任务系统”，因为 ownership 和交互不同：Goal 驱动持续执行，Plan 约束协作模式，Todo 只表达当前工作列表。

结构化状态最终写入 Session Event，或者交给明确的持久化服务保存。UI 只显示根据这些记录算出的当前状态。插件不能只在内存设置 `isPlanning` 后改变 prompt，否则恢复会话时会丢失 Plan 模式。

## Settings、Credentials 与 Identity

Settings 是用户可编辑、持久的部署选择；Credentials 通过 reference 解析 secret；anonymous identity 提供非敏感稳定标识。模型配置只保存 credential ref，不把 API key 写入 Session 或 UI transport。部署 tunable 进入 validated Config/Settings，协议常量和安全 invariant 保持固定。

## 设计评价

这种拆分避免一个“万能 memory service”同时处理来源可信度、资源释放和持久化，也允许单独替换每类状态的实现。缺点是模型输入由多个插件共同生成。排查 token 或模型行为时，需要同时检查 `systemPrompt.assemble()` 的结果、动态内容快照、从 Session Log 算出的历史消息，以及当前 preset 安装了哪些内容。
