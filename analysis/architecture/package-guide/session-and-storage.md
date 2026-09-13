# Session、存储与用户数据 Packages

## Session Persistence

- [`session/session-persistence`](../../../packages/session/session-persistence/README.md)（D）：单写者 SessionHandle、create/open/stat/list、flush 与错误类型。
- [`session/session-persistence-jsonl`](../../../packages/session/session-persistence-jsonl/README.md)（P）：JSONL 代际、读写句柄、live 事件路由、write-behind 与文件租约。
- [`session/session-checkpoint-policy`](../../../packages/session/session-checkpoint-policy/README.md)（X）：模型请求、pre-step 和顶层工具前等待 durability。

## Session 格式

- [`session/session-format`](../../../packages/session/session-format/README.md)（S）：逻辑格式解码、恢复与相邻迁移链。
- [`session/session-format-catalog`](../../../packages/session/session-format-catalog/README.md)（S）：当前版本与受支持历史代际的静态目录。
- [`session/session-log-deepseek`](../../../packages/session/session-log-deepseek/README.md)（X）：安装当前产品认识的 Session 事件定义与校验。

## Projection、Stats、Title 与 Telemetry

- [`session/session-projection`](../../../packages/session/session-projection/README.md)（D）：纯 fold projection registry。
- [`session/session-projection-cache`](../../../packages/session/session-projection-cache/README.md)（P/X）：按 event seq 增量缓存，可从日志重建。
- [`session/session-stats`](../../../packages/session/session-stats/README.md)（X）：从 turn/step/message/usage 事件投影统计。
- [`session/session-title`](../../../packages/session/session-title/README.md)（D/X）：标题 provider 协调与 `session/title` 投影。
- [`session/session-title-llm`](../../../packages/session/session-title-llm/README.md)（P）：公共 LLM 标题生成实现。
- [`session/session-title-first-prompt-llm`](../../../packages/session/session-title-first-prompt-llm/README.md)（P/X）：首个合格 prompt 的标题策略。
- [`session/session-title-all-prompts-llm`](../../../packages/session/session-title-all-prompts-llm/README.md)（P/X）：基于所有合格 prompt 的刷新策略。
- [`session/session-telemetry`](../../../packages/session/session-telemetry/README.md)（D/X）：从 Session/Agent 生命周期生成中立 telemetry。
- [`session/session-telemetry-otel`](../../../packages/session/session-telemetry-otel/README.md)（P）：OpenTelemetry exporter。

## Session Query

- [`session-query/session-query`](../../../packages/session-query/session-query/README.md)（D）：session corpus、bounded read、lineage、event relationship 与过滤。
- [`session-query/session-query-sqlite`](../../../packages/session-query/session-query-sqlite/README.md)（P）：SQLite/FTS 查询 Provider。
- [`session-query/tool-session-query`](../../../packages/session-query/tool-session-query/README.md)（C）：模型检索历史 Session 工具。
- [`session-query/session-log-export`](../../../packages/session-query/session-log-export/README.md)（C/X）：会话日志导出。

## Attachment 与 Spill

- [`attachment/attachment`](../../../packages/attachment/attachment/README.md)（D）：durable attachment identity、metadata 和内容验证。
- [`attachment/attachment-local`](../../../packages/attachment/attachment-local/README.md)（P）：本地 content-addressed attachment 存储。

## Storage

- [`storage/storage`](../../../packages/storage/storage/README.md)（D）：非 Session 数据的通用 storage hub。
- [`storage/storage-domain`](../../../packages/storage/storage-domain/README.md)（X）：按领域命名/隔离 storage。
- [`storage/storage-json`](../../../packages/storage/storage-json/README.md)（P）：JSON 文件 backend。
- [`storage/storage-sqlite`](../../../packages/storage/storage-sqlite/README.md)（P）：SQLite backend。

## Settings、Credentials 与 Workspace

- [`settings/settings`](../../../packages/settings/settings/README.md)（D）：validated user settings namespace/section。
- [`settings/settings-file`](../../../packages/settings/settings-file/README.md)（P）：文件持久设置 Provider 与原子写。
- [`credentials/credentials`](../../../packages/credentials/credentials/README.md)（D）：credential reference 解析能力；公共配置只携带 ref。
- [`credentials/credentials-local`](../../../packages/credentials/credentials-local/README.md)（P）：环境变量优先于 `.env` 的本地 Provider。
- [`workspace/workspace`](../../../packages/workspace/workspace/README.md)（D/X）：Workspace 实体、identity 和路径语义，供 context/host/UI 共享。
