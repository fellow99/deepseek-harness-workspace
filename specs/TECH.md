# TECH.md — 父工程技术选型

> 工程：`deepseek-harness-workspace`
> 记录整个工作区（父工程 + 四个交付壳 + 上游/运行时）的技术栈、版本与选型理由。
> 版本以各子工程实际清单为准：`package.json`、`oh-package.json5`、`gradle/libs.versions.toml`、`runtime.lock.json`。
> Last Updated: 2026-11-01

---

## 1. 技术栈总览

| 层 | 技术 | 版本/基线 | 作用域 | 用途 |
|----|------|-----------|--------|------|
| 上游宿主 | deepseek-harness（dsh） | `dsh-v0.2.1-rc.2` | 全工作区 | Agent Host、Web UI、profile/bundle 插件机制 |
| 插件市场 | dsh-market（`dshmarket`） | 各壳分别锁定（如 1.66.42 / v1.26.0） | 四个壳 | 可视化插件市场 |
| 桌面运行时 | Electron | `43.0.0` | dsh-desktop | Chromium + Node 主进程 |
| 桌面构建 | Electron Forge | `^7.11.2` | dsh-desktop | start/package/make 生命周期 |
| 桌面前端构建 | Vite | `^7.3.6` | dsh-desktop | main/preload/renderer 编译 |
| 桌面语言 | TypeScript | `^5.6.0`（strict） | dsh-desktop | 类型安全 |
| HarmonyOS Electron 运行时 | harmonypc-electron | Electron 37 / Node 22.17.0 | dsh-desktop-hos | Electron-on-HarmonyOS |
| HarmonyOS UI | ArkTS / ArkUI（Stage） | 随 SDK | hos / ark | 应用入口、窗口、Web 容器 |
| HarmonyOS Web 容器 | ArkWeb / `web_engine` HAR | 随系统 / 本工程 | hos / ark | 渲染 dsh Web UI |
| HarmonyOS 构建 | hvigor / DevEco CLT | 4.0+ / 6.1.1.280 | hos / ark | HAP 构建与签名 |
| HarmonyOS SDK | HarmonyOS SDK | API 24（compatible 17） | hos / ark | 编译/打包 |
| HarmonyOS Node（hos） | Electron 内置 Node | 22.17.0 | dsh-desktop-hos | 主进程跑 dsh Host |
| HarmonyOS Node（ark） | `@fellow99/node-runtime`（`libnode.so`） | 24.2.0（NODE_MODULE_VERSION 137） | dsh-desktop-ark | 进程内 `node::Start` |
| HarmonyOS NAPI 桥 | `@fellow99/node-bridge` | 0.1.0（ohpm） | dsh-desktop-ark | `dlopen` + `node::Start` |
| Android 语言 | Kotlin | 2.2.20（jvmTarget 17） | dsh-desktop-android | 外壳 / 能力桥 |
| Android 构建 | Gradle / AGP | 8.12 / 8.10.1 | dsh-desktop-android | APK 构建 |
| Android SDK | Android SDK | compile/target 36，min 29，arm64-v8a | dsh-desktop-android | 系统 WebView 壳 |
| Android Node | `aosp-node-runtime`（`libnode.so`） | 24.22.1 small-icu，SHA-256 锁定 | dsh-desktop-android | 进程内 Node |
| Android NAPI 桥 | `io.github.fellow99:node-bridge` | 0.2.0（Maven Central） | dsh-desktop-android | JNI 槽位表桥 |
| 包管理器（Node 侧） | pnpm（纯 JS） | 10.34.6（ark/android 随包） | ark / android / hos | 插件市场安装通道 |
| 包管理器（构建） | npm / pnpm / corepack | 各工程锁定 | 全部 | 依赖安装与构建 |
| 工作区组织 | git submodule | 8 个 | 父工程 | 固定上游/兄弟工程版本 |
| 文档体系 | Markdown + specs-as-built | — | 全部 | as-built 规范、计划、验收 |

---

## 2. 四个交付壳的技术对比

| 维度 | dsh-desktop | dsh-desktop-hos | dsh-desktop-ark | dsh-desktop-android |
|------|-------------|-----------------|-----------------|---------------------|
| 目标平台 | Windows / Linux / macOS | HarmonyOS 2in1 / tablet | HarmonyOS 2in1 / tablet | Android arm64-v8a |
| 渲染引擎 | 内置 Chromium（Electron 43） | 内置 Chromium（Electron 37） | 系统 ArkWeb | 系统 WebView |
| Node 来源 | Electron 内置 | Electron 内置（22.17.0） | `libnode.so` 24.2.0（ohpm） | `libnode.so` 24.22.1（jniLibs） |
| Host 运行位置 | Electron 主进程 | Electron 主进程 | 应用进程内 `node::Start` | 应用进程内 `node::Start` |
| 数据面 | 同源 loopback | 同源 + 局域网 IP（loopback 隔离适配） | 同源 loopback + 一次性 token | 同源 loopback + 一次性 token |
| 上游 patch | 3 个 | 13 个 | 0 个（profile/bundle/seam + 平台垫片） | 0 个（profile/bundle/seam + 垫片 S1–S5） |
| 包形态 | exe/dmg/deb/rpm/zip | HAP | HAP | APK（full/store flavor） |
| 签名 | 未签名 | debug/release 签名（AppGallery） | debug/release 签名（AppGallery） | keystore + CI secrets |
| 构建工作流 | GitHub Actions CI/Release | 本地 DevEco / hvigor | 本地 DevEco CLT / hvigor | GitHub Actions tag 构建 |
| 特色能力 | 托盘、原生通知、窗口状态 | Electron 全量 API、better-sqlite3 | 零 ACL 默认（compat）、小包体 | SAF、保活、纯 JS 搜索 |

---

## 3. 选型理由

### 3.1 为什么统一“进程内 Host + 同源 Web UI”

- dsh 已完成 Host/Client 拆分，webserver 同时服务 SPA dist 与 `/api`。
- 四个壳都让 Host 跑在应用进程内，渲染层直接加载该 webserver：
  - 零 CORS、零鉴权、零自定义协议、零新 IPC 载体；
  - 复用上游 `WebApiClient`（HTTP 上行 + WebSocket 下行）；
  - 上游 Web UI 不做 fork，升级 dsh 只需重跑构建管线。
- 这是父工程最核心的架构约束（见 `constitution.md`）。

### 3.2 为什么桌面用 Electron / 鸿蒙有两条路线 / Android 用 WebView

- **Electron 桌面**：需要成熟的窗口、托盘、通知、系统集成；Electron 43 的 Node 版本满足 dsh 要求。
- **HarmonyOS Electron 版（hos）**：最大化桌面保真度，复用 Electron 运行时与全部 Node API；代价是包体大、需要 JIT ACL。
- **HarmonyOS Edition（ark）**：不打包 Chromium，使用系统 ArkWeb + `libnode.so`；包体小、默认无 ACL（JIT-less），代价是平台能力缺口需要 seam 插件/垫片补齐。
- **Android Edition**：系统 WebView + `libnode.so`，与 ark 架构同构，便于规格互译；使用 Kotlin/JNI 桥与 SAF 权限模型。

### 3.3 为什么 pnpm 以纯 JS 随包

- ark/android 无法在应用域 spawn 随包 ELF（无二进制证书 / SELinux 限制），也不能依赖系统 node/pnpm。
- 将 pnpm 的纯 JS 安装引擎物化进包内，由宿主进程内 worker 调用，绕开 spawn/symlink/hardlink 限制。
- 市场通过官方 `desktopProfiles` / `desktopPnpm` 契约接入，无需修改 dsh-market 源码。

### 3.4 为什么用 submodule 组织工作区

- 四个壳的构建脚本都期望 `../deepseek-harness`、`../dsh-market`、`../harmonypc-electron` 同级目录。
- submodule 固定上游/兄弟工程版本，保证“同一工作区可复现构建”。
- 通用插件/技能放父工程 `dsh-plugins/`、`skills/`；壳专用放各自 `plugins/`、`skills/`。

---

## 4. 版本策略

| 对象 | 策略 |
|------|------|
| dsh 上游 | 每个壳锁定一个 tag；升级 = 新增 `patches/<new-tag>/` + 更新构建脚本 pin + 全量回归 |
| Electron | 桌面锁定 `43.0.0`；hos 锁定 Electron 37（随 harmonypc-electron） |
| Node（ark） | 与 `@fellow99/node-runtime` 版本号一致（24.2.0），SHA-256 锁定 |
| Node（android） | 与 `aosp-node-runtime` 版本号一致（24.22.1），`runtime.lock.json` 锁定 |
| 应用版本 | SemVer；Android/HarmonyOS 的 `versionCode = major*10000 + minor*100 + patch`，发布严格递增 |
| dsh-market | 各壳按自身基线锁定（桌面 CI 当前 `v1.26.0`；ark/android 随载荷锁定） |

---

## 5. 明确否决的选型

| 选项 | 否决理由 |
|------|----------|
| fork / 长期 patch dsh 前端 | 上游 rc 迭代快，fork 等于放弃升级 |
| 自研原生聊天 UI（ArkUI/Compose） | 必须跟随上游前端演进，重写等于永久分叉 |
| 在 ArkWeb/WebView 内嵌第二个 JS 引擎（QuickJS/Hermes） | 不是 Node，Host 起不来 |
| 远端/云端 Node | 违背本地离线与隐私边界 |
| 随包 ELF 并在应用域 spawn（ark/android） | 无二进制证书 / SELinux 拒绝 |
| 固定端口 | 端口冲突与多实例风险；统一 `--port 0` |
| 在组件内直接调用后端 API（绕过 Host） | 破坏同源数据面与鉴权边界 |

---

## 6. 开发与验证工具

| 工具 | 用途 |
|------|------|
| `git` + submodule | 工作区版本固定 |
| `npm` / `pnpm` / `corepack` | 依赖安装与构建 |
| Electron Forge CLI | 桌面 start/package/make |
| DevEco Studio / hvigor / hdc | HarmonyOS 构建、签名、真机安装与日志 |
| Android Gradle Plugin / adb | APK 构建、安装、logcat |
| `uitest` / `snapshot_display` / `hilog` | HarmonyOS UI 自动化与证据 |
| `aapt` / `apksigner` / `zipalign` | APK 版本、签名、对齐校验 |
| Markdown specs + `requesting-code-review` / `receiving-code-review` / `git-commit` | 规范驱动、评审、提交 |
