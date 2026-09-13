# 测试、构建与静态检查

## 源码检查和发布物检查不能混用

静态检查和多数测试通过 tsconfig paths 读取 `src`，必须在没有旧构建产物时也能通过。验证发布内容的测试则先 build，再用普通 Node 读取 `lib`。如果混在一起，本地测试可能误读旧 `lib`，也可能漏掉 package exports 错误。

Host 和 Client 分别使用不同的 TypeScript 编译配置。同时包含两种程序的包使用 `tsconfig.host.json` 与 `tsconfig.client.json`，根配置只汇总引用；仅有一种程序的包使用自己的叶配置。Client 不能引入只在 Node 中可用的依赖。`tsdown` 生成运行时代码，`tsc -b` 检查 TypeScript 项目依赖图。

## 测试层级

| 层级 | 证明内容 | 不足以替代 |
|---|---|---|
| Unit/Vitest | 纯逻辑、输入边界、状态机 | Loader 组合和完整产品输出 |
| Contract test | 多个 Provider 遵守同一 Definition | 实际 bundle 配置 |
| REAL composition | Loader 从真实 `cordis.yml` 装载服务 | 模型或协议输出 |
| Snapshot | 不调用真实模型的完整应用输出 | Provider 的真实 API |
| E2E | 真实 DeepSeek/provider 行为 | 无 key CI 的确定性覆盖 |
| Built smoke | exports、NodeNext、打包安装 | 源码逻辑全覆盖 |

用户或模型能看到的插件必须有真实 Loader 组合测试，不能只有单元测试。模型可见行为变化还要更新不调用真实模型的 snapshot，并通过可运行 example 生成。Mock 只替换外部服务或不确定输入，不能手工组装 Context 来冒充 Loader 路径。

## Coverage 与属性测试

CI 用 `test:coverage` 检查覆盖率，要求 `packages/*/*/src` 每个文件达到 100%；普通 `test` 不检查这项要求。Session、JSON、repair 等代码使用 fast-check/property test 生成多种事件序列和边界输入。覆盖率只能说明代码被执行过，不能说明 bundle 配置正确，因此 package 仍要有真实 Loader 测试。

## 生成文档也参与一致性检查

工具、配置、Cordis API、module graph、persistence catalog 和 client catalog 都从源码生成。检查脚本会在生成结果过期时失败。这样不必手写容易过期的清单，但公开 JSDoc 也会直接影响生成文档。`type-equiv` 检查文档中的类型片段是否与源码声明一致。

## 运行时检查和测试各自发现什么

测试验证预先写出的场景。Runtime invariant 在实际运行时检查事件和状态之间的关系，例如模型请求是否等于 Session Log 算出的消息。它用于发现某种插件组合破坏了这些关系，不重复单元测试中的固定样例。静态检查确认声明 companion 的 package 正确导出 invariant，未声明的包解释省略原因；发布物检查验证实际导出。

## 为什么不默认全跑

仓库包含多个平台、真实 API、网站和生成器，因此本地检查要对应实际改动：行为改动运行相关测试；模型输出改动运行 snapshot；文档改动运行 `doc-sync`；发布内容改动运行 build、hygiene 和 built smoke。CI 负责完整覆盖率和各平台检查。开发者仍要判断一次改动影响了哪些包和输出。

## 文档检查

`doc-sync` 检查正式文档和 package README 的链接、段落物理行、字数预算、生成文档、双语配对、JSDoc 和依赖图是否最新。`analysis/` 是独立分析，不在当前 `verify-md-links`、`verify-md-wrap` 和 doc typecheck 的 glob 中，因此它使用相同的一段一行与相对链接规则，但需要额外对 `analysis/**/*.md` 运行链接、fragment 和段落检查；仅有一次通过的 `doc-sync` 不能证明本目录有效。
