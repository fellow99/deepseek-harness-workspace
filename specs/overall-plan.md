# overall-plan.md — 整体技术方案

> 工程：`deepseek-harness-workspace`
> 本文档是 `overall-spec.md` 的技术对应物：描述整个工作区 **HOW** 建成，以及四个交付壳的实现策略。
> 具体模块实现见各子工程 `specs/<module>/plan.md`。
> Last Updated: 2026-11-01

---

## 1. 技术上下文

### 1.1 运行时环境

| 版本 | 运行时 | 构建环境 | 产物 |
|------|--------|----------|------|
| dsh-desktop | Electron 43（内置 Node） | Node 22 + Electron Forge + Vite | exe/dmg/deb/rpm/zip |
| dsh-desktop-hos | Electron 37 / Node 22.17（harmonypc-electron） | DevEco Studio / hvigor + Node 18+ | HAP |
| dsh-desktop-ark | ArkWeb + `libnode.so` 24.2.0（`node::Start`） | DevEco CLT / hvigor + Node 18+ | HAP |
| dsh-desktop-android | 系统 WebView + `libnode.so` 24.22.1（JNI） | Gradle 8.12 / AGP 8.10.1 / JDK 17 | APK |

### 1.2 关键依赖

| 依赖 | 版本 | 作用 |
|------|------|------|
| `deepseek-harness` | `dsh-v0.2.1-rc.2` | Host、Web UI、profile/bundle |
| `dsh-market` | 各壳锁定 | 插件市场 |
| `harmonypc-electron` | Electron 37 / Node 22.17 | HarmonyOS Electron 运行时 |
| `@fellow99/node-bridge` / `@fellow99/node-runtime` | 0.1.0 / 24.2.0 | ark 的 NAPI 桥与 Node 引擎 |
| `io.github.fellow99:node-bridge` | 0.2.0 | android 的 JNI 桥 |
| `aosp-node-runtime` | 24.22.1 | android 的 Node 引擎 |
| pnpm（纯 JS） | 10.34.6 | ark / android / hos 插件安装 |
| better-sqlite3 / node:sqlite | ABI 138 / 内置 | 会话持久化与 FTS |

---

## 2. 宪法合规检查

| 原则 | 状态 | 说明 |
|------|------|------|
| C1 上游唯一事实源，只做装配 | ✅ | 四壳均复用上游 Web UI；平台差异走 profile/bundle/seam/patch |
| C2 进程内 Host + 同源数据面 | ✅ | 四壳统一 `runProfile` / `node::Start` + 同源 webserver |
| C3 平台真相不隐藏 | ✅ | 能力矩阵 + `not-implemented` 纪律 |
| C4 证据优先 | ✅ | 各壳 `logs/`、`docs/`、`SPECS_CHECKLIST` 记录证据 |
| C5–C8 编号规则 | ✅ | 本次整理统一为 0XX/1XX/2XX，见 `constitution.md` |
| C9–C11 文档分工/as-built | ✅ | 父工程只写整体；子工程写实现；无 API/data-model 文档 |
| C12 Conventional Commits | ✅ | 本次提交按 `docs:` / `chore:` 等前缀 |
| C13 代码审查 | ✅ | 本次整理使用 `requesting-code-review` / `receiving-code-review` |
| C15 安全与合规 | ✅ | 签名材料/凭据不入库；权限最小化 |

---

## 3. 实现策略总览

### 3.1 共同策略：Host 装配 + 同源 UI

四个壳共享同一套装配范式：

1. 准备 dsh 运行树（构建 / 物化 / 解包）。
2. 在应用进程内启动 Host（`runProfile` 或 `node::Start` + `bootstrap`）。
3. 读取实际端口与 URL。
4. 渲染层同源加载该 URL。
5. 壳层能力通过 dsh 插件/能力桥暴露；不新增 IPC 载体。
6. 退出时优雅关闭 Host（或明确终态）。

### 3.2 dsh-desktop

- 主进程 `host.ts` 动态 import dsh `runProfile`，`desktop` profile 启动 Host。
- `windows.ts` 创建 BrowserWindow 并加载 loopback URL；窗口状态持久化 + F11。
- `tray.ts` / `notifications.ts` 订阅 `ctx` 事件。
- 构建：`scripts/build-dsh.mjs` 应用 3 个 patch、构建 dsh + dsh-market；`collect-dsh.mjs` 物化；`fetch-runtime.mjs` 准备便携 Node/pnpm。
- 打包：Electron Forge maker（Squirrel/deb/rpm/dmg/zip）；CI 三平台矩阵。

### 3.3 dsh-desktop-hos

- ArkTS `EntryAbility` 启动 Electron-on-HarmonyOS 运行时；主进程 `src-main/main.js` 编排 Host 与窗口。
- 三阶段构建：`collect-runtime`（copy 运行时 + 定制）→ `build-dsh`（13 patch + 构建）→ `collect-dsh`（物化 + sharp/sqlite/preset 适配）。
- HarmonyOS 适配：HOME 沙箱、loopback 隔离（wildcard + Host/Origin 改写）、symlink/hardlink 替代、fs 原语 patch。
- 打包：`dsh-dist.tar.gz` 单文件 + hvigor assembleHap；签名走本地 `signing.*.local.json`。

### 3.4 dsh-desktop-ark

- ArkTS `EntryAbility` / `Index.ets` 承载 ArkWeb；`@fellow99/node-bridge` 在应用进程内 `node::Start`。
- `payload/bootstrap.js` 解包 `dsh.tar.gz` → `host.js` 启动 Host；`ready.json` 通告端口/token。
- 适配：`ohos` profile + `@dsh-ohos/bundle` + seam 插件（subprocess/shell/git/market/branding/workspace-picker 等）+ 平台垫片（白名单见 `102-payload-pipeline`）。
- 运行形态：`compat`（`--jitless`，零 ACL）默认；`jit` 可选。
- 构建：`assemble-payload` → `package-payload` → `hvigorw assembleHap`；`runtime.lock.json` 锁定引擎。

### 3.5 dsh-desktop-android

- Kotlin `MainActivity` + `DshRuntime` 启动 `libnode.so`（JNI 槽位表桥）；`bootstrap.js` 解包裸 ustar 载荷。
- `WebShellView` 加载 `http://127.0.0.1:<port>/?token=...`；`HostCapabilities` 提供 17 项冻结能力。
- 适配：`android` profile + `@dsh-android/bundle` + seam 插件 + 垫片 S1–S5 + 物化期补丁。
- 构建：`assemble-payload` → `package-payload` → `provision-engine` → `gradlew assembleFull*`；GitHub Actions tag 构建 + Release。
- 保活：`KeepAliveService`（dataSync FGS）仅在 agent turn 期间运行。

---

## 4. 跨工程横切关注点

### 4.1 版本与锁

- 上游 dsh tag、Node 引擎、桥、market 版本必须成对锁定；升级走 `011-version-*`。
- SHA-256 锁定：`runtime.lock.json`（ark/android）、`runtime/`（desktop）、`dsh.version`（载荷）。
- 应用版本：SemVer；Android/HarmonyOS `versionCode = major*10000+minor*100+patch`。

### 4.2 错误处理

- Host 启动失败 → 返回 `null` / `FAILED` 终态 + 错误页；不阻塞壳启动。
- 能力 handler 永不抛：结构化 `{ok:false,error,hint}`。
- 未捕获异常：记录日志，不静默退出（平台允许时）。
- 载荷/运行时校验失败：阻止进入半可用状态。

### 4.3 日志与诊断

- 统一前缀/通道：desktop `[dsh-desktop]`；hos `[dsh-harmony]` + hilog；ark `app.log` + `ready.json` + `/facts`；android logcat + `/_android/facts`。
- 诊断信息包含版本、运行时、Host 状态、错误、日志尾；debug-only 端点需门禁。
- 不记录 token/凭据/API Key。

### 4.4 安全

- Electron：sandbox + contextIsolation + nodeIntegration false + fuse + 导航白名单。
- HarmonyOS：默认零 ACL（ark compat）；host 侧鉴权；壳不改写 token。
- Android：loopback-only 明文；full/store flavor 权限分离；SAF 两段式。
- 签名材料 gitignored / Secrets；仓库无二进制。

### 4.5 性能

- 载荷流式解包 + 幂等 marker；避免一次性读入内存。
- 端口 0、单实例；避免多实例争用。
- 包体：ark 不打包 Chromium；android 单 ABI；desktop 便携运行时按需。

---

## 5. 测试与验收策略

| 层级 | 方式 | 证据 |
|------|------|------|
| L0 静态/单元 | 脚本断言、YAML/JSON 校验、类型检查、`selfcheck` | 命令输出 |
| L1 本机集成 | 启动应用、对话、工具调用、插件安装、能力桥 | 截图/日志 |
| L2 真机/模拟器 | HarmonyOS 真机、Android 模拟器/真机 | hilog/logcat + 截图 |
| L3 构建/发布 | CI 绿、HAP/APK 签名、版本/哈希校验 | Actions 日志、`aapt`/`apksigner` |
| 回归 | 四版能力矩阵逐项核对 | `docs/能力对照清单.md` + 各壳 `SPECS_CHECKLIST` |

验收原则：

- 每个 `FR-*` 至少在 `spec.md` 有验收场景；关键能力在 `test-cases.md` 或 `SPECS_CHECKLIST` 有证据。
- 未实现能力标 `planned` / `not-implemented`，不得在测试中当成通过。
- 版本升级后必须重跑能力矩阵并登记差异。

---

## 6. 构建与发布

| 壳 | 构建命令（摘要） | 产物 | 发布 |
|----|------------------|------|------|
| desktop | `npm run build:dsh` → `npm run make` | 三平台安装包 | GitHub Releases |
| hos | `collect-runtime` → `build-dsh` → `collect-dsh` → `build-debug.ps1` | HAP | AppGallery / 侧载 debug |
| ark | `assemble-payload --deploy --apply` → `build-hap.sh` | HAP | AppGallery / 侧载 |
| android | `assemble-payload` → `package-payload` → `provision-engine` → `gradlew assembleFull*` | APK | GitHub Releases / Play（规划） |

发布检查：

- 版本号/构建号单调递增。
- 上游 tag、运行时版本、market 版本与文档一致。
- 权限/签名/包体预算符合平台要求。
- 已知限制写入发布说明。

---

## 7. 风险与缓解

| 风险 | 影响 | 缓解 |
|------|------|------|
| 上游 dsh rc 漂移 | patch/profile 失效、构建失败 | 按 tag 锁定 + patch 幂等 + 升级回归 |
| HarmonyOS 平台限制 | 能力缺口 | 能力矩阵登记 + seam/垫片 + 明确降级 |
| Node 不可重启（ark/android） | 崩溃即终态 | 单实例 + 崩溃日志 + 重启应用入口 |
| 签名/证书限制 | 无法分发 | 提前准备签名材料；debug/release 分离 |
| 包体过大 | 安装/审核失败 | 不打包 Chromium、单 ABI、载荷压缩 |
| 文档与实现漂移 | 误导开发/验收 | as-built + 证据 + 定期 `SPECS_CHECKLIST` 对账 |

---

## 8. 实施顺序（本次整理）

1. 建立父工程宪法与编号规则。
2. 统一父工程 0XX 模块（011/012/013），合并 `010-assets` → `013-branding`。
3. 各子工程按基准改号（1XX），解决 `012` 冲突（pnpm → 110）。
4. 补齐各子工程 `013-branding`；desktop/android 补 `012-building-workflow`。
5. 更新 README / SPECS_CHECKLIST / constitution / 交叉引用。
6. 代码审查（requesting-code-review + receiving-code-review）。
7. 按 Conventional Commits 分工程提交。
