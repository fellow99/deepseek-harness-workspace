# ARCHITECTURE.md — 整体架构

> 工程：`deepseek-harness-workspace`
> 本文档描述整个工作区的整体架构：分层、四个交付壳的运行时模型、共享数据面、构建/发布拓扑与关键决策。
> 各子工程的具体实现见其 `specs/ARCHITECTURE.md` 与模块 `plan.md`。
> Last Updated: 2026-11-01

---

## 1. 架构总览

工作区不是单一应用，而是**一个上游宿主 + 四个交付壳 + 一个运行时/市场生态**的聚合：

```
┌────────────────────────────────────────────────────────────────────┐
│                         用户 / 操作系统                              │
│   Windows · Linux · macOS        HarmonyOS 2in1/tablet        Android │
└───────┬──────────────────────────────┬──────────────────────────┬──┘
        │                              │                          │
┌───────▼────────┐        ┌────────────▼───────────┐   ┌──────────▼─────────┐
│  dsh-desktop   │        │ dsh-desktop-hos        │   │ dsh-desktop-ark    │
│  Electron 43   │        │ Electron 37 on Harmony │   │ ArkWeb + libnode   │
│  Chromium+Node │        │ Chromium + Node 22.17  │   │ Node 24.2 (in-proc)│
└───────┬────────┘        └────────────┬───────────┘   └──────────┬─────────┘
        │                              │                          │
        │                    ┌─────────▼─────────┐       ┌────────▼─────────┐
        │                    │ dsh-desktop-android│      │ 系统能力 / HAP     │
        │                    │ WebView + libnode  │      │ ArkWeb / ArkUI     │
        │                    │ Node 24.22 (in-proc)│      └────────────────────┘
        │                    └─────────┬─────────┘
        └──────────────┬───────────────┴───────────────┬────────────────────┘
                       │                               │
              ┌────────▼────────┐              ┌───────▼────────┐
              │  dsh Host       │              │  dsh Web UI    │
              │  webserver      │  same-origin │  (upstream)    │
              │  apiProxy       │◄────────────►│  WebApiClient  │
              │  connection     │  HTTP + WS   │                │
              └────────┬────────┘              └────────────────┘
                       │
        ┌──────────────┼───────────────────────┐
        │              │                       │
┌───────▼──────┐ ┌─────▼──────────┐ ┌──────────▼─────────┐
│ dsh-market   │ │ dsh-plugins /  │ │ 平台 seam 插件      │
│ (dshmarket)  │ │ 壳专用 plugins │ │ (@dsh-ohos/android) │
└──────────────┘ └────────────────┘ └────────────────────┘
```

**核心不变量：**

1. dsh Host 在应用进程内启动，webserver 同时服务 SPA dist 与 `/api`。
2. 渲染层同源加载该 webserver，复用上游 `WebApiClient`（HTTP + WebSocket）。
3. 不 fork 上游 Web UI，不新造业务协议；平台差异通过 profile / bundle / seam 插件 / 垫片 / 少量 patch 解决。
4. 四个壳共享同一套规格编号与能力语义，便于跨平台互译。

---

## 2. 分层架构

| 层 | 职责 | 主要落点 |
|----|------|----------|
| L1 交付壳层 | 应用生命周期、窗口、托盘、通知、权限、单实例 | Electron main / EntryAbility / MainActivity |
| L2 运行时层 | 提供 Node 运行时并启动 dsh Host | Electron Node / `libnode.so` + NAPI 桥 |
| L3 Host 层 | dsh Host、webserver、apiProxy、connection | `runProfile` / `node::Start` + bootstrap |
| L4 适配层 | profile / bundle / patch / seam 插件 / 垫片 | `profiles/`、`packages/`、`patches/`、`payload/shims/` |
| L5 能力层 | 文件、shell、搜索、git、通知、剪贴板、市场、品牌 | dsh 工具 + 平台能力桥 |
| L6 展示层 | 上游 dsh Web UI（未修改） | webserver 服务的 SPA dist |

---

## 3. 四个壳的运行时模型

### 3.1 dsh-desktop（Electron）

```
Electron main (Node)
  runProfile('desktop', ['--port','0'])
    → { ctx, shutdown, port, url }
  BrowserWindow.loadURL(`http://127.0.0.1:${port}/`)
  tray / notifications 订阅 ctx 事件
```

- 数据面：loopback 同源。
- 平台适配：3 个 dsh patch（HMR off、native picker off、welcome notice off）。
- 生命周期：关窗即退出；`before-quit` 优雅关闭 Host。

### 3.2 dsh-desktop-hos（Electron on HarmonyOS）

```
EntryAbility (ArkTS) → Electron-on-HarmonyOS runtime
  → main process (Node 22.17)
    → dsh Host（webserver 0.0.0.0:<port>）
  → BrowserWindow 加载局域网 IP / 同源 URL
```

- 数据面：HarmonyOS loopback 隔离 → 绑定 wildcard + 在 session 内改写 `Host`/`Origin` 回 loopback。
- 平台适配：13 个 dsh patch（symlink→copy、allow-all-interfaces、flock/hardlink fallback、fs 原语、可写根、rebrand 等）。
- 安全边界：仅对本应用内嵌渲染进程改写；局域网其他设备仍受 loopback-only 特权围栏限制。

### 3.3 dsh-desktop-ark（ArkWeb + Node）

```
EntryAbility / Index.ets (ArkWeb)
  → node::Start(libnode.so) in app process
    → bootstrap.js 解包 payload → host.js 启动 dsh Host
  → ArkWeb 加载 http://127.0.0.1:<port>/?token=...
  → 303 签名 cookie → 同源 Web UI
```

- 数据面：loopback 同源 + 一次性 token → HttpOnly/SameSite cookie。
- 平台适配：`ohos` profile + `@dsh-ohos/bundle` + seam 插件 + 可审计平台垫片（白名单见 `102-payload-pipeline`）。
- 运行形态：默认 `compat`（`--jitless`，零 ACL）；可选 `jit`（需 `ALLOW_WRITABLE_CODE_MEMORY`）。
- 单进程、单实例、Node 不可重启：崩溃即终态。

### 3.4 dsh-desktop-android（WebView + Node）

```
MainActivity (Kotlin/View)
  → DshRuntime 启动 libnode（JNI 槽位表桥）
  → bootstrap.js 解包 payload → dsh Host（127.0.0.1:<port>）
  → WebShellView 加载 http://127.0.0.1:<port>/?token=...
  → HostCapabilities（Kotlin↔Node JSON 桥）
  → KeepAliveService（agent turn 期间 dataSync FGS）
```

- 数据面：loopback 同源 + 一次性 token；`networkSecurityConfig` 仅放行 loopback 明文。
- 平台适配：`android` profile + `@dsh-android/bundle` + seam 插件 + 垫片 S1–S5 + 物化期补丁。
- 权限：full flavor 含 All-files；store flavor SAF-only；按需触发，不冷启批量弹窗。

---

## 4. 共享数据面

四个壳都遵循**同源数据面**：

```
Renderer (dsh Web UI)
   │  HTTP  /api/*        → dsh webserver → apiProxy → Host 插件
   │  WebSocket /api/ws   → dsh connection
   ▼
dsh Host（应用进程内）
```

- **不新增 IPC 载体**：壳层能力通过 dsh 插件/能力桥暴露为 Host 能力，而不是自定义 renderer↔main 协议。
- **鉴权**：桌面/Electron 版依赖 loopback 边界；ark/android 使用一次性 token → 签名 cookie；壳不解析 token。
- **CORS**：不需要；渲染层与 API 同源。
- **失败语义**：Host 启动失败 → 兜底页/错误页；不以白屏或静默重试掩盖。

---

## 5. 构建与发布拓扑

### 5.1 版本与上游锁定

```
deepseek-harness (tag dsh-v0.2.1-rc.2)
        │
        ├── dsh-desktop        → patches/dsh-v0.2.1-rc.2/ + build-dsh
        ├── dsh-desktop-hos    → patches/dsh-v0.2.1-rc.2/ + build-dsh
        ├── dsh-desktop-ark    → assemble-payload（profile/bundle/seam）→ dsh.tar.gz
        └── dsh-desktop-android→ assemble-payload → dsh.tar（裸 ustar）
```

- 桌面：Electron Forge `make` 产出 exe/dmg/deb/rpm/zip；GitHub Actions 三平台矩阵。
- hos：`collect-runtime → build-dsh → collect-dsh → tar → hvigor assembleHap`；AppGallery 签名。
- ark：`assemble-payload → package-payload → hvigor assembleHap`；ohpm 依赖 + `runtime.lock.json`。
- android：`assemble-payload → package-payload → provision-engine → gradlew assembleFull*`；GitHub Actions tag 构建 + Release。

### 5.2 发布物流

| 壳 | 产物 | 分发 |
|----|------|------|
| desktop | 安装包（未签名） | GitHub Releases |
| hos | HAP（debug/release 签名） | AppGallery / 侧载 debug |
| ark | HAP（debug/release 签名） | AppGallery / 侧载 |
| android | APK（full/store） | GitHub Releases / Play（规划） |

---

## 6. 关键架构决策

| 决策 | 理由 | 代价 |
|------|------|------|
| Host 进程内运行 | 同源、零 CORS、零自定义协议、复用上游 | Host 崩溃 = 应用级崩溃（MVP 接受） |
| 不 fork 上游 UI | 跟随上游演进，升级成本低 | 平台差异需 profile/seam/patch 适配 |
| 平台差异走扩展点 | 上游零补丁是目标 | 需要维护 seam 插件与垫片 |
| ark/android 使用进程内 Node | 本地离线、隐私、无远端依赖 | Node 不可重启；平台限制多 |
| pnpm 纯 JS 随包 | 绕开 spawn/symlink 限制 | 包体增大；需 worker 调度 |
| 统一 specs 编号 | 跨工程互译、复用需求 | 改号需要同步索引与 `FR-*` |
| 证据优先 | 避免“看起来可用”的假结论 | 文档维护成本高 |

---

## 7. 安全与合规边界

| 边界 | 规则 |
|------|------|
| 渲染进程 | `sandbox:true` + `contextIsolation:true` + `nodeIntegration:false`（Electron） |
| 导航 | 仅允许应用自身 URL；外部 URL 交系统浏览器 |
| 明文网络 | 仅 loopback；Android `networkSecurityConfig` 不放行通配域名 |
| 凭据 | 不入库、不进日志；签名材料 gitignored / Secrets |
| 权限 | 最小化、按需触发；store flavor 不声明 All-files |
| 上游 | 不 fork；patch 必须幂等、可审计、按 tag 分目录 |
| 二进制 | 不入库；SHA-256 锁定；构建期投放 |

---

## 8. 与各子工程文档的关系

- 父工程（本目录）：整体规划、设计、共同需求、跨工程编号与能力语义。
- 子工程 `specs/`：具体技术实现、独立需求、模块 `spec.md` / `plan.md`。
- 能力语义以 `docs/能力对照清单.md` 为跨版本事实源；各子工程 `overall-spec.md` 与模块 `spec.md` 为工程内事实源。
- 任一跨工程差异（如应用名 `DSH Desktop` vs `DSH Desktop Ark`）必须在对应模块 spec 中显式登记。
