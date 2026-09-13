# 安全与正确性机制

## Prompt 和工具列表不能代替权限检查

隐藏工具名、在系统提示中禁止操作，或者不向模型展示某个 schema，都不能真正阻止调用。权限必须在 `tools/pre-execute`、最终拒绝检查、FS policy、sandbox wrapper 或 Provider 操作中执行，并覆盖 SDK、nested dispatch 等其他调用路径。UI permission preset 只负责选择政策，真正的允许或拒绝仍在执行路径发生。

## 审批与权限

`user-approval` 定义异步审批能力，`permission-presets` 把用户选择映射到工具政策，`tool-ask-user` 与 `user-questions` 处理模型主动询问。工具 pipeline 的 `ask` 决策暂停 dispatch；取消审批必须与 caller abort 合并，不能在用户拒绝后继续 body。

通用 `ApprovalRequest` 只携带 Agent、工具名、可选 call id、reason 和 signal，不重复传递参数或任意执行上下文。UI 用 call id 把问题附到已经展示的、由 Registry 快照并冻结的工具调用；Provider 后续补出的默认 cwd、timeout 或其他 resolved spec 不会自动进入审批 payload。Sandbox escalation 的 reason 另外明确包含目标 mode 和模型给出的 justification。Secret 不应进入工具展示、reason、审批 audit 或错误文本。

## Sandbox 与文件政策

Sandbox 约束子进程 OS 权限，FS provider/policy 约束 API 文件访问，observation policy约束编辑前置观察。三层解决不同威胁，不能互相替代。Native confinement 还受平台能力限制；“sandbox enabled”不等于所有平台同等隔离。

## 信任边界

同一进程内、已经由 TypeScript 类型约束的值不重复做运行时验证。来自 JSON、配置、文件、持久日志、worker、其他进程、网络协议和模型工具参数的数据必须验证；验证失败要明确报错。跨这些入口记录参数或结果时，系统使用可无损往返的 JSON 快照。

## 运行时检查事件和状态的关系

每个 package 提供 `./invariant`，由 diagnostics runtime 安装。有效 invariant 检查 owned relationship，例如：loop-built request 等于 Session 派生消息；tool execution 阶段有序；turn/step 嵌套合法；registry notification 与数据一致。纯示例或方法存在性不是运行 invariant。

Invariant failure 说明系统内部契约已被破坏，应尽早暴露而非降级。Package 没有合理运行关系时必须给出 package-specific 空说明，防止形式化占位。

## 创建、取消和释放规则

状态只能在提交后对外发布。一次异步操作只有一个对象负责开始、取消和等待结束。收到取消后要等待操作真正停止；listener 的错误不能逃出其负责范围；资源按创建的相反顺序释放；启动尚未完成时收到 dispose，也要等启动工作结束并清理结果。Agent factory 的 `FactoryOwnership`、工具 signal fusion 和 persistence write-behind 都遵守这些规则。详情见[防御模式参考](../../docs/defensive-patterns.md)。

## 类型与身份

跨 package/wire opaque id 使用 `Branded<B>`，closed union switch 用 `assertNever`；merge-extensible union 使用有说明的 default。类型不能防御外部数据，但能防止同进程把 SessionId、JobId、call id 混用。

## 主要剩余风险

- 插件顺序影响 waterfall，错误组合可能形成政策空隙；
- 第三方插件与 host 同进程，Cordis scope 不是安全隔离；
- sandbox 能力受 OS/Provider 实现限制；
- 模型摘要和外部 subagent 不能提供无损语义；
- 事件和注册分散使人工 review 容易漏掉替代调用路径。

项目通过默认 bundle、真实组合测试、运行时关系检查、snapshot 和静态依赖检查降低风险，但第三方插件与 host 在同一进程运行，部署者仍要判断是否信任它们及其 Provider。
