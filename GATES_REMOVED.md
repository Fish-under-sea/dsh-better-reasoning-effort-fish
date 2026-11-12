# 已移除的质量门禁清单

> 移除日期：2026-11-12。按维护者要求临时移除，待后续统一重新添加。
> 最后包含完整门禁的提交：`95397f1`（可从该提交恢复下列文件）。

## 本次移除

| 位置 | 移除内容 | 影响 |
|------|----------|------|
| `.github/workflows/ci.yml` | 整个 workflow 删除 | 不再在 PR / push `master` 时运行 Node 22.19.0 与 Node 24 矩阵的 `npm ci`、`npm run typecheck`、`npm test`、`npm run build` |
| `.github/workflows/publish.yml` | 删除 `Typecheck`、`Test` 两个步骤 | tag 发布只安装依赖、构建、发布；不再先跑类型检查与测试 |
| `package.json` | 删除 `check:styles` script，并从 `build:client` 去掉 `npm run check:styles &&` | 构建不再检查 `STYLES` / `RUNNER_STYLES` 模板字符串里的反引号 |
| `scripts/check-styles.mjs` | 文件删除 | 反引号门禁脚本不再存在 |

**保留的 `publish.yml` 步骤**：`Build` 仍然是打包必需步骤，不是质量门禁。发布命令使用 `npm publish --ignore-scripts`，`lib/` 必须事先构建，否则 tarball 不带产物。

**仍保留的手动命令**：`npm run typecheck`、`npm test`、`npm run build` 脚本没有删除，仍可本地手动运行，只是不再被 CI 或发布流程强制执行。

## 同时移除的深色模式分支（本次视觉改动）

| 位置 | 移除内容 |
|------|----------|
| `src/client/runner-styles.ts` | 169 处 `body:not([data-ds-dark-theme]) ` 选择器前缀；浅色规则改为无条件生效，滑块八款主题统一使用浅色配色 |
| `src/client/styles.ts` | 1 处 `body:not([data-ds-dark-theme])` reduced-motion 选择器前缀 |
| `src/client/radiation.ts` | `document.body.hasAttribute('data-ds-dark-theme')` 判断；固定使用浅色原子配色（`isDark = false`） |
| `src/client/ComposerSlider.tsx` | 监听 `data-ds-dark-theme` 的 `MutationObserver` 与清理逻辑 |

深色基础配色块仍留在 `runner-styles.ts` 中作为底层声明，但会被后面无条件生效的浅色规则覆盖。重新添加深色模式时，应从这些基础块恢复统一的门禁/选择器，而不是逐条补回。

## 重新添加时的检查清单

- [ ] 从 `95397f1` 恢复 `.github/workflows/ci.yml`：`git show 95397f1:.github/workflows/ci.yml > .github/workflows/ci.yml`
- [ ] 在 `publish.yml` 的 `Install dependencies` 与 `Build` 之间加回 `Typecheck`、`Test` 步骤
- [ ] 从 `95397f1` 恢复 `scripts/check-styles.mjs`，并在 `package.json` 加回 `check:styles` script 与 `build:client` 里的调用
- [ ] 统一重新设计深色模式门禁（`runner-styles.ts`、`styles.ts`、`radiation.ts`、`ComposerSlider.tsx`），不要只恢复其中一部分
- [ ] 恢复后跑一次完整 `npm run typecheck`、`npm test`、`npm run build`，再更新本文件
