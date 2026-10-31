# 013-branding 技术方案（父工程 As-Built）

> 本文档记录整个工作区品牌化的实际实现与物料分发路径。
> Module: 013-branding（原 `010-assets` 并入）
> 对应规格: [spec.md](./spec.md)
> Last Updated: 2026-11-01

## 1. 技术上下文

### 1.1 物料源

父工程 `assets/`：

| 物料 | 用途 |
|------|------|
| `logo.svg` | 官网 favicon/wordmark、矢量场景 |
| `logo.png` | 桌面窗口/托盘、HarmonyOS 窗口/状态栏、Android 源图 |
| `logo.ico` | Windows 打包/安装器图标 |
| `logo.icns` | macOS 打包图标 |
| `logo-1024-with-bg.png` | HarmonyOS 应用图标（不透明底） |
| `logo-200-with-bg.png` | 不透明底备用位图 |

### 1.2 各工程实现落点

| 工程 | 应用名 | 图标落点 | Web 品牌位 |
|------|--------|----------|------------|
| dsh-desktop | `productName = DSH Desktop` | `resources/{icon.ico,icon.icns,icon.png,tray.png}`；`forge.config.ts` | 无（复用上游 UI） |
| dsh-desktop-hos | `AppScope` `app_name` / `EntryAbility_label` | `AppScope/.../media/*`、`window_icon`、`electron_white` | `dsh-rebrand.patch` |
| dsh-desktop-ark | `AppScope` `app_name = DSH Desktop Ark` | `AppScope/.../media/app_icon.png`、`entry/.../media/{startIcon,icon}.png` | `205-plugin-branding` |
| dsh-desktop-android | `res/values/strings.xml` `app_name` | `res/mipmap-*/ic_launcher*.png`、`res/drawable-*/ic_stat_dsh.png` | `205-plugin-branding` |
| dsh-desktop-website | 站点文案 | `images/logo.svg` | — |

---

## 2. 宪法合规

| 原则 | 状态 | 说明 |
|------|------|------|
| C1 上游零补丁 | ✅ / ⚠️ | hos 的 Web 品牌位用 patch（已登记）；其余不动上游 |
| C5 013 模块 | ✅ | 原 010-assets 并入 013，父工程 + 四子工程均有 013 |
| C8 编号即契约 | ✅ | 013/205 分工写入宪法与各 README |
| C11 as-built | ✅ | 图标/字符串/脚本路径均来自实际文件 |
| C15 安全 | ✅ | 物料为公开资源；签名材料不属本模块 |

---

## 3. 各工程实现

### 3.1 dsh-desktop

- 物料复制：`assets/logo.ico → resources/icon.ico`，`logo.icns → icon.icns`，`logo.png → icon.png` / `tray.png`。
- 打包：`forge.config.ts` 的 `packagerConfig.icon = 'resources/icon'`（按平台自动补扩展名），`MakerSquirrel.setupIcon = 'resources/icon.ico'`。
- 运行时：`BrowserWindow.icon` 指向 `resources/icon.png`（打包态 `process.resourcesPath`）；`Tray` 用 `tray.png`，失败回退空图标 + warning。
- 应用名：`package.json` `productName`；窗口标题/托盘 tooltip/兜底页标题 `DSH Desktop`。
- `extraResource` 包含 `icon.png` / `tray.png`，保证打包态可解析。

### 3.2 dsh-desktop-hos

- 应用名：`AppScope/resources/base/element/string.json` `app_name = DSH Desktop`；三 locale `EntryAbility_label = DSH Desktop`。
- 应用图标：`app_icon.png` / `startIcon.png` / `icon.png` / `product_logo_32.png` 使用不透明底 `logo-1024-with-bg.png`。
- 窗口图标：新增 `window_icon.png`（透明 `logo.png`），由 `electron/src/main/module.json5` 的 ability `icon` 引用。
- 状态栏图标：`web_engine/.../resfile/resources/app/electron_white.png`（透明），列入 `OVERLAY_FILES`。
- in-app 标题：`src-main/main.js` 的 loading/页面/窗口/about 标题统一 `DSH Desktop`；打包副本同步。
- Web 品牌位：`patches/dsh-v0.2.1-rc.2/dsh-rebrand.patch` 替换 `brand.localBuild`（中英）、`hero.headline`（中英）与 hero logo；`dsh-disable-welcome-notice.patch` 关闭首次引导自动弹窗。

### 3.3 dsh-desktop-ark

- 应用名：`AppScope` `app_name = DSH Desktop Ark`；`entry` `EntryAbility_label = DSH Desktop Ark`。
- 图标：`AppScope/.../media/app_icon.png`、`entry/.../media/{startIcon,icon}.png`。
- 标题：`Index.ets` 启动/错误覆盖层、`rawfile/boot.html` 标题 `DSH Desktop Ark`。
- Web 品牌位：`packages/branding-ohos`（`205-plugin-branding`）使用槽位 + 自有 locale 命名空间 + `webServer.tapIndex` + index 注入，上游零补丁。

### 3.4 dsh-desktop-android

- 应用名：`res/values/strings.xml` `app_name = DSH Desktop`；Manifest `android:label="@string/app_name"`。
- 启动图标：`scripts/generate-brand-assets.py` 从 `../assets/logo.png` 生成五密度 `ic_launcher.png` / `ic_launcher_round.png`（白底 + 86% logo；圆形裁切）。
- 通知图标：同脚本生成五密度 `ic_stat_dsh.png`（白色 alpha 剪影，内缩 92%）。
- Manifest：`android:icon="@mipmap/ic_launcher"`、`android:roundIcon="@mipmap/ic_launcher_round"`。
- Web 品牌位：`packages/branding-android`（`205-plugin-branding`，移植自 branding-ohos）。

### 3.5 dsh-desktop-website

- `images/logo.svg` 从父工程复制；`index.html` / `index_zh.html` 添加 favicon 与 wordmark。

---

## 4. 物料分发与生成

| 工程 | 方式 | 命令/脚本 |
|------|------|-----------|
| desktop | 复制 | 手工/构建前复制（`resources/`） |
| hos | 复制/覆盖 | 物料覆盖 `AppScope/.../media/*` |
| ark | 复制 | 物料复制到 `AppScope`/`entry` media |
| android | 生成 | `python scripts/generate-brand-assets.py` |
| website | 复制 | `assets/logo.svg → images/logo.svg` |

**原则**：物料源只读；目标路径固定；生成脚本幂等；缺失源文件大声失败。

---

## 5. 平台规范

| 平台 | 应用图标 | 窗口/状态栏 | 通知 |
|------|----------|-------------|------|
| Windows | `.ico` 多分辨率 | `.png` | — |
| macOS | `.icns` | `.png` | — |
| Linux | `.png` | `.png` | — |
| HarmonyOS | 不透明底 PNG | 透明 PNG | 状态栏图标 |
| Android | 五密度 legacy/round | — | 白色 alpha 剪影 |

---

## 6. 与 205 的边界

| 模块 | 负责 | 不负责 |
|------|------|--------|
| 013-branding | 应用名、launcher/启动/窗口/通知图标、启动标题、物料源 | Web UI 侧栏/hero/标题 |
| 205-plugin-branding | Web UI 品牌位、hero、document title、首次引导 | 应用图标/包名/签名 |

---

## 7. 测试与证据

- 桌面：`npm run make` 后检查安装包/窗口/托盘图标；打包态资源解析。
- hos：真机桌面名/图标、窗口/状态栏图标、Web 品牌位（中英）。
- ark：真机桌面名/图标、boot 标题、Web 品牌位（205）。
- android：启动器/通知栏/Web 品牌位；生成脚本重复运行稳定性。
- 官网：favicon/wordmark。
- 证据：截图、`aapt`/资源检查、`grep` 名称一致性。

---

## 8. 文件清单

| 文件/目录 | 用途 |
|-----------|------|
| `assets/*` | 唯一物料源 |
| `dsh-desktop/resources/*`、`forge.config.ts`、`src/main/{windows,tray}.ts`、`index.html` | 桌面品牌 |
| `dsh-desktop-hos/AppScope/*`、`electron/.../string.json`、`src-main/main.js`、`patches/.../dsh-rebrand.patch` | hos 品牌 |
| `dsh-desktop-ark/AppScope/*`、`entry/.../media/*`、`entry/.../Index.ets`、`rawfile/boot.html` | ark 应用级品牌 |
| `dsh-desktop-android/scripts/generate-brand-assets.py`、`res/mipmap-*`、`res/drawable-*`、`res/values/strings.xml`、`AndroidManifest.xml` | Android 品牌 |
| `packages/branding-ohos` / `packages/branding-android` | Web 品牌位（205） |
| `dsh-desktop-website/images/logo.svg`、`index*.html` | 官网品牌 |
