# 来源与署名（NOTICE）

本仓库是 **dsh-better-reasoning-effort** 插件的**个人客制化版（fork）**，**不是原创项目**。

## 原始项目

| 项目 | 内容 |
|------|------|
| 插件名 | **dsh-better-reasoning-effort** —— 为第三方模型声明思考强度（reasoning effort）与输入模态 |
| 原作者 | **HaoyueQin** |
| GitHub | [@HaoyueQin](https://github.com/HaoyueQin) |
| 原仓库 | <https://github.com/HaoyueQin/dsh-better-reasoning-effort> |
| 上游 npm 包名 | `dsh-better-reasoning-effort` |
| 许可证 | **MIT** |
| 本版基线 | 上游 `master` = **v0.5.2**（commit `52b584c`） |

## 本版（Fish-under-sea/dsh-better-reasoning-effort-fish）做了什么

只做**一处修复**，未改动上游的思考强度编辑、自动适配、自动填充、Composer 滑块、每模型默认档与请求头接管逻辑：

**把官方 `settings.models.provider-card` 席位完整让出，请求头编辑器改挂官方卡片自身的编辑器容器。**

原因是官方该席位为 **keyed slot**：注册表对同一 `(key, priority)` 只接受一个条目，第二个注册直接抛错。dsh-web 全家桶的 `@linxin666/dsh-client-ui-model-capabilities`（模型能力面板）注册的正是同一个 `llm-pi-ai` key、同一个默认优先级 `0`，于是两者只有一个能注册成功，另一个的整块界面**静默消失**（失败方用 `try/catch` 吞掉了异常，界面上没有任何报错）。因 dsh-web 聚合包走异步壳行加载，实际被挤掉的是**模型能力面板**（每模型思考强度 / 图片输入）。

- 新增 `src/client/injection/provider-card.ts`：锚定官方动作行 `[class*="editorActions"]` 的父元素（该容器只在卡片被编辑时渲染，故「容器存在」即「卡片打开」），并做三级 route 解析（新建卡片 → 官方 route 标签 → 卡片标题）。
- 改动 `src/client/index.ts`：不再注册该席位。
- 改动 `src/client/injection/models-page.ts`：把卡片注入接入页面扫描与 fiber teardown。
- 删除上游的 `src/client/injection/provider-card-slot.ts`（席位版实现）。
- 测试：`tests/provider-card.spec.tsx`〔新增 11 条〕；`tests/client.spec.tsx` 改为断言该席位**未被注册**。

## 原作者的权利

- 插件全部功能与代码的著作权归**原作者**所有。
- 本仓库**完整保留**上游 `LICENSE` 原文与版权声明，未做任何修改。
- 上游原始 README 完整保留为 [`README.original.md`](README.original.md) 与 [`README_ZH.original.md`](README_ZH.original.md)，未做任何修改。
- 本版新增代码同样以 **MIT** 发布；再分发时请保留本文件与上游署名。

## 身份字段的改动（必须三处同步）

`package.json` 只改身份与元数据字段（`name` / `version` / `description` / `author` / `repository` / `homepage` / `bugs` / `keywords` / `files` / `publishConfig`）；`peerDependencies`、`devDependencies`、构建与测试脚本保持上游不变。

**包名牵动三处，必须一致，否则插件加载失败（`loaded without registering`）：**

| 位置 | 值 |
|------|-----|
| `package.json` 的 `name` | `dsh-better-reasoning-effort-fish` |
| `cordis.patch.yml` 插件行的 `name` 与 `id` | `dsh-better-reasoning-effort-fish` |
| `src/constants.ts` 的 `PLUGIN_ID` | `dsh-better-reasoning-effort-fish` |

同时把 host 侧的路由前缀从 `/dsh-better-reasoning-effort/*` 改为 `/dsh-better-reasoning-effort-fish/*`（`PROBE_PATH` / `AUTOFILL_CONFIG_PATH` / `HEADERS_CONFIG_PATH`），使本版与上游**可以并存而不撞车**；实际使用仍建议只装一个。

浏览器端的滑块开关偏好键刻意**保留上游的 `dsh-better-reasoning-effort.slider.enabled`**，这样从上游切到本版时偏好延续；若两者并存，该开关会同步（两者本就是同一个开关的语义）。