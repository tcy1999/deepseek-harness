# Package 指南

本指南覆盖当前 `packages/*/*` 下的全部 package。条目按架构职责分组，不复制 README 的完整 API；每个名称链接到包目录中的 README，括号内给出角色，后面给出源码关注点。

- [核心运行时](core.md)：`core`、LLM、Typert 与 runtime diagnostics。
- [执行能力](runtime-capabilities.md)：FS、进程、Shell、Terminal、Sandbox、LSP、Web、代码执行、spill 和 E2B。
- [Agent 能力](agent-capabilities.md)：上下文、技能、压缩、目标、计划、任务、工作流、交互和扩展。
- [Session 与存储](session-and-storage.md)：持久化、投影、查询、附件、设置、凭据、storage 和 workspace。
- [集成与协议](integration-and-protocols.md)：boot、bundle、API、SDK、ACP、hooks、MCP、subagent 和 examples。
- [Web 与 Client](web-and-client.md)：host 与全部浏览器插件。
- [支持与工具库](support-and-utilities.md)：测试支撑和零依赖 util。

## 角色缩写

| 标记 | 含义 |
|---|---|
| D | Service Definition 或公共协议 |
| P | Provider/具体后端 |
| C | Consumer（工具、命令、UI） |
| X | Composition/桥接/横切 policy |
| S | Support/纯工具库 |

一个包可以承担多个角色，但完整能力要在 bundle 或应用组合里闭合。定位实现时通常先读 `src/index.ts`，再搜索 `register`、`ctx.on/waterfall/effect`、`SessionEventMap` 和 package tests；组合包应优先读 `cordis.patch.yml`。
