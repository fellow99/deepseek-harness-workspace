# 公共物料资源 测试用例

> Module: 010-assets
> 对应规格: [spec.md](./spec.md)
> 对应方案: [plan.md](./plan.md)
> Last Updated: 2026-09-09

## 1. 测试环境

| 环境 | 用途 |
|------|------|
| deepseek-harness-desktop（Windows/Linux） | `npm run package` 编译 + 运行时图标验证 |
| deepseek-harness-harmony（真机已连线） | 编译 + 部署真机，验证桌面名/图标 |
| deepseek-harness-desktop-website | 浏览器打开验证 favicon/wordmark |

## 2. 功能测试用例

### TC-001 desktop 物料就位

- **步骤**：检查 `deepseek-harness-desktop/resources/` 目录。
- **预期**：存在 `icon.ico`、`icon.png`、`icon.icns`、`tray.png`。

### TC-002 desktop 打包图标

- **前置**：`npm run package` 完成。
- **步骤**：查看打包产物 `out/` 下 exe 与安装器图标。
- **预期**：显示产品 LOGO（非 Electron 默认图标）。

### TC-003 desktop 窗口图标

- **步骤**：运行应用，观察窗口标题栏/任务栏图标。
- **预期**：显示产品 LOGO。

### TC-004 desktop 托盘图标

- **步骤**：运行应用，观察系统托盘图标。
- **预期**：显示产品 LOGO（非空图标）。

### TC-005 harmony 物料就位

- **步骤**：检查 `deepseek-harness-harmony/AppScope/resources/base/media/`。
- **预期**：`app_icon.png`、`startIcon.png` 内容为产品 LOGO。

### TC-006 harmony OS 桌面名

- **前置**：编译并部署真机。
- **步骤**：查看系统桌面 App 名称。
- **预期**：显示「Deepseek Harness Harmony」（非「Electron」）。

### TC-007 harmony 应用图标

- **前置**：编译并部署真机。
- **步骤**：查看桌面 App 图标。
- **预期**：显示产品 LOGO。

### TC-008 harmony in-app 标题

- **步骤**：启动 App，观察加载页标题。
- **预期**：显示「Deepseek Harness Harmony」（非「DeepSeek Harness」）。

### TC-009 website favicon

- **步骤**：浏览器打开官网。
- **预期**：标签页显示产品 LOGO favicon。

### TC-010 website wordmark

- **步骤**：浏览器打开官网（中/英文页）。
- **预期**：顶部站标与底部品牌区显示产品 LOGO 图片。

## 3. 边界与负向用例

### TC-101 物料源缺失

- **步骤**：临时移除某物料源后执行分发。
- **预期**：复制/构建脚本大声失败，不产生残缺产物。

### TC-102 图标加载失败降级

- **步骤**：模拟托盘图标路径不可解析。
- **预期**：记录 warn，不阻塞应用启动。

## 4. 测试通过标准

- TC-001~TC-010 全部通过。
- TC-101~TC-102 按预期降级/失败，无崩溃。

## 5. 实测结果

> 待测试阶段填写。
