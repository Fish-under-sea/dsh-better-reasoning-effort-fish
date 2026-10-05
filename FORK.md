# 本 fork 的维护说明

本仓库是 **dsh-better-reasoning-effort 的上游客制化版（fork）**：基线为上游 `master` = **v0.5.2**（commit `52b584c`）。

改动范围、署名、身份字段（包名 / 插件行 / `PLUGIN_ID` 三处同步）的完整说明见 **[NOTICE.md](NOTICE.md)**；本文件只留维护者需要的东西。

## 跟随上游

```bash
git remote add upstream https://github.com/HaoyueQin/dsh-better-reasoning-effort.git   # 首次
git fetch upstream && git merge upstream/master
```

合并时留意三处本版已改、上游仍在维护的地方：

| 路径 | 本版的差异 |
|------|-----------|
| `src/client/injection/provider-card-slot.ts` | **已删除** —— 本版不再占用官方 `settings.models.provider-card` 席位 |
| `src/client/injection/provider-card.ts` | **新增** —— 请求头编辑器改挂官方卡片的编辑器容器 |
| `src/client/index.ts` / `injection/models-page.ts` | 装配段与扫描接入相应改动 |
| `src/constants.ts` | 插件 id 与三个 host 路由前缀都已带 `-fish`（与上游并存不撞车） |

身份字段（`package.json` 的 `name`、`cordis.patch.yml` 的行名、`src/constants.ts` 的 `PLUGIN_ID`）以本版为准；上游若改动它们，合并时取本版值。

## 发布

```bash
npm run build                                  # 生成 lib/（host + client + 类型）
npm publish --access public --ignore-scripts   # 本机首发；--ignore-scripts 跳过 prepare 的重复构建
git tag v0.5.3 && git push origin v0.5.3       # 之后交给 .github/workflows/publish.yml
```

CI 走 npm 的 **Trusted Publishing（OIDC）**，仓库里不存 token；但**首次走 CI 前**需要在本包的 npm 页面配置一次发布者（Settings → Trusted Publisher → GitHub Actions，填 `Fish-under-sea/-dsh-better-reasoning-effort-fish` 与 workflow 名 `publish.yml`）。

## 本机开发备注（DSH 文件沙箱）

本机 DSH 运行在文件沙箱下，**禁止子进程通过管道捕获输出**（`EPERM`），因此：

- `vitest` 启动即失败 —— vite 在 Windows 上无条件执行一次 `exec("net use")` 探测网络驱动器映射，该调用的同步 `EPERM` 落在它自己的 `try` 块之外。先跑一次幂等补丁脚本（只改本地 `node_modules`，重装依赖后需重跑）：

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