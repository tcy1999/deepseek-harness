# 可替换能力：Definition、Provider、Consumer

## 仓库为什么称它为“能力接缝”

“能力接缝”是仓库对一组可替换组件的专用名称，不是泛指抽象接口。Service Definition 声明操作；Provider 实现操作；Consumer 通过这些操作提供模型工具、命令、UI 或另一项服务。一项能力至少要有这三个角色，bundle 再选择实际安装的 Provider。完整关系见[官方能力图](../../docs/capability-seams.md)。

例如 shell：`shell/shell` 定义 request/spec 与 backend registry，`bash-local`、`bash-sandbox`、`pwsh-*` 是 Provider，`tool-bash`、`tool-pwsh` 是模型 Consumer。Bundle 决定装哪个 Provider。若工具包直接 spawn，它既绕过 subprocess/sandbox，又使远程执行需要 fork 工具实现。

## Request/Spec 分离

部署可变默认值在 Provider 或 owning implementation 的 `resolve(request): spec` 阶段显式物化，`run(spec)` 不用隐藏的 `?? default`。这让实际 Provider invocation、诊断和测试面对一个完整 spec，也让不同 Provider 对缺省值的解释有明确 owner；但 Tool Runtime 的日志和通用 `ApprovalRequest` 不会自动保存这个 spec。某个 resolved 字段若必须进入审计或审批，Consumer 要在相应边界前物化并明确传递，不能仅依赖 body 内部的 `resolve()`。

## 为什么分成多个包

只有这些角色需要独立替换时才分包。工具只依赖 Definition；本地 Provider 可以依赖 Node；E2B Provider 可以依赖远端 SDK；Web UI 只依赖公开类型。负责装配的 bundle 是唯一需要同时知道 Definition、Provider 和 Consumer 的包。简单且不会单独替换的能力可以把三个角色放在同一个包中。

## 替换 Provider 会影响什么

FS 与 subprocess Provider 共同定义执行世界；shell Provider 消费 subprocess，terminal 和 LSP 也建立在相同底层能力上。把这两项指向远程 sandbox，就能移动多个高层 Consumer，无需为每个工具增加 `-e2b` 分支。执行环境专题见[09](09-execution-environment.md)。

Subagent 也使用同一分工：Consumer 始终是 delegation tools；Provider 可以在进程内 fork、spawn 新 Agent，经 SDK/ACP 连接另一运行时，或委托给 Claude Code/Codex。共同接口只承诺如何提交任务、停止任务和读取结果，不要求所有 Provider 使用相同进程模型。

## 设计检查

设计一项新的可替换能力时，应回答：

- 是否存在至少一个当前 Provider 和 Consumer，而非为未来预留？
- Definition 是否只含所有 Consumer 需要的语义，没有工具 schema、Loader 或 UI 字段？
- 默认值由能解释它的 owner 显式解析了吗？
- Provider 错误是否在公共失败模型中可表达？
- 注册和替换能否撤销；已经开始的操作继续使用旧 Provider，还是切换到新 Provider？
- policy 在最终执行点强制了吗？

接口过宽会让所有 Consumer 都依赖某个 Provider 的细节；接口过窄会迫使 Consumer 绕开接口直接调用实现。仓库根据当前 Consumer 的实际需求定义接口，并用 package README、subsystem reference 和 contract test 记录和验证这些要求。
