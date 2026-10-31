# 品牌化 功能规格（父工程）

> Module: 013-branding（原 `010-assets` 并入）
> 范围：父工程 + `dsh-desktop`、`dsh-desktop-hos`、`dsh-desktop-ark`、`dsh-desktop-android`（官网 `dsh-desktop-website` 作为物料消费者一并约束）
> Status: Implemented（as-built；各工程实现程度不同）
> Last Updated: 2026-11-01

## 1. 模块概述

### 1.1 目的（Why this module exists）

本模块统一管理整个工作区的**产品身份**：公共物料源（logo/图标）、各交付版本的应用名/应用图标/窗口图标/通知图标/启动标题，以及应用级品牌与 Web UI 品牌位的边界。

`010-assets` 的“公共物料资源”职责并入本模块；品牌化不再只是“复制图标”，还包括应用名、标题、通知图标与 Web 品牌位的跨工程一致性。

### 1.2 解决的问题

- 四个交付版本曾各自为政：桌面用 Electron 默认图标、HarmonyOS 显示 “Electron”、Android 用占位图标。
- 应用级品牌（launcher/通知/窗口）与 Web UI 品牌（侧栏/hero/标题）容易混淆，导致重复实现或遗漏。
- 物料来源不统一，换 logo 需要多处修改。
- 平台规范不同：HarmonyOS 应用图标要求不透明底；Android 通知图标要求白色剪影；桌面需要 ico/icns/png。

### 1.3 范围

**包含：**

- 父工程 `assets/` 作为唯一物料源。
- 四个交付版本的应用级品牌：应用名、应用/启动/窗口/通知图标、启动/窗口标题。
- 品牌物料的分发/生成规则（复制或脚本生成）。
- 应用级品牌与 `205-plugin-branding`（Web UI 品牌位）的职责边界。
- 官网 favicon/wordmark 的物料消费（不作为主交付版本）。

**不包含：**

- Web UI 内部品牌位（侧栏 logo/名称、hero、document title）的具体实现——由各工程的 `205-plugin-branding` 负责。
- 包名、签名、版本号、商店 listing 文案（属 `108-compliance-release` / `011-version-*`）。
- 商标注册、法务审核。

## 2. 用户故事

- 作为用户，我希望四个版本在各自平台上都显示统一的产品 logo 与名称，而不是上游/框架默认身份。
- 作为用户，我希望 HarmonyOS/Android 的桌面图标、窗口图标、通知图标符合平台规范且清晰可辨。
- 作为维护者，我希望换 logo/改产品名只改一个物料源与少量固定字符串，不碰业务逻辑。
- 作为维护者，我希望应用级品牌与 Web 品牌位职责清晰，不重复实现。

## 3. 产品身份基线

| 交付版本 | 应用级名称 | Web 品牌名 | 品牌位实现 |
|----------|------------|------------|------------|
| dsh-desktop | `DSH Desktop` | 上游 UI（未做 Web 品牌位替换） | 应用级：`013-branding` |
| dsh-desktop-hos | `DSH Desktop` | `DSH Desktop`（`dsh-rebrand` patch） | 应用级 013 + patch |
| dsh-desktop-ark | `DSH Desktop Ark` | `DSH Desktop`（`205-plugin-branding`） | 应用级 013 + 205 |
| dsh-desktop-android | `DSH Desktop` | `DSH Desktop`（`205-plugin-branding`） | 应用级 013 + 205 |

> 应用名 `DSH Desktop Ark` 用于区分同一设备上的 Electron 版；该差异必须在 ark 的 013 spec 中显式登记。

## 4. 功能需求

### 4.1 物料源

- FR-013-001：父工程 `assets/` MUST 是唯一物料源，至少包含 `logo.svg`、`logo.png`、`logo.ico`、`logo.icns`、`logo-1024-with-bg.png`（不透明底）、`logo-200-with-bg.png`。
- FR-013-002：物料更新 MUST 按固定文件名覆盖；MUST NOT 要求各子工程改代码引用路径。
- FR-013-003：物料源缺失时，分发/生成脚本 MUST 大声失败，MUST NOT 产生残缺产物。

### 4.2 应用级品牌（各子工程）

- FR-013-004：每个交付版本 MUST 有稳定的应用名，并与窗口/启动标题一致。
- FR-013-005：桌面版 MUST 提供 Windows `.ico`、macOS `.icns`、Linux `.png` 打包图标，以及窗口/托盘图标。
- FR-013-006：HarmonyOS 版 MUST 提供应用图标、启动图标、窗口图标；应用图标 MUST 为不透明底，窗口/状态栏图标 MAY 为透明底。
- FR-013-007：Android 版 MUST 提供五密度启动图标（方形/圆形）与通知小图标；通知小图标 MUST 为白色 + alpha 剪影。
- FR-013-008：应用级品牌实现 MUST NOT 修改 `deepseek-harness` 上游源码（Web 品牌位除外，见 4.3）。
- FR-013-009：实现 Web 品牌位的版本，其应用级品牌 MUST 与 `205-plugin-branding` 的 Web 品牌名保持一致；未实现 Web 品牌位的版本（desktop）MUST 在 spec 中登记该差异。允许平台后缀差异，但 MUST 登记。

### 4.3 Web 品牌位边界

- FR-013-010：Web UI 内部品牌位 MUST 由 `205-plugin-branding`（或等效模块）负责，MUST NOT 混入应用级图标/名称模块。
- FR-013-011：Web 品牌位 MUST 优先使用上游扩展点（槽位、locale 命名空间、index 注入、配置项）；Electron 版使用 patch 替换的方式 MUST 在 spec 中登记。
- FR-013-012：应用级品牌与 Web 品牌位的名称差异 MUST 在对应子工程 013 spec 中显式说明，避免被误判为缺陷。

### 4.4 一致性

- FR-013-013：四个版本的品牌名 MUST 来自同一产品命名基线（`DSH Desktop` / `DSH Desktop Ark`）。
- FR-013-014：同一版本内，应用名、窗口标题、Web 品牌名 MUST 一致（允许已登记的 Ark 后缀差异）。
- FR-013-015：品牌变更 MUST 同步更新各子工程 013 spec、README 与 `SPECS_CHECKLIST`。

## 5. 关键实体

| 实体 | 说明 | 关键属性 |
|------|------|----------|
| 物料源 | 父工程 `assets/` | svg/png/ico/icns + 不透明底变体 |
| 应用身份 | 各版本应用名 | `DSH Desktop` / `DSH Desktop Ark` |
| 应用图标 | 桌面/HarmonyOS/Android | 格式、密度、透明度要求 |
| 通知图标 | Android | 白色 + alpha 剪影 |
| Web 品牌位 | 205/rebrand | 侧栏、hero、标题 |

## 6. 验收场景

### 场景：桌面版品牌

- Given 打包并运行 dsh-desktop
- When 查看安装包、窗口、托盘
- Then 图标为产品 logo，应用名为 `DSH Desktop`

### 场景：HarmonyOS 版品牌

- Given 安装 HAP 到真机
- When 查看桌面、窗口、状态栏、Web UI
- Then 应用名为 `DSH Desktop`（ark 为 `DSH Desktop Ark`），图标正确，Web 品牌位为 `DSH Desktop`

### 场景：Android 版品牌

- Given 安装 APK
- When 查看启动器、通知栏、Web UI
- Then 应用名为 `DSH Desktop`，启动图标正确，通知小图标为白色剪影，Web 品牌位为 `DSH Desktop`

### 场景：物料源缺失

- Given 临时移除 `assets/logo.png`
- When 执行物料分发/生成
- Then 脚本失败并指出缺失物料，不产生残缺产物

## 7. 非功能需求

- **平台合规**：HarmonyOS 应用图标不透明底；Android 通知图标白色剪影；桌面多格式。
- **体积**：图标为 KB–MB 级，不显著影响包体。
- **可维护性**：物料源唯一；生成/复制路径固定。
- **可重复性**：生成脚本幂等。
- **证据**：真机/打包截图为准。

## 8. 假设与约束

- 假设 `assets/logo.png` 为透明底品牌图，尺寸满足各平台缩放。
- 约束：不改上游源码；Web 品牌位走扩展点/patch，并在 205/patch 中登记。
- 约束：包名/签名/版本号不属本模块。
- 约束：官网不作为主交付版本，但其物料消费遵循同一物料源。

## 9. 依赖

- 物料：父工程 `assets/`。
- 下游：`dsh-desktop`（resources/forge/window/tray）、`dsh-desktop-hos`（AppScope/electron/main.js/rebrand patch）、`dsh-desktop-ark`（AppScope/entry/205）、`dsh-desktop-android`（res/manifest/generate-brand-assets/205）、`dsh-desktop-website`（favicon/wordmark）。
