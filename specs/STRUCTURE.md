# STRUCTURE.md — 父工程目录与文件结构

> 工程：`deepseek-harness-workspace`（submodule 容器 / 聚合工作区）
> 生成方式：扫描工作区实际目录（2026-11-01），排除 `node_modules`、`oh_modules`、`build`、`out`、`dsh-dist`、`.git`、`.hvigor`、`.gradle`、`.kotlin`、`logs` 等生成目录。
> 用途：为父工程规范文档与各子工程 specs 提供目录索引；新增/删除模块时同步更新本文件。

---

## 1. 顶层目录树

```
deepseek-harness-workspace/
├── README.md / README_zh.md          # 工作区总览、四版能力矩阵、版本表
├── LICENSE                           # MIT
├── .gitmodules                       # 8 个 submodule 定义
├── .gitignore
│
├── assets/                           # 全产品公共物料源（logo / 截图 / 协议文本）
│   ├── logo.{svg,png,ico,icns}
│   ├── logo-1024-with-bg.png         # HarmonyOS 应用图标（不透明底）
│   ├── logo-200-with-bg.png
│   ├── screenshots/hos/
│   ├── privacy-policy.txt
│   └── user-agreement.txt
│
├── docs/                             # 父工程级文档（非 specs）
│   ├── 能力对照清单.md                # 四版 42 项能力矩阵（源码核实）
│   ├── 社区推广方案.md
│   ├── hos-promotion-channels.md
│   └── upstream-contribution/        # 5 个上游 patch 提案材料
│
├── specs/                            # 父工程规范文档（本文件所在目录）
│   ├── README.md / STRUCTURE.md / TECH.md / ARCHITECTURE.md
│   ├── constitution.md / overall-spec.md / overall-plan.md
│   ├── 011-version-0.1.5/ ~ 0.2.1/    # 版本更新需求（原 101-version-*）
│   ├── 012-building-workflow/         # 构建工作流（公共）
│   └── 013-branding/                  # 品牌化（公共，原 010-assets）
│
├── dsh-desktop/                      # [submodule] Electron 桌面壳
├── dsh-desktop-hos/                  # [submodule] Electron on HarmonyOS Edition
├── dsh-desktop-ark/                  # [submodule] HarmonyOS Edition（ArkWeb + Node）
├── dsh-desktop-android/              # [submodule] Android Edition（WebView + Node）
├── deepseek-harness/                 # [submodule] dsh 上游宿主
├── dsh-market/                       # [submodule] 可视化插件市场
├── harmonypc-electron/               # [submodule] Electron-on-HarmonyOS 运行时
├── dsh-desktop-website/              # [submodule] 产品官网
├── dsh-plugins/                      # 通用 dsh 插件（当前仅 README）
├── skills/                           # 通用 dsh 技能（当前仅 README）
└── harmonypc-electron-versions/      # 运行时归档 + Node header 指南（非 submodule）
```

---

## 2. 目录角色

| 目录 | 类型 | 角色 | 被谁消费 |
|------|------|------|----------|
| `dsh-desktop` | submodule | Electron 桌面壳（Windows/Linux/macOS） | — |
| `dsh-desktop-hos` | submodule | Electron on HarmonyOS Edition（HAP） | — |
| `dsh-desktop-ark` | submodule | HarmonyOS Edition（ArkWeb + 进程内 Node，HAP） | — |
| `dsh-desktop-android` | submodule | Android Edition（系统 WebView + 进程内 Node，APK） | — |
| `deepseek-harness` | submodule | 被封装的上游宿主（dsh） | 四个壳工程 |
| `dsh-market` | submodule | 内置可视化插件市场（`dshmarket`） | 四个壳工程 |
| `harmonypc-electron` | submodule | Electron-on-HarmonyOS 运行时 | `dsh-desktop-hos` |
| `dsh-desktop-website` | submodule | 产品官网 | — |
| `dsh-plugins` | 普通目录 | 通用/可共享 dsh 插件 | 任意壳工程 |
| `skills` | 普通目录 | 通用/可共享 dsh 技能 | 任意壳工程 |
| `harmonypc-electron-versions` | 普通目录 | 运行时发布归档与重建指南 | harmony 工具链 |
| `assets` | 普通目录 | 全产品公共物料源 | 四个壳工程 + 官网 |
| `specs` | 普通目录 | 父工程规范文档 | 全员 |

---

## 3. 各子工程目录结构（摘要）

### 3.1 dsh-desktop

```
dsh-desktop/
├── src/main/             # Electron 主进程：host / windows / tray / notifications / lifecycle
├── src/preload/          # 历史薄 IPC（011-version-0.1.5 起运行时不再依赖）
├── src/renderer/         # 兜底 loading 页
├── profiles/desktop/     # dsh desktop profile（cordis.patch.yml + package.json）
├── patches/<dsh-tag>/    # 按 dsh tag 分目录的幂等 patch
├── scripts/              # build-dsh / collect-dsh / fetch-runtime
├── resources/            # 应用/窗口/托盘图标
├── runtime/              # 便携 Node + pnpm（生成物，gitignored）
├── dsh-dist/             # 物化后的 dsh 运行树（生成物）
├── .github/workflows/    # ci.yml / release.yml
└── specs/                # 000/012/013/1XX/2XX 规范
```

### 3.2 dsh-desktop-hos

```
dsh-desktop-hos/
├── AppScope/             # 应用身份：app.json5、app_name、应用图标
├── electron/             # Electron-on-HarmonyOS 入口模块（ELF/资源）
├── web_engine/           # ArkUI Web 桥接 HAR（Adapter / AdapterBind）
├── src-main/             # 自研主进程编排（main.js 单文件）
├── profiles/desktop/     # dsh desktop profile
├── patches/<dsh-tag>/    # 13 个 dsh patch（含 rebrand / filesystem 适配）
├── plugins/              # 工程内专用 dsh 插件（harmony-plugin-*）
├── skills/               # 工程内专用技能（harmony-runtime-capabilities）
├── runtime-overlays/     # 运行时定制覆盖层
├── native/               # 原生构建辅助
├── scripts/              # collect-runtime / build-dsh / collect-dsh / build-debug
├── dsh-dist/             # 物化后的 dsh 运行树（生成物）
└── specs/                # 013/1XX/2XX/3XX 规范
```

### 3.3 dsh-desktop-ark

```
dsh-desktop-ark/
├── AppScope/             # 应用身份：app.json5、app_name、应用图标
├── entry/                # HAP 模块：EntryAbility、Index.ets、HostCapabilities、resfile 载荷
├── payload/              # Node 侧 bootstrap/host/selfcheck + 平台垫片
├── profiles/ohos/        # ohos profile
├── packages/             # @dsh-ohos/* seam 插件（subprocess/shell/git/market/branding/...）
├── libs/                 # 本地原生库（生成物/不入库）
├── scripts/              # assemble-payload / package-payload / build-hap / verify-upstream
└── specs/                # 013/101-108/110/119/2XX 规范
```

### 3.4 dsh-desktop-android

```
dsh-desktop-android/
├── app/                  # Kotlin 外壳：DshRuntime / WebShellView / HostCapabilities / KeepAliveService
├── packages/             # @dsh-android/* seam 插件（bundle/subprocess/shell/git/market/branding/...）
├── payload/              # Node 侧 bootstrap/host + 平台垫片
├── profiles/android/     # android profile
├── scripts/              # assemble-payload / package-payload / provision-engine / generate-brand-assets
├── gradle/               # version catalog / wrapper
├── .github/workflows/    # android-build.yml
└── specs/                # 000/012/013/1XX/2XX 规范
```

---

## 4. 父工程 specs 目录结构

```
specs/
├── README.md                     # 文档索引
├── STRUCTURE.md                  # 本文件
├── TECH.md                       # 技术选型
├── ARCHITECTURE.md               # 整体架构
├── constitution.md               # 宪法原则 + 编号规则
├── overall-spec.md               # 整体规格
├── overall-plan.md               # 整体技术方案
├── 011-version-0.1.5/            # 版本更新（spec/tasks/test-cases）
├── 011-version-0.1.7/
├── 011-version-0.2.0/            # spec/plan/tasks/test-cases
├── 011-version-0.2.1/
├── 012-building-workflow/        # spec/plan
└── 013-branding/                 # spec/plan/tasks/test-cases
```

> 按需求，不生成 `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。

---

## 5. 编号与目录映射（全工作区）

| 编号 | 父工程 | dsh-desktop | dsh-desktop-hos | dsh-desktop-ark | dsh-desktop-android |
|------|--------|-------------|-----------------|-----------------|---------------------|
| 000 | — | — | — | — | `000-scaffold`（000 例外） |
| 011 | `011-version-*` | — | — | — | — |
| 012 | `012-building-workflow` | `012-building-workflow` | —（鸿蒙暂无） | —（鸿蒙暂无） | `012-building-workflow` |
| 013 | `013-branding` | `013-branding` | `013-branding` | `013-branding` | `013-branding` |
| 101 | — | `101-runtime-launch-orchestration` | `101-runtime-launch-orchestration` | `101-runtime-launch-orchestration` | `101-runtime-launch-orchestration` |
| 102 | — | — | `102-payload-pipeline` | `102-payload-pipeline` | `102-payload-pipeline` |
| 103 | — | `103-window-shell` | `103-window-shell` | `103-arkweb-shell` | `103-webview-shell` |
| 104 | — | — | `104-host-capability-bridge` | `104-host-capability-bridge` | `104-host-capability-bridge` |
| 105 | — | `105-upstream-adapter-bundle` | `105-upstream-adapter-bundle` | `105-upstream-adapter-bundle` | `105-upstream-adapter-bundle` |
| 106 | — | — | — | `106-workspace-picker` | `106-workspace-picker` |
| 107 | — | — | — | `107-diagnostics-observability` | `107-diagnostics-observability` |
| 108 | — | — | — | `108-compliance-release` | `108-compliance-release` |
| 109 | — | — | — | —（改到 119） | `109-agent-keepalive` |
| 110 | — | — | `110-pnpm-integration` | `110-pnpm-integration` | `110-pnpm-integration` |
| 111 | — | — | `111-build-pipeline` | — | — |
| 112 | — | — | `112-tool-bash` | — | — |
| 113 | — | `113-tray` | — | — | — |
| 114 | — | `114-lifecycle-safety` | `114-lifecycle-safety` | — | — |
| 115 | — | `115-host` | `115-host` | — | — |
| 116 | — | `116-notification` | — | — | — |
| 117 | — | — | `117-runtime-provisioning` | — | — |
| 118 | — | `118-preload-bridge` | — | — | — |
| 119 | — | — | — | `119-jitless` | — |
| 201 | `201-dsh-market` | `201-dsh-market` | `201-dsh-market` | `201-dsh-market` | `201-dsh-market` |
| 202 | — | — | `202-plugin-fs-mutate` | — | — |
| 203 | — | — | `203-plugin-workspace-picker` | — | — |
| 204 | `204-plugin-git` | — | `204-plugin-git` | `204-plugin-git` | `204-plugin-git` |
| 205 | `205-plugin-branding` | — | — | `205-plugin-branding` | `205-plugin-branding` |
| 206 | — | — | — | — | `206-plugin-fs-search` |
| 301 | — | — | `301-skill-runtime-capabilities` | — | — |

---

## 6. 关键配置与生成物

| 类别 | 位置 | 说明 |
|------|------|------|
| 工作区 submodule 定义 | `.gitmodules` | 8 个 submodule |
| 父工程物料源 | `assets/` | logo / 协议 / 截图 |
| 父工程文档 | `docs/` | 能力矩阵、推广方案、上游提案 |
| 父工程规范 | `specs/` | 本目录 |
| desktop 打包 | `dsh-desktop/forge.config.ts` | 图标、maker、extraResource |
| desktop 构建 | `dsh-desktop/scripts/build-dsh.mjs` | dsh patch + 构建 + market |
| desktop CI | `dsh-desktop/.github/workflows/{ci,release}.yml` | 三平台构建 |
| hos 构建 | `dsh-desktop-hos/scripts/{collect-runtime,build-dsh,collect-dsh}.mjs` | 三阶段 |
| hos 签名 | `dsh-desktop-hos/signing.*.local.json`（gitignored） | 本机签名材料 |
| ark 构建 | `dsh-desktop-ark/scripts/{assemble-payload,package-payload,build-hap}.mjs/.sh` | 载荷 + HAP |
| ark 依赖 | `dsh-desktop-ark/oh-package.json5` | `@fellow99/node-bridge` / `@fellow99/node-runtime` |
| android 构建 | `dsh-desktop-android/gradlew` + `app/build.gradle.kts` | APK / flavor / 签名 |
| android CI | `dsh-desktop-android/.github/workflows/android-build.yml` | tag 触发 |
| android 品牌资产 | `dsh-desktop-android/scripts/generate-brand-assets.py` | 五密度图标 |
| 生成物（不入库） | `out/`、`dsh-dist/`、`runtime/`、`build/`、`jniLibs/`、`.hvigor/`、`logs/` | 各工程 `.gitignore` 管理 |

---

## 7. 维护规则

1. 新增/删除 submodule 或顶层目录 → 更新 §1、§2、§5。
2. 新增/改号 specs 模块 → 更新 §4、§5 及对应子工程 README/SPECS_CHECKLIST。
3. 生成物目录不写入源码树；二进制不入库（签名材料、`libnode.so`、载荷归档等）。
4. 目录名、编号、`FR-*` 前缀必须一致（见 `constitution.md`）。
