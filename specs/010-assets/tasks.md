# 010-assets 任务拆解

> 对应方案: [plan.md](./plan.md)
> 依赖顺序：自上而下，标号靠前的先做。
> 状态：待开发。

## 阶段一：物料分发（复制）

- [ ] T1 复制 desktop 物料：`assets/logo.ico` → `deepseek-harness-desktop/resources/icon.ico`；`assets/logo.png` → `resources/icon.png` + `resources/tray.png`；`assets/logo.icns` → `resources/icon.icns`
- [ ] T2 复制 harmony 物料：`assets/logo.png` → 覆盖 `deepseek-harness-harmony/AppScope/resources/base/media/app_icon.png` 与 `startIcon.png`
- [ ] T3 复制 website 物料：`assets/logo.svg` → `deepseek-harness-desktop-website/images/logo.svg`

## 阶段二：deepseek-harness-desktop 接线

- [ ] T4 `forge.config.ts`：`packagerConfig` 增加 `icon`，`MakerSquirrel` 增加 `setupIcon`
- [ ] T5 `src/main/windows.ts`：BrowserWindow 增加 `icon:` 选项
- [ ] T6 `src/main/tray.ts`：用 `nativeImage.createFromPath(...)` 替换空占位图标
- [ ] T7 `index.html`：添加 favicon `<link rel="icon">`
- [ ] T8 更新 `resources/README.md`（图标文件已就位）

## 阶段三：deepseek-harness-harmony 接线

- [ ] T9 `AppScope/resources/base/element/string.json`：`app_name` → `Deepseek Harness Harmony`
- [ ] T10 `electron/src/main/resources/base/element/string.json`：`EntryAbility_label` → `Deepseek Harness Harmony`
- [ ] T11 `electron/src/main/resources/en_US/element/string.json`：同上
- [ ] T12 `electron/src/main/resources/zh_CN/element/string.json`：同上
- [ ] T13 `src-main/main.js`：标题 3 处（28/48/495）→ `Deepseek Harness Harmony`，并镜像到 `web_engine/src/main/resources/resfile/resources/app/main.js`

## 阶段四：deepseek-harness-desktop-website 接线

- [ ] T14 `index.html`：favicon + `.brand`/`.footer-brand` 换 logo 图
- [ ] T15 `index_zh.html`：同上
- [ ] T16 `style.css`：logo 图尺寸/对齐规则（如需要）

## 阶段五：验证

- [ ] T17 desktop：`npm run package` 编译，验证 exe/安装器图标
- [ ] T18 desktop：运行时验证窗口/托盘图标
- [ ] T19 harmony：编译并部署真机，验证桌面名 + 图标
- [ ] T20 website：浏览器验证 favicon + wordmark

## 依赖关系

```
T1 ──→ T4 ──→ T5 ──→ T6 ──→ T7 ──→ T8
T2 ──→ T9 ──→ T10/T11/T12 ──→ T13
T3 ──→ T14 ──→ T15 ──→ T16
T1 + T2 + T3 ──→ T17 ──→ T18 ──→ T19 ──→ T20
```
