# 204 · GIT 插件（21 工具 · 功能规格 · 父工程）

> Module: 204-plugin-git ｜ 范围：`dsh-desktop-hos` / `dsh-desktop-ark` / `dsh-desktop-android`（`desktop` 无此模块：其宿主可直接用系统 git）
> Status: Implemented（as-built）｜ Last Updated: 2026-11-01
> 关联：各壳 `204-plugin-git/plan.md`；上游 `ctx.fs` / `ctx.sandbox` / `ctx.credentials` / `ctx.tools`；`110`（契约风格参考）

---

## 1. 模块概述

### 1.1 目的（Why）

在**无法依赖系统 git 二进制**的壳（HarmonyOS / Android：应用域禁止 spawn 任意二进制，且无预装 git）上，以**纯 JS 的 `isomorphic-git`** 实现
**21 个 git 工具**，供 agent 直接在会话工作区内完成 init/clone/status/diff/log/commit/branch/checkout/reset/merge/pick/revert/stash/tag/remote/fetch/pull/push/config，并在设置页提供**Git 凭据**管理。**不改上游源码**（C1），以 seam 插件形态交付。

### 1.2 解决的问题

- **无系统 git**：移动/鸿蒙应用域不能 spawn `git`；纯 JS 实现使其可用。
- **工作区围栏与提权**：仓库根必须先经 `ctx.fs` 解析；根在会话工作区外时按策略拒绝并走既有提权审批（fail-closed）。
- **凭据安全**：口令不进对话/日志/参数明文；只经凭据库与 `.git-credentials`（`0600`）存放读取。
- **isomorphic-git 的能力缺口**：缺 `diff`/`show`/`revert`/`rebase`，需宿主闭包补 diff、自建 show/revert，rebase 明确不交付。
- **回调方言坑**：适配层 promises 方法必须是 `async`，否则 isomorphic-git 判定为 callback 方言，导致所有 git 调用永久挂起。

### 1.3 范围

**包含**：插件包形态与挂载平面、21 工具面、仓库根围栏与提权、路径包含校验、凭据库与设置页、依赖物化、调试通道（loopback）。
**不包含**：`git rebase`、系统级 git 配置管理、上游源码改动。

---

## 2. 用户故事

- 作为**设备用户**，让 agent 在工作区内完成常见 git 操作（状态、提交、分支、推送），无需设备装 git。
- 作为**安全负责人**，越界仓库先审批，口令不外泄，`.git-credentials` 权限正确。
- 作为**打包者**，git 依赖随包物化、断言完整，缺产物即构建失败。

---

## 3. 功能需求

> RFC 2119。实现映射见 `plan.md`。

### 3.1 插件身份与挂载平面

- **FR-204-001**：本模块 MUST 以 seam 插件形态交付（包 → 载荷内 `node_modules/@dsh-*/<name>/`），MUST NOT 改动上游源码、MUST NOT 新增上游补丁。
- **FR-204-002**：插件 MUST 经（各自）profile 的 `cordis.patch.yml` 的 `- insert:` **单行**挂载于 host 平面，同时交付两物：① 21 个工具（host 平面注册，对 agent 可见）；② `dsh.client` 声明被 client 扫描，纳入设置页启动图。挂载后 MUST NOT 出现重复注册/重复工具名。
- **FR-204-003**：exceptions：若常规做法是把插件挂到 agent preset 行，则「需 client 面的插件」MUST 改走 host 行（否则 `dsh.client` 不被扫描、设置页失效）；该例外 MUST 在对应 `plugins/README.md` 与本模块登记。
- **FR-204-004**：插件 MUST 在启动期自报一行可检索的可见性日志（工具条数），使「行是否真激活」可判定。
- **FR-204-005**：插件依赖的上游服务 MUST 通过注入/查询取得；缺失服务 MUST NOT 让整棵插件树挂起（宿主平面某行 pending 会拖垮其余行）。

### 3.2 工具面（恰好 21 个）

- **FR-204-010**：工具面 MUST 恰好为：`git_init`、`git_clone`、`git_status`、`git_diff`、`git_log`、`git_show`、`git_add`、`git_commit`、`git_branch`、`git_checkout`、`git_reset`、`git_merge`、`git_cherry_pick`、`git_revert`、`git_stash`、`git_tag`、`git_remote`、`git_fetch`、`git_pull`、`git_push`、`git_config`。同名工具 MUST NOT 注册两次。
- **FR-204-011**：每个工具 MUST 用上游 `defineTool` 产出（`ctx.tools.register()` 会拒绝行内 `required: true` 简写，裸 schema 会被拒并抛错）。
- **FR-204-012**：仓库相关工具 MUST 接受 `dir`（仓库根）并解析到同一会话工作区；`dir` 省略 MUST 报错而非猜测。
- **FR-204-013**：`git_diff` MUST 支持 `workdir`/`staged`/`head`/`refs` 四种模式、pathspec 过滤与 `stat` 汇总；二进制（含 NUL）MUST 只报 `Binary files … differ`，MUST NOT 输出内容级差异。`git_diff` 在非 `refs` 模式收到 `ref/ref2` MUST 明确报错并指向 `mode:"refs"`。
- **FR-204-014**：`git_status` MUST 按 staged/modified/deleted/untracked/conflicted 分组；冲突判定 MUST 依据工作区文件里的完整冲突标记组。
- **FR-204-015**：`git_revert` MUST 自建实现且 fail-closed：目标提交非恰好 1 个父提交、或工作区该路径已分叉时 MUST 拒绝并列出冲突路径，MUST NOT 静默覆盖。
- **FR-204-016**：`git_remote` MUST 提供只读的服务端 refs 探测（不落地），用于连通性/认证校验。
- **FR-204-017**：网络类工具（clone/fetch/pull/push）MUST 透传取消信号并有墙钟预算；超时 MUST 如实报告已用时与建议（`depth`/`singleBranch`）。
- **FR-204-018**：任何工具 MUST NOT 在参数或返回值携带凭据明文；远端地址进入输出前 MUST 抹掉 userinfo。
- **FR-204-019**：网络协议面 MUST 只支持 `http(s)`（各壳实现）；`git://`/ssh 若未实现 MUST 明确不支持，MUST NOT 假装可用。

### 3.3 仓库根围栏与提权

- **FR-204-020**：每次调用 MUST 先经 `ctx.fs` 把 `dir` 解析成进程可见的绝对仓库根，该根是后续全部 I/O 的唯一来源。
- **FR-204-021**：仓库根落在会话工作区之外且策略非 `danger-full-access` 时 MUST 以 `FS_SANDBOX_DENIED` 拒绝并给出同轮提权提示。
- **FR-204-022**：写类工具 MUST 暴露 `sandbox_permissions` + `justification` 提权参数；审批 MUST 经既有 approval 服务、一次性、fail-closed；拒绝 MUST 如实上报。
- **FR-204-023**：文件适配层 MUST 对**每个**路径做仓库根包含校验（词法 + realpath 双检）；`..` 逃逸、绝对路径、软链指向根外 MUST 拒绝且不落到 `node:fs`。
- **FR-204-024**：适配层 promises 方法 MUST 是 `async` 函数（同步抛错会被 isomorphic-git 判成 callback 方言 → 永久挂起）。

### 3.4 凭据

- **FR-204-030**：凭据 MUST 以 `credentialRef = GIT_CREDENTIAL_<HOST>` 存放，`<HOST>` = `host[:port]` 大写化、非字母数字 → `_`；值格式 `username:password`，按**第一个** `:` 切分。
- **FR-204-031**：凭据来源 MUST 两处且优先级固定：设置页保存过的主机以 dsh 凭据库为准，其余以 `<dsh 配置目录>/.git-credentials` 为准。
- **FR-204-032**：`.git-credentials` MUST 是 git 原生格式（一行一条带 userinfo 的 URL）、权限 `0600`、可手工编辑；写入 MUST 保留其它行与注释；删除只删匹配行，删空即删文件。
- **FR-204-033**：工具实际用到的凭据 SHOULD 镜像回该文件（尽力而为；配置目录只读/写满 MUST NOT 让已有凭据操作失败）。
- **FR-204-034**：缺凭据时 MUST 报出 key 名、设置入口、`.git-credentials` 路径与可直接照抄的占位行，MUST NOT 把裸 `401` 抛给模型。
- **FR-204-035**：凭据被拒 MUST 点名 key 名与文件路径，最多重试 1 次，MUST NOT 无限重试。
- **FR-204-036**：无账号（公开）仓库 MUST NOT 触发认证回调，零凭据可完成 clone/fetch/pull 与全部本地操作。
- **FR-204-037**：提交身份（`user.name`/`user.email`）MUST NOT 进凭据库；默认值 MUST 是明显占位值，MUST NOT 含真实个人信息。

### 3.5 设置页

- **FR-204-040**：设置页 MUST 提供「Git 凭据」区，可保存/查看状态/删除，并就地展示 ref 命名规则。
- **FR-204-041**：设置页 MUST NOT 回显口令明文（输入框 `type=password`，行内只显示主机名与 ref 名）。
- **FR-204-042**：状态读取失败 MUST 显示失败原因与形态诊断，MUST NOT 把「读不到」显示成「未知」而不给线索；只有读取成功才允许显示「已保存」。
- **FR-204-043**：设置页 MUST NOT 成为绕过围栏的路径：只写凭据库，不写仓库、不执行 git。

### 3.6 依赖与调试

- **FR-204-050**：`isomorphic-git` 及其运行时依赖 MUST 在**构建期**物化（随包自带），MUST NOT 依赖设备联网安装；MUST 有产物断言（入口存在、版本等于 pin、闭包完整）。
- **FR-204-051**：`jsdiff` 等可由宿主闭包提供的依赖 MUST 复用宿主闭包（不重复物化）。
- **FR-204-052**：debug 构建 MAY 通过 `--inspect` 挂宿主 Node inspector（**仅回环**），release MUST NOT 开启；调试通道 MUST NOT 改变业务行为。

---

## 4. 跨壳差异矩阵（as-built）

| 维度 | `desktop` | `dsh-desktop-hos` | `dsh-desktop-ark` | `dsh-desktop-android` |
|---|---|---|---|---|
| 是否提供 | 否（用系统 git） | 是 | 是 | 是 |
| 插件包 | — | `harmony-plugin-git` | `packages/git-ohos` | `packages/git-android` |
| 挂载 | — | desktop profile `insert` 单行（host 平面，有意例外） | `bundle` `insert` 行 | `bundle` `insert` 行 |
| git 依赖 | — | `isomorphic-git` + 宿主 `jsdiff` | `isomorphic-git` | `isomorphic-git` 1.44.2（闭包随包） |
| rebase | — | 不交付 | 不交付 | 不交付 |
| 网络协议 | — | http(s) | http(s) | 仅 http(s)（`git://` 不支持） |
| 证据 | — | 见其 logs | 见其 logs | 21 工具注册 + clone/push 实证 |

---

## 5. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 21 个工具 | agent 可见的 git 动作 | 名称、参数（`dir` + 工具专属）、提权参数 |
| 仓库根 | 围栏决策点 | 绝对路径、是否在工作区内 |
| 凭据 ref | 凭据键 | `GIT_CREDENTIAL_<HOST>`、值 `username:password` |
| `.git-credentials` | 凭据文件 | `0600`、git 原生格式、可编辑 |
| 文件适配层 | isomorphic-git 的 fs 适配 | async、双检包含、`ctx.fs` 唯一来源 |

---

## 6. 验收场景

- **工具注册**：启动日志自报 21 个工具；无重复名称。
- **本地流程**：`git_init` → `git_add` → `git_commit` → `git_status`/`git_log` 正确。
- **网络流程（凭据/公开）**：公开仓库 clone/fetch/pull 通过；私有仓库经 `.git-credentials`/凭据库认证。
- **越界提权**：仓库根在工作区外 → `FS_SANDBOX_DENIED` + 提权提示；批准后放行，拒绝则如实失败。
- **diff 四模式**：四模式 + pathspec + stat；二进制只报 differ。
- **revert fail-closed**：分叉/多父提交被拒并列出冲突路径。
- **凭据安全**：设置页不回显口令；缺凭据报可照抄占位行；凭据被拒最多重试 1 次。
- **路径安全**：`..`/绝对/软链出根被拒且不落 `node:fs`。

---

## 7. 非功能需求

- **零上游改动**：`deepseek-harness` 工作树零改动、零新补丁。
- **体量**：载荷增量 SHOULD 控制在 ~10 MiB 量级（isomorphic-git 闭包；jsdiff 复用宿主）。
- **健壮性**：适配层 async；网络工具可取消有预算；缺服务不致挂起整树。

---

## 8. 假设与约束

- 移动/鸿蒙应用域禁止 spawn 任意二进制；纯 JS git 是唯一可行路径。
- `isomorphic-git` 能力缺口（diff/show/revert/rebase）按 §3 处理。
- 网络协议以 `http(s)` 为主；无 `ssh`（无系统 ssh agent）。

---

## 9. 依赖

- 上游：`deepseek-harness` 的 `ctx.fs`/`ctx.sandbox`/`ctx.credentials`/`ctx.tools`/approval（只读）；`isomorphic-git`（纯 JS）。
- 工程内：各壳载荷管线与 bundle 适配层；`110`（契约风格）。
