# 012-building-workflow 技术方案（父工程 As-Built）

> 本文档记录整个工作区构建工作流的实际实现与跨工程约定。
> Module: 012-building-workflow
> 对应规格: [spec.md](./spec.md)
> Last Updated: 2026-11-01

## 1. 技术上下文

### 1.1 工作流分布

| 工程 | 工作流文件 | 触发 | 产物 |
|------|-----------|------|------|
| dsh-desktop | `.github/workflows/ci.yml` | push main / PR | 三平台 `out/make/` |
| dsh-desktop | `.github/workflows/release.yml` | tags `v*` / `workflow_dispatch` | 三平台安装包 + GitHub Release |
| dsh-desktop-android | `.github/workflows/android-build.yml` | tags `v*` / `workflow_dispatch` | APK + GitHub Release |
| dsh-desktop-hos | —（无 Actions） | DevEco / hvigor 本地 | HAP |
| dsh-desktop-ark | —（无 Actions） | DevEco CLT / hvigor 本地 | HAP |

### 1.2 运行环境

| 项 | 桌面版 | Android 版 |
|----|--------|------------|
| Runner | windows-latest / ubuntu-latest / macos-latest | ubuntu-latest |
| Node | 22（setup-node） | 22（setup-node） |
| 包管理器 | npm + corepack/pnpm | npm + pnpm@11.28.5 |
| JDK | — | 17 |
| Android SDK | — | compileSdk 36 / build-tools 36.0.0 |
| 签名 | 无（未签名产物） | GitHub Secrets（KEYSTORE_BASE64 等） |

---

## 2. 宪法合规

| 原则 | 状态 | 说明 |
|------|------|------|
| C1 上游唯一事实源 | ✅ | 只 checkout 上游并应用既有 patch/装配，不改上游源码 |
| C4 证据优先 | ✅ | Run summary 输出上游 ref、引擎/载荷/APK 哈希 |
| C12 Conventional Commits | ✅ | 工作流变更按 `ci:` / `build:` 提交 |
| C15 安全与合规 | ✅ | 签名材料零入库；Secrets 注入；二进制不入库 |
| C5/C6 编号规则 | ✅ | 模块号 012，与父工程/子工程一致 |

---

## 3. 桌面版实现

### 3.1 CI（`.github/workflows/ci.yml`）

1. `actions/checkout@v7` 检出壳工程。
2. `git clone` dsh（锁定 tag）与 dsh-market（锁定 tag）到 `../` 同级目录。
3. `actions/setup-node@v7`（Node 22，npm 缓存）。
4. `corepack enable`（dsh 内部嵌套 pnpm 调用）。
5. Linux 安装 `fakeroot` / `rpm`。
6. `npm ci` → `npm run build:dsh` → `npm run make`。
7. `actions/upload-artifact@v7` 上传 `out/make/`，`if-no-files-found: error`。

并发组 `ci-${{ github.ref }}`，`cancel-in-progress: true`。

### 3.2 Release（`.github/workflows/release.yml`）

- 触发：tags `v*` / `workflow_dispatch`（输入 `dsh_ref`、`dsh_market_ref`、`version`）。
- build job：与 CI 相同的三平台矩阵与构建步骤。
- release job：`actions/download-artifact@v8` 汇总 → `chmod -R +x` 恢复执行位 → `softprops/action-gh-release@v3` 创建 Release，匹配 exe/nupkg/deb/rpm/dmg/zip。

### 3.3 关键约束

- dsh / dsh-market 必须同级目录，`actions/checkout` 的 `path` 不能越出 workspace，故用 `git clone`。
- 版本锁定点：workflow 的 `--branch`、`scripts/build-dsh.mjs` 的 pin、`patches/<dsh-tag>/`。
- 无签名/公证/自动更新；产物为本地可安装包。

---

## 4. Android 版实现

### 4.1 触发与版本

- `push` tags `v*`；`workflow_dispatch` 手动彩排。
- `versionName` / `versionCode` 从 tag 派生，注入 Gradle 属性与载荷 `dsh.version`。
- `versionCode = MAJOR*10000 + MINOR*100 + PATCH`。

### 4.2 构建步骤（摘要）

1. checkout 主仓 + `aosp-node-runtime`（锁定 commit/归档）。
2. 下载引擎归档 → `archiveSha256` 对账 → `tools/provision.mjs` 八项 ELF 门禁 → `provision-engine.mjs` 投放 jniLibs。
3. checkout dsh（tag）与 dsh-market（tag）；Node 22 + pnpm@11.28.5。
4. `assemble-payload --deploy --apply` → `package-payload`（注入版本）。
5. setup-java 17 + Android SDK（build-tools 36.0.0 / platforms 36）。
6. `:app:assembleFullDebug`（彩排）或 `:app:assembleFullRelease`（tag + Secrets 签名）。
7. `aapt dump badging` 校验版本；`apksigner` 校验证书；收集 APK sha256。
8. `upload-artifact` + `gh release create`（或 action-gh-release）。

### 4.3 签名与安全

- `keystore.properties` / `*.keystore` gitignored；`keystore.properties.sample` 入库。
- CI 通过 `KEYSTORE_BASE64` / `KEYSTORE_PASSWORD` / `KEY_ALIAS` / `KEY_PASSWORD` 物化。
- release 构建缺任一 Secret 即失败；彩排仍出 debug 包。
- 二进制（jniLibs、dsh-payload、keystore）不入库。

---

## 5. 父工程协同

- 版本更新 `011-version-*` 的任务清单必须列出各 workflow 的 tag 更新。
- 父工程 README 版本表与 submodule 指针在发布后同步。
- 四版能力矩阵在版本升级后重跑，差异登记到 `docs/能力对照清单.md`。

---

## 6. 测试与证据

| 检查 | 方式 | 证据 |
|------|------|------|
| YAML 合法性 | 解析 + 结构断言 | CI 运行 |
| 版本注入 | `aapt dump badging` / `version` 输出 | 构建日志 |
| 引擎/载荷哈希 | sha256 对账 | Run summary |
| 签名 | `apksigner verify --print-certs` | Run 日志 |
| 制品 | Release 附件下载 | Release 页面 |
| 三平台 | matrix 结果 | Actions 页面 |

---

## 7. 风险与缓解

| 风险 | 缓解 |
|------|------|
| 上游 tag 不可用/漂移 | 锁定 tag + clone 失败即失败；升级走 `011-version-*` |
| pnpm/corepack 缺失 | 显式 `corepack enable`；Android 固定 pnpm 版本 |
| 签名 Secrets 缺失 | release job 硬失败；不回落 debug |
| 制品执行位丢失 | `chmod -R +x` |
| 包体超预算 | 载荷压缩 + 单 ABI + 体积核对 |

---

## 8. 文件清单

| 文件 | 用途 |
|------|------|
| `dsh-desktop/.github/workflows/ci.yml` | 桌面版三平台 CI |
| `dsh-desktop/.github/workflows/release.yml` | 桌面版三平台发布 |
| `dsh-desktop-android/.github/workflows/android-build.yml` | Android tag 构建/发布 |
| `dsh-desktop/scripts/build-dsh.mjs` | dsh patch + 构建 + market |
| `dsh-desktop-android/scripts/assemble-payload.mjs` | Android 载荷装配 |
| `dsh-desktop-android/scripts/package-payload.mjs` | Android 载荷打包 |
| `dsh-desktop-android/scripts/provision-engine.mjs` | Android 引擎投放 |
| `dsh-desktop-android/app/build.gradle.kts` | 版本注入/签名/flavor |
