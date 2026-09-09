# 010-assets 技术方案

> 本文档为「公共物料资源」的技术实现方案。
> Module: 010-assets
> 对应规格: [spec.md](./spec.md)
> 对应任务: [tasks.md](./tasks.md)
> 对应测试: [test-cases.md](./test-cases.md)
> Status: Specified（待开发）
> Last Updated: 2026-09-09

## 1. 技术上下文

### 1.1 运行环境

- **父工程**：submodule 容器工作区，持有唯一物料源 `assets/`，无构建产物。
- **deepseek-harness-desktop**：Electron `^43` + Electron Forge + Vite + TypeScript；应用图标经 `packagerConfig.icon`（打包）与 `BrowserWindow.icon`/`Tray`（运行时）加载。
- **deepseek-harness-harmony**：HarmonyOS Stage 模型 ArkTS；应用名经 `EntryAbility_label` / `app_name` 字符串资源；图标经 `$media:app_icon` / `$media:startIcon`。
- **deepseek-harness-desktop-website**：纯静态站点（HTML/CSS/JS），无构建。

### 1.2 物料源

| 物料 | 大小 | 目标用途 |
|------|------|----------|
| `logo.ico` | ~169KB（多分辨率） | desktop Windows 打包/安装图标 |
| `logo.icns` | ~2KB | desktop macOS 打包图标 |
| `logo.png` | ~4KB | desktop 窗口/托盘 + harmony 应用图标 |
| `logo.svg` | ~2KB（矢量） | website favicon/wordmark |

## 2. 宪法合规检查

| 原则 | 状态 | 说明 |
|------|------|------|
| 架构：零上游改动 | ✅ | 不改 dsh/dsh-market/harmonypc-electron 源码 |
| 架构：只写装配代码 | ✅ | 纯物料复制 + 引用接线，无业务逻辑 |
| 安全：不引入敏感信息 | ✅ | logo 为公开资源 |
| 代码质量：源码即真理 | ✅ | 引用点均已对照源码核实（行号见 spec §4） |
| Git：不改动分支 | ✅ | 各子工程在当前分支工作 |

> 合规结论：全部 ✅。

## 3. 研究结论

### 3.1 关键决策与理由

| 决策 | 理由 |
|------|------|
| 物料复制到 `resources/`（desktop） | Electron Forge 约定 `resources/` 存放应用图标；`packagerConfig.icon` 直接引用源文件路径 |
| 托盘图标复用 `logo.png` | 托盘图标即小尺寸位图；`logo.png` 满足，无需单独 `tray.png`（如有尺寸问题再单独生成） |
| harmony 图标**覆盖** `app_icon.png`/`startIcon.png` 内容 | 保持 `$media:` 引用不变，零代码改动，仅替换位图字节 |
| harmony 名改动覆盖三 locale + app_name + in-app | 需求明确「全部统一」，避免局部残留「Electron」或「DeepSeek Harness」 |
| website 用 `logo.svg` 作 favicon + wordmark | SVG 矢量自适应缩放，适合站点；favicon 现代浏览器支持 SVG |
| `product_logo_32.png` 保留不动 | 未被引用，非本模块职责；清理需另行决策 |

### 3.2 源码级核实（引用点）

- desktop 托盘空占位：`src/main/tray.ts:12-13`（`nativeImage.createEmpty()`）。
- desktop 窗口无图标：`src/main/windows.ts:73-97`（`new BrowserWindow({...})` 无 `icon:`）。
- desktop 打包无图标：`forge.config.ts:12-22`（`packagerConfig` 无 `icon:`）、`:26-28`（`MakerSquirrel` 无 `setupIcon`）。
- harmony 显示名错误：`electron/src/main/resources/{base,en_US,zh_CN}/element/string.json:13`（`EntryAbility_label = "Electron"`）。
- harmony 应用名：`AppScope/resources/base/element/string.json:5`（`app_name = "DeepSeek Harness"`）。
- harmony in-app 标题：`src-main/main.js:28/48/495` + 打包副本 `web_engine/src/main/resources/resfile/resources/app/main.js`。
- website 无 favicon/logo：`index.html`/`index_zh.html` 无 `<link rel="icon">`，wordmark 为纯文本 `.brand`/`.footer-brand`。

## 4. 数据模型

无新增数据实体。仅涉及静态资源文件与字符串常量。

| 变更类型 | 说明 |
|----------|------|
| 文件复制 | 物料从父工程 `assets/` 复制到各子工程目标目录 |
| 配置修改 | `forge.config.ts`（icon 接线） |
| 字符串修改 | harmony 显示名/应用名/标题 |
| HTML 修改 | website favicon + wordmark；desktop 兜底页 favicon |

## 5. 接口契约

### 5.1 deepseek-harness-desktop

| 文件 | 变更 |
|------|------|
| `resources/icon.ico`（新） | 复制 `logo.ico` |
| `resources/icon.png`（新） | 复制 `logo.png` |
| `resources/icon.icns`（新） | 复制 `logo.icns` |
| `resources/tray.png`（新） | 复制 `logo.png` |
| `forge.config.ts`（改） | `packagerConfig.icon` + `MakerSquirrel.setupIcon` |
| `src/main/windows.ts`（改） | BrowserWindow `icon:` 选项 |
| `src/main/tray.ts`（改） | `nativeImage.createFromPath(...)` 替换空占位 |
| `index.html`（改） | `<link rel="icon">` favicon |

### 5.2 deepseek-harness-harmony

| 文件 | 变更 |
|------|------|
| `AppScope/resources/base/media/app_icon.png`（覆盖） | 替换为 `logo.png` 内容 |
| `AppScope/resources/base/media/startIcon.png`（覆盖） | 替换为 `logo.png` 内容 |
| `AppScope/resources/base/element/string.json`（改） | `app_name` → `Deepseek Harness Harmony` |
| `electron/src/main/resources/base/element/string.json`（改） | `EntryAbility_label` → `Deepseek Harness Harmony` |
| `electron/src/main/resources/en_US/element/string.json`（改） | 同上 |
| `electron/src/main/resources/zh_CN/element/string.json`（改） | 同上 |
| `src-main/main.js`（改） | 标题 3 处 → `Deepseek Harness Harmony` |
| `web_engine/src/main/resources/resfile/resources/app/main.js`（改） | 镜像同步 |

### 5.3 deepseek-harness-desktop-website

| 文件 | 变更 |
|------|------|
| `images/logo.svg`（新） | 复制 `logo.svg` |
| `index.html`（改） | favicon + `.brand`/`.footer-brand` 换 logo 图 |
| `index_zh.html`（改） | 同上 |
| `style.css`（改，如需要） | logo 图尺寸/对齐规则 |

## 6. 实现策略

### 6.1 架构模式

**复制 + 接线**：物料从唯一源复制到目标，代码侧仅做引用接线（无业务逻辑、无上游改动）。

### 6.2 关键实现细节

- **desktop 图标解析**：`forge.config.ts` 的 `icon` 字段在 Windows 用 `.ico`、macOS 用 `.icns`、Linux 用 `.png`；Electron Packager 按平台自动匹配，跨平台可提供分平台字段或直接给 `.ico`（Windows 主目标）。
- **desktop 运行时图标路径**：开发态用 `__dirname`/项目相对路径；打包态图标已由 Packager 内嵌，窗口/托盘图标在打包态可由 `process.resourcesPath` 或直接内嵌资源解析（实现时按现有 `windows.ts`/`tray.ts` 的资源加载方式对齐）。
- **harmony 图标**：直接覆盖 PNG 字节，`$media:` 引用不变，最稳。
- **website**：favicon 用 `<link rel="icon" type="image/svg+xml" href="images/logo.svg">`；wordmark 用 `<img src="images/logo.svg" class="brand-logo">`，CSS 补尺寸规则。

### 6.3 错误处理

- 复制物料：源文件缺失时构建/复制脚本大声失败（避免残缺产物）。
- 图标路径解析失败：`nativeImage.createFromPath` 返回空时记录 warn（沿用现有空占位兜底），不阻塞启动。

## 7. 测试考虑

- **desktop**：`npm run package` 后验证 exe/安装器图标；运行时验证窗口/托盘图标。
- **harmony**：编译 + 部署真机，验证桌面名「Deepseek Harness Harmony」与图标。
- **website**：浏览器打开验证 favicon + wordmark。
- 详见 [test-cases.md](./test-cases.md)。

## 8. 文件清单

| 文件 | 用途 |
|------|------|
| `specs/010-assets/spec.md` | 功能规格 |
| `specs/010-assets/plan.md` | 技术方案（本文档） |
| `specs/010-assets/tasks.md` | 任务拆解 |
| `specs/010-assets/test-cases.md` | 测试用例 |
| `deepseek-harness-desktop/resources/icon.{ico,png,icns}` + `tray.png`（新） | 复制物料 |
| `deepseek-harness-desktop/forge.config.ts`（改） | 打包图标 |
| `deepseek-harness-desktop/src/main/windows.ts`（改） | 窗口图标 |
| `deepseek-harness-desktop/src/main/tray.ts`（改） | 托盘图标 |
| `deepseek-harness-desktop/index.html`（改） | favicon |
| `deepseek-harness-harmony/AppScope/resources/base/media/{app_icon,startIcon}.png`（覆盖） | 应用图标 |
| `deepseek-harness-harmony/AppScope/resources/base/element/string.json`（改） | app_name |
| `deepseek-harness-harmony/electron/src/main/resources/*/element/string.json`（改×3） | EntryAbility_label |
| `deepseek-harness-harmony/src-main/main.js` + resfile 副本（改） | in-app 标题 |
| `deepseek-harness-desktop-website/images/logo.svg`（新） | favicon/wordmark 物料 |
| `deepseek-harness-desktop-website/index.html` + `index_zh.html`（改） | favicon + wordmark |

## 9. 与规格的交叉引用

| 规格需求 | 实现位置 |
|----------|----------|
| FR-001-001（物料源） | 父工程 `assets/` |
| FR-001-002（复制分发） | 各子工程 `resources/`、`AppScope/.../media/`、`images/` |
| FR-001-003~006（desktop） | `forge.config.ts` + `windows.ts` + `tray.ts` + `index.html` |
| FR-001-007~010（harmony） | 3×`string.json` + `app_name` + media + `main.js` |
| FR-001-011~012（website） | `index.html`/`index_zh.html` |
