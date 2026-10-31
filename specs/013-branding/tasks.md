# 013-branding 任务拆解（父工程）

> 对应方案: [plan.md](./plan.md)
> 状态：as-built 整理（历史任务已随各子工程完成；本清单用于跨工程核对）
> Last Updated: 2026-11-01

## 阶段一：物料源与分发

- [x] T1 父工程 `assets/` 建立唯一物料源（svg/png/ico/icns + 不透明底变体）
- [x] T2 desktop：`logo.{ico,icns,png}` → `dsh-desktop/resources/{icon.ico,icon.icns,icon.png,tray.png}`
- [x] T3 hos：不透明底 `logo-1024-with-bg.png` → `AppScope/.../media/{app_icon,startIcon,icon,product_logo_32}.png`；透明 `logo.png` → `window_icon.png` / `electron_white.png`
- [x] T4 ark：`logo.png` → `AppScope/.../media/app_icon.png`、`entry/.../media/{startIcon,icon}.png`
- [x] T5 android：`assets/logo.png` → `scripts/generate-brand-assets.py` 生成五密度 mipmap + 通知图标
- [x] T6 website：`logo.svg` → `dsh-desktop-website/images/logo.svg`

## 阶段二：应用身份接线

- [x] T7 desktop：`productName` / 窗口标题 / 托盘 tooltip / favicon
- [x] T8 hos：`app_name` + 三 locale `EntryAbility_label` + `src-main/main.js` 标题
- [x] T9 ark：`app_name` / `EntryAbility_label` / `Index.ets` / `boot.html` 标题
- [x] T10 android：`strings.xml` `app_name` + Manifest `label` / `icon` / `roundIcon`

## 阶段三：图标加载

- [x] T11 desktop：`forge.config.ts` `packagerConfig.icon` + `MakerSquirrel.setupIcon` + `extraResource`
- [x] T12 desktop：`BrowserWindow.icon` + `Tray` 图标加载与降级
- [x] T13 hos：`electron/src/main/module.json5` 的 ability `icon` 引用 `window_icon`
- [x] T14 android：Manifest 引用 mipmap；通知使用 `ic_stat_dsh`

## 阶段四：Web 品牌位边界

- [x] T15 hos：`dsh-rebrand.patch` 替换 `brand.localBuild` / `hero.headline` / hero logo
- [x] T16 hos：`dsh-disable-welcome-notice.patch` 关闭首次引导自动弹窗
- [x] T17 ark：`205-plugin-branding` 零补丁实现 Web 品牌位
- [x] T18 android：`205-plugin-branding` 移植实现 Web 品牌位

## 阶段五：验证

- [x] T19 desktop：打包图标 / 窗口 / 托盘验证
- [x] T20 hos：真机桌面名/图标、窗口/状态栏、Web 品牌位
- [x] T21 ark：真机桌面名/图标、boot 标题、Web 品牌位
- [x] T22 android：启动器/通知/Web 品牌位、生成脚本幂等
- [x] T23 website：favicon/wordmark

## 依赖关系

```
T1 → T2/T3/T4/T5/T6 → T7/T8/T9/T10 → T11..T18 → T19..T23
```
