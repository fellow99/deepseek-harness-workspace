[English](./README.md) | 中文

---

# DeepSeek Harness Workspace

> 本仓库是一个聚合工程（workspace），把将 [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) 封装为一等桌面应用所需的全部工程汇聚到一处——Windows/Linux/macOS 走 Electron 桌面壳，鸿蒙端则分为**两个独立 edition**：**Electron on HarmonyOS Edition**（`dsh-desktop-hos`，保留 Electron 运行时）与 **HarmonyOS Edition**（`dsh-desktop-ark`，ArkWeb + 进程内 Node 运行时）。所有仓库均以 **git submodule** 形式固定版本，使本 workspace 的目录结构恰好还原各封装工程构建脚本所期望的兄弟目录布局（`../deepseek-harness`、`../dsh-market`、`../harmonypc-electron`、`../ohos-node-bridge`）。

---

## 这是什么

DeepSeek Harness（`dsh`）是 DeepSeek AI 开源的 Agent 运行框架（harness），采用"一切皆插件"架构；其原生入口是 `dsh web`（浏览器 Web UI）。三个姊妹封装工程将该 Web UI 变成原生桌面应用——完整复用 dsh 前端，并在进程内托管 dsh Host：

- **`dsh-desktop`** —— 面向 Windows / Linux / macOS 的 Electron 桌面壳
- **`dsh-desktop-hos`** —— **Electron on HarmonyOS Edition**：鸿蒙移植版（2in1 / 平板，打包为 HAP），运行在 Electron-on-HarmonyOS 运行时之上
- **`dsh-desktop-ark`** —— **HarmonyOS Edition**：同一个 dsh Host 跑在嵌入式 Node 运行时上，由系统 ArkWeb 渲染（不含 Electron、不打包 Chromium）

两个鸿蒙 edition 背后的运行时构件单独发行：**`ohos-node-bridge`** 是把真实 Node.js 运行时（`NAPI bootstrap + dlopen + node::Start`）嵌进应用自身进程的 ArkTS 组件库，其 `runtime/` 目录即 SHA-256 锁定的发行版缓存。三个封装工程消费完全相同的上游构件；本 workspace 正是承载这些构件（以 submodule 形式并列存放）的容器，各封装工程可以在一个地方完成克隆、构建与开发，其 `../兄弟目录` 引用在 workspace 内部即可解析。

## 仓库布局

```
deepseek-harness-workspace/            # 本仓库 —— submodule 容器
├── dsh-desktop/          # [submodule] Electron 桌面封装（Windows/Linux/macOS）
├── dsh-desktop-hos/      # [submodule] Electron on HarmonyOS Edition（2in1/平板，HAP）
├── dsh-desktop-ark/      # [submodule] HarmonyOS Edition —— ArkWeb + 进程内 Node（HAP）
├── deepseek-harness/                  # [submodule] dsh —— 被封装的 Agent 主机（上游）
├── dsh-market/                        # [submodule] 可视化插件市场（npm 包 "dshmarket"）
├── ohos-node-bridge/                  # [submodule] 内嵌 Node.js（node::Start）的 ArkTS 库 + runtime/ 发行缓存
├── dsh-plugins/                       # （普通目录）通用、可共享的 dsh 插件 —— 一插件一目录，命名 dsh-plugin-XXX
├── skills/                            # （普通目录）通用、可共享的 dsh 工具技能 —— 一技能一目录，功能性 kebab-case 名
├── harmonypc-electron/                # [submodule] Electron-on-HarmonyOS 运行时（Electron 37 / Node 22.17.0）
├── dsh-desktop-website/  # [submodule] 三个封装工程的产品官网
└── harmonypc-electron-versions/       # （普通目录，非 submodule）运行时发行版归档 + Electron 头文件指南

### 各角色一览

| 目录 | 角色 | 被谁消费 |
|---|---|---|
| `dsh-desktop` | Electron 桌面壳 —— 主封装工程 | — |
| `dsh-desktop-hos` | **Electron on HarmonyOS Edition** —— 基于 Electron 运行时的鸿蒙移植版 | — |
| `dsh-desktop-ark` | **HarmonyOS Edition** —— ArkWeb 壳 + 进程内 Node；不含 Electron | — |
| `ohos-node-bridge` | 内嵌 Node（`node::Start`）的 ArkTS 库 + `runtime/` 发行缓存 | 两个鸿蒙 edition |
| `deepseek-harness` | 被封装的 Agent 主机（`dsh`，上游源码引用） | 三个封装工程 |
| `dsh-market` | 内置可视化插件市场 | 三个封装工程 |
| `dsh-plugins` | **通用、可共享**的 dsh 插件，一插件一目录，命名 `dsh-plugin-XXX`。目前除自身 README 外为空（见 [`dsh-plugins/README.md`](dsh-plugins/README.md)）；封装工程专用的插件放在各自工程内 | 任一封装工程 |
| `skills` | **通用、可共享**的 dsh 工具技能，一技能一目录，功能性 kebab-case 名。目前除自身 README 外为空（见 [`skills/README.md`](skills/README.md)）；封装工程专用的技能放在各自工程内 | 任一封装工程 |
| `harmonypc-electron` | Electron-on-HarmonyOS 运行时（原生 SO + ArkTS 桥接层） | 仅 `dsh-desktop-hos` |
| `dsh-desktop-website` | 三个封装工程的产品官网 | — |
| `harmonypc-electron-versions` | Electron-on-HarmonyOS 发行版归档（v34/v37/v40）+ 用于重编译原生模块（如 `better-sqlite3`）的 Node 头文件指南 | harmony 工具链 |

### 插件约定

第一方 dsh 插件分**两层**，按"谁能消费它"区分：

| 位置 | 层级 | 命名 |
|---|---|---|
| `dsh-plugins/`（本 workspace） | **通用 / 可共享** —— 不依赖任何特定封装工程的补丁，任何壳都能消费 | `dsh-plugin-XXX` |
| `<封装工程>/plugins/`（如 [`dsh-desktop-hos/plugins/`](dsh-desktop-hos/plugins/)） | **封装工程专用** —— 依赖该壳自己的补丁集 / profile / 运行期适配；编译期打入包内，运行期**全部默认加载** | harmony 壳为 `harmony-plugin-XXX` |

两层中目录名都同时是包 `name`（裸包名 / 非 scoped），`XXX` 描述该插件新增的功能。判断归哪一层的一句问法：*「把它装到另一个 dsh 壳上，它还能工作吗？」* 能 → 通用；不能 → 封装工程专用。

插件均为纯 ESM（`lib/index.js`，无构建步骤）。各消费壳在收集阶段把需要的插件物化进自己随包分发的 `dsh-dist/node_modules/`，再由 agent preset 的一行挂载。

> 目前唯一的插件——`delete` / `move` 文件工具——属**封装工程专用**：它需要 harmony 封装的 `dsh-fs-remove-primitive` 补丁提供 `ctx.fs.remove`，因此位于 [`dsh-desktop-hos/plugins/harmony-plugin-fs-mutate/`](dsh-desktop-hos/plugins/harmony-plugin-fs-mutate/)。这也是 `dsh-plugins/` 目前除自身 README 外为空的原因。详见 [`dsh-plugins/README.md`](dsh-plugins/README.md) 与 [`dsh-desktop-hos/plugins/README.md`](dsh-desktop-hos/plugins/README.md)。

### 技能约定

工具技能与插件采用**同一套两层分工**：

| 位置 | 层级 | 被谁消费 |
|---|---|---|
| `skills/`（本 workspace） | **通用 / 可共享** —— 在任何 dsh 壳上都成立 | 任一封装工程 |
| `<封装工程>/skills/`（如 [`dsh-desktop-hos/skills/`](dsh-desktop-hos/skills/)） | **封装工程专用** —— 描述该壳自己的运行期（沙箱、打包路径、能力缺口） | 仅该封装工程 |

与插件不同：**技能名不加前缀**，一律用功能性 kebab-case。因为技能的 `name` 是**模型可见标识**，也是人工 `/name` 要输入的字符串 —— 归属由**目录**表达，不由名字表达。约定「目录名 == frontmatter `name`」，但注意 dsh **并不校验**这一点。

技能形态为 `<技能名>/SKILL.md`（目录包，可携带资源）或 `<技能名>.md`；技能根**只扫描顶层**，嵌套的 `**/SKILL.md` 不会被发现。frontmatter 必填 `name` + `description`，可选 `whenToUse` / `metadata` / `disable-model-invocation` / `user-invocable`（驼峰旧键会被拒绝）。

**加载语义 —— 不要描述成"启动时即加载"。** 模型自动看到的只是技能**目录**（名称 + 截断到 500 字的描述），它在会话**首次请求之前**作为一条持久 user-role 消息注入；技能**正文按需加载**（模型调 `skill` 工具，或人工打 `/name`），且不缓存。

**通用技能不会自动生效。** 每个壳自行选择采纳；dsh 提供的机制是 `customSkillDirs` —— 自带的 `cordis` preset 就是这样把它自己的 `skills/` 指给 `skill-filesystem` 的。

> 目前唯一的技能——`harmony-runtime-capabilities`——属**封装工程专用**（它讲的是 HarmonyOS HAP 的沙箱、`hmdfs` 与本构建的能力缺口），因此位于 [`dsh-desktop-hos/skills/harmony-runtime-capabilities/`](dsh-desktop-hos/skills/harmony-runtime-capabilities/)。这也是 `skills/` 目前除自身 README 外为空的原因。详见 [`skills/README.md`](skills/README.md) 与 [`dsh-desktop-hos/skills/README.md`](dsh-desktop-hos/skills/README.md)。

### 当前版本

| 工程 | 版本 | 对应 dsh 版本 |
|---|---|---|
| `dsh-desktop` | **0.2.1** | `dsh-v0.2.1-rc.2` |
| `dsh-desktop-hos` | **0.2.1** | `dsh-v0.2.1-rc.2` |
| `dsh-desktop-ark` | **0.1.0**（scaffold） | `dsh-v0.2.1-rc.2` |
| `ohos-node-bridge` | **0.1.0**（库） · 内置 Node **v24.2.0** | — |
| `deepseek-harness`（submodule 固定版本） | — | `dsh-v0.2.1-rc.2` |

各封装工程按自身固定的上游 dsh tag，构建脚本据此选择对应的 `patches/<dsh-tag>/`。任一工程升级时，
请同步更新本表与各产品 README。

## 共享架构

`dsh` 已完成 **Host/Client 拆分**，其 Web 服务器**同时托管 SPA dist 与 `/api`**。因此三个封装工程采用相同的**进程内 Host + webserver + 同源数据面**方案：

- 封装工程在进程内托管 dsh Host（webserver + apiProxy + connection 统一绑定一个端口）：Electron 系封装在 Electron 的 Node 模式主进程中运行；HarmonyOS Edition 则在 ArkTS 应用进程内嵌的 Node 运行时上运行。
- 就绪后，视图加载该 webserver，运行**未经修改的标准 dsh Web UI**，复用其既有 `WebApiClient`（HTTP 上行 + WebSocket 下行）。
- 结果：**零 CORS、零鉴权、零自定义协议、零新增 IPC 载体 —— 上游零改动**（仅有幂等补丁）。

### 各平台的差异适配

| 方面 | 桌面端（Electron） | 鸿蒙端 · Electron on HarmonyOS Edition | HarmonyOS Edition（ArkWeb + Node） |
|---|---|---|---|
| 目标平台 | Windows / Linux / macOS | HarmonyOS 2in1 / 平板（HAP） | HarmonyOS 2in1 / 平板（HAP） |
| 运行时 | Electron 43（随包） | Electron 37 on HarmonyOS（`harmonypc-electron`） | 系统 ArkWeb + Node v24.2.0（`ohos-node-bridge`） |
| Web 服务器绑定 | `127.0.0.1:<随机端口>` | `0.0.0.0:<随机端口>` + 局域网 IP + `Host`/`Origin` 改写（HarmonyOS 回环网络隔离） | `127.0.0.1:<随机端口>`（同源回环） |
| dsh 补丁 | 3 个（禁用 HMR · 禁用原生目录选择器 · 关闭欢迎提示） | 13 个（flock 与硬链接回退 · `ctx.fs` remove / writeBytes / chmod · 追加可写根 · 附件 fsync · 品牌改写） | 0 个（profile/bundle + seam 插件 + 受审计垫片） |
| 状态 | v0.2.1 —— 打包可用；托盘 / 通知 / 剪贴板 / 窗口状态 | v0.2.1 —— 已在 HarmonyOS 6.1.0.135（API 24）真机验证；终端 / 沙箱缺失 | v0.1.0 scaffold —— 编译打包通过（含运行时 44 MB HAP）；真机验证待做 |

## 快速开始

```bash
git clone --recursive https://github.com/fellow99/deepseek-harness-workspace.git
cd deepseek-harness-workspace
git submodule update --init --recursive   # 拉取/刷新全部固定版本的 submodule
```

本 workspace 的目录布局已满足各封装工程的兄弟目录前置要求（`../deepseek-harness`、`../dsh-market`、`../harmonypc-electron`、`../ohos-node-bridge`）。构建与运行说明由各封装工程各自维护、可能与本概述存在出入：

- **桌面端**：见 [`dsh-desktop/README.md`](dsh-desktop/README.md)（`npm run build:dsh` 应用补丁并构建 dsh + dsh-market，随后 `npm start` / `npm run package`）。
- **Electron on HarmonyOS Edition**：见 [`dsh-desktop-hos/README.md`](dsh-desktop-hos/README.md)（三阶段构建 `collect-runtime → build-dsh → collect-dsh`，随后经 DevEco Studio / hvigor 构建并签名 HAP）。
- **HarmonyOS Edition**：见 [`dsh-desktop-ark/README.md`](dsh-desktop-ark/README.md)（构建 bridge HAR → 投放运行时 → 装配载荷 → `hvigorw assembleHap`；规范文档见 [`dsh-desktop-ark/specs/`](dsh-desktop-ark/specs/)）。

> Electron-on-HarmonyOS 构建的第一阶段从 `../harmonypc-electron` submodule 拷贝运行时；HarmonyOS Edition 则从 `../ohos-node-bridge/runtime/` 取 `libnode.so`，并由 `ohos-node-bridge/runtime.lock.json` 以 SHA-256 锁定。

## Submodule 清单

全部条目记录在 [`.gitmodules`](.gitmodules)：

| 路径 | 远端 |
|---|---|
| `dsh-desktop` | https://github.com/fellow99/dsh-desktop.git |
| `dsh-desktop-hos` | https://github.com/fellow99/dsh-desktop-hos.git |
| `dsh-desktop-ark` | https://github.com/fellow99/dsh-desktop-ark.git |
| `ohos-node-bridge` | https://github.com/fellow99/ohos-node-bridge.git |
| `deepseek-harness` | https://github.com/deepseek-ai/deepseek-harness.git |
| `dsh-market` | https://github.com/dsh-market/dsh-market.git |
| `harmonypc-electron` | https://atomgit.com/jianguoxu/harmonypc-electron.git |
| `dsh-desktop-website` | https://github.com/fellow99/dsh-desktop-website.git |

如需推进固定版本（例如接入新的 dsh 版本），可执行 `git submodule update --remote`，随后提交新的指针。

## License

[MIT](LICENSE) © 2026 fellow99
