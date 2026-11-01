# 201 · dsh-market 插件市场（功能规格 · 父工程）

> Module: 201-dsh-market ｜ 范围：`desktop` / `dsh-desktop-hos` / `dsh-desktop-ark` / `dsh-desktop-android`
> Status: Implemented（as-built；各壳证据见各自 `logs/` 与本文 §10）｜ Last Updated: 2026-11-01
> 关联：`013-branding`（应用级品牌）、`110-pnpm-integration`（进程内 pnpm 引擎）、各壳 `201-dsh-market/plan.md`

---

## 1. 模块概述

### 1.1 目的（Why）

把 `dsh-market`（npm 包 `dshmarket`，DeepSeek Harness 的可视化插件市场）**内置**进每个壳所封装的 dsh 环境：
设置页出现「插件市场」，用户可浏览社区目录并**安装 / 更新 / 配置 / 卸载**插件——开发态与打包态都可用，且不需要设备侧自装 Node/pnpm。

`dshmarket` 是前后端混合的 Cordis 插件：宿主端（`lib/`）挂载 `/dsh-market/*` 路由、读写 profile 目录、触发安装；浏览器端（`client/`）注入设置页 UI。本模块负责**物化、profile/bundle 声明、插件管理通道接线与运行配置**，**不改 dsh-market 与 dsh 的上游源码**（C1）。

### 1.2 解决的问题

- **零命令行逛/装插件**：用户无需自装 Node/pnpm、无需敲 `dsh plugin add`，在设置页即可浏览与一键装卸。
- **打包应用没有系统 Node/pnpm**：打包进 APK/HAP/安装包的应用没有可用的系统 Node/pnpm；且移动/鸿蒙沙箱**禁止 spawn 任意子进程**或对 spawn 的 node/pnpm 以 `SIGSYS` 杀死。本模块让市场的安装/卸载走**进程内 pnpm**（引擎由 `110-pnpm-integration` 提供），无需改市场源码。
- **profile 正确性**：市场默认操作 `web` profile，各壳用 `desktop`/自有 profile；必须显式注入 profile 名，否则装到错误目录。
- **进程生命周期归属**：市场不得擅自 spawn 独立 dsh 进程重启；重启由壳掌控（`allowRestart: false`）。
- **产物新鲜度（见 §10）**：市场 `lib/` 一旦落后于 `src/`，会出现「某个 UI 动作 404/405」（如 `sidebar entry: HTTP 405`）——本模块要求在构建期拒绝陈旧产物。

### 1.3 范围

**包含**：市场的构建期物化（产物 + 运行时依赖闭包）、版本固定与断言、profile bundle 声明与运行配置注入、插件管理通道（`desktopProfiles`/`desktopPnpm` 契约 或 Electron 侧便携 Node/pnpm）、运行期接线与复制、产物新鲜度门禁、能力与限制披露。

**不包含**：市场自身 UI/目录/备份/WebDAV 等功能（由市场提供）；进程内 pnpm 引擎实现（属 `110`）；修改市场或上游源码。

---

## 2. 用户故事

- 作为**最终用户**，打开设置就能看到「插件市场」，浏览并一键安装插件，无需自装 Node/pnpm。
- 作为**最终用户**，安装/卸载真的生效且重启不复活；市场不擅自重启我的应用。
- 作为**打包者**，市场产物随构建自动物化；缺产物、版本不符或**产物落后于源码**时构建明确失败，而不是产出「入口消失 / 某动作 405」的残包。

---

## 3. 功能需求

> RFC 2119：MUST / SHOULD / MAY。各壳实现映射见 `plan.md`。

### 3.1 构建期物化与版本锁定

- **FR-201-001**：装配/收集阶段 MUST 把市场产物（`package.json`、`cordis.patch.yml`、`lib/`、`client/`）物化到 dsh 运行树的 `node_modules/dshmarket/`，SHOULD 排除 `src/`、`tests/`、`scripts/`、`site/`、`locale/`、`assets/`、`tsconfig*` 等开发物。
- **FR-201-002**：市场工程缺失或未构建（缺 `lib/index.js` / `client/client.js`）时 MUST 硬失败，并给出 `npm i && npm run build` 的下一步；MUST NOT 静默产出缺市场的包。
- **FR-201-003**：系统 MUST 递归复制市场的**运行时依赖闭包**到市场自身 `node_modules`；`@deepseek-ai/*` MUST 从载荷宿主闭包解析，**MUST NOT** 复制第二份。
- **FR-201-004**：物化 MUST 幂等。
- **FR-201-005**：每个壳 MUST 以固定常量**pin** 市场版本并在构建期**断言**（如 `assertDshMarketVersion()`）；版本不一致 MUST 失败。各壳 pin 值 MUST 记录在 `plan.md` 矩阵，并 SHOULD 对齐到工作区 `dsh-market` 子模块版本。

### 3.2 profile / bundle 声明与运行配置

- **FR-201-006**：壳 MUST 把 `dshmarket` 声明为（各自）profile 的 bundle，使市场随组合自动加载，且为市场行注入运行配置 `{ profile: <壳 profile>, allowRestart: false }`（市场不得擅自重启壳）。
- **FR-201-007**：ark/android 类「市场随载荷 bundle 解析」的壳 MUST NOT 在 profile `dependencies` 里声明 `dshmarket`（按需以 bundle 锚点解析；见各壳 plan）。
- **FR-201-008**：市场行 MUST NOT 被注入会令其进入「官方 Electron 桌面分支」的配置（除显式 `profile`/`allowRestart` 外）。

### 3.3 插件管理通道（装卸能力）

- **FR-201-010**：宿主 MUST 在 market bundle mount **之前**提供 `desktopProfiles` 服务，市场经特性检测（`ctx.get('desktopProfiles')`）采用宿主提供的包管理器（由 `110` 实现）。
- **FR-201-011**：宿主 MUST 通过 `desktopPnpm`（实现市场 `DesktopPnpmLike`）提供 `runPlugin(args, invokingDir, signal)`，实现 MUST 为**进程内**（不 spawn node/pnpm 子进程）——适用于 `hos` / `ark` / `android`。
- **FR-201-012**：安装 MUST 经 `desktopPnpm.runPlugin(['add', '<spec>'], dir)`；卸载 MUST 经 `['remove','<name>']` 并同步从 profile bundle 列表 reconcile。
- **FR-201-013**：`desktop`（Electron）MUST 以**内置便携 Node + pnpm + dsh CLI** 打通装卸通道（Electron 允许 spawn，但应用不带系统 Node/pnpm）。
- **FR-201-014**：当宿主未提供契约时，系统 MUST 显式记录「安装通道不可用」（不静默），并给出降级/报错路径。
- **FR-201-015**：装卸 MUST 通过宿主既有的持久化/激活路径完成；对「本宿主客户端模块图不可在线重放」的壳，UI MUST 提示「重启应用后生效」，MUST NOT 在市场侧 hot mount。

### 3.4 运行期接线

- **FR-201-016**：`hos`/`ark`/`android` MUST 在 Host 启动前把市场产物就位（复制或直接解析），并把 `DSH_PNPM_ENGINE` 指向随包引擎（`<runRoot>/node_modules/dsh-market-pnpm/bin/pnpm.cjs` 或等价）；MUST 幂等，失败 MUST NOT 阻塞 Host 启动（记日志）。
- **FR-201-017**：`hos` MUST 在 Host 启动前把市场复制到 profile 目录（**复制而非 symlink**，鸿蒙沙箱禁止 symlink），且幂等、失败不阻塞启动。

### 3.5 产物新鲜度与健壮性（含 405 修复）

- **FR-201-020**：构建/装配期 MUST 拒绝「**源码比产物新**」的 `dsh-market`（`src/*.ts` 新于对应 `lib/*.js` 即失败），避免打出缺路由的旧 `lib/`。**现象**：市场 UI 的某个动作返回 404/405（典型：`sidebar entry: HTTP 405`，见 §10）。
- **FR-201-021**：将市场产物复制进载荷的壳 MUST 在源目录比已物化副本更新时**重新复制**（避免沿用陈旧副本）。
- **FR-201-022**：外果子资源（插件客户端模块、图标、API 请求）失败时 MUST 只记录日志；MUST NOT 因单个子资源失败显示整页壳级错误（仅主文档加载失败才整页报错）。

### 3.6 能力与限制披露

- **FR-201-023**：文档 MUST 披露：`git:` 源插件、需要 lifecycle 构建脚本的插件不可安装；「放行构建脚本并重试」在仅支持 `ignoreScripts` 的宿主无效。
- **FR-201-024**：文档 MUST 披露：profile 位于载荷内时，换包（新 sha/分代）后已装插件不保留。
- **FR-201-025**：`store`/受限发行线的市场 MAY 为只读（不允许安装），MUST 在 UI 与文档登记。

---

## 4. 跨壳差异矩阵（as-built）

| 维度 | `desktop`(Electron) | `dsh-desktop-hos`(Electron-on-HarmonyOS) | `dsh-desktop-ark`(ArkWeb+Node) | `dsh-desktop-android`(WebView+Node) |
|---|---|---|---|---|
| 市场版本 pin | `v1.26.0`（历史，见其 spec） | `1.66.7` | `1.66.42` | `1.67.0` |
| 物化位置 | dsh-dist / 可解析 | `dsh-dist/node_modules/dshmarket` | `<stage>/node_modules/dshmarket` | `<run>/node_modules/dshmarket` |
| 走哪条安装通道 | 内置便携 Node+pnpm+dsh CLI（spawn） | 进程内 pnpm（契约） | 进程内 pnpm（契约，`--jitless` 兼容层入口） | 进程内 pnpm（契约） |
| 运行期是否复制到 profile | 否（Electron 解析） | 是（复制，禁 symlink） | 否（bundle 锚点解析） | 否（bundle 锚点解析） |
| JIT/JITLESS | JIT | JIT（Electron） | `--jitless`（`DSH_OHOS_JITLESS_ENTRY`） | JIT |
| 产物新鲜度门禁 | 构建期构建 | 构建期**无条件 `npm run build`**（天然新鲜） | 构建期 `assertMarketBuilt` | 构建期 `assertMarketBuilt`（2026-11-01 补） |

> pin 值存在历史漂移（1.26.0 / 1.66.7 / 1.66.42 / 1.67.0）；按 FR-201-005，各壳 SHOULD 对齐到工作区子模块当前版本。工作区 `dsh-market` 子模块当前为 `1.67.0`。

---

## 5. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| `dshmarket` 产物 | 内置市场包 | `lib/`（host）、`client/client.js`（web）、`cordis.patch.yml`（插入 `dsh-market` 行） |
| 运行时依赖闭包 | 市场自身依赖 | `undici`、`js-yaml`、`argparse` 等；不含 `@deepseek-ai/*` |
| profile 声明 | 加载市场入口 | `dsh.profile.bundles` 含 `dshmarket` + 运行配置 |
| `desktopProfiles`/`desktopPnpm` | 宿主提供的进程内包管理器契约 | `current.dir`、`desktopPnpm.runPlugin`、`pluginActivation` |
| 版本常量 | 各壳 pin | `DSH_MARKET_VERSION`（每壳一份，构建期断言） |

---

## 6. 验收场景

- **市场就位**：Given 市场已构建；When 装配；Then `node_modules/dshmarket/{package.json,cordis.patch.yml,lib/index.js,client/client.js}` 存在，运行时依赖可解析，且无 `@deepseek-ai/` 副本。
- **缺产物即失败**：Given 缺 `lib/index.js`；When 装配；Then 硬失败并给出 `npm run build` 提示。
- **陈旧产物即失败**：Given `src/routes.ts` 比 `lib/routes.js` 新；When 装配；Then 硬失败（不产出 405 残包）。
- **设置页出现市场**：Given 应用就绪；When 打开设置；Then 有「插件市场」入口并能拉取目录（或如实记录网络不可达）。
- **安装插件（进程内）**：When 点安装；Then 插件落盘于 profile、bundle 行追加、UI 提示重启、无 node/pnpm 子进程、无整页错误。
- **重启生效 / 卸载干净**：重启后新行参与加载；卸载后目录消失、bundle 行移除、重启不复活。
- **不擅自重启**：Given `allowRestart:false`；Then 市场安装不会触发独立 dsh 进程重启。

---

## 7. 非功能需求

- **可重复/幂等**：物化、复制、配置生成、启动接线可重复执行。
- **健壮性**：构建期缺产物/陈旧产物硬失败；运行期失败不阻塞启动。
- **安全**：市场只在同源 loopback 暴露路由；安装通道 `ignoreScripts`；凭据/口令不进日志、不进 specs 样例（C15）。
- **体积**：市场 + 依赖约数 MiB 量级。

---

## 8. 假设与约束

- 市场为同级源码（`../dsh-market`），版本以常量固定并断言。
- 设备网络可达 registry 与市场目录源是市场可用前提；不可达时如实记录（C3）。
- 移动/鸿蒙应用域禁止 spawn 任意 node/pnpm；装卸必须进程内（`hos`/`ark`/`android`）。
- profile 位于载荷内时，换包后已装插件不保留（FR-201-024）。

---

## 9. 依赖

- 上游：`dsh-market`（只读）、`deepseek-harness` 的 bundle/profile/client 机制（只读）。
- 工程内：`110-pnpm-integration`（进程内 pnpm 引擎与契约）、各壳载荷管线（`102`/`112`）、bundle 适配层（`105`）。

---

## 10. 修复记录：`sidebar entry: HTTP 405`

| 项 | 内容 |
|---|---|
| 现象 | 设置 → 插件市场 → 行「想要更快打开？可在侧栏显示」点「显示」→ 报 `sidebar entry: HTTP 405` |
| 报错来源 | 客户端 `GET/POST /dsh-market/sidebar-entry`（市场 `client/sidebar-entry.ts`）；宿主侧路由在 `src/routes.ts` 定义 |
| 根因 | 工作区 `dsh-market` 的**构建产物 `lib/` 落后于源码 `src/`**：`/dsh-market/sidebar-entry` 路由只存在于 `src/routes.ts`，而打出的 `lib/routes.js` 是旧构建（缺该路由）→ 请求落到兜底 → HTTP 405 |
| 证据（android，2026-11-01） | 修复前 `grep -c sidebar-entry dsh-market/lib/routes.js` = **0**；`src/routes.ts` 含该路由且 mtime 比 `lib/routes.js` 新 5 天 |
| 修复 | ① 重建 `dsh-market`（`npm run build`）→ `lib/routes.js` 含该路由；② `dsh-desktop-android` 装配加 `assertMarketBuilt()`（`scripts/assemble-payload.mjs`），`src/*.ts` 新于 `lib/*.js` 即构建失败（FR-201-020） |
| 各壳状态 | `ark`：早已有 `assertMarketBuilt`（本模块 FR 来源）；`android`：本次补齐并验证；`hos`：`scripts/build-dsh.mjs` **构建期无条件 `npm run build`**（天然新鲜，不受影响）；`desktop`：构建期构建 |
| 回归证据 | android 真机 MIX（Android 8.0/API26）重装后，设置→插件市场→「显示」不再 405（2026-11-01，See `dsh-desktop-android/logs/`） |
| 预防 | 以 FR-201-020/021 固化：**产物落后于源码即构建失败**，禁止再产出缺路由的残包 |

---

## 11. 修复记录：插件安装失败（Android 无 `/tmp` → pnpm `temp-dir` ENOENT）

| 项 | 内容 |
|---|---|
| 现象 | 设置 → 插件市场 → 点「安装」→「确认安装」，任务失败；市场只显示 `dsh-market-pnpm/dist/pnpm.cjs` 的 4 行栈尾（`createNewStoreController` → `pkg-manager/client` → `binary-fetcher`） |
| 真实错误（host stderr） | `Error: ENOENT: no such file or directory, lstat '/tmp'`，位于 pnpm `temp-dir@2.0.0` 的**模块加载期** `realpathSync(os.tmpdir())` |
| 根因 | **Android 没有 `/tmp`**；libnode 的 `os.tmpdir()` 回退到 `/tmp`，而 pnpm 在 `createNewStoreController`（store 初始化）时 eager require `fetching/binary-fetcher` → `tempy` → `temp-dir`，模块体即 `realpathSync(os.tmpdir())` → ENOENT → 整个 pnpm store 初始化失败 → 所有插件安装失败 |
| 为何只有 android 复现 | 桌面/宿主有 `/tmp`（或 `%TEMP%`）；`hos`/`ark` 走各自引擎封装。Android 应用域无 `/tmp` |
| 修复（android） | ① `packages/market-android/lib/pnpm-worker.mjs`：在 `import(pnpm)` **之前** `mkdir <profile>/.pnpm-cache/tmp` 并设置 `TMPDIR/TMP/TEMP` + 覆盖 `os.tmpdir()` 指向它；② 强制 `package-import-method=copy`（避开 reflink/hardlink 的 Android 原生/权限问题）；③ 装配期物化 `@reflink/reflink-android-arm64` 存根（防御下一层 reflink require）；④ `pnpm-inprocess.js` 失败时把**完整 worker stderr** 写入 host 日志（市场 UI 只保留栈尾、丢消息首行，导致此前无法定位） |
| 各壳状态 | `ark`：其 pnpm worker 亦需保证 `os.tmpdir()` 可用（HarmonyOS 同样可能无 `/tmp`）——按本 FR 精神核对；`hos`：Electron/HarmonyOS 走各自链路；`desktop`：有系统 `/tmp`/`%TEMP%` 不受影响 |
| 回归证据 | android 真机 MIX（API26）：安装 `dsh-unlag` 成功 → 落盘 `profiles/android/node_modules/dsh-unlag` → 卡片「✓ 已安装」、`已安装 (1)`（2026-11-01，见 `dsh-desktop-android/logs/20261101-1/evidence/45-market-install-success.png`） |
| 预防 | 以「临时目录必须在 import pnpm 前就位」为装配/运行前置；`pnpm-inprocess` 失败时务必保留完整 stderr（诊断可观测性） |
