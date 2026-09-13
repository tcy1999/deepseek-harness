# 执行与外部能力 Packages

## Filesystem

- [`fs/fs`](../../../packages/fs/fs/README.md)（D）：文件操作接口与 `fs/*` policy events。
- [`fs/fs-local`](../../../packages/fs/fs-local/README.md)（P）：本地文件系统 Provider。
- [`fs/fs-observation-policy`](../../../packages/fs/fs-observation-policy/README.md)（X）：跟踪 Agent 已观察内容，约束基于旧/未读状态的写入。
- [`fs/fs-sandbox`](../../../packages/fs/fs-sandbox/README.md)（P/X）：将 FS 请求映射到 sandbox 执行世界。
- [`fs/tool-fs`](../../../packages/fs/tool-fs/README.md)（C）：基础文件读写模型工具。
- [`fs/tool-fs-search`](../../../packages/fs/tool-fs-search/README.md)（C）：使用 shell/搜索后端的文件发现工具。
- [`fs/tool-str-replace-editor`](../../../packages/fs/tool-str-replace-editor/README.md)（C）：基于精确字符串替换的编辑工具，依赖观察和写入政策。

## Subprocess、Sandbox 与 Shell

- [`subprocess/subprocess`](../../../packages/subprocess/subprocess/README.md)（D）：进程启动、signal、等待和 process-tree 生命周期。
- [`subprocess/subprocess-local`](../../../packages/subprocess/subprocess-local/README.md)（P）：本地进程与平台化后代回收。
- [`sandbox/sandbox`](../../../packages/sandbox/sandbox/README.md)（D）：进程 confinement request/spec 接口。
- [`sandbox/sandbox-local`](../../../packages/sandbox/sandbox-local/README.md)（P）：本机 bwrap/Landlock/Seatbelt 等 Provider/选择逻辑。
- [`sandbox/sandbox-policy`](../../../packages/sandbox/sandbox-policy/README.md)（X）：把部署安全政策应用到 spawn 路径。
- [`sandbox/sandbox-windows-acl`](../../../packages/sandbox/sandbox-windows-acl/README.md)（P）：Windows ACL confinement Provider。
- [`shell/shell`](../../../packages/shell/shell/README.md)（D）：shell backend registry、request/spec 显式解析。
- [`shell/shell-env`](../../../packages/shell/shell-env/README.md)（X）：环境变量组合与执行环境解析。
- [`shell/bash-local`](../../../packages/shell/bash-local/README.md)（P）：经 subprocess seam 的本地 Bash。
- [`shell/bash-sandbox`](../../../packages/shell/bash-sandbox/README.md)（P/X）：受 sandbox policy 约束的 Bash。
- [`shell/pwsh-local`](../../../packages/shell/pwsh-local/README.md)（P）：本地 PowerShell。
- [`shell/pwsh-sandbox`](../../../packages/shell/pwsh-sandbox/README.md)（P/X）：受限 PowerShell。
- [`shell/tool-bash`](../../../packages/shell/tool-bash/README.md)（C）：一次性 Bash 模型工具。
- [`shell/tool-bash-persistent`](../../../packages/shell/tool-bash-persistent/README.md)（C）：将长生命周期命令接入 terminal 的 Bash 工具。
- [`shell/tool-pwsh`](../../../packages/shell/tool-pwsh/README.md)（C）：PowerShell 模型工具。

## Terminal 与 LSP

- [`terminal/terminal`](../../../packages/terminal/terminal/README.md)（D）：owner-scoped persistent session registry。
- [`terminal/terminal-bash`](../../../packages/terminal/terminal-bash/README.md)（P）：Bash PTY 创建、输入、增量输出与退出管理。
- [`terminal/tool-terminal`](../../../packages/terminal/tool-terminal/README.md)（C）：终端创建、写入、读取、关闭工具。
- [`lsp/lsp`](../../../packages/lsp/lsp/README.md)（D）：语言服务器 registry 与请求协议。
- [`lsp/lsp-stdio`](../../../packages/lsp/lsp-stdio/README.md)（P）：通过 subprocess stdio 驱动 LSP。
- [`lsp/tool-lsp`](../../../packages/lsp/tool-lsp/README.md)（C）：模型 LSP 操作。

## Web 能力

- [`web/web`](../../../packages/web/web/README.md)（D）：搜索/抓取能力 registry 和公共结果模型。
- [`web/web-fetch-http`](../../../packages/web/web-fetch-http/README.md)（P）：HTTP fetch Provider 与响应边界处理。
- [`web/web-search-deepseek`](../../../packages/web/web-search-deepseek/README.md)（P）：DeepSeek 搜索 Provider。
- [`web/web-search-exa`](../../../packages/web/web-search-exa/README.md)（P）：Exa 搜索 Provider。
- [`web/web-search-perplexity`](../../../packages/web/web-search-perplexity/README.md)（P）：Perplexity 搜索 Provider。
- [`web/tool-web`](../../../packages/web/tool-web/README.md)（C）：模型搜索与抓取工具。

## Code Runtime、Spill 与 E2B

- [`code-runtime/code-runtime`](../../../packages/code-runtime/code-runtime/README.md)（D/C）：代码执行能力与 Code Mode bridge。
- [`code-runtime/code-runtime-worker-thread`](../../../packages/code-runtime/code-runtime-worker-thread/README.md)（P）：worker thread 执行 Provider；worker 消息是需验证的 wire boundary。
- [`spill/spill`](../../../packages/spill/spill/README.md)（D）：大结果外置存储能力。
- [`spill/spill-local`](../../../packages/spill/spill-local/README.md)（P）：本地 spill 存储。
- [`spill/spill-policy`](../../../packages/spill/spill-policy/README.md)（X）：在完整工具结果处决定外置并返回引用。
- [`e2b/e2b`](../../../packages/e2b/e2b/README.md)（P/S）：E2B sandbox 连接与共享资源。
- [`e2b/fs-e2b`](../../../packages/e2b/fs-e2b/README.md)（P）：E2B 文件系统 Provider。
- [`e2b/subprocess-e2b`](../../../packages/e2b/subprocess-e2b/README.md)（P）：E2B subprocess Provider。
