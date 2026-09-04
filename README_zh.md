[English](./README.md) | 中文

---

# DeepSeek Harness Workspace

> 本仓库是一个聚合工程（workspace），把将 [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) 封装为一等桌面应用所需的全部工程汇聚到一处——Windows/Linux 端基于 Electron 桌面壳，鸿蒙端基于 "Electron-on-HarmonyOS" 运行时。所有仓库均以 **git submodule** 形式固定版本，使本 workspace 的目录结构恰好还原各封装工程构建脚本所期望的兄弟目录布局（`../deepseek-harness`、`../dsh-market`、`../harmonypc-electron`）。

---

## 这是什么

DeepSeek Harness（`dsh`）是 DeepSeek AI 开源的 Agent 运行框架（harness），采用"一切皆插件"架构；其原生入口是 `dsh web`（浏览器 Web UI）。两个姊妹封装工程将该 Web UI 变成原生桌面应用——完整复用 dsh 前端，并在进程内托管 dsh Host：

- **`deepseek-harness-desktop`** —— 面向 Windows / Linux 的 Electron 桌面壳（macOS 后续支持）
- **`deepseek-harness-harmony`** —— 面向鸿蒙的桌面移植版（2in1 / 平板，打包为 HAP），基于 Electron-on-HarmonyOS 运行时

两个封装工程消费完全相同的上游构件。本 workspace 正是承载这些构件（以 submodule 形式并列存放）的容器：两个封装工程可以在一个地方完成克隆、构建与开发，其 `../兄弟目录` 引用在 workspace 内部即可解析。

## 仓库布局

```
deepseek-harness-workspace/            # 本仓库 —— submodule 容器
├── deepseek-harness-desktop/          # [submodule] Electron 桌面封装（Windows/Linux）
├── deepseek-harness-harmony/          # [submodule] 鸿蒙桌面封装（2in1/平板，HAP）
├── deepseek-harness/                  # [submodule] dsh —— 被封装的 Agent 主机（上游）
├── dsh-market/                        # [submodule] 可视化插件市场（npm 包 "dshmarket"）
├── harmonypc-electron/                # [submodule] Electron-on-HarmonyOS 运行时（Electron 37 / Node 22.17.0）
├── deepseek-harness-desktop-website/  # [submodule] 两个封装工程的产品官网
└── harmonypc-electron-versions/       # （普通目录，非 submodule）运行时发行版归档 + Electron 头文件指南
```

### 各角色一览

| 目录 | 角色 | 被谁消费 |
|---|---|---|
| `deepseek-harness-desktop` | Electron 桌面壳 —— 主封装工程 | — |
| `deepseek-harness-harmony` | 鸿蒙桌面移植版（以 desktop 为基准） | — |
| `deepseek-harness` | 被封装的 Agent 主机（`dsh`，上游源码引用） | 两个封装工程 |
| `dsh-market` | 内置可视化插件市场 | 两个封装工程 |
| `harmonypc-electron` | Electron-on-HarmonyOS 运行时（原生 SO + ArkTS 桥接层） | 仅 harmony |
| `deepseek-harness-desktop-website` | 两个封装工程的产品官网 | — |
| `harmonypc-electron-versions` | Electron-on-HarmonyOS 发行版归档（v34/v37/v40）+ 用于重编译原生模块（如 `better-sqlite3`）的 Node 头文件指南 | harmony 工具链 |

## 共享架构

`dsh` 已完成 **Host/Client 拆分**，其 Web 服务器**同时托管 SPA dist 与 `/api`**。因此两个封装工程采用相同的**进程内 Host + webserver + 同源数据面**方案：

- 封装工程的主进程动态导入 dsh 的 `runProfile('desktop')`，在进程内托管 dsh Host（webserver + apiProxy + connection 统一绑定一个端口）。
- 就绪后，渲染进程对该 webserver 执行 `loadURL(...)`，运行**未经修改的标准 dsh Web UI**，复用其既有 `WebApiClient`（HTTP 上行 + WebSocket 下行）。
- 结果：**零 CORS、零鉴权、零自定义协议、零新增 IPC 载体 —— 上游零改动**（仅有幂等补丁）。

### 各平台的差异适配

| 方面 | 桌面端（Electron） | 鸿蒙端（Electron-on-HarmonyOS） |
|---|---|---|
| 目标平台 | Windows + Linux（macOS 后续） | HarmonyOS 2in1 / 平板（HAP） |
| Web 服务器绑定 | `127.0.0.1:<随机端口>` | `0.0.0.0:<随机端口>`，渲染进程经局域网 IP 连接并改写 `Host`/`Origin`（HarmonyOS NEXT 回环网络隔离） |
| dsh 补丁 | 2 个（禁用 HMR、禁用原生目录选择器） | 4 个（共享的 2 个 + 符号链接→`cpSync` 拷贝、放行全接口绑定） |
| MVP 能力 | 脚手架与 dsh 消费已完成；托盘 / 通知 / 无边框窗口 / 剪贴板图片粘贴（实时状态见其 README） | 已在真机验证 —— HarmonyOS 6.1.0.135（API 24）；Web UI 全功能可用；会话持久化与插件市场已内置；终端 / 进程沙箱暂缺（aarch64 构件缺失） |

## 快速开始

```bash
git clone --recursive https://github.com/fellow99/deepseek-harness-workspace.git
cd deepseek-harness-workspace
git submodule update --init --recursive   # 拉取/刷新全部固定版本的 submodule
```

本 workspace 的目录布局已满足各封装工程的兄弟目录前置要求（`../deepseek-harness`、`../dsh-market`、`../harmonypc-electron`）。构建与运行说明由各封装工程各自维护、可能与本概述存在出入：

- **桌面端**：见 [`deepseek-harness-desktop/README.md`](deepseek-harness-desktop/README.md)（`npm run build:dsh` 应用 2 个补丁并构建 dsh + dsh-market，随后 `npm start` / `npm run package`）。
- **鸿蒙端**：见 [`deepseek-harness-harmony/README.md`](deepseek-harness-harmony/README.md)（三阶段构建 `collect-runtime → build-dsh → collect-dsh`，随后经 DevEco Studio / hvigor 构建并签名 HAP）。

> 鸿蒙构建的第一阶段从 `../harmonypc-electron` submodule 拷贝运行时；若需要使用除固定 submodule 之外的运行时版本，`harmonypc-electron-versions/` 中存有独立的发行版归档。

## Submodule 清单

全部条目记录在 [`.gitmodules`](.gitmodules)：

| 路径 | 远端 |
|---|---|
| `deepseek-harness-desktop` | https://github.com/fellow99/deepseek-harness-desktop.git |
| `deepseek-harness-harmony` | https://github.com/fellow99/deepseek-harness-harmony.git |
| `deepseek-harness` | https://github.com/deepseek-ai/deepseek-harness.git |
| `dsh-market` | https://github.com/dsh-market/dsh-market.git |
| `harmonypc-electron` | https://atomgit.com/jianguoxu/harmonypc-electron.git |
| `deepseek-harness-desktop-website` | https://github.com/fellow99/deepseek-harness-desktop-website.git |

如需推进固定版本（例如接入新的 dsh 版本），可执行 `git submodule update --remote`，随后提交新的指针。

## License

[MIT](LICENSE) © 2026 fellow99
