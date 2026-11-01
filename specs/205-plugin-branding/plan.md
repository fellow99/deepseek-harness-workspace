# 205 · 品牌位（Web UI 内部 · 技术方案 · 父工程）

> HOW：ark/android 两壳同构。WHAT 见 [spec.md](./spec.md)（`FR-205-*`）。
> 细案：`dsh-desktop-ark/specs/205-plugin-branding/plan.md`、`dsh-desktop-android/specs/205-plugin-branding/{spec,plan}.md`。

## 1. 统一架构（两半体）

```
seam 插件包（branding-*）
 ├─ Host 半边（lib/index.js）：inject=['webServer']
 │    ├─ webServer.tapIndex(html => 产品名标题)           ← 页面标题
 │    └─ ctx.on('webserver/index-inject', t => t.push({kind:'style', text: HERO_CSS})) ← hero 标语隐藏
 └─ Client 半边（lib/client.js）：dsh.client { platform:'web', inject:[…] }
      ├─ slots.register('sidebar.brand.mark',  SidebarMark)
      ├─ slots.register('sidebar.brand.name',  SidebarName)
      ├─ slots.register('conversation.hero.brand.mark', HeroBrand)
      ├─ locale.register(NS, {zh,en})                    ← 自有命名空间
      └─ document.title 观察器（标题守卫，幂等，产品名收尾）
```

## 2. 各壳落点

| 关注点 | `dsh-desktop-ark` | `dsh-desktop-android` |
|---|---|---|
| 包 | `packages/branding-ohos` | `packages/branding-android`（移植自 ark） |
| Host | `lib/index.js`（`tapIndex` + `index-inject` HERO_CSS） | 同 |
| Client 槽位 | 三槽位 + locale + 标题守卫 | 同 |
| 图标几何 | 同源 `assets/logo.svg` | 同源 `assets/logo.png`（#1296db 气泡） |
| hero 锚点 | `data-dsh-ohos-hero` 属性 + `:has()` | `data-dsh-android-hero` 属性 + `:has()` |
| 挂载 | `bundle-ohos` `insert` 行 | `bundle-android` `insert` 行 |
| 上游行抑制 | `ui-brand-official` disabled；`welcomeNoticeVersion` 设置 | 同 |
| 应用级图标/名（013） | 由 `013`/AppScope 处理 | 见 android `205` spec 的实现记录（归属 013） |

## 3. hero 标语隐藏规则（`:has()` 锚点，抗漂移）

```css
/* 以品牌位锚点(:has 命中 branded hero)为前置，隐藏不含品牌位的兄弟 span（上游 titleGroup） */
div:has(> span > [data-slot="conversation.hero.brand.mark"] [data-dsh-*-hero]) > span:not(:has([data-dsh-*-hero])) { display:none !important }
```

- 锚点属性保证规则只命中 hero 那一行，不会选中品牌位自身。
- 上游若改 hero 结构 → 规则不匹配 → 标语恢复可见（安全回退，FR-205-012），不误伤其它元素。

## 4. 标题守卫

- `document.title` 观察器：仅当标题不是以产品名收尾时改写；无变化不写入（幂等）。
- 上游把标题改成「会话名 — 上游产品名」时，仍以产品名收尾（保留会话名）。

## 5. 构建期门禁

| 锚点 | 漂移即失败并指明同步位置 |
|---|---|
| 欢迎提示版本（`WELCOME_NOTICE_VERSION`） | `welcomeNoticeVersion` 配置需同步 |
| `brand.localBuild` 中英文文案 | locale 覆盖需同步 |
| 被覆盖的 bundle 行 id（如 `ui-brand-official`） | 行 id 变更需同步 |
| 两半体品牌位锚点属性 | 宿主/浏览器属性须同值 |

## 6. 验证

| 层 | 判据 |
|---|---|
| 构建期 | 门禁通过（锚点/文案/行 id/属性一致） |
| 运行期 | 侧栏/hero 显示本产品身份；标题收尾为产品名；无上游标语/鱼形图标 |
| 回退 | 人为改 hero 结构 → 上游标语可见、无空白错位 |

## 7. 风险

- `:has()` 浏览器前提（WebView ≥105）→ 已由 G-WV 门禁（≥110）覆盖。
- 上游文案/结构漂移 → 构建期门禁 + 安全回退兜底。
- 与 `013-branding` 职责边界 → 两处 spec 显式登记，避免重复/遗漏。
