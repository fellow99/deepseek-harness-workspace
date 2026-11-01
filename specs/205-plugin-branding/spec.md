# 205 · 品牌位（Web UI 内部 · 功能规格 · 父工程）

> Module: 205-plugin-branding ｜ 范围：`dsh-desktop-ark` / `dsh-desktop-android`（`hos`/`desktop` 无独立 205：其 Electron 窗口标题/应用名在各自外壳层处理）
> Status: Implemented（as-built）｜ Last Updated: 2026-11-01
> 关联/边界：`013-branding`（应用级：安装图标、应用名、首启门禁）——**与本模块职责分离**，详见 §3.7。

---

## 1. 模块概述

### 1.1 目的（Why）

在**不改上游 Web 前端**（C1）的前提下，把 dsh Web UI 内部仍带「上游产品身份」的品牌位，替换为本产品（DSH Desktop）身份：
侧栏品牌图标位 + 品牌名称位、会话空态 hero 品牌位、以及页面标题。使用上游公开口子（槽位注册、自有 locale 命名空间、`webServer.tapIndex`、`webserver/index-inject` 注入行）实现。

### 1.2 解决的问题

- 上游侧栏显示**鱼形图标**与 `brand.localBuild` 文案；会话空态显示 `hero.headline` 标语与 `hero.preview` 徽标——均属上游身份。
- 上游会把 `document.title` 改写成「会话名 — 上游产品名」；需保证标题**以本产品名收尾**（保留会话名部分）。
- 需保证「官方品牌位」行与「预览版说明」弹窗在不同构建环境下不泄漏上游身份。

### 1.3 范围

**包含**：侧栏品牌图标/名称位、hero 品牌位、页面标题与标题守卫、上游身份的抑制（预览说明/官方品牌位行）、构建期锚点门禁、浏览器能力前提登记。
**不包含**：应用安装图标、应用名、启动闪屏/首启引导（属 `013-branding`）；任何上报/遥测。

---

## 2. 用户故事

- 作为**最终用户**，我在应用里看到的是「DSH Desktop」的图标、名称与标题，而不是上游产品身份。
- 作为**维护者**，换产品名/换图标只需改一处宿主常量 + 一处浏览器常量，不会散落多处。
- 作为**打包者**，若上游改了被我覆盖的锚点（文案/行 id/hero 结构），构建期即失败或界面安全回退，而不是空白/错位。

---

## 3. 功能需求

> RFC 2119。实现映射见 `plan.md`。

### 3.1 品牌位占用

- **FR-205-001**：侧栏品牌图标位与品牌名称位 MUST 由本产品占用；MUST NOT 呈现上游鱼形图标或 `brand.localBuild` 文案（任何语言）。
- **FR-205-002**：会话空态 hero 品牌位 MUST 呈现本产品图标 + 产品名；MUST NOT 呈现上游 `hero.headline` 标语与 `hero.preview` 徽标。
- **FR-205-003**：页面/窗口标题 MUST 为产品名；上游随后改写成「会话名 — 上游产品名」时，标题 MUST 仍以本产品名收尾（会话名部分保留）。

### 3.2 只用上游公开口子

- **FR-205-004**：实现 MUST 只使用上游公开口子（槽位注册、自有 locale 命名空间、`webServer.tapIndex`、`webserver/index-inject` 的注入行），MUST NOT 修改上游工作树。
- **FR-205-005**：产品名与图标几何 MUST 各只有一个来源常量；改产品名/换图标 MUST 不超过两处改动（一处宿主、一处浏览器）。

### 3.3 上游身份抑制

- **FR-205-006**：上游「预览版说明」首次弹窗 MUST NOT 自动出现（经上游自己的配置项 `welcomeNoticeVersion` 等于上游 `WELCOME_NOTICE_VERSION` 实现）。
- **FR-205-007**：DeepSeek 首次配置对话框 MUST NOT 自动弹出；用户从设置显式进入时 MAY 仍可使用（能力不删）。
- **FR-205-008**：上游「官方品牌位」行（`ui-brand-official`）MUST 被禁用，避免不同构建环境出现上游官方标识。
- **FR-205-009**：实现 MUST NOT 伪造 `dshDesktop` 全局或其它「我是官方桌面壳」开关；MUST NOT 因本模块启用任何上报/遥测。

### 3.4 构建期门禁与安全回退

- **FR-205-010**：欢迎提示版本、`brand.localBuild` 中英文文案、被覆盖的 bundle 行 id 若发生变化，构建期门禁 MUST 失败并指明需同步的位置。
- **FR-205-011**：宿主与浏览器两半边的品牌位锚点属性 MUST 同值，不一致时门禁 MUST 失败。
- **FR-205-012**：上游若改动 hero 结构导致隐藏规则不再匹配，界面 MUST 回退为「上游标语可见」，MUST NOT 出现空白、错位或误隐藏其它元素。

### 3.5 渲染与资源

- **FR-205-013**：品牌位渲染 MUST NOT 引入新的网络请求或图片资源（图标内联 SVG，避免 CSP/离线问题）。
- **NFR-205-014**：标题守卫 MUST 幂等（无变化不写入），MUST NOT 造成观察器自激或可感知性能开销。
- **NFR-205-015**：实现 MUST 对目标浏览器内核可用；如使用 `CSS :has()`，MUST 在 `plan.md` 登记为浏览器能力前提（Chrome/WebView ≥105）。
- **NFR-205-016**：模块 MUST 随载荷分发（各壳 `packages/branding-*`），MUST NOT 要求额外构建步骤。

### 3.6 与 `013-branding` 的边界

- **FR-205-017**：应用安装图标、应用名、启动闪屏/首启门禁属 `013-branding`；本模块只负责 **Web UI 内部**品牌位与页面标题。二者可同时存在，差异 MUST 在两处 spec 显式登记。
  - `dsh-desktop-android` 的 `205` 文档同时记录了应用图标/名称（源自 `013`）作为实现说明——其**归属仍为 `013`**，Web 品牌位归属本模块。

---

## 4. 跨壳差异矩阵（as-built）

| 维度 | `desktop` | `dsh-desktop-hos` | `dsh-desktop-ark` | `dsh-desktop-android` |
|---|---|---|---|---|
| 是否提供 | 否（Electron 层处理标题/应用名） | 否（Electron-on-HarmonyOS 外壳层） | 是 | 是 |
| 插件包 | — | — | `packages/branding-ohos` | `packages/branding-android` |
| 槽位 | — | — | `sidebar.brand.mark`/`sidebar.brand.name`/`conversation.hero.brand.mark` | 同 ark（移植） |
| 页面标题 | — | — | `webServer.tapIndex` + 标题守卫 | 同 ark |
| hero 标语隐藏 | — | — | `webserver/index-inject` style 行（`:has()` 锚点） | 同 ark |
| 品牌物料 | — | — | 同源 `assets/logo.svg` 几何 | 同源 `assets/logo.png`（#1296db 气泡） |

---

## 5. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 品牌位锚点 | 槽位占用者的标记属性 | 宿主/浏览器两边同值 |
| 产品名常量 | 名称单一来源 | 与 `app_name` 同源 |
| 标题守卫 | `document.title` 观察 | 幂等、以产品名收尾 |
| hero 隐藏规则 | `:has()` 锚点选择器 | 结构变则不匹配→回退上游标语 |

---

## 6. 验收场景

- **侧栏品牌位**：应用就绪后侧栏显示本产品图标+名称，无鱼形图标/`brand.localBuild`。
- **hero 品牌位**：会话空态显示本产品图标+名称，无上游标语/Preview 徽标。
- **页面标题收尾**：进入会话后标题为「会话名 — DSH Desktop」（收尾为本产品名）。
- **上游身份抑制**：首装不弹「预览版说明」；官方品牌位行未出现。
- **门禁**：人为改动被覆盖的文案/行 id → 构建失败并指明位置。
- **安全回退**：模拟上游 hero 结构变化 → 界面回退上游标语可见，不空白/不误隐藏。

---

## 7. 非功能需求

- **零上游改动**：上游工作树零改动、零补丁。
- **无新增网络/资源**：图标内联 SVG；不引入外部图片/CSP 风险。
- **幂等/低开销**：标题守卫幂等。
- **随包分发**：无额外构建步骤。

---

## 8. 假设与约束

- 目标浏览器内核支持所用选择器（`:has()` 需 Chrome/WebView ≥105；各壳 G-WV 门禁已 ≥110）。
- 上游槽位/locale/config 口子存在且稳定；若漂移由构建期门禁兜底。
- 上游 hero/标题结构变化时以安全回退为准（FR-205-012）。

---

## 9. 依赖

- 上游：`deepseek-harness` 的槽位、locale、`webServer`（`tapIndex` / `index-inject`）（只读）。
- 工程内：各壳 bundle 适配层（`insert` 挂载 + 覆盖行）、`013-branding`（应用级品牌物料同源）。
