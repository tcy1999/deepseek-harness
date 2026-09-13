# 工具怎样注册和执行

## 同一份工具定义用于生成 Prompt 和执行调用

[`ToolRuntime`](../../packages/core/tools/src/index.ts)保存定义、scope、schema、执行 body、并发分类、结果 finalizer 和 UI presentation。模型看到的是当前 Agent scope 的 schema；执行时再次按同一 scope 解析定义，不能把“prompt 中隐藏”当权限控制。

通过 `defineTool()` 注册的工具会在执行 body 前验证模型参数，并根据声明的 output schema 验证 canonical value；直接注册原始 `ToolDefinition` 的工具仍要自行验证输入，但同样必须声明 registry-enforced output。Registry 会把调用参数快照为 lossless JSON 并冻结，防止调用后对象突变改变日志或 observer 看到的事实。

工具自己声明 UI presentation。Call view 只有 `generic`、`terminal`、`diff` 三种 card；`locations` 是 `generic`/`diff` call 上的可选文件定位 metadata，不是第四种 card。Result view 除对应的三种 card 外还支持 `search`、`read` 和 `web`。展示层按 tagged union 渲染，不从结果文本或工具名猜测语义。

## 有序执行阶段

```text
createExecution / JSON snapshot / presentation-mode denial
  -> tools/pre-execute waterfall
  -> optional approval ask
  -> monotonic guards
  -> scheduler reservation
  -> tools/execute waterfall
  -> definition body
  -> tools/post-execute waterfall
  -> definition finalizeContent
  -> immutable final notification/result
```

`tools/pre-execute` 是可扩展政策，之后的 monotonic guard 只能增加拒绝，不能被后续 listener 重新允许。真正安全决策必须在执行路径强制，schema omission 或 prompt filtering 只改善模型体验。Pipeline invariant 检查每次 execution 的阶段顺序。

所有异常被物化为 `ToolExecutionResult`，而不是从 executor 抛到 Agent Loop；unknown tool、invalid args、listener failure 和 body failure 因此都能成为模型可见 tool result。Pipeline 自身仍区分还需 post-process 的结果与已经 final 的结果，避免一个阶段失败后错误地继续调用 listener。

## 并发调度

工具定义可根据参数判断 `isConcurrencySafe`。只有明确返回 `true` 才会并行；没有定义、判断报错、工具未知或工具隐藏时都按互斥调用处理。Agent Loop 的 `maxParallelToolCalls` 限制并行数量；互斥调用开始前要等待已经运行的并行调用结束。这个分类只决定调度方式，工具仍要自己保护共享文件、进程或其他资源。

## 取消后仍要等待已经开始的工具结束

调用 body 前取消会产生 `ABORTED_BEFORE_DISPATCH`。body 已经开始时，执行器会发送取消信号并等待 Promise 结束，然后把成功结果改为 `ABORTED`；工具自己返回的结构化错误仍可保留。这样 turn 结束后不会有旧工具继续写文件或发布事件。Around wrapper 如果替换 signal，Registry 会把新旧两个 signal 合并，确保调用方仍能取消。

## Code Mode

Code presentation mode 把多个直接工具折叠为唯一 `run_code` transport。Registry 在执行入口再次执行 collapse：模型直接调用被折叠工具会在审批和 policy 之前拒绝，nested sub-dispatch 才可绕过。判断使用 Agent 的有效 mode 而不是部署默认，避免 preset Agent 宣告 code mode 却仍能直调 native tool。实现集中在 `collapses()` 和 `createExecution()`。

## 哪些工具调用信息写入 Session Log

Agent Loop 为 assistant tool call 写调用事实，执行后写 model-facing result content；直接顶层调用还可把 `presentationMeta(args, value)` 产生的 JSON metadata 一并持久化，供重放时重建 result card。Canonical tool `value` 只在本次执行内供 Code Mode binding 和 presenter 使用，不写入 Session，也不能从重放恢复。Code-mode 子调用用独立 dispatch 事件记录最终 content/isError，但不会再次进入模型 history，也不计算 presentation metadata。

结果过大时 retention/spill policy 在完整结果大小已知处处理，不能只限制 body 的原始文本。工具还可 `deferContext()`，把后续上下文作为明确输入进入下一 step，而不是修改当前请求。
