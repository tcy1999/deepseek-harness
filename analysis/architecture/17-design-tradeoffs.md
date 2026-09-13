# 设计取舍与综合评价

## 几乎所有功能都由插件安装

**收益：** 部署、Provider、政策、UI 和 Loop 都可替换；HMR 与测试使用相同的注册和释放规则。**成本：** 行为分散在注册者、listener 和 patch 顺序中，只看静态调用图无法确定运行时安装了什么。**约束：** 所有注册都必须能撤销；配置错误必须明确失败；系统要提供最终配置、注册表和运行时关系检查等诊断信息。

相比固定 application container，这更适合实验性 Agent 产品和多入口部署；对单一 CLI 则可能过度模块化。项目通过 bundle 提供 opinionated 默认组合，减轻最终用户装配成本。

## 模型历史只从 Event Log 生成

**收益：** 请求可以重建；fork、resume、UI、查询和 telemetry 读取同一份记录；流式过程也能回放。**成本：** 新增模型输入时要定义事件和消息计算规则；日志增长较快；格式变化需要严格处理。**没有采用的做法：** 直接维护可变 messages 数组更简单，但恢复后无法确认发送给模型的历史和之前相同。

追加式日志不等于 event sourcing 整个产品：PTY、进程、live Agent、approval waiter 等仍是实时资源，日志只记录模型与用户需要的事实。这个边界是合理的，否则恢复外部资源将产生虚假保证。

## 接口、实现和调用方分包

**收益：** Consumer 不绑定本地实现，更换 FS 和 subprocess Provider 可以让多个高层工具一起改用远程环境。**成本：** 包数量达到两百以上，理解一个功能需要跨包。**约束：** 只有角色需要独立替换时才拆分；Definition 只包含当前 Consumer 需要的操作；bundle 必须同时安装接口、实现和调用方。

这是代码库最显著的组织成本。Package guide、generated module graph 和 capability docs 是必要基础设施，而不是可选文档。

## Waterfall 扩展

**收益：** 重试、权限检查、回放和请求配置可以通过 listener 添加，无需修改发起操作的服务；listener 也能包裹异步流。**成本：** 忘记调用 `next()` 会阻止后续操作；listener 顺序会改变结果；每层都要明确处理自己的错误。**约束：** 事件文档要说明调用模式；运行时检查关键关系；事件要携带正确的 Agent scope。

它比无序 event bus 更能表达中间件，但没有静态 middleware stack 那么直观。对于安全决策，项目在 waterfall 后增加 monotonic guard 或最终 executor enforcement，避免 listener 顺序成为唯一防线。

## 发布前完整配置

**收益：** 监听者不会看到只完成一半配置的 Agent；preset 的工具、prompt 和权限规则在第一次请求前已经安装。**成本：** create/resume 必须处理配置期间取消、配置成功后创建者又释放，以及通知已经发出后失败等情况。**约束：** setup 只安装组件，不启动 Agent；同步 `commit()` 完成后才加入 registry。

这比“先注册再异步装插件”更可靠，尤其适合 UI/ACP 立即订阅 created event 的环境。

## 后台写盘，并在外部操作前确认落盘

**收益：** 记录每个 chunk 时不用等待磁盘；发起新的模型请求或外部操作前，相关历史又已经落盘。**成本：** 内存、磁盘和远程查询可能短暂处于不同进度；所有会产生外部影响的入口都必须先等待 checkpoint。**其他做法：** 每条事件同步写盘更简单但更慢；只在退出时 flush，则进程崩溃后可能只留下外部动作，没有留下导致该动作的历史。

当前设计平衡良好，风险在于新增外部行为时忘记 checkpoint；因此 checkpoint policy 与真实路径测试很重要。

## 共享服务，但按 Agent 筛选注册项

**收益：** 大部分 runtime service 可共享，贡献按 Agent 筛选，preset 局部覆盖。**成本：** scope carrier 和 isolate realm 容易混淆，直接读全局 registry 可能泄漏工具。**约束：** `scopeTarget`/`scopeOf`、agent-scoped context、执行时二次解析。

如果所有服务都复制到每个 Agent，隔离直观但资源昂贵且跨 Agent provider 难共享；当前混合模型更灵活，但要求 package 作者理解两种隔离。

## 严格失败而非兼容降级

当前 Pre-release 策略允许拒绝旧磁盘格式、未知 required event、无效 bundle 和不完整配置。**收益：** 不会静默生成错误模型上下文。**成本：** 升级需要明确迁移或清理，外部消费者兼容性低；现阶段的磁盘格式和 package 边界都不构成稳定兼容承诺。

## 总体判断

这些设计解决四个不同问题：插件 effect 负责安装和卸载；Session Event 负责恢复模型看到的历史；Definition/Provider/Consumer 负责替换实现；projection 负责从同一份日志生成模型消息、标题、统计和 UI 数据。复杂度来自一个功能经常同时涉及这四部分。对于需要多个 Provider、多个 UI 或协议入口、会话恢复和动态组合的 Agent 平台，这些成本有价值；如果产品只使用固定模型和固定工具，当前拆分会显得过重。

Pre-release 重构可以调整 package 边界，但当前设计仍要求：模型请求可以从日志重建；权限在真正执行操作的位置检查；Agent 配置完成后才对外可见；每个注册项和异步资源都有明确 owner 负责取消、等待结束并释放。
