# 公共物料资源 功能规格

> Module: 010-assets
> 需求编号: 010-assets
> 需求名称: 公共物料资源
> Status: Specified（规格说明，待开发）
> Git 分支: main（不改动）
> Last Updated: 2026-09-09

## 1. 模块概述

### 1.1 目的（Why this module exists）

本模块统一管理本系统（父工程 + 各子工程）的**公共物料资源**——即产品 LOGO 与图标，并定义各子工程接入物料的方式。

当前各子工程使用 Electron 默认图标 / 占位图标 / 纯文本 wordmark，缺乏统一的产品视觉标识。本模块将父工程 `assets/` 下的统一 logo 物料（`logo.icns`/`logo.ico`/`logo.png`/`logo.svg`）以**复制**方式分发到各子工程，并更新各子工程代码以**加载对应物料**，实现全产品线视觉统一。

### 1.2 解决的问题（What pain points it addresses）

- **视觉不统一**：desktop 打包产物为 Electron 默认图标、托盘为空占位图标；harmony 应用图标为占位；website 无 favicon 与 logo 图。
- **应用名错误**：harmony App 在操作系统桌面显示名当前为「Electron」，不符合产品命名。
- **物料零散**：无统一物料来源，各子工程各自为政。

### 1.3 范围（Scope）

**包含**：

- 父工程 `assets/` 公共物料（logo 四种格式）作为唯一物料源。
- 物料以**复制**方式分发到三个子工程：`deepseek-harness-desktop`、`deepseek-harness-harmony`、`deepseek-harness-desktop-website`。
- 各子工程更新代码加载对应物料（应用图标、窗口图标、托盘图标、favicon、wordmark）。
- harmony App 显示名统一为「Deepseek Harness Harmony」。
- 本模块文档落于父工程 `specs/010-assets/`。

**不包含**：

- 各子工程自身的 `specs/` 规范文档**不**新增本模块（按需求注明）。
- 不修改 deepseek-harness（dsh）、dsh-market、harmonypc-electron 三个上游/运行时的源码。
- 不改变各子工程 Git 分支（均在当前分支工作）。

## 2. 物料清单

父工程 `assets/` 目录存放本产品 LOGO 与图标：

| 物料 | 路径 | 用途 |
|------|------|------|
| `logo.icns` | `assets/logo.icns` | macOS 应用图标（.icns） |
| `logo.ico` | `assets/logo.ico` | Windows 应用图标（多分辨率 .ico） |
| `logo.png` | `assets/logo.png` | 通用位图（窗口/托盘/web/鸿蒙应用图标） |
| `logo.svg` | `assets/logo.svg` | 矢量源（web favicon/wordmark） |

## 3. 用户故事

- 作为用户，我希望桌面应用（Windows/Linux/macOS）的安装包图标、窗口图标、任务栏/托盘图标显示统一的产品 LOGO，而非 Electron 默认图标。
- 作为用户，我希望鸿蒙设备桌面上 App 的图标与名称正确显示为产品 LOGO 与「Deepseek Harness Harmony」。
- 作为用户，我希望访问产品官网时看到统一的产品 LOGO（favicon + 站标）。

## 4. 各子工程物料接入功能点

> 本节列举各子工程「所需物料的功能点」，作为接入清单。

### 4.1 deepseek-harness-desktop（Electron 桌面壳）

| 功能点 | 当前状态 | 目标 |
|--------|----------|------|
| 应用图标（打包/安装） | `forge.config.ts` `packagerConfig` 无 `icon:`；`MakerSquirrel` 无 `setupIcon` | 用 `logo.ico`（Windows）/`logo.icns`（macOS）/`logo.png`（Linux）接线 |
| 窗口图标 | `src/main/windows.ts` BrowserWindow 无 `icon:` 选项 | 用 `logo.png` 接线 |
| 托盘图标 | `src/main/tray.ts:12-13` 空占位 `nativeImage.createEmpty()` | 用 `logo.png` 生成真实托盘图标 |
| 兜底页 favicon | `index.html` 无 `<link rel="icon">` | 添加 favicon 指向 logo |

### 4.2 deepseek-harness-harmony（HarmonyOS 桌面版）

| 功能点 | 当前状态 | 目标 |
|--------|----------|------|
| OS 桌面显示名 | `electron/src/main/resources/{base,en_US,zh_CN}/element/string.json` `EntryAbility_label = "Electron"` | 改为 `Deepseek Harness Harmony` |
| 应用名（app_name） | `AppScope/resources/base/element/string.json` `app_name = "DeepSeek Harness"` | 改为 `Deepseek Harness Harmony` |
| 应用图标 | `AppScope/resources/base/media/app_icon.png`（被 `$media:app_icon` 引用） | 用 `logo.png` 替换内容 |
| 启动图标 | `AppScope/resources/base/media/startIcon.png`（被 `$media:startIcon` 引用） | 用 `logo.png` 替换内容 |
| in-app 标题 | `src-main/main.js:28/48/495` `"DeepSeek Harness"` | 改为 `Deepseek Harness Harmony` |

> 注：`product_logo_32.png` 当前未被任何代码引用，属候选物料，可保留或清理（见 plan）。

### 4.3 deepseek-harness-desktop-website（产品官网）

| 功能点 | 当前状态 | 目标 |
|--------|----------|------|
| favicon | 无 `<link rel="icon">` | 添加 favicon 指向 logo |
| wordmark（顶部站标 + 底部） | `index.html`/`index_zh.html` 纯文本 `.brand` / `.footer-brand` | 替换为 logo 图片 |

> 注：`images/light.png` / `images/dark.png` 是产品**截图**（主题切换用），非 logo，保持不动。

## 5. 功能需求

### 5.1 物料源

- FR-001-001：父工程 `assets/` 下必须存在 `logo.icns`、`logo.ico`、`logo.png`、`logo.svg` 四种物料，作为唯一物料源。

### 5.2 物料分发（复制）

- FR-001-002：系统 MUST 以**复制**方式将所需物料分发到各子工程对应目录（见 §4 与 plan 文件清单），不修改物料源。

### 5.3 deepseek-harness-desktop 接入

- FR-001-003：`forge.config.ts` MUST 增加 `packagerConfig.icon`（Windows `.ico` / macOS `.icns` / Linux `.png`）与 `MakerSquirrel.setupIcon`。
- FR-001-004：`src/main/windows.ts` BrowserWindow MUST 增加 `icon:` 选项指向 logo.png。
- FR-001-005：`src/main/tray.ts` MUST 用真实 logo 图片（`nativeImage.createFromPath`）替换空占位图标。
- FR-001-006：`index.html` MUST 添加 favicon `<link rel="icon">` 指向 logo。

### 5.4 deepseek-harness-harmony 接入

- FR-001-007：三个 locale 的 `EntryAbility_label` MUST 由 `Electron` 改为 `Deepseek Harness Harmony`。
- FR-001-008：`app_name` MUST 由 `DeepSeek Harness` 改为 `Deepseek Harness Harmony`。
- FR-001-009：`app_icon.png`（及 `startIcon.png`）内容 MUST 替换为 `logo.png`。
- FR-001-010：`src-main/main.js` 的 in-app 标题字符串 MUST 统一为 `Deepseek Harness Harmony`（并镜像到打包副本）。

### 5.5 deepseek-harness-desktop-website 接入

- FR-001-011：`index.html` 与 `index_zh.html` MUST 添加 favicon 指向 logo。
- FR-001-012：`index.html` 与 `index_zh.html` 的顶部 `.brand` 与底部 `.footer-brand` 文本 wordmark MUST 替换为 logo 图片。

## 6. 关键实体

| 实体 | 说明 | 关键属性 |
|------|------|----------|
| 物料源 | 父工程 `assets/` | `logo.icns`/`logo.ico`/`logo.png`/`logo.svg` |
| 分发目标（desktop） | `deepseek-harness-desktop/resources/` | `icon.ico`/`icon.png`/`icon.icns`/`tray.png` |
| 分发目标（harmony） | `deepseek-harness-harmony/AppScope/resources/base/media/` | `app_icon.png`/`startIcon.png` |
| 分发目标（website） | `deepseek-harness-desktop-website/images/` | `logo.svg`（或 `logo.png`） |
| 应用名（harmony） | 字符串资源 | `EntryAbility_label`、`app_name` = `Deepseek Harness Harmony` |

## 7. 验收场景

### 场景：desktop 应用图标生效

- Given `npm run package` 完成
- When 查看打包产物 exe/安装器图标
- Then 显示产品 LOGO（非 Electron 默认图标）

### 场景：desktop 窗口/托盘图标

- Given 应用运行
- When 观察窗口图标与系统托盘图标
- Then 显示产品 LOGO

### 场景：harmony OS 桌面名

- Given 编译并部署到真机
- When 查看系统桌面 App 名称
- Then 显示「Deepseek Harness Harmony」（非「Electron」）

### 场景：harmony 应用图标

- Given 编译并部署到真机
- When 查看桌面 App 图标
- Then 显示产品 LOGO

### 场景：website favicon + wordmark

- Given 打开官网
- When 观察浏览器标签页图标与页面站标
- Then 显示产品 LOGO（favicon + logo 图）

## 8. 非功能需求

- **可维护性**：物料源唯一（父工程 `assets/`）；分发为一次性复制，后续物料更新按相同路径重发。
- **体积**：logo 物料极小（KB 级），对安装包体积无影响。
- **兼容性**：desktop 需同时支持 Windows（`.ico`）、macOS（`.icns`）、Linux（`.png`）；harmony 需 `.png`；web 需 `.svg`/`.png`。

## 9. 假设与约束

- **假设**：`logo.png` 分辨率/格式满足 Electron 托盘与 BrowserWindow 图标要求（实现阶段如尺寸不适再缩放）。
- **假设**：`logo.png` 满足 HarmonyOS 应用图标要求（实现阶段如格式不适再转换）。
- **约束**：不改动上游（dsh/dsh-market/harmonypc-electron）源码。
- **约束**：不改动 Git 分支；各子工程在其当前分支提交。
- **约束**：各子工程 `specs/` 不新增本模块文档。

## 10. 依赖

**上游（物料源）**：

- 父工程 `assets/`（logo 四种格式）——唯一物料源。

**下游（消费本模块）**：

- `deepseek-harness-desktop`：`forge.config.ts`、`src/main/windows.ts`、`src/main/tray.ts`、`index.html`、`resources/`。
- `deepseek-harness-harmony`：`AppScope/`（app.json5、string.json、media/）、`electron/src/main/resources/*/element/string.json`、`src-main/main.js`。
- `deepseek-harness-desktop-website`：`index.html`、`index_zh.html`、`images/`。
