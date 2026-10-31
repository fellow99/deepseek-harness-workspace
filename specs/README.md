# 规格文档索引（父工程）

**项目名称：** DeepSeek Harness Workspace（`deepseek-harness-workspace`）
**版本：** 工作区聚合（各子工程版本见下表）
**技术栈：** Electron 43 / Electron-on-HarmonyOS 37 / ArkTS + ArkWeb / Kotlin + WebView；统一封装 deepseek-harness
**文档生成时间：** 2026-11-01
**最后更新：** 2026-11-01（按 0XX / 1XX / 2XX 编号规则整理）

---

## 一、文档总览

| 层级 | 分类 | 文档数量 | 说明 |
|------|------|---------|------|
| 整体 | 项目级顶层文档 | 7 | 索引、结构、技术、架构、宪法、整体规格、整体方案 |
| 模块 | 公共需求模块（0XX） | 20 | `011-version-*`（4 个版本）、`012-building-workflow`、`013-branding` |
| **合计** | **7 根文件 / 6 模块目录 / 20 模块文件** | | |

> 按需求约定，父工程**不生成** `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。
> 编号规则见 [constitution.md](./constitution.md)：`0XX` = 各子工程公共需求，`1XX` = 各工程基础功能（以 android 为基准），`2XX` = 补丁/插件。

---

## 二、项目级顶层文档

| 文档 | 路径 | 说明 |
|------|------|------|
| **文档索引** | [README.md](./README.md) | 本文件 |
| **项目结构** | [STRUCTURE.md](./STRUCTURE.md) | 工作区目录树、子工程角色、编号映射 |
| **技术选型** | [TECH.md](./TECH.md) | 全工作区技术栈、版本、四个壳对比、否决项 |
| **整体架构** | [ARCHITECTURE.md](./ARCHITECTURE.md) | 分层、四壳运行时模型、同源数据面、构建拓扑 |
| **宪法原则** | [constitution.md](./constitution.md) | 产品/架构原则、0XX/1XX/2XX 编号规则、文档分工、提交/审查/安全 |
| **整体规格** | [overall-spec.md](./overall-spec.md) | 跨工程 WHAT/WHY：身份、运行时、界面、能力、数据、生态、诊断、发布、版本 |
| **整体方案** | [overall-plan.md](./overall-plan.md) | 跨工程 HOW：技术上下文、宪法合规、四壳实现策略、横切关注点、测试/发布/风险 |

---

## 三、公共需求模块（0XX）

### 011 — 版本更新（版本需求）

> 上游 dsh 升级 + 各封装壳版本更新的跨工程需求。原 `101-version-*` 统一改号为 `011-version-*`；历史 `101-version-*` 不再使用。

| 版本 | 文档 | 说明 |
|------|------|------|
| 0.1.5 | [011-version-0.1.5/spec.md](./011-version-0.1.5/spec.md) | 升级到 `dsh-v0.1.5-rc.2`；窗口改系统标题栏、窗口状态、F11 |
| 0.1.7 | [011-version-0.1.7/spec.md](./011-version-0.1.7/spec.md) | desktop 升级到 `dsh-v0.1.7-rc.2`；hos 本轮不发版 |
| 0.2.0 | [011-version-0.2.0/spec.md](./011-version-0.2.0/spec.md) | desktop/hos 升级到 `dsh-v0.2.0-rc.2`；patch 适配 |
| 0.2.1 | [011-version-0.2.1/spec.md](./011-version-0.2.1/spec.md) | desktop/hos 升级到 `dsh-v0.2.1-rc.2`；ark/android 对齐 |
| 任务/用例 | `011-version-*/{tasks,test-cases}.md` | 各版本任务与验收 |

### 012 — 构建工作流（building-workflow）

> 全工作区构建工作流：桌面版三平台 CI/Release、Android tag 构建/Release、父工程协同；鸿蒙工程暂无 GitHub Actions。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [012-building-workflow/spec.md](./012-building-workflow/spec.md) | 构建工作流功能规格 |
| 技术方案 | [012-building-workflow/plan.md](./012-building-workflow/plan.md) | 工作流文件、步骤、版本锁定、签名注入 |

### 013 — 品牌化（branding）

> 原 `010-assets` 并入。统一物料源 + 四个交付版本的应用名/图标/标题 + 与 `205-plugin-branding`（Web 品牌位）的边界。

| 文档 | 链接 | 说明 |
|------|------|------|
| 功能规格 | [013-branding/spec.md](./013-branding/spec.md) | 品牌化功能规格 |
| 技术方案 | [013-branding/plan.md](./013-branding/plan.md) | 物料源、各工程落点、平台规范 |
| 任务拆解 | [013-branding/tasks.md](./013-branding/tasks.md) | 跨工程任务核对 |
| 测试用例 | [013-branding/test-cases.md](./013-branding/test-cases.md) | 四版品牌验收 |

---

## 四、各子工程版本与 specs 入口

| 工程 | 版本 | 技术路线 | specs 入口 |
|------|------|----------|------------|
| `dsh-desktop` | 0.2.1 | Electron 43 | [dsh-desktop/specs/README.md](../dsh-desktop/specs/README.md) |
| `dsh-desktop-hos` | 0.2.1 | Electron 37 on HarmonyOS | [dsh-desktop-hos/specs/README.md](../dsh-desktop-hos/specs/README.md) |
| `dsh-desktop-ark` | 0.2.1 | ArkWeb + Node 24.2.0 | [dsh-desktop-ark/specs/README.md](../dsh-desktop-ark/specs/README.md) |
| `dsh-desktop-android` | 0.1.0 | WebView + Node 24.22.1 | [dsh-desktop-android/specs/README.md](../dsh-desktop-android/specs/README.md) |
| `deepseek-harness` | `dsh-v0.2.1-rc.2` | 上游宿主 | — |
| `dsh-market` | 各壳锁定 | 插件市场 | — |

---

## 五、模块编号一览

| 编号 | 模块名 | 英文名 | 父工程目录 | 子工程使用 |
|------|--------|--------|------------|------------|
| 011 | 版本更新 | version | `011-version-0.1.5` ~ `0.2.1` | 父工程 |
| 012 | 构建工作流 | building-workflow | `012-building-workflow` | desktop / android（hos/ark 暂无） |
| 013 | 品牌化 | branding | `013-branding` | desktop / hos / ark / android |

> 子工程 1XX/2XX 模块不在此重复；完整映射见 [STRUCTURE.md](./STRUCTURE.md) §5。

---

## 六、模块文档结构规范

父工程模块目录至少包含：

| 文件 | 命名 | 说明 |
|------|------|------|
| 功能规格 | `spec.md` | WHAT/WHY：用户故事、`FR-*`、验收场景、NFR、边界 |
| 技术方案 | `plan.md` | HOW：技术上下文、宪法合规、实现策略、文件清单 |

> 按需扩展：`tasks.md`（任务拆解）、`test-cases.md`（验收用例）。
> 父工程不生成 `api.md`、`data-model.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。

---

## 七、快速导航

| 目标读者 | 推荐阅读顺序 |
|---------|-------------|
| **新加入开发者** | constitution.md → STRUCTURE.md → overall-spec.md → 子工程 specs/README.md |
| **架构师 / Tech Lead** | ARCHITECTURE.md → TECH.md → overall-plan.md → 各子工程 ARCHITECTURE.md |
| **桌面开发** | dsh-desktop/specs/README.md → 101/103/105/115 等模块 |
| **HarmonyOS 开发** | dsh-desktop-ark/specs/README.md 或 dsh-desktop-hos/specs/README.md |
| **Android 开发** | dsh-desktop-android/specs/README.md（1XX 编号基准） |
| **构建/发布** | 012-building-workflow → 各壳 build pipeline / compliance 模块 |
| **测试 / QA** | overall-spec.md 验收场景 → 各子工程 SPECS_CHECKLIST.md → 模块 test-cases.md |
| **产品经理** | overall-spec.md → 013-branding → 各子工程 overall-spec.md |

---

## 八、维护规则

- 新增/删除子工程或模块：更新本文件、`STRUCTURE.md`、`constitution.md` 与相关子工程 README/SPECS_CHECKLIST。
- 编号变更：先更新宪法与迁移映射，再改目录与所有交叉引用。
- 文档内容以源码为事实源；未实现能力标 `planned` / `not-implemented`，并附证据或缺口说明。
- 提交注释遵循 `git-commit` 技能的 Conventional Commits 前缀；审查遵循 `requesting-code-review` / `receiving-code-review`。

---

**文档维护者：** DeepSeek Harness Workspace 开发团队
