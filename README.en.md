<div align="center">

# dsh-better-reasoning-effort-fish

**Edit reasoning effort and input modalities for third-party models right inside the official Models page card in DSH**

[![npm](https://img.shields.io/npm/v/dsh-better-reasoning-effort-fish?style=flat-square&label=npm&color=cb3837)](https://www.npmjs.com/package/dsh-better-reasoning-effort-fish)
![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![node](https://img.shields.io/badge/node-%5E22.19.0%20%7C%7C%20%3E%3D24-339933?style=flat-square)
![DSH](https://img.shields.io/badge/DSH-%E2%89%A5%200.1.5%2Dalpha.1-4b6ef6?style=flat-square)
![plugin](https://img.shields.io/badge/plugin-client%20%2B%20host-6b7280?style=flat-square)

<img src="icon.svg" alt="dsh-better-reasoning-effort-fish" width="96">

[简体中文](README.md) · **English**

</div>

## Original Author & Attribution (Please Read First)

| Item | Details |
|------|---------|
| Plugin name | **dsh-better-reasoning-effort** — declares reasoning effort and input modalities for third-party models |
| Original author | **HaoyueQin** · GitHub [@HaoyueQin](https://github.com/HaoyueQin) |
| Original repository | <https://github.com/HaoyueQin/dsh-better-reasoning-effort> |
| Upstream npm package | `dsh-better-reasoning-effort` (**belongs to the original author; this fork cannot reuse the name**) |
| This fork's npm package | **`dsh-better-reasoning-effort-fish`** |
| License | **MIT**, copyright held by the original author (the [`LICENSE`](LICENSE) file in this repo **is unmodified**) |
| Fork baseline | Upstream `master` = **v0.5.2** (commit `52b584c`) |
| Nature of this fork | **Personal customization (fork)** — neither original nor official; the upstream README is preserved intact as [`README.original.md`](README.original.md) / [`README_ZH.original.md`](README_ZH.original.md) |

> Full details on attribution and the scope of changes are in [`NOTICE.md`](NOTICE.md). Please preserve the original author's credit when redistributing.

## What Problem Does This Solve

The `llm-pi-ai` adapter natively supports per-model `reasoningEfforts` and `input` declarations, but the official Models page editor deliberately keeps both fields out of reach. As a result, third-party models get **no thinking-level picker** in the Composer — only the official DeepSeek API can set reasoning effort, hand-declared models are treated as **text-only**, and configuring any of this meant hand-writing `settings.yaml` blocks.

This plugin brings both configuration surfaces back into the UI: edit them inside the official model editor card, with one-click auto-adapt (built-in model knowledge base + wire-protocol inference).

| Capability | Description |
|------------|-------------|
| **In-page editor** | An "Options" block appears in each model row's disclosure on the official Models page (reasoning effort, input modalities, endpoint compatibility); changes commit with the card's own **Save** and are discarded together on **Cancel** |
| **Auto-adapt** | One click fills recommended levels, wire spellings and modalities: built-in knowledge base (65 entries across 15 vendors) + wire-protocol inference + same-origin `/models` probe; every suggestion is labeled by confidence |
| **Auto-fill** | Models without a declaration receive a recommended one at boot, and newly added models mid-session are filled in too (disable with `autofill: false` / `modalityAutofill: false`) |
| **Three intents** | All levels off = unset (back to inheritance); only `off` checked = disable reasoning; levels armed = write the declaration |
| **Composer effort slider** | The official model menu popup is replaced with an upstream-style level slider (drag / keyboard, optimistic commit with rollback); the bottom-right trigger button is untouched |
| **Per-model default effort** | A "Default effort" picker on each model row, stored in the settings document; every new session starts the model at that level |
| **Request headers & User-Agent** | Edit the official `headers` field inside the provider card (masked display, path-merged), with per-origin fetch-layer `user-agent` takeover; same-origin `/models` probes are covered |
| **Defensive injection** | Everything keys off the official page's DOM; if an official upgrade changes the structure, injection pauses automatically — the official page is unaffected |

**What this fork changes from upstream: it yields the `settings.models.provider-card` slot so it no longer conflicts with dsh-web's model-capabilities plugin.** See [Compatibility & Boundaries](#compatibility--boundaries).

## In Action

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/banner-zh-dark.svg">
    <img src="docs/banner-zh.svg" alt="DSH Better Reasoning Effort" width="720">
  </picture>
</p>

<p align="center">
  <img src="assets/models-page-effort-editor.png" alt="The thinking-effort editor injected into a model row on the official Models page" width="720">
</p>

## Installation

Requires DeepSeek Harness **`0.1.5-alpha.1` or later** (per the peerDependencies range); compiled and gated against `0.2.0-rc.2`. Node requirement: `^22.19.0 || >=24`.

Install from npm (recommended):

```sh
# Desktop profile
dsh plugin --profile desktop add dsh-better-reasoning-effort-fish

# Web profile
dsh plugin --profile web add dsh-better-reasoning-effort-fish
```

Link a local checkout (for source changes):

```sh
npm install && npm run build
dsh plugin --profile desktop add link:/path/to/dsh-better-reasoning-effort-fish
dsh plugin --profile web add link:/path/to/dsh-better-reasoning-effort-fish
```

**Restart DSH** and hard-refresh the browser after installation.

> DSH bundles pnpm with a `minimum-release-age` of 24 hours: installs within a day of a release resolve to the newest version old enough. Pin the version to install immediately, e.g. `dsh-better-reasoning-effort-fish@0.5.5`.

## Usage

1. Configure a third-party provider (API key, etc.) on the official Models page.
2. Expand a model row: the editor block sits below the official capacity fields.
   - Check levels (off / minimal / low / medium / high / xhigh / max) and fill wire values (e.g. give `high` the spelling `ultra` — the gateway receives `ultra` when you pick High in the Composer);
   - Toggle **Image input** under *Input modalities* to declare what the model accepts;
   - Click **Auto-adapt** to fill recommended levels and modalities — reference capacities appear as read-only hints you can copy into the official fields;
   - Changes are **pending** immediately and land when you press the card's own **Save**; **Cancel** (or a reload) discards them along with the card's fields.
3. On a compatible protocol, an *Endpoint compatibility* section appears at the bottom — thinking budget field / vLLM priority on `openai-completions`, `max_output_tokens` handling on `openai-responses`.
4. Request headers: expand the **provider card**; the header area sits above the editor block (masked display, saved with the card) — fill in `user-agent` and other custom headers.
5. All levels off + Save = unset the declaration (back to inheritance); only `off` checked + Save = disable reasoning (`false`); *Clear declaration* on the modality row + Save = back to inheriting the provider default.

Declared models are immediately selectable for reasoning effort in the Composer; models with image input declared accept attachments end to end.

## Configuration

The host side accepts the following optional config keys (written to the plugin's `config` block in `settings.yaml`):

| Key | Default | Values | Description |
|-----|---------|--------|-------------|
| `autofill` | `true` | `true` / `false` | Auto-fill undeclared models with recommended declarations at boot |
| `modalityAutofill` | `true` | `true` / `false` | Whether the boot fill also covers input-modality declarations |
| `probeTimeoutMs` | `15000` | positive integer | `/models` probe request timeout (ms) |
| `bootRetryDelaysMs` | `[1000, 2000, 4000, 8000, 16000, 30000]` | positive integer array | Retry delays after boot auto-fill failures (ms) |
| `defaultGuard` | `true` | `true` / `false` | Map effort-less calls on forced-thinking ladders to the vendor default instead of sending `thinking: disabled` |

Example:

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

## Compatibility & Boundaries

**This fork makes exactly one fix**: the request-header editor no longer occupies the official `settings.models.provider-card` slot — it mounts on the card's own editor container instead.

The reason: the official slot is a **keyed slot** — the registry accepts only one entry per `(key, priority)` pair, and a second registration throws. The dsh-web suite's `@linxin666/dsh-client-ui-model-capabilities` (model-capabilities panel) registers the same `llm-pi-ai` key at the same default priority `0`. When both plugins occupy the slot, only one registers successfully; the other's entire UI **silently disappears**. Because dsh-web loads via an async shell row, the loser is the **model-capabilities panel** (per-model reasoning effort / image input).

| | Before fix | After fix |
|---|---|---|
| `settings.models.provider-card` slot | Occupied by this plugin | **Fully yielded to `model-capabilities`** |
| Model-capabilities panel | Silently disappeared | Displays normally |
| Request-header editor | In the card (occupying the slot) | **Still in the card** — mounted on the card's own editor container |

Changes land in [`src/client/injection/provider-card.ts`](src/client/injection/provider-card.ts) (new), [`src/client/index.ts`](src/client/index.ts) (no longer registers the slot), and [`src/client/injection/models-page.ts`](src/client/injection/models-page.ts) (wired into page scanning). Regression guards live in [`tests/client.spec.tsx`](tests/client.spec.tsx) and [`tests/provider-card.spec.tsx`](tests/provider-card.spec.tsx).

The two plugins have separate plugin ids, host route prefixes (`/dsh-better-reasoning-effort-fish/*`), and package names — they can coexist in one profile. In practice, **install only one** to avoid the same settings being edited by two UIs simultaneously.

The Composer slider is **adapted from [HanaAyane's dsh-reasoning-effort](https://github.com/HanaAyane/dsh-reasoning-effort)** (MIT). If you used that plugin, remove it first to avoid two effort controls on the same seat:

```sh
dsh plugin --profile web remove dsh-reasoning-effort
```

**Known limitations**

- Injection depends on the official Models page's DOM (aria-label / class); an official upgrade may pause injection until adapted — the official page is unaffected meanwhile.
- The auto-adapt probe route answers **loopback and IP-literal hosts only**, and **never follows redirects** — a gateway listing its models only behind a 30x yields no endpoint evidence; auto-adapt falls back to the knowledge base and protocol inference.
- `reasoningEfforts` declarations are suggestions — what an endpoint actually accepts is up to its docs; tweak in the UI.
- Endpoint-compatibility switches are never auto-filled by design: they describe gateway behavior, not model capabilities.
- The modality vocabulary follows pi-ai's core (`text` / `image` today); wider gateway support (PDF / audio / video) cannot be declared until the core vocabulary grows — this is by design.
- Name-heuristic modality advice (vision-flavored ids) is deliberately labeled low-confidence; verify before relying on it.
- Self-hosted relays: auto-fill pins `supportsDeveloperRole: false` on routes no official host claims; explicit values are never overwritten.
- Forced-thinking models (ladders without `off`, e.g. GLM-5.3): effort-less calls map to the vendor default instead of sending `thinking: disabled` — set `defaultGuard: false` to restore raw behavior.
- **Credentials inside `headers` are not redacted on disk**: the read-only view masks them, but the settings document still holds them in clear text — treat it like an API key.
- **The request-header section depends on the official card's editor container**: if the official build changes that structure, the section stops appearing — it never breaks the page. This is the fork's tradeoff vs. upstream: no slot occupancy, in exchange for coexistence with dsh-web plugins.
- **Only one `user-agent` rewrite should be active**: sibling header plugins land on the same layer; the last writer wins. The plugin detects and reports known siblings but cannot cover unknown cases.
- The request-layer takeover relies on the official adapter creating a fresh SDK client per request — an end-to-end test guards this boundary and fails loudly if it changes.

See [`docs/compatibility-notes.md`](docs/compatibility-notes.md) for more compatibility details, and [`docs/supported-models.md`](docs/supported-models.md) for the supported-models list.

## Development & Testing

```sh
npm run typecheck   # tsc strict check
npm test            # vitest: knowledge base / inference / autofill / DOM injection / writing
npm run build       # lib/*.js + lib/client.js
```

Tests are vitest-driven. Run `npm install` first to install dependencies (a fresh clone has no `node_modules`), then `npm test` to run the suite; build with `npm run build` (`lib/` is a build artifact, gitignored and not present in clones).

When developing under the DSH file sandbox, `vitest` and esbuild need extra handling (the sandbox blocks subprocess pipes) — see [`FORK.md`](FORK.md) for the full commands.

## Relationship to the Aggregation Package

This package lives in a **standalone repository** (<https://github.com/Fish-under-sea/dsh-better-reasoning-effort-fish>) — it is not inside the `dsh-fish` monorepo. However, it is one of the members of the aggregation package [`@fish-under-sea/dsh-fish`](https://github.com/Fish-under-sea/dsh-fish) and can be installed alongside the other plugins in that suite.

## License

[MIT](LICENSE), copyright held by the original author **HaoyueQin**; new code in this fork is also released under MIT. Please preserve [`LICENSE`](LICENSE) and [`NOTICE.md`](NOTICE.md) when redistributing.
