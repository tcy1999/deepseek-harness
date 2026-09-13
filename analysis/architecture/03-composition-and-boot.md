# 组合与启动

## Profile 和 Bundle

Profile 是用户拥有的部署组合；bundle 是可分发 patch 层。Profile 的 `package.json#dsh.profile.bundles` 给出有序 bundle 列表，bundle 的 `package.json#dsh.bundle.patch` 指向自己的 patch。实现位于 [`profile.ts`](../../packages/boot/app-boot/src/profile.ts)。

组合从空 entry 列表开始，依次应用 bundle、profile 的 `cordis.patch.yml`、home 级和命令行 overlay。Patch 按 row id 寻址，目标 row 的 `config` 是整块替换而非深合并，因此上层必须重述完整配置。这刻意让最终配置可读且不会偷偷继承下层新增字段，但提高了升级时配置漂移风险；`--dump-config` 是检查实际启动树的权威诊断。

`dsh-base` 提供所有 profile 共用的模型、工具、会话、执行环境、设置和遥测骨架；`dsh-web-app` 与 `dsh-headless` 添加不同入口。bundle 的 TypeScript `index.ts` 很薄，真正产品组合在 [`base/cordis.patch.yml`](../../packages/bundle/base/cordis.patch.yml)、[`web-app/cordis.patch.yml`](../../packages/bundle/web-app/cordis.patch.yml)和 [`headless/cordis.patch.yml`](../../packages/bundle/headless/cordis.patch.yml)。分析部署时应先读 patch，而不是只读包入口。

## CLI 到 Loader

[`apps/cli/src/bin.ts`](../../apps/cli/src/bin.ts)解析命令后按分支动态加载 run、dump-config 或 plugin 管理逻辑。`app-boot` 负责 `.env`、Harness home、profile 解析、patch 合成、Loader guard、路径解析和等待插件树 settle，见 [`packages/boot/app-boot/src/index.ts`](../../packages/boot/app-boot/src/index.ts)。

Loader 的 `baseUrl` 指向 profile 目录，使 out-of-tree plugin 能作为 profile 依赖解析；in-box bundle 优先从 dsh 安装位置解析，再回退 profile。这个双锚点同时满足官方 bundle 的版本一致性和用户插件扩展。一个标为 bundle 却没有 `dsh.bundle` manifest 的包会在加载时失败，不会被当成空层跳过。

## 配置决定实际安装哪些插件

`cordis.yml` 的 entry 表示插件树，`config` 和 `disabled` 可使用 `!!js`，其他 metadata 保持字面值。裸包名必须出现在对应 resolver manifest 的 dependencies 中，`verify-cordis-config` 防止源码环境能解析而打包安装后失败。

只根据配置就能发现的问题会在加载时直接报错，例如重复 Agent identity、`toolOrder` 重名或缺少 `<unlisted-tools>`，以及缺失 bundle manifest。依赖运行时注册信息的问题则在第一次能够检查时报错：`toolOrder` 引用的工具是否存在，要等 prompt assembly 得到当前 scope 的已注册工具全集后才能验证。

## 热重载

长生命周期入口监视 profile patch 并让 Loader reconcile entry 树。正确 HMR 依赖所有贡献可撤销、异步任务可停止、服务替换不保留旧 listener。这里 Cordis effect 模型与产品配置模型相互依赖：patch 只描述目标树，fiber disposal 才真正清理旧树。

## 设计取舍

通过配置选择插件后，同一组 package 可以组成 headless、Web、测试或用户定制产品。Patch 按固定顺序应用，同一 row 的 config 整块替换，因此最终值来自哪一层可以确定。代价是单看入口文件无法知道实际行为；还要查看 patch 顺序、row id 和完整 config。仓库提供 config catalog、`dump-config`、真实 Loader 组合测试和 manifest 检查来发现配置错误。
