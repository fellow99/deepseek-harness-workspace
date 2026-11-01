# 204 · GIT 插件（技术方案 · 父工程）

> HOW：三壳（hos/ark/android）的统一做法 + 落点。WHAT 见 [spec.md](./spec.md)（`FR-204-*`）。
> 各壳细案：`dsh-desktop-hos/specs/204-plugin-git/plan.md`、`dsh-desktop-ark/…/plan.md`、`dsh-desktop-android/…/plan.md`。

## 1. 统一架构

```
seam 插件包（host 面：21 工具 + client 面：设置页）
   │  cordis.patch.yml `insert` 单行挂载（host 平面）
   ▼
ctx.tools.register(defineTool(...))            ← 21 个工具（对 agent 可见）
ctx.fs.resolve(dir)+processPath → 仓库根(绝对)  ← 围栏决策点
   ├─ 工作区内 → 文件适配层(async, 双检包含) → isomorphic-git
   └─ 工作区外 → FS_SANDBOX_DENIED + approval(fail-closed)
ctx.credentials + <dsh 配置目录>/.git-credentials（0600）
```

## 2. 各壳落点

| 关注点 | `dsh-desktop-hos` | `dsh-desktop-ark` | `dsh-desktop-android` |
|---|---|---|---|
| 插件包 | `plugins/harmony-plugin-git` | `packages/git-ohos` | `packages/git-android` |
| 挂载 | `profiles/desktop/cordis.patch.yml` `insert`（host 平面，FR-204-003 例外，已在 `plugins/README.md` 登记） | `packages/bundle-ohos/cordis.patch.yml` `insert` | `packages/bundle-android/cordis.patch.yml` `insert` |
| 工具实现 | `lib/tools/*.js`（一动词一文件） | `lib/tools/*.js` | `lib/tools/*.js` |
| git 引擎 | `isomorphic-git`（依赖随包；`jsdiff` 用宿主闭包） | `isomorphic-git` | `isomorphic-git@1.44.2`（嵌套闭包随包 vendor） |
| 依赖物化 | 构建期 | `assemble-payload.mjs::collectGitRuntime` | `assemble-payload.mjs`（git 闭包断言 + 随包 vendor） |
| 凭据文件 | `<dsh 配置目录>/.git-credentials` | 同 | 同（`0600`，临时文件+rename） |
| 网络协议 | http(s) | http(s) | 仅 http(s)（`git://` 不识别） |

## 3. 关键实现要点（跨壳共有）

- **defineTool**：每个工具必须经 `defineTool`，禁止行内 `required:true` 简写；抛错发生在 `apply()` 内时宿主不记 `__hostError`，易误判为「插件无痕迹」。
- **文件适配层**：所有 promises 方法 `async`；每个路径做 `ctx.fs` 解析 + 词法包含 + realpath 包含双检；拒绝则返回 `FS_SANDBOX_DENIED`。
- **diff**：`workdir/staged/head/refs` 四模式；`head` 由宿主 `jsdiff` 生成 unified diff；二进制（含 NUL）只报 `Binary files … differ`。
- **revert**：自建（上游无 reversal），fail-closed；单父提交 + 工作区未分叉才执行。
- **凭据**：`GIT_CREDENTIAL_<HOST>`；`.git-credentials` 原生格式、`0600`、保留其它行；缺凭据给可照抄占位行；拒一次即止（最多重试 1 次）。
- **调试**：debug 允许 `--inspect` 挂宿主 Node inspector（仅回环），release 关闭。

## 4. 验证

| 层 | 手段 | 判据 |
|---|---|---|
| 构建期 | 装配脚本 | 入口存在、isomorphic-git 闭包完整、版本等于 pin；缺即失败 |
| 运行期 | 设备/模拟器 | 启动日志「21 tools registered」；`git_status`/`commit`/`log` 正确 |
| 网络 | 设备 + 远端 | clone/push（http(s)）通过；凭据认证/公开仓库 |
| 安全 | 用例 | 越界 `FS_SANDBOX_DENIED`；路径逃逸被拒；设置页不回显口令 |

## 5. 风险

- 依赖 `isomorphic-git` 闭包完整性（external 依赖必须随包供给）→ 构建期断言。
- 适配层非同 async → git 调用永久挂起（FR-204-024）。
- 网络不可达/DNS（如 github 直连）→ 如实报告，不伪造。
