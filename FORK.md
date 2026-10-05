# 本 fork 的说明（相对上游 0.5.2）

本机维护的 **dsh-better-reasoning-effort** fork，基线来自上游 `master`（0.5.2，
commit `52b584c`）。相对上游只有**一处必要改动**，其余与上游一致。

## 改动一：请求头编辑器让出官方席位

官方「模型」页的 `settings.models.provider-card` 是 **keyed slot**：注册表对同一
`(key, priority)` 只接受一个条目，第二个注册会直接抛错（`@deepseek-ai/dsh-client-ui-slots`
的 `SlotCore.register`）。dsh-web 全家桶里的 `@linxin666/dsh-client-ui-model-capabilities`
（模型能力面板）注册的正是同一个 key `llm-pi-ai`、同一个默认优先级 `0`。

原实现的请求头编辑器（issue #12）也占这个席位，于是：

- 两者只有一个能注册成功，**另一个的整块 UI 静默消失**（失败方通常还用自己的
  `try/catch` 吞掉了异常，界面上没有任何报错）；
- dsh-web 聚合包走的是**异步壳行**（`await import()` 之后才 `ctx.plugin`），滑块是直接
  插件行，因此实际赢家是滑块 —— 被挤掉的是**模型能力面板**（每模型思考强度 / 图片输入），
  也就是「单独设置模型参数」那块界面。

现在：

- `src/client/index.ts` **不再注册**该席位，席位完整留给 `model-capabilities`；
- 请求头编辑器改由 [`src/client/injection/provider-card.ts`](src/client/injection/provider-card.ts)
  挂进官方卡片**自己的编辑器容器**：锚点是官方动作行 `[class*="editorActions"]` 的父元素，
  该容器只在卡片被编辑时渲染，所以「容器存在」就等于「卡片已打开」——不需要 `_editor`
  类名探测、不需要自己的 MutationObserver、也不需要 `data-edit` 显隐开关；
- route 解析（`routeOfProviderCard`）按可靠性排序：新建卡片（携带 `Provider ID` 字段）
  直接拒绝 → 官方 route 标签 → 卡片标题（仅在标签被隐藏时，且必须是 route key 形状）；
- 两侧功能因此都保留：**模型能力面板回来了，请求头编辑器仍在卡片里**。

回归防护落在测试里：`tests/client.spec.tsx` 断言该席位**没有被注册**，
`tests/provider-card.spec.tsx` 覆盖卡片内挂载、幂等、卡片收起后的卸载、route 变化重渲染
与三种 route 解析分支。

> 上游仍在维护 `src/client/injection/provider-card-slot.ts`；本 fork 已删除该文件并接管
> 相应的装配段，因此跟随上游合并时这里会冲突 —— 预期如此，保留本 fork 的行为即可。

## 跟随上游

```bash
git remote add upstream https://github.com/HaoyueQin/dsh-better-reasoning-effort.git   # 首次
git fetch upstream && git merge upstream/master
```

## 本机开发备注（DSH 文件沙箱）

本机 DSH 运行在文件沙箱下，**禁止子进程通过管道捕获输出**（`EPERM`），因此：

- `vitest` 启动即失败 —— vite 在 Windows 上无条件执行一次 `exec("net use")` 探测网络
  驱动器映射，该调用的同步 `EPERM` 落在它自己的 `try` 块之外；
  先跑一次幂等补丁脚本（只改本地 `node_modules`，重装依赖后需重跑）：

  ```bash
  node scripts/sandbox-vite-patch.mjs
  ```

- vitest 默认的 fork 池同样被拦（IPC 走管道），测试请用线程池：

  ```bash
  node node_modules/vitest/vitest.mjs run --pool=threads
  ```

- 构建（esbuild 需启动自身二进制服务并通过管道通信）需在放宽沙箱下执行：

  ```bash
  node scripts/build-host.mjs && node scripts/build-client.mjs && npx tsc -p tsconfig.build.json
  ```

英文版 `README.md` 未同步本节内容，差异只记录在这份中文说明里。