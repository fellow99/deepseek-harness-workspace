[中文](./README_zh.md) | English

---

# DeepSeek Harness Workspace

> A single-repo workspace aggregating every project required to ship [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) as a first-class desktop app — on Windows/Linux via an Electron shell, and on HarmonyOS via the "Electron-on-HarmonyOS" runtime. All repositories are pinned as **git submodules** so the workspace layout reproduces the exact sibling-directory structure each wrapper's build scripts expect (`../deepseek-harness`, `../dsh-market`, `../harmonypc-electron`).

---

## What is this

DeepSeek Harness (`dsh`) is an open-source agent harness by DeepSeek AI, built on an "everything is a plugin" architecture; its native entry is `dsh web` (a browser Web UI). Two sibling wrapper projects turn that Web UI into native desktop apps — fully reusing the dsh frontend and hosting the dsh Host in-process:

- **`deepseek-harness-desktop`** — an Electron desktop shell for Windows / Linux (macOS later)
- **`deepseek-harness-harmony`** — a HarmonyOS desktop port (2in1 / tablet, packaged as a HAP) on the Electron-on-HarmonyOS runtime

Both wrappers consume the same upstream building blocks. This workspace is the container that keeps those building blocks (as submodules) side by side, so the two wrapper projects can be cloned, built, and developed in one place — with their `../sibling` references resolving inside the workspace.

## Repository layout

```
deepseek-harness-workspace/            # this repo — submodule container
├── deepseek-harness-desktop/          # [submodule] Electron desktop wrapper (Windows/Linux)
├── deepseek-harness-harmony/          # [submodule] HarmonyOS desktop wrapper (2in1/tablet, HAP)
├── deepseek-harness/                  # [submodule] dsh — the wrapped agent host (upstream)
├── dsh-market/                        # [submodule] visual plugin marketplace (npm pkg "dshmarket")
├── harmonypc-electron/                # [submodule] Electron-on-HarmonyOS runtime (Electron 37 / Node 22.17.0)
├── deepseek-harness-desktop-website/  # [submodule] website of the two wrappers
└── harmonypc-electron-versions/       # (plain dir, not a submodule) runtime release archives + Electron header guides
```

### Roles at a glance

| Directory | Role | Consumed by |
|---|---|---|
| `deepseek-harness-desktop` | Electron desktop shell — the primary wrapper | — |
| `deepseek-harness-harmony` | HarmonyOS desktop port (benchmarked against desktop) | — |
| `deepseek-harness` | The wrapped agent host (`dsh`, upstream source reference) | both wrappers |
| `dsh-market` | Built-in visual plugin marketplace | both wrappers |
| `harmonypc-electron` | Electron-on-HarmonyOS runtime (native SOs + ArkTS bridge) | harmony only |
| `deepseek-harness-desktop-website` | Product website of the two wrappers | — |
| `harmonypc-electron-versions` | Electron-on-HarmonyOS release archives (v34/v37/v40) + Node-header guides for rebuilding native modules like `better-sqlite3` | harmony tooling |

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
| dsh patches | 2 (disable HMR, disable native picker) | 4 (shared 2 + symlink→`cpSync` copy, allow-all-interfaces) |
| MVP capabilities | Scaffold + dsh consumption complete; tray / notifications / frameless window / clipboard image paste (see README for live status) | Verified on device — HarmonyOS 6.1.0.135 (API 24); Web UI fully functional; session persistence + plugin marketplace in; terminal / process sandbox out (aarch64 artifacts missing) |

## Getting started

```bash
git clone --recursive https://github.com/fellow99/deepseek-harness-workspace.git
cd deepseek-harness-workspace
git submodule update --init --recursive   # refresh all pinned submodules
```

The workspace layout already satisfies each wrapper's sibling prerequisites (`../deepseek-harness`, `../dsh-market`, `../harmonypc-electron`). Build and run instructions are maintained inside each wrapper and may drift from this overview:

- **Desktop**: see [`deepseek-harness-desktop/README.md`](deepseek-harness-desktop/README.md) (`npm run build:dsh` applies the 2 patches and builds dsh + dsh-market, then `npm start` / `npm run package`).
- **HarmonyOS**: see [`deepseek-harness-harmony/README.md`](deepseek-harness-harmony/README.md) (three-stage build `collect-runtime → build-dsh → collect-dsh`, then HAP build + signing via DevEco Studio / hvigor).

> The harmony build's first stage copies the runtime from the `../harmonypc-electron` submodule; `harmonypc-electron-versions/` holds the standalone release archives if you need a runtime version other than the pinned submodule.

## Submodules

All entries are recorded in [`.gitmodules`](.gitmodules):

| Path | Remote |
|---|---|
| `deepseek-harness-desktop` | https://github.com/fellow99/deepseek-harness-desktop.git |
| `deepseek-harness-harmony` | https://github.com/fellow99/deepseek-harness-harmony.git |
| `deepseek-harness` | https://github.com/deepseek-ai/deepseek-harness.git |
| `dsh-market` | https://github.com/dsh-market/dsh-market.git |
| `harmonypc-electron` | https://atomgit.com/jianguoxu/harmonypc-electron.git |
| `deepseek-harness-desktop-website` | https://github.com/fellow99/deepseek-harness-desktop-website.git |

Advance the pinned commits (e.g. to pick up a new dsh release) with `git submodule update --remote`, then commit the new pointers.

## License

[MIT](LICENSE) © 2026 fellow99
