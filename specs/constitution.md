# constitution.md — 父工程宪法原则

> 工程：`deepseek-harness-workspace`
> 状态：**约束性文档**。与本文件冲突的编号/文档结构一律视为缺陷。
> 依据：工作区 README、各子工程 `specs/constitution.md`、`docs/能力对照清单.md`、`specs-as-built` 方法论。
> Last Updated: 2026-11-01

---

## 一、适用范围

本宪法约束整个工作区：

- 父工程 `deepseek-harness-workspace`
- `dsh-desktop`（Electron 桌面壳）
- `dsh-desktop-hos`（Electron on HarmonyOS Edition）
- `dsh-desktop-ark`（HarmonyOS Edition）
- `dsh-desktop-android`（Android Edition）

各子工程可在本宪法之上追加更严格的工程原则，但**不得违反**本文件的编号、文档分工与产品边界。

---

## 二、产品与架构原则

### C1 上游是唯一事实源，壳工程只做装配

- 不 fork、不改写 dsh Web 前端；不重写原生聊天 UI。
- 平台适配优先使用上游扩展点：profile / bundle / seam 插件 / 配置项 / 槽位。
- 必须修改上游时，使用**按 dsh tag 分目录、幂等、可审计**的 patch；patch 数量与理由必须登记。
- **违反症状**：`deepseek-harness` 工作区出现未登记改动；`specs/` 出现“我们改了前端”。

### C2 进程内 Host + 同源数据面

- dsh Host 在应用进程内启动；webserver 同时服务 SPA dist 与 `/api`。
- 渲染层同源加载该 webserver，复用上游 `WebApiClient`（HTTP 上行 + WebSocket 下行）。
- 不新增自定义业务协议、不新增 renderer↔main IPC 载体、不引入 CORS/鉴权舞蹈（ark/android 的一次性 token 属 Host 鉴权，不是壳自造协议）。
- **违反症状**：自建 WebSocket/HTTP 协议；组件直接调用平台 API 绕过 Host。

### C3 平台真相不隐藏

- 能力未实现就回 `not-implemented` + hint；未注册就回 `unregistered`；不返回伪造的成功。
- 降级（无沙箱、无 PTY、无 JIT、纯 JS 搜索等）必须在 UI 与文档中显式登记。
- **违反症状**：返回 `{ok:true}` 但实际没做事；文档写“已支持”却无证据。

### C4 证据优先

- 任何“可用/通过”的结论必须附可复现证据：命令输出、截图、`ready.json`、`/facts`、logcat/hilog 片段。
- 禁止作为判据：一条日志、目录存在、模拟器截图、`uitest dumpLayout` 单独作为唯一证据。
- **违反症状**：验收表只有 ✅ 没有证据链接/片段。

---

## 三、规范编号规则（强制）

### C5 0XX — 各子工程公共、共同的需求模块

`0XX` 只用于**跨子工程共享**的需求模块。整体规范由父工程 `specs/` 承载；子工程如需实现，使用同一编号并只写本工程实现。

| 编号 | 模块 | 规则 |
|------|------|------|
| `011-version-<VERSION>` | 版本更新 | 原 `101-version-*` **统一改号为 `011-version-*`**；父工程创建 `specs/011-version-<VERSION>/`。历史 `101-version-*` 不再使用。 |
| `012-building-workflow` | 构建工作流 | 父工程创建 `specs/012-building-workflow/`；有 CI 的工程（desktop / android）使用同号模块；**鸿蒙工程暂无**（hos / ark 经 DevEco/hvigor 本地构建，不建 012）。 |
| `013-branding` | 品牌化 | 原 `010-assets` **并入 `013-branding`**；父工程创建 `specs/013-branding/`；**各子工程如果没有就必须补充本模块文档**。 |

补充规则：

- `000-scaffold`：工程搭建基线，保留 `000` 例外编号，不参与 0XX 公共模块编号（当前仅 android 使用）。
- `3XX`：技能类规范预留（当前 hos `301-skill-runtime-capabilities`）。

### C6 1XX — 各工程自己的基础功能模块

- 用于各工程自己的基础功能模块（运行时、外壳、载荷、能力桥、适配层、发布等）。
- 因为各工程基础功能模块有不少共通，**编号尽量与 `dsh-desktop-android` 基准对齐**。
- 允许平台后缀差异（如 `103-webview-shell` / `103-arkweb-shell` / `103-window-shell`），但槽位语义一致。

**基准编号（dsh-desktop-android）：**

| 编号 | 语义 | 说明 |
|------|------|------|
| 101 | runtime-launch-orchestration | 运行时启动/入口编排 |
| 102 | payload-pipeline | 载荷/产物装配与解包 |
| 103 | webview-shell | 渲染外壳（WebView/ArkWeb/窗口） |
| 104 | host-capability-bridge | 宿主能力桥/预加载 IPC |
| 105 | upstream-adapter-bundle | profile/bundle/patch 适配层 |
| 106 | workspace-picker | 工作区选择 |
| 107 | diagnostics-observability | 诊断与可观测 |
| 108 | compliance-release | 合规与发布 |
| 109 | agent-keepalive | Agent 保活 |
| 110 | pnpm-integration | 纯 JS pnpm 集成 |

**扩展编号（工作区统一约定）：**

| 编号 | 语义 | 当前使用 |
|------|------|----------|
| 111 | build-pipeline | hos 构建编排 |
| 112 | tool-bash | hos 命令执行工具 |
| 113 | tray | desktop 系统托盘 |
| 114 | lifecycle-safety | desktop / hos 生命周期与安全 |
| 115 | host | desktop / hos Host 宿主 |
| 116 | notification | desktop 原生通知 |
| 117 | runtime-provisioning | hos 运行时供给 |
| 118 | preload-bridge | desktop 薄 IPC 桥（历史） |
| 119 | jitless | ark 无 JIT 模式（原 109） |

### C7 2XX — 各工程的补丁、插件模块

- 因为各工程补丁、插件模块有不少共通，**编号尽量对齐**。
- 当前统一目录：

| 编号 | 模块 | 使用工程 |
|------|------|----------|
| 201 | dsh-market 插件市场 | desktop / hos / ark / android |
| 202 | plugin-fs-mutate | hos |
| 203 | plugin-workspace-picker | hos |
| 204 | plugin-git | hos / ark / android |
| 205 | plugin-branding（Web 品牌位） | ark / android |
| 206 | plugin-fs-search | android |

> `013-branding`（应用级品牌/图标/名称）与 `205-plugin-branding`（Web UI 内部品牌位）职责分离，允许同时存在；两者差异必须在 spec 中显式登记。

### C8 编号即契约

- 目录名格式：`<NNN>-<name>`，编号与名称一经发布不得随意变更；改号必须提供迁移映射。
- `spec.md` / `plan.md` 内的需求前缀必须与目录号一致（如 `FR-013-*`、`FR-204-*`）。
- 新增/改号模块必须同步更新：
  - 本文件与父工程 `specs/README.md`、`STRUCTURE.md`；
  - 子工程 `specs/README.md`、`SPECS_CHECKLIST.md`；
  - 所有交叉引用与 `FR-*` 前缀。
- **违反症状**：目录号与 `FR-*` 不一致；README/清单缺项；父工程与子工程编号冲突。

---

## 四、规范文档分工（强制）

### C9 父工程规范文档的职责

父工程 `specs/` 关注**整体规划、整体设计，以及各子工程共同的需求**：

| 文档 | 职责 |
|------|------|
| `README.md` | 文档索引、模块导航、阅读顺序 |
| `STRUCTURE.md` | 整个工作区目录结构、子工程角色、编号映射 |
| `TECH.md` | 整个工作区技术选型与版本策略 |
| `ARCHITECTURE.md` | 整个工作区架构、四壳运行时模型、数据面、构建拓扑 |
| `constitution.md` | 宪法原则、编号规则、文档分工 |
| `overall-spec.md` | 跨工程整体规格（技术无关） |
| `overall-plan.md` | 跨工程整体技术方案 |
| `011-version-*` | 跨工程版本更新需求 |
| `012-building-workflow` | 跨工程构建工作流整体规范 |
| `013-branding` | 跨工程品牌化整体规范 |

父工程**不写**：`API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`（按需求约定）。

### C10 子工程规范文档的职责

各子工程 `specs/` 关注**本工程具体技术实现、独立的需求**：

- 本工程模块 `spec.md` / `plan.md` / `tasks.md` / `test-cases.md`；
- 本工程 `overall-*`、`ARCHITECTURE.md`、`TECH.md`、`STRUCTURE.md`、`constitution.md`；
- 本工程特有的 1XX / 2XX / 3XX 模块；
- 对父工程 0XX 公共模块的实现说明（同号，只写本工程实现，不复制父工程整体规划）。

**禁止**：子工程重复父工程的整体规划/整体架构长文；应链接父工程对应文档。

### C11 文档内容规则

- **as-built**：文档描述“实际建成什么”，不写未实现的愿望；未实现标 `planned` / `not-implemented`。
- **源码即真理**：每条声明可追溯到源码/配置/脚本；不确定处标 `[NEEDS CLARIFICATION]`，每份 spec 最多 3 处。
- **WHAT / HOW 分离**：`spec.md` 技术无关（不写框架名/文件路径）；`plan.md` 写实现细节并交叉引用 `FR-*`。
- **RFC 2119**：需求使用 MUST / SHOULD / MAY。
- **无占位符**：禁止 `TODO`、`TBD`、`[fill in]` 作为交付内容。
- **中文 + 英文技术术语**：正文中文，技术名保留英文。

---

## 五、工程与流程原则

### C12 Conventional Commits（强制）

- 提交注释必须且只能使用 `git-commit` 技能规定的 type 前缀：
  `feat` / `fix` / `docs` / `style` / `refactor` / `perf` / `test` / `build` / `ci` / `chore` / `revert`。
- 格式：`<type>[(scope)]: <description>`；描述用现在时、祈使语气、<72 字符。
- 严禁其它前缀（如 `update:`、`修改:`、`add:`）。
- **违反症状**：`git log` 出现非 Conventional Commits 前缀。

### C13 代码审查（强制）

- 完成任务、重大功能、合并前，使用 `requesting-code-review` 技能发起审查。
- 收到反馈后，使用 `receiving-code-review` 技能：先核实、再实现；不盲从、不表演式认同。
- Critical 立即修；Important 在继续前修；Minor 登记。
- **违反症状**：无审查记录直接合并；对反馈未核实即改。

### C14 规范驱动

- 新功能遵循 `specs-based-devflow`：规格 → 开发 → 审查 → 测试 → 修复 → 回归。
- 模块文档成对产出：`spec.md` + `plan.md`；必要时补 `tasks.md` / `test-cases.md`。
- 版本更新（`011-version-*`）必须包含：patch 适配、版本号、构建、验证、文档同步。

### C15 安全与合规

- 渲染进程：`sandbox:true` + `contextIsolation:true` + `nodeIntegration:false`（Electron）。
- 明文网络仅 loopback；Android `networkSecurityConfig` 不放行通配域名。
- 凭据/签名材料/API Key 不入库、不进日志、不进 specs 样例。
- 权限最小化、按需触发；store flavor 不声明 All-files。
- 二进制不入库；`libnode.so`、载荷归档、keystore 等由脚本投放/SHA-256 锁定。
- **违反症状**：仓库出现 `.p12`/`.cer`/`keystore`；日志可 grep 到 token；`git status` 出现 `jniLibs/**`。

### C16 变更最小化

- 修 Bug 只做最小改动，严禁顺便重构。
- 章节/编号整理属文档变更，必须与代码变更分开提交。
- 涉及跨工程编号调整时，先更新本宪法与迁移映射，再改目录与引用。

---

## 六、检查方式（可执行）

| 原则 | 检查点 | 方式 |
|------|--------|------|
| C1 | 上游零补丁/已登记 patch | `git -C deepseek-harness status --porcelain` 为空；patch 目录与 `specs` 登记一致 |
| C2 | 同源数据面 | 渲染层 URL 与 webserver 同源；无自定义协议 |
| C3 | 能力诚实 | `selfcheck` / `/facts` / 能力桥返回 `implemented/not-implemented` |
| C4 | 证据 | `logs/<date>-N/` 或 `docs/` 下有命令输出/截图/JSON |
| C5–C8 | 编号一致 | 目录树、README、SPECS_CHECKLIST、`FR-*` 前缀互相对账 |
| C9–C10 | 文档分工 | 父工程无 API/data-model 文档；子工程不复制父工程整体长文 |
| C11 | as-built | `grep -rn "TODO\|TBD" specs/` 为空；`planned` 与实现状态一致 |
| C12 | 提交规范 | `git log --oneline` 前缀合法 |
| C13 | 审查 | 评审记录/修复记录可追溯 |
| C15 | 安全 | `git ls-files` 无签名/密钥；日志无 token |

---

## 七、修订记录

| 日期 | 变更 |
|------|------|
| 2026-11-01 | 建立父工程宪法；写入 0XX/1XX/2XX 编号规则、版本/构建/品牌模块规则、文档分工；`101-version-*` → `011-version-*`，`010-assets` → `013-branding`。 |
