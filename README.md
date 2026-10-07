<div align="center">

# dsh-better-reasoning-effort-fish

**第三方模型的思考强度与输入模态，直接改在官方「模型」页的编辑卡里**

[![npm](https://img.shields.io/npm/v/dsh-better-reasoning-effort-fish?style=flat-square&label=npm&color=cb3837)](https://www.npmjs.com/package/dsh-better-reasoning-effort-fish)
![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![node](https://img.shields.io/badge/node-%5E22.19.0%20%7C%7C%20%3E%3D24-339933?style=flat-square)
![DSH](https://img.shields.io/badge/DSH-%E2%89%A5%200.1.5%2Dalpha.1-4b6ef6?style=flat-square)
![plugin](https://img.shields.io/badge/plugin-client%20%2B%20host-6b7280?style=flat-square)

<img src="icon.svg" alt="dsh-better-reasoning-effort-fish" width="96">

**简体中文** · [English](README.en.md)

</div>

## 原作者与授权（请先读）

| 项目 | 内容 |
|------|------|
| 插件名 | **dsh-better-reasoning-effort** —— 为第三方模型声明思考强度与输入模态 |
| 原作者 | **HaoyueQin** · GitHub [@HaoyueQin](https://github.com/HaoyueQin) |
| 原仓库 | <https://github.com/HaoyueQin/dsh-better-reasoning-effort> |
| 上游 npm 包名 | `dsh-better-reasoning-effort`（**归原作者，本版不能沿用**） |
| 本版 npm 包名 | **`dsh-better-reasoning-effort-fish`** |
| 许可证 | **MIT**，版权归原作者（本仓库 [`LICENSE`](LICENSE) **未做任何修改**） |
| 本版基线 | 上游 `master` = **v0.5.2**（commit `52b584c`） |
| 本版性质 | **个人客制化版（fork）**，非原创、非官方；上游 README 完整保留为 [`README.original.md`](README.original.md) / [`README_ZH.original.md`](README_ZH.original.md) |

> 署名与改动范围的完整说明见 [`NOTICE.md`](NOTICE.md)。再分发时请保留原作者署名。

## 解决什么问题

`llm-pi-ai` 适配器原生支持每模型声明 `reasoningEfforts`（思考强度）与 `input`（输入模态），但官方「模型」页编辑卡刻意不暴露这两个字段。结果：第三方模型在 Composer 里**没有思考档位选择器**，只有官方 DeepSeek API 能设思考强度，手工声明的模型被当作**纯文本**，想配置只能手写 `settings.yaml` 块。

本插件把这两份配置能力搬回 UI：在官方模型编辑卡内直接编辑，加一键自动适配（内置模型知识库 + 线协议推断）。

| 能力 | 说明 |
|------|------|
| **官方页内注入** | 官方模型行展开区出现「选项」编辑块（思考强度、输入模态、端点兼容），随卡片自身的**保存**统一提交，**取消**则一起丢弃 |
| **自动适配** | 一键填入推荐档位、线上取值与模态：内置知识库（15 家厂商 65 个条目）+ 线协议推断 + 同源 `/models` 探测，每条建议标注置信度 |
| **自动填充** | 启动时补齐未声明模型的推荐声明，会话中新增的也会补写（`autofill: false` / `modalityAutofill: false` 可关） |
| **三种意图** | 全不勾 = 取消声明（回到继承）；只勾 off = 禁用推理；勾选档位 = 写入声明 |
| **Composer 思考强度滑块** | 官方模型菜单弹出体换成上游风格的档位滑块（拖动 / 键盘，乐观提交、被拒回滚），右下角触发钮保持原样 |
| **每模型默认思考强度** | 模型行上的「默认思考强度」选择器，写进设置文档，新会话打开该模型即用它 |
| **请求头与 User-Agent** | 提供商卡片内编辑官方 `headers` 字段（掩码显示、路径合并），并按 origin 精确接管 fetch 层的 `user-agent`；同源 `/models` 探测一并覆盖 |
| **防御式注入** | 一切锚定官方页 DOM；官方升级改变结构时注入自动暂停，官方页不受影响 |

**本版（本 Fork）相对上游的改动：让出官方模型能力面板的 `settings.models.provider-card` 席位，避免与 dsh-web 的模型能力插件冲突。** 详见[兼容与边界](#兼容与边界)。

## 效果

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/banner-zh-dark.svg">
    <img src="docs/banner-zh.svg" alt="DSH Better Reasoning Effort" width="720">
  </picture>
</p>

<p align="center">
  <img src="assets/models-page-effort-editor.png" alt="官方「模型」页模型行展开区内注入的思考强度编辑器" width="720">
</p>

## 安装

需要 DeepSeek Harness **`0.1.5-alpha.1` 或更高**（按 peerDependencies 范围），当前对照 `0.2.0-rc.2` 编译与门禁。Node 要求 `^22.19.0 || >=24`。

从 npm 安装（推荐）：

```sh
# 桌面版
dsh plugin --profile desktop add dsh-better-reasoning-effort-fish

# Web 版
dsh plugin --profile web add dsh-better-reasoning-effort-fish
```

链接本地检出（改源码用）：

```sh
npm install && npm run build
dsh plugin --profile desktop add link:/path/to/dsh-better-reasoning-effort-fish
dsh plugin --profile web add link:/path/to/dsh-better-reasoning-effort-fish
```

安装后**重启 DSH** 并强制刷新浏览器。

> DSH 内置 pnpm 的 `minimum-release-age` 为 24 小时：版本发布后一天内的安装会解析到「足够旧的最新版」。写明版本号可立刻安装，例如 `dsh-better-reasoning-effort-fish@0.5.5`。

## 使用

1. 在官方「模型」页配置第三方供应商（API Key 等）。
2. 展开某个模型行：官方容量字段下方是编辑块。
   - 勾选档位（off / minimal / low / medium / high / xhigh / max），填线上取值（如给 `high` 填 `ultra`，Composer 选 High 时网关收到 `ultra`）；
   - 在「输入模态」区勾选**图片输入**，声明模型接受什么；
   - 点「自动适配」填推荐档位与模态 —— 参考容量以只读提示出现，可自行照抄进官方输入框；
   - 改动**即时进入待写入**，点卡片自身的**保存**时一并落盘；**取消**（或刷新）则与卡片字段一起丢弃。
3. 协议兼容时，底部会出现「端点兼容」分区 —— `openai-completions` 上设思考预算字段 / vLLM 优先级，`openai-responses` 上设 `max_output_tokens` 的处理方式。
4. 请求头：展开**提供商卡片**，编辑块上方就是请求头区域（掩码显示、随卡片保存），可填 `user-agent` 等自定义请求头。
5. 全不勾 + 保存 = 取消声明（回到继承）；只勾 off + 保存 = 禁用推理（`false`）；模态行「清除声明」+ 保存 = 回到继承提供方默认。

声明后的模型在 Composer 里立即可选思考强度；声明了图片输入的模型可以端到端传附件。

## 配置

host 侧接受以下可选配置项（写入 `settings.yaml` 的插件 `config` 块）：

| 配置键 | 默认值 | 可选值 | 说明 |
|--------|--------|--------|------|
| `autofill` | `true` | `true` / `false` | 启动时自动填充未声明模型的推荐声明 |
| `modalityAutofill` | `true` | `true` / `false` | 上述填充是否连带输入模态声明 |
| `probeTimeoutMs` | `15000` | 正整数 | `/models` 探测请求超时（毫秒） |
| `bootRetryDelaysMs` | `[1000, 2000, 4000, 8000, 16000, 30000]` | 正整数数组 | 启动时自动填充失败后的重试延迟（毫秒） |
| `defaultGuard` | `true` | `true` / `false` | 强制思考梯子上的无档位调用落厂商默认档，而非发送 `thinking: disabled` |

示例：

```yaml
- insert:
    - id: dsh-better-reasoning-effort-fish
      name: dsh-better-reasoning-effort-fish
      config:
        autofill: true
        modalityAutofill: true
        probeTimeoutMs: 15000
        bootRetryDelaysMs: [1000, 2000, 4000, 8000, 16000, 30000]
        defaultGuard: true
```

## 兼容与边界

**本 Fork 只做一处修复**：请求头编辑器不再占用官方 `settings.models.provider-card` 席位，改挂官方卡片自身的编辑器容器。

原因是官方该席位为 **keyed slot** —— 注册表对同一 `(key, priority)` 只接受一个条目，第二个注册直接抛错。dsh-web 全家桶的 `@linxin666/dsh-client-ui-model-capabilities`（模型能力面板）注册的正是同一个 `llm-pi-ai` key、同一个默认优先级 `0`。两者都占时只有一个能注册成功，另一个的整块界面**静默消失**。因 dsh-web 走异步壳行加载，实际被挤掉的是**模型能力面板**（每模型思考强度 / 图片输入）。

| | 修复前 | 修复后 |
|---|---|---|
| `settings.models.provider-card` 席位 | 本插件抢占 | **完整让给 `model-capabilities`** |
| 模型能力面板 | 静默消失 | 正常显示 |
| 请求头编辑器 | 在卡片内（占席位） | **仍在卡片内** —— 改挂卡片自己的编辑器容器 |

改动落在 [`src/client/injection/provider-card.ts`](src/client/injection/provider-card.ts)（新增）、[`src/client/index.ts`](src/client/index.ts)（不再注册席位）与 [`src/client/injection/models-page.ts`](src/client/injection/models-page.ts)（接入页面扫描）。回归防护见 [`tests/client.spec.tsx`](tests/client.spec.tsx) 与 [`tests/provider-card.spec.tsx`](tests/provider-card.spec.tsx)。

两者的插件 id、host 路由前缀（`/dsh-better-reasoning-effort-fish/*`）与包名都已分开，可在同一 profile 并存；实际使用建议**只装一个**，避免同一份设置被两套 UI 同时编辑。

Composer 滑块**改编自 [HanaAyane 的 dsh-reasoning-effort](https://github.com/HanaAyane/dsh-reasoning-effort)**（MIT）。如果用过上游那个插件，请先移除以免同一席位出现两个档位控件：

```sh
dsh plugin --profile web remove dsh-reasoning-effort
```

**已知限制**

- 注入依赖官方「模型」页的 DOM（aria-label / class）；官方升级可能让注入暂停直至适配 —— 期间官方页不受影响。
- 自动适配探测路由只应答**回环与 IP 字面量 host**，且**从不跟随重定向** —— 只在 30x 后面列模型的网关拿不到端点证据，自动适配回退到知识库与协议推断。
- `reasoningEfforts` 声明是建议 —— 端点真正接受什么以它的文档为准，请在 UI 里微调。
- 端点兼容开关刻意永不自动填充：它们描述的是网关行为而非模型能力。
- 模态词汇跟随 pi-ai 核心（当前 `text` / `image`）；更宽的网关支持（PDF / 音频 / 视频）在核心词汇扩充前声明不了，这是设计使然。
- 命名启发式的模态建议（视觉风味 id）刻意标注低置信度，使用前请核对。
- 自建中转：对没有官方 host 认领的路由，自动填充会钉 `supportsDeveloperRole: false`；显式值永不被覆盖。
- 强制思考模型（无 `off` 的梯子，如 GLM-5.3）：无档位调用自动落厂商默认档而不是发 `thinking: disabled` —— 设 `defaultGuard: false` 可恢复原行为。
- **`headers` 中的凭据在磁盘上不脱敏**：只读视图会掩码，但设置文档仍明文保存 —— 请当 API key 对待。
- **请求头区域依赖官方卡片的编辑器容器**：官方若改变该结构，请求头区域会停止出现 —— 绝不弄坏官方页面。这也是本版相对上游的取舍：不再使用官方席位，换来与 dsh-web 插件共存。
- **同一时间只应有一个 `user-agent` 改写器**：同类 header 插件落在同一层，后写者赢；插件会检测并提示已知同类，但不覆盖未知情况。
- 请求层接管依赖官方适配器每请求新建 SDK 客户端 —— 有端到端测试守护该边界，变化时会响亮地失败而不是静默失效。

更多兼容性细节见 [`docs/compatibility-notes.md`](docs/compatibility-notes.md)，支持的模型列表见 [`docs/supported-models.md`](docs/supported-models.md)。

## 开发与测试

```sh
npm run typecheck   # tsc 严格检查
npm test            # vitest：知识库 / 推断 / 自动填充 / DOM 注入 / 写入
npm run build       # lib/*.js + lib/client.js
```

测试由 vitest 驱动，需先 `npm install` 安装依赖（全新 clone 不含 `node_modules`），再 `npm test` 运行；构建用 `npm run build`（`lib/` 为构建产物，已被 gitignore，clone 里不含）。

本机在 DSH 文件沙箱下开发时，`vitest` 与 esbuild 需要额外处理（沙箱禁止子进程管道）—— 完整命令见 [`FORK.md`](FORK.md)。

## 与聚合包的关系

本包是**独立仓库**（<https://github.com/Fish-under-sea/dsh-better-reasoning-effort-fish>），不在 `dsh-fish` 单仓内；但它是聚合包 [`@fish-under-sea/dsh-fish`](https://github.com/Fish-under-sea/dsh-fish) 的成员之一，可与聚合包内其他插件一起安装。

## 许可

[MIT](LICENSE)，著作权归原作者 **HaoyueQin**；本版新增代码同样以 MIT 发布，再分发请保留 [`LICENSE`](LICENSE) 与 [`NOTICE.md`](NOTICE.md)。
