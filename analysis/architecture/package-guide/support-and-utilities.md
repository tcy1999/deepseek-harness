# 测试支撑与 Utilities

## Test Support

- [`test-support/session-snapshot`](../../../packages/test-support/session-snapshot/README.md)（S）：ACP/headless keyless transcript harness 与 fixture replay。
- [`test-support/agent-loop-testkit`](../../../packages/test-support/agent-loop-testkit/README.md)（S）：Agent Loop 单元/集成测试构造和等待辅助；不能替代真实 Loader composition。
- [`test-support/client-runtime`](../../../packages/test-support/client-runtime/README.md)（S）：浏览器 Cordis/UI 测试 runtime。
- [`test-support/llm-mock-server`](../../../packages/test-support/llm-mock-server/README.md)（S）：可控模型 wire mock server。
- [`test-support/llm-replay`](../../../packages/test-support/llm-replay/README.md)（P/S）：按录制内容实现 `llm/stream` replay。
- [`test-support/loader-smoke`](../../../packages/test-support/loader-smoke/README.md)（S）：从真实 `cordis.yml` 启动 Loader 的组合 smoke。

## Utilities

- [`util/atomic-write`](../../../packages/util/atomic-write/README.md)（S）：临时文件、flush/rename 等原子替换原语。
- [`util/brand`](../../../packages/util/brand/README.md)（S）：零运行时开销的 `Branded<B>` opaque id。
- [`util/home-paths`](../../../packages/util/home-paths/README.md)（S）：Harness home 与平台路径解析。
- [`util/launch-environment`](../../../packages/util/launch-environment/README.md)（S）：子进程启动环境构造。
- [`util/native-command`](../../../packages/util/native-command/README.md)（S）：跨平台 native command/argv 解析。
- [`util/output-retention`](../../../packages/util/output-retention/README.md)（S）：对完整输出执行字节/行保留和截断。
- [`util/timeout`](../../../packages/util/timeout/README.md)（S）：可取消 timeout/deadline 原语。

这些包刻意保持 Harness 依赖少或为零。若 utility 开始知道 Agent、Session 或 Cordis scope，通常说明行为应回到 owning capability package。
