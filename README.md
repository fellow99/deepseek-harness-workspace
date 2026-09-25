[中文](./README_zh.md) | English

---

# DeepSeek Harness Workspace

> A single-repo workspace aggregating every project required to ship [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) as a first-class desktop app — on Windows/Linux via an Electron shell, and on HarmonyOS via the "Electron-on-HarmonyOS" runtime. All repositories are pinned as **git submodules** so the workspace layout reproduces the exact sibling-directory structure each wrapper's build scripts expect (`../deepseek-harness`, `../dsh-market`, `../harmonypc-electron`).

---

## What is this

DeepSeek Harness (`dsh`) is an open-source agent harness by DeepSeek AI, built on an "everything is a plugin" architecture; its native entry is `dsh web` (a browser Web UI). Two sibling wrapper projects turn that Web UI into native desktop apps — fully reusing the dsh frontend and hosting the dsh Host in-process:

- **`dsh-desktop`** — an Electron desktop shell for Windows / Linux (macOS later)
- **`dsh-desktop-hos`** — a HarmonyOS desktop port (2in1 / tablet, packaged as a HAP) on the Electron-on-HarmonyOS runtime

Both wrappers consume the same upstream building blocks. This workspace is the container that keeps those building blocks (as submodules) side by side, so the two wrapper projects can be cloned, built, and developed in one place — with their `../sibling` references resolving inside the workspace.

## Repository layout

```
deepseek-harness-workspace/            # this repo — submodule container
├── dsh-desktop/          # [submodule] Electron desktop wrapper (Windows/Linux)
├── dsh-desktop-hos/          # [submodule] HarmonyOS desktop wrapper (2in1/tablet, HAP)
├── deepseek-harness/                  # [submodule] dsh — the wrapped agent host (upstream)
├── dsh-market/                        # [submodule] visual plugin marketplace (npm pkg "dshmarket")
├── dsh-plugins/                       # (plain dir) generic, shareable dsh plugins — one dir per plugin, named dsh-plugin-XXX
├── skills/                            # (plain dir) generic, shareable dsh tool skills — one dir per skill, kebab-case name
├── harmonypc-electron/                # [submodule] Electron-on-HarmonyOS runtime (Electron 37 / Node 22.17.0)
├── dsh-desktop-website/  # [submodule] website of the two wrappers
└── harmonypc-electron-versions/       # (plain dir, not a submodule) runtime release archives + Electron header guides
```

### Roles at a glance

| Directory | Role | Consumed by |
|---|---|---|
| `dsh-desktop` | Electron desktop shell — the primary wrapper | — |
| `dsh-desktop-hos` | HarmonyOS desktop port (benchmarked against desktop) | — |
| `deepseek-harness` | The wrapped agent host (`dsh`, upstream source reference) | both wrappers |
| `dsh-market` | Built-in visual plugin marketplace | both wrappers |
| `dsh-plugins` | **Generic, shareable** dsh plugins — one directory per plugin, named `dsh-plugin-XXX`. Currently empty but for its own README (see [`dsh-plugins/README.md`](dsh-plugins/README.md)); wrapper-specific plugins live inside their wrapper | any wrapper |
| `skills` | **Generic, shareable** dsh tool skills — one directory per skill, kebab-case name. Currently empty but for its own README (see [`skills/README.md`](skills/README.md)); wrapper-specific skills live inside their wrapper | any wrapper |
| `harmonypc-electron` | Electron-on-HarmonyOS runtime (native SOs + ArkTS bridge) | harmony only |
| `dsh-desktop-website` | Product website of the two wrappers | — |
| `harmonypc-electron-versions` | Electron-on-HarmonyOS release archives (v34/v37/v40) + Node-header guides for rebuilding native modules like `better-sqlite3` | harmony tooling |

### Plugin convention

First-party dsh plugins are split into **two tiers**, distinguished by who can consume them:

| Location | Tier | Naming |
|---|---|---|
| `dsh-plugins/` (this workspace) | **Generic / shareable** — depends on no wrapper-specific patch, so any shell can consume it | `dsh-plugin-XXX` |
| `<wrapper>/plugins/` (e.g. [`dsh-desktop-hos/plugins/`](dsh-desktop-hos/plugins/)) | **Wrapper-specific** — depends on that wrapper's own patch set / profile / runtime adaptations; baked into the package at build time and **all loaded by default** | `harmony-plugin-XXX` for the harmony wrapper |

In both tiers the directory name is also the package `name` (a bare, unscoped name), where `XXX` names the function the plugin adds. The test for which tier a plugin belongs to: *"would it still work if installed into another dsh shell?"* Yes → generic; no → wrapper-specific.

Plugins are plain ESM (`lib/index.js`, no build step). Each consuming wrapper materializes the plugins it wants into its own shipped `dsh-dist/node_modules/` at collect time, then mounts them from an agent-preset row.

> The only plugin written so far — the `delete`/`move` file tools — is **wrapper-specific**: it needs the harmony wrapper's `dsh-fs-remove-primitive` patch for `ctx.fs.remove`, so it lives at [`dsh-desktop-hos/plugins/harmony-plugin-fs-mutate/`](dsh-desktop-hos/plugins/harmony-plugin-fs-mutate/). That is also why `dsh-plugins/` is currently empty but for its own README. See [`dsh-plugins/README.md`](dsh-plugins/README.md) and [`dsh-desktop-hos/plugins/README.md`](dsh-desktop-hos/plugins/README.md).

### Skill convention

Tool skills follow the same **two-tier split** as plugins:

| Location | Tier | Consumed by |
|---|---|---|
| `skills/` (this workspace) | **Generic / shareable** — applies to any dsh shell | any wrapper |
| `<wrapper>/skills/` (e.g. [`dsh-desktop-hos/skills/`](dsh-desktop-hos/skills/)) | **Wrapper-specific** — describes that wrapper's own runtime (sandbox, packaging paths, capability gaps) | that wrapper only |

Unlike plugins, **skill names carry no prefix**: a skill is named by function in kebab-case, because its `name` is a *model-visible identifier* and the string a human types after `/name` — ownership is expressed by the **directory**, not the name. The convention is that the directory name equals the frontmatter `name`; note that dsh does **not** validate this.

A skill is either `<name>/SKILL.md` (directory bundle, may carry resources) or `<name>.md`, and only the **top level** of a skill root is scanned — a nested `**/SKILL.md` is not discovered. Frontmatter requires `name` + `description`, plus optional `whenToUse` / `metadata` / `disable-model-invocation` / `user-invocable` (the camelCase legacy keys are rejected).

**Loading semantics — do not describe this as "loaded at startup".** The model automatically sees only the skill **catalog** (name, plus a description truncated to 500 chars), injected as a durable user-role message before a session's **first request**. A skill **body** loads on demand — when the model calls the `skill` tool, or a human types `/name` — and bodies are never cached.

**Generic skills do not load themselves.** Each wrapper opts in; dsh's mechanism is `customSkillDirs`, which is how the shipped `cordis` preset points `skill-filesystem` at its own `skills/`.

> The only skill written so far — `harmony-runtime-capabilities` — is **wrapper-specific** (it documents the HarmonyOS HAP sandbox, `hmdfs`, and this build's capability gaps), so it lives at [`dsh-desktop-hos/skills/harmony-runtime-capabilities/`](dsh-desktop-hos/skills/harmony-runtime-capabilities/). That is why `skills/` is currently empty but for its own README. See [`skills/README.md`](skills/README.md) and [`dsh-desktop-hos/skills/README.md`](dsh-desktop-hos/skills/README.md).

### Current versions

| Project | Version | dsh version |
|---|---|---|
| `dsh-desktop` | **0.1.5** | `dsh-v0.1.5-rc.2` |
| `dsh-desktop-hos` | **0.1.5** | `dsh-v0.1.5-rc.2` |
| `deepseek-harness` (submodule pin) | — | `dsh-v0.1.5-rc.2` |

Both wrappers pin the same upstream dsh tag, which selects `patches/dsh-v0.1.5-rc.2/` in each project's
build script. When either wrapper advances, update this table together with the two product READMEs.

## Shared architecture

`dsh` has completed its **Host/Client split**, and its webserver **serves both the SPA dist and `/api`**. Both wrappers therefore use the same **in-process Host + webserver + same-origin data plane**:

- The wrapper's main process dynamically imports dsh's `runProfile('desktop')` and hosts the dsh Host in-process (webserver + apiProxy + connection, all on one bound port).
- Once ready, the renderer does `loadURL(...)` against that webserver and runs the **standard, unmodified dsh Web UI**, reusing its existing `WebApiClient` (HTTP uplink + WebSocket downlink).
- Result: **zero CORS, zero auth, zero custom protocol, zero new IPC carrier — zero upstream changes** (only idempotent patches).

### Platform-specific adaptations

| Aspect | Desktop (Electron) | HarmonyOS (Electron-on-HarmonyOS) |
|---|---|---|
| Target platforms | Windows + Linux (macOS later) | HarmonyOS 2in1 / tablet (HAP) |
| Webserver binding | `127.0.0.1:<free port>` | `0.0.0.0:<free port>` + renderer connects via LAN IP with `Host`/`Origin` rewrite (HarmonyOS NEXT loopback network isolation) |
| dsh patches | 2 (disable HMR, disable native picker) | 5 (shared 2 + symlink→`cpSync` copy, allow-all-interfaces, flock openharmony stub) |
| MVP capabilities | Scaffold + dsh consumption complete; tray / notifications / native title bar / clipboard image paste (see README for live status) | Verified on device — HarmonyOS 6.1.0.135 (API 24); Web UI fully functional; session persistence + plugin marketplace in; terminal / process sandbox out (aarch64 artifacts missing) |

## Getting started

```bash
git clone --recursive https://github.com/fellow99/deepseek-harness-workspace.git
cd deepseek-harness-workspace
git submodule update --init --recursive   # refresh all pinned submodules
```

The workspace layout already satisfies each wrapper's sibling prerequisites (`../deepseek-harness`, `../dsh-market`, `../harmonypc-electron`). Build and run instructions are maintained inside each wrapper and may drift from this overview:

- **Desktop**: see [`dsh-desktop/README.md`](dsh-desktop/README.md) (`npm run build:dsh` applies the 2 patches and builds dsh + dsh-market, then `npm start` / `npm run package`).
- **HarmonyOS**: see [`dsh-desktop-hos/README.md`](dsh-desktop-hos/README.md) (three-stage build `collect-runtime → build-dsh → collect-dsh`, then HAP build + signing via DevEco Studio / hvigor).

> The harmony build's first stage copies the runtime from the `../harmonypc-electron` submodule; `harmonypc-electron-versions/` holds the standalone release archives if you need a runtime version other than the pinned submodule.

## Submodules

All entries are recorded in [`.gitmodules`](.gitmodules):

| Path | Remote |
|---|---|
| `dsh-desktop` | https://github.com/fellow99/dsh-desktop.git |
| `dsh-desktop-hos` | https://github.com/fellow99/dsh-desktop-hos.git |
| `deepseek-harness` | https://github.com/deepseek-ai/deepseek-harness.git |
| `dsh-market` | https://github.com/dsh-market/dsh-market.git |
| `harmonypc-electron` | https://atomgit.com/jianguoxu/harmonypc-electron.git |
| `dsh-desktop-website` | https://github.com/fellow99/dsh-desktop-website.git |

Advance the pinned commits (e.g. to pick up a new dsh release) with `git submodule update --remote`, then commit the new pointers.

## License

[MIT](LICENSE) © 2026 fellow99
