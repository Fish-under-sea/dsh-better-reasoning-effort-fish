# 本 fork 的维护说明

本仓库是 **dsh-better-reasoning-effort 的个人客制化版（fork）**，不是原创或官方发行版。当前基线为 [HaoyueQin/dsh-better-reasoning-effort](https://github.com/HaoyueQin/dsh-better-reasoning-effort) `master` **v0.5.4**，固定 commit **`2d68b65cc641cd85e95384c859a2bc3de23d9e08`**；前一基线为 v0.5.2 `52b584c`。

改动范围、素材来源、MIT 许可及身份字段约束见 **[NOTICE.md](NOTICE.md)**；完整用户说明见 [中文 README](README.md) / [English README](README.en.md)。

## 跟随上游：按基线 diff 移植

本仓库是**没有共同历史的独立仓库**，不能照搬 `git merge upstream/master`；也不应通过 `--allow-unrelated-histories` 将上游全树合并进来。应取得固定提交，审阅旧基线到新基线的 diff，再逐路径移植，同时保留本 Fork 差异。不要把「上游当前 master」的后续变化混入本次固定同步。

以下仅作维护命令示例，`upstream` remote 已存在则跳过添加；不要求切分支或提交：

```bash
git remote add upstream https://github.com/HaoyueQin/dsh-better-reasoning-effort.git
git fetch upstream

# 本次同步已采用的变更范围
git diff --stat 52b584c 2d68b65cc641cd85e95384c859a2bc3de23d9e08
git diff 52b584c 2d68b65cc641cd85e95384c859a2bc3de23d9e08 -- src tests scripts package.json
```

不要把上述 diff 整体覆盖应用到工作树：逐文件核对交叠改动，将上游行为移入本地实现。下一次同步应从本次 **`2d68b65cc641cd85e95384c859a2bc3de23d9e08`** 到新的固定提交计算 diff，而不是重新从 `52b584c` 起算。

### 必须保留的 Fork 差异

| 路径 / 范围 | 本版约束 |
|-------------|----------|
| `src/client/injection/provider-card-slot.ts` | 保持不使用该席位实现，不恢复对 `settings.models.provider-card` 的注册 |
| `src/client/injection/provider-card.ts` | 请求头编辑器继续挂官方卡片编辑器容器，保留 route 解析与 teardown |
| `src/client/index.ts` / `injection/models-page.ts` | 保留不占席位的装配与扫描接入 |
| `src/constants.ts` | Fish 插件 id；`PROBE_PATH` / `AUTOFILL_CONFIG_PATH` / `HEADERS_CONFIG_PATH` 使用 `/dsh-better-reasoning-effort-fish/*` |
| `package.json` / `cordis.patch.yml` / `src/constants.ts` | 包名与插件行 `name` 一致为 `@fish-under-sea/dsh-better-reasoning-effort-fish`；行 `id` 与 `PLUGIN_ID` 保持 `dsh-better-reasoning-effort-fish`；元数据保留 Fish 身份 |
| `src/client/ComposerSlider.tsx` / `model-theme.ts` / `radiation.ts` / `runner-styles.ts` | 保留八角色、深浅色、辐射效果、拖动加速与 reduced-motion 静态路径；模型 id / name 优先、provider 兜底、未知 DeepSeek |
| Composer 滑块偏好 | 不加角色开关、不强制开启；保留整个滑块的 `dsh-better-reasoning-effort.slider.enabled` 偏好 |
| `scripts/build-client.mjs` / `src/client/assets.d.ts` / `assets/` | 八张运行 PNG 经 esbuild `dataurl` 内嵌，不引入静态资源路由；保留完整第三方 MIT 许可 |
| 六份根目录说明文档 | 用户 README 保持 Fish 说明；两份 `original` 备份保持固定上游提交原文与身份 |

本次同步还包括官方 `modelOverrides` 目录安全写入、未知模型不自动填充，以及默认档守护与活动报告。DSH peer 范围新增 `^0.2.1-alpha.1`、DSH devDependencies 对齐上游 `^0.2.1-alpha.1`；本次发布版本为 `0.5.10`，不要误改为上游包名或版本 `0.5.4`。依赖声明对齐不等于本 Fork 已在真实 DSH 验证。

### 原始 README 备份

每次更新基线时，从**同一固定提交**原样更新两份署名备份；不将其改写成 Fish 身份，不向原文插入 Fork 说明：

```bash
git show 2d68b65:README.md > README.original.md
git show 2d68b65:README.zh.md > README_ZH.original.md
```

原文内相对链接也保留原样；它们是来源快照而不是本 Fork 的用户入口。

## 视觉来源与资源

视觉与相关代码来自 [HanaAyane/dsh-reasoning-effort PR #9](https://github.com/HanaAyane/dsh-reasoning-effort/pull/9)，贡献者 [xihucuyudaichi](https://github.com/xihucuyudaichi)，固定 commit [`651d24edc1ff45fbce5339700b40f32c5ff3ddea`](https://github.com/HanaAyane/dsh-reasoning-effort/commit/651d24edc1ff45fbce5339700b40f32c5ff3ddea)，不是浮动跟随该仓库的 master。

- 八款主题：DeepSeek / OpenAI / Claude / Gemini / Kimi / GLM / Qwen / MiniMax。
- `assets/readme/all-themes.jpg` 是素材 PR 的参考预览，**不是本插件真实 DSH 运行截图**。
- `assets/third-party/dsh-reasoning-effort.LICENSE` 完整保留来源 MIT 版权、授权及免责条款；不得只保留链接或短句替代完整许可。
- `assets/` 的八张 `*-runner-strip.png` 运行素材经 `dataurl` 内嵌进客户端 bundle；不要把参考预览当成运行资源，也不要另开资源路由。

## 发布记录：0.5.10

`0.5.10` 是包名改为 `@fish-under-sea/dsh-better-reasoning-effort-fish` 后的首个发布版本，内容为上游 v0.5.4 同步与八主题功能。

- **npm**：`@fish-under-sea/dsh-better-reasoning-effort-fish@0.5.10`。未加 scope 的旧包 `0.5.9` 已由作者删除；scoped 包首次发布使用 `npm stage publish` 提交暂存队列，由维护者在 npm 网页完成 2FA 审批后上线。
- **GitHub Release**：`v0.5.10`，资产为同一 commit 构建的 `fish-under-sea-dsh-better-reasoning-effort-fish-0.5.10.tgz`，可直接用于 `dsh plugin add <tarball URL>`。`npm pack` 对 scoped 包会去掉 `@` 并把 `/` 换成 `-`，`release.yml` 的附件 glob 已同步。
- **历史版本**：`v0.5.9` 与 `v0.5.8` 的 tag 与 release 保留为历史，不移动、不覆盖；其中 `v0.5.9` 是未加 scope 的版本，npm 侧已删除，不再使用。
- **后续 tag 发布**：`publish.yml` 使用 npm Trusted Publishing。scoped 包需要先在 npm 网页重新配置 GitHub Actions 发布者（仓库名 + `publish.yml`）；未配置前，新的 tag 发布会失败，应继续在本机完成首次发布。`publish.yml` 对已经存在于 registry 的版本会跳过重复发布，避免本机发布后 tag 推送把 CI 标红。
- **已知展示问题**：npm 页面 README 取的是根目录 `README.original.md`（npm 的 `{README,README.*}` 选取顺序），因此页面安装示例仍可能显示上游包名；`npm install` 内容不受影响。修正需把两份 `original` 备份移出根目录（不再匹配 `README.*`）后发布新版本。

发布不改变验证边界：本地 `typecheck`、`npm test`（24 个测试文件、518 项）与 `build` 已通过，浏览器交互与真实 DSH 运行仍未验证。

## 本机开发备注（DSH 文件沙箱）

质量门禁已在 2026-11-12 按维护者要求临时移除：CI 工作流、`publish.yml` 的 typecheck/test、构建里的 `check:styles` 以及滑块的深色模式分支。完整清单与恢复步骤见 [`GATES_REMOVED.md`](GATES_REMOVED.md)；`npm run typecheck`、`npm test`、`npm run build` 仍可手动运行。

以下为既有开发环境的排障参考，不是本次已执行或要求自动执行的步骤。遇到权限限制应遵循当前环境审批，不绕过沙箱。

- 在曾出现 `EPERM` 的环境中，vite 在 Windows 探测网络驱动器的 `exec("net use")` 可能被阻止。仓库提供的幂等补丁只修改本地 `node_modules`，重装依赖后需重跑：

  ```bash
  node scripts/sandbox-vite-patch.mjs
  ```

- 若默认 fork 池的 IPC 管道被限制，可改用线程池：

  ```bash
  node node_modules/vitest/vitest.mjs run --pool=threads
  ```

- esbuild 需要启动自身二进制服务并通过管道通信；如被拦，应先通过环境批准的方式取得执行权限。常规构建入口：

  ```bash
  npm run build
  ```

构建中的 `build:client` 包含样式检查；不要为了绕开环境问题跳过项目已有检查。`lib/` 为 gitignore 中的构建产物，源码 clone 不自带。
