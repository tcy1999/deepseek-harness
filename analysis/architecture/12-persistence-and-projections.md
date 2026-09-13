# Session 怎样保存，以及怎样供 UI 和查询读取

## 三项分开的工作

1. `core/session` 保存当前进程中的追加式日志，并检查事件顺序；
2. `session-persistence` 定义保存、加载和 checkpoint，JSONL 和 SQLite 提供具体存储；
3. projection、query、title、stats 和 telemetry 根据日志计算读取结果。

这里的 projection（投影）就是“按规则读取 Session Event，并计算出另一种方便使用的数据”。例如，同一份日志可以计算出模型消息、会话标题、使用量统计和 UI 节点。投影可以缓存，但不能修改日志，也不能成为另一份会话事实。

把三项工作分开后，数据库 schema 不会反过来决定 Agent 记录哪些事件，UI 查询也不会成为写入会话状态的另一条路径。

## 后台写盘由谁协调

[`session-persistence`](../../packages/session/session-persistence/src/coordinator.ts)监听 Session 的创建、事件和释放，把新增事件放入后台写盘队列，并在 flush 或 dispose 时等待写盘完成。它用 revision 和 preparation 状态防止加载、首次保存和并发追加互相覆盖。Backend 负责文件原子替换或数据库事务；coordinator 决定当前 Session 的各批事件按什么顺序交给 backend。

Write-behind 减少每个 chunk 等待磁盘的时间，但内存日志可能暂时领先磁盘。`session-checkpoint-policy` 会在发起模型请求、执行顶层工具和开始新 step 前等待 checkpoint，保证新的外部操作开始前，导致该操作的历史已经保存。如果 checkpoint 失败，后续操作不会执行。

## JSONL 与 SQLite

JSONL backend 维护一份逻辑追加式 JSONL，但默认物理文件是 `.jsonl.zstd`：header 和每批 append 分别写成带 checksum 的独立 Zstandard frame，连续 chunk 还可打包成一条 storage row，因此默认文件不能直接逐行人工读取。只有 `compression: 'none'` 才保存原始 `.jsonl` 文本。Backend 会 `fsync` 每批写入，回滚失败的部分 append，并在加载时截断可恢复的 torn tail；SQLite 则提供事务、索引和可查询的事件存储。

两种 backend 必须提供相同的保存、加载和 checkpoint 操作，并明确拒绝不支持的格式，不能猜测如何迁移。SQLite schema 版本只能递增；Session Event 外层格式使用另一个版本号，两者记录不同内容。

`storage` 家族保存非 Session 数据，提供 JSON/SQLite backend 和各类业务数据的读写接口。它不替代 session persistence，因为 Session Log 还需要检查事件顺序、修复崩溃留下的不完整 turn、等待 checkpoint，并重建模型消息。

## 附件与 Workspace 不是 Session backend 的附属表

[`attachment`](../../packages/attachment/attachment/src/index.ts)先验证并原子保存不可变图片字节，再允许 `user/message` 等模型可见事件记录可序列化的 `ImageAttachmentRef`。Session Log 只保存 content-addressed id、media type、字节数和尺寸，不保存浏览器路径、object URL、provider URL 或 base64；Provider adapter 发送模型请求时再通过 store 读取并校验 digest/metadata。本地实现把对象放在 `DSH_HOME/attachments/v1/objects`，不同 Session 或 fork 可以共享同一对象，因此当前没有自动 GC。

[`workspace`](../../packages/workspace/workspace/src/index.ts)则使用 `storage-domain` 保存 Workspace record、显示顺序、session candidate account 和 archive set。Session 归组仍要同时满足 account 中有 id，并且持久 Session header 的 canonical cwd 等于 Workspace path；删除 Workspace registration 不删除目录、live Session 或 Session log，只会让保留的 Session 回到 Ungrouped。它不写 Session Event，也不进入模型请求。

## 从日志计算读取结果

`session-projection` 注册计算规则；规则按事件顺序逐个更新结果。`session-projection-cache` 按 `seq` 缓存增量结果；title 和 stats 提供具体规则。缓存只用于加速，删除缓存后仍能从完整日志重建。只有事件写入日志后才能更新读取结果，不能先向 UI 发布预测状态。

Title Provider 根据符合条件的 prompt 生成 `session/title` event；`first-prompt` 只用首条 prompt，`all-prompts` 可以使用全部 prompt。公共 Service 处理手动刷新、失败后的备用方案、取消，以及多个结果同时返回时采用最后一个结果。标题写成事件而不是直接修改 Session metadata，因此可以重放标题来源和更新顺序。

## Query

`session-query` 定义可以查询哪些 Session、每次最多读取多少内容、父子 Session 关系和事件关系；SQLite Provider 添加全文检索；`tool-session-query` 给模型使用；`session-log-export` 负责导出。查询结果返回 Session 和事件 id，不暴露数据库内部行。大小限制应用到最终结果，防止单个巨大事件绕过条数限制。

## Telemetry

`session-telemetry` 根据 Session Event 以及 Agent 的创建和释放生成不依赖具体监控 Provider 的数据，OTel 包负责导出。遥测只读取已经提交的事件，不能阻止 Session 写入；导出失败也不能改变 Agent 行为。原始 Session 可能包含模型内容，部署者要在 exporter 配置中限制发送哪些数据。

## 不同读取方式何时能看到新事件

当前进程写入 Session 后可以立即读取；投影可以同步或增量更新；checkpoint 完成后才能保证 write-behind 已经落盘；远程查询只能看到 backend 已提交的数据。因此不同读取方式可能短暂看到不同进度。系统在发起新的模型请求或外部操作前等待 checkpoint，避免动作已经发生而相关历史尚未保存。
