# 013-branding 测试用例（父工程）

> Module: 013-branding
> 对应规格: [spec.md](./spec.md) ｜ 对应方案: [plan.md](./plan.md)
> Last Updated: 2026-11-01

## 1. 测试环境

| 环境 | 用途 |
|------|------|
| dsh-desktop（Windows/Linux/macOS） | 打包 + 运行时图标/名称 |
| dsh-desktop-hos（HarmonyOS 真机） | 桌面名/图标、窗口/状态栏、Web 品牌位 |
| dsh-desktop-ark（HarmonyOS 真机） | 桌面名/图标、boot 标题、Web 品牌位 |
| dsh-desktop-android（模拟器/真机） | 启动器/通知/Web 品牌位 |
| dsh-desktop-website（浏览器） | favicon/wordmark |

## 2. 功能测试用例

### TC-013-001 物料源完整

- **步骤**：检查父工程 `assets/`。
- **预期**：存在 `logo.svg`、`logo.png`、`logo.ico`、`logo.icns`、`logo-1024-with-bg.png`、`logo-200-with-bg.png`。

### TC-013-002 桌面打包图标

- **前置**：`npm run make` 完成。
- **步骤**：查看安装器与可执行文件图标。
- **预期**：显示产品 logo（非 Electron 默认图标）。

### TC-013-003 桌面窗口/托盘

- **步骤**：运行应用，观察窗口/任务栏/托盘。
- **预期**：图标为产品 logo；托盘 tooltip 为 `DSH Desktop`。

### TC-013-004 桌面应用名

- **步骤**：检查 `package.json` `productName` 与窗口标题。
- **预期**：均为 `DSH Desktop`。

### TC-013-005 HarmonyOS 应用名/图标

- **前置**：安装 HAP 到真机。
- **步骤**：查看系统桌面与设置。
- **预期**：hos 为 `DSH Desktop`；ark 为 `DSH Desktop Ark`；图标为产品 logo。

### TC-013-006 HarmonyOS 图标合规

- **步骤**：检查 `app_icon.png` / `startIcon.png` / `icon.png` / `product_logo_32.png`。
- **预期**：均为不透明底；窗口/状态栏图标为透明底。

### TC-013-007 HarmonyOS boot/in-app 标题

- **步骤**：冷启动应用，观察 boot/loading 页与窗口标题。
- **预期**：hos 为 `DSH Desktop`；ark 为 `DSH Desktop Ark`；无上游文案。

### TC-013-008 HarmonyOS Web 品牌位

- **步骤**：加载 dsh Web UI，查看侧栏/hero/标题（中英文）。
- **预期**：品牌名为 `DSH Desktop`，无上游 “DSH Local Build”/鱼形 logo；首次启动无自动弹窗。

### TC-013-009 Android 启动器

- **前置**：安装 `fullDebug` APK。
- **步骤**：查看启动器/最近任务/设置。
- **预期**：应用名 `DSH Desktop`；方形/圆形图标正确。

### TC-013-010 Android 通知图标

- **步骤**：触发 agent 完成/审批通知。
- **预期**：小图标为白色剪影 `ic_stat_dsh`，不是白方块。

### TC-013-011 Android Web 品牌位

- **步骤**：加载 dsh Web UI，查看侧栏/hero/标题。
- **预期**：品牌名为 `DSH Desktop`，无上游文案。

### TC-013-012 Android 生成脚本幂等

- **步骤**：连续运行两次 `python scripts/generate-brand-assets.py`。
- **预期**：两次输出一致；无报错。

### TC-013-013 官网品牌

- **步骤**：浏览器打开官网中英文页。
- **预期**：favicon/wordmark 为产品 logo。

## 3. 边界与负向用例

### TC-013-101 物料源缺失

- **步骤**：临时移除 `assets/logo.png` 后执行 Android 生成脚本。
- **预期**：脚本失败并指出缺失路径；不产生残缺图标。

### TC-013-102 托盘图标加载失败

- **步骤**：临时移走 `tray.png` 后启动 desktop。
- **预期**：记录 warning，回退空图标，应用不崩溃。

### TC-013-103 名称不一致扫描

- **步骤**：`grep` 各工程应用名/标题/205 品牌常量。
- **预期**：同一版本内一致；ark 的 `DSH Desktop Ark` 差异已在 spec 登记。

## 4. 实测记录

| 用例 | 工程 | 状态 | 证据 |
|------|------|------|------|
| TC-013-001~004 | desktop | ✅ | 打包/运行时截图（历史记录） |
| TC-013-005~008 | hos | ✅ | 真机截图（0.2.0/0.2.1） |
| TC-013-005~008 | ark | ✅ | 真机 `ready.json` + 截图（0.2.1） |
| TC-013-009~012 | android | 🟡 | 源码/模拟器证据；真机回归待做 |
| TC-013-013 | website | ✅ | 浏览器检查 |
