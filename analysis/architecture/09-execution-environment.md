# 执行环境：FS、Subprocess、Shell、Terminal、Sandbox 与 LSP

## FS 和 Subprocess 必须指向同一个执行环境

`fs` 决定工具看到哪些文件；`subprocess` 决定进程在哪里启动；`sandbox` 限制进程权限；`shell` 把用户或模型请求转换为可执行 spec；`terminal` 管理长时间运行的 PTY；`lsp` 在相同文件和进程环境中启动语言服务器。Bundle 必须为这些能力选择指向同一环境的 Provider。

本地组合通常是 `fs-local + subprocess-local + sandbox-local/policy + bash-local`；E2B 组合替换 FS 和 subprocess，使高层工具保持不变。设计重点不是统一所有 API，而是让共享环境身份的能力一起替换。

## Filesystem

[`dsh-fs`](../../packages/fs/fs/src/index.ts)定义文件操作和 `fs/*` policy events。`fs-local` 实现本地访问，`fs-sandbox` 将路径映射到 sandbox 能力，`fs-observation-policy` 追踪模型已经观察过的内容，防止编辑工具基于未读旧状态写入。`tool-fs`、`tool-fs-search` 和 `tool-str-replace-editor` 是不同交互粒度的 Consumer。

路径安全不能只靠工具 schema：Provider 和 policy listener 在最终操作处解析 workspace、允许根、符号链接和访问模式。观察政策是语义安全而非 OS confinement；真正进程权限由 sandbox 层处理。

## Subprocess 与 Sandbox

[`dsh-subprocess`](../../packages/subprocess/subprocess/src/index.ts)定义 spawn、等待、signal 和 process-tree cleanup；`subprocess-local` 负责平台差异和后代进程回收。Sandbox 不自己执行命令，而是把 argv 包装为 bwrap/Landlock/Seatbelt/Windows ACL 等受限启动，或声明本地直通。

这样包装启动命令后，shell、terminal、LSP 和其他 subprocess Consumer 都使用同一套进程权限限制。但它只能限制通过 `dsh-subprocess` 启动的进程；直接调用 Node `spawn` 会绕过限制，因此包依赖规则和代码审查必须禁止这种调用。Windows ACL backend 和 Unix wrapper 提供的保证并不完全相同，权限政策只能承诺当前平台实际支持的限制。

## Shell

`shell/shell` 将 model request 与 resolved execution spec 分开，Provider 在 resolve 阶段物化 cwd、environment 和 executable。`bash-local`/`pwsh-local` 通过 subprocess 执行，sandbox variants 选择受限环境，`shell-env` 负责环境组合。`tool-bash-persistent` 并不是简单加长 timeout，而是把命令路由到持久 terminal。

结果保留使用统一 output-retention，确保 stdout/stderr、包装说明和 metadata 的完整结果满足字节/行限制。限制在 chunk 层会遗漏多 chunk 合计和多字节字符。

## Terminal

[`terminal`](../../packages/terminal/terminal/src/index.ts)按创建者管理 terminal session；`terminal-bash` 管理 PTY、输入、增量读取和退出；`tool-terminal` 提供创建、写入、读取和关闭操作。Terminal 通常属于创建它的 Agent scope。销毁 Agent 时必须先停止 loop，再关闭 terminal，避免工具继续使用已经卸载的服务。

持久终端保存的是实时资源，不进入 Session Log；模型只通过 tool call/result 得到可重建观察。恢复一个 Session 不会复活旧 PTY，这是日志事实与外部资源生命周期的明确分界。

## LSP

`lsp` 定义 server registry 与请求语义，`lsp-stdio` 通过 subprocess 启动标准输入输出协议，`tool-lsp` 提供模型操作。LSP 文档位置、workspace root 和文件内容必须与 FS Provider 一致，否则模型编辑远端文件却查询本地语言服务器。

## 失败与释放

进程/PTY/LSP 都采用“请求取消不等于资源已经停止”。Owner 先发 signal，再等待退出或执行升级终止，最后撤销 registry。相关防御规则见[防御模式](../../docs/defensive-patterns.md)。这比 fire-and-forget cleanup 成本高，但避免 HMR、Agent dispose 或 timeout 后留下孤儿进程。
