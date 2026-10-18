# 规格说明 — 版本更新 0.2.1

- **需求编号**：101-version-0.2.1
- **需求名称**：版本更新：0.2.1
- **日期**：2026-09-30
- **状态**：已确认

## 1. 概述

将 deepseek-harness 上游从 `dsh-v0.2.0-rc.2` 升级到 `dsh-v0.2.1-rc.2`，并随之把两个封装壳
`dsh-desktop`、`dsh-desktop-hos` 的产品版本从 0.2.0 升到 **0.2.1**，同步更新 `dsh-desktop-website`
的版本展示与父工程 README 版本表。

deepseek-harness 已由用户手动切换到本地分支 `Branch_dsh-v0.2.1-rc.2`（tag
`dsh-v0.2.1-rc.2`，当前 HEAD 即该 tag，工作树干净）。**Git 分支不改动**（除 dsh-desktop 按惯例
打 `v0.2.1` tag 外）。

## 2. 目标 / 非目标

### 目标
- dsh-desktop：基于 `dsh-v0.2.1-rc.2` 重建补丁、版本改 0.2.1、`pnpm i && npm run package` 重新打包、本地功能测试（API KEY/工作区/对话）、打 `v0.2.1` tag。
- dsh-desktop-hos：基于 `dsh-v0.2.1-rc.2` 重建补丁、版本改 0.2.1（versionCode 1007→1009）、重新编译构建、部署真机功能测试。
- dsh-desktop-website：版本展示与下载说明更新到 0.2.1。
- 父工程 README / README_zh 版本表同步。

### 非目标
- 不改动 deepseek-harness 上游源码（只通过 patch 做幂等适配）。
- 不新增功能、不重构壳工程逻辑。
- 不改动 Git 分支结构（dsh-desktop 打 tag 除外）。

## 3. 补丁适配结论（基于 dsh-v0.2.0-rc.2 → dsh-v0.2.1-rc.2 diff 分析）

补丁按 dsh 版本分目录存放（`patches/<dsh-tag>/`）。从 `dsh-v0.2.0-rc.2` 到 `dsh-v0.2.1-rc.2`
的目标文件漂移情况决定了每个补丁是「原样复制」还是「重新锚定 / 重写」。

> 漂移判定方法：对每个补丁 `+++ b/<file>` 列出的目标文件，执行
> `git -C deepseek-harness diff dsh-v0.2.0-rc.2 dsh-v0.2.1-rc.2 -- <file>`，
> 有改动即 DRIFT（需重新锚定），无改动即 CLEAN（原样复制）。

### 3.1 dsh-desktop（基线 dsh-v0.2.0-rc.2 → dsh-v0.2.1-rc.2，共 3 个补丁）

| 补丁 | 目标文件 | 漂移 | 处置 |
|---|---|---|---|
| dsh-disable-hmr | apps/cli/src/profile-boot.ts | 已变（`initializeProfileFromDefault` 重构） | **重锚**：`overlays:` 锚点行仍在（line 321），仅行号偏移，补丁逻辑不变（`DSH_DISABLE_HMR` 未设置时为 no-op；保留以对齐 0.2.0 行为）。 |
| dsh-disable-native-picker | packages/host/directory-picker-auto/src/resolve.ts | 已变 | **重锚**：函数体从 `if (facts.bindHost !== '127.0.0.1')` 改为 `if (facts.allowsRemoteAuthorities \|\| !isLoopbackHost(facts.bindHost))`，在函数首行插入 `if ('electron' in process.versions) return 'browse'`。 |
| dsh-disable-welcome-notice | ui-settings-models/{src/client/index.ts, tests/apply.client.spec.ts} | 已变（+ModelReasoningEfforts 等大量新增，index.ts 仅 -2 行） | **重锚**：index.ts 移除 onboarding 注册（welcome-notice + deepseek-official）；spec 同步移除相应 onboarding 断言（spec 从 ~330 行增到 353 行，新增 ModelReasoningEfforts 相关测试须保留）。 |

> 说明：构建链 `build:lib:client` / `build:web` 走 `tsc -b` + `tsdown`，不运行 vitest 单测；
> 但为保持上游可构建/可测的一致性，welcome-notice 补丁仍同步维护 spec（与 0.2.0 做法一致）。

### 3.2 dsh-desktop-hos（基线 dsh-v0.2.0-rc.2 → dsh-v0.2.1-rc.2，共 13 个补丁）

| 补丁 | 目标文件 | 漂移 | 处置 |
|---|---|---|---|
| dsh-allow-all-interfaces | packages/bundle/web-app/src/startup.ts | 已变 | **重锚**（需核对 startup.ts 变更点） |
| dsh-disable-native-picker | packages/host/directory-picker-auto/src/resolve.ts | 已变 | **重锚**（同 desktop §3.1） |
| dsh-disable-welcome-notice | ui-settings-models/{index.ts, apply.client.spec.ts} | 已变 | **重锚**（同 desktop §3.1，hos 侧通常锚点更保守） |
| dsh-rebrand | locale/{en,zh}.ts、ui-conversation locales、EmptyHero.tsx、FishLogo.tsx | 文件有改动，但补丁 hunk 未命中变更区 | **原样沿用**：实测精确锚点应用成功（实际核对：locales.ts:77,457 / EmptyHero.tsx / FishLogo.tsx） |
| dsh-flock-openharmony | native/system/.../flock.ts | 未变 | 原样复制 |
| dsh-hardlink-to-rename | session-persistence-jsonl/{generation,index,openharmony}.ts | index.ts 有改动，generation/openharmony 未变 | **原样沿用**：实测精确锚点应用成功，三处 post-image 均在目标树中 |
| dsh-fs-hardlink-fallback | packages/fs/fs-local/src/fsio.ts | 未变 | 原样复制 |
| dsh-fs-remove-primitive | fs-local/{index,remove}.ts、fs-sandbox/index.ts、fs/{index,types}.ts | 未变 | 原样复制 |
| dsh-fs-write-bytes | fs/{types,index}.ts、fs-local/{fsio,index}.ts、fs-sandbox/index.ts | 未变 | 原样复制 |
| dsh-extra-writable-roots | packages/sandbox/sandbox/src/roots.ts | 未变 | 原样复制 |
| dsh-disable-lefthook-postinstall | package.json | 已变 | **重锚**（核对 scripts 变更行） |
| dsh-attachment-durable-walk-sandbox | packages/attachment/attachment-local/src/store.ts | 未变 | 原样复制 |
| dsh-fs-chmod-primitive | fs-local/chmod.ts、fs-local/index.ts、fs-sandbox/index.ts、fs/{index,types}.ts | 未变 | 原样复制 |

> 最终 HOS 补丁数：**13**（7 原样 + 6 重建/重锚），与 0.2.0 的 13 个 .patch 一致（0.2.0 目录内第 14 项是 `REMOVE-dsh-symlink-to-copy.md` 说明文件，非补丁）。
> 注意：dsh-symlink-to-copy 在 0.2.0 已被上游覆盖（见 `patches/dsh-v0.2.0-rc.2/REMOVE-dsh-symlink-to-copy.md`），0.2.1 继续不纳入。

## 4. 版本与构建产物

| 工程 | 旧版本 | 新版本 | 关键文件 |
|---|---|---|---|
| dsh-desktop | 0.2.0 | **0.2.1** | package.json `version`、build-dsh.mjs `patchDir` pin |
| dsh-desktop-hos | 0.2.0（versionCode 1007） | **0.2.1**（versionCode **1009**） | AppScope/app.json5、build-dsh.mjs `patchDir` pin |
| dsh-desktop-website | 展示 0.2.0 | 展示 **0.2.1** | index.html、index_zh.html、README.md |

> versionCode：0.2.0 用 1007；0.2.1 按 0.1.5→0.1.7(1005→1007) 的 +2 规律取 **1009**。

## 5. 验收标准（详见 test-cases.md）

- dsh-desktop：`pnpm i && npm run package` 成功；启动后设 DeepSeek API KEY、建工作区、一次对话均正常；打 `v0.2.1` tag。
- dsh-desktop-hos：编译构建成功；部署真机后设 API KEY、建工作区、一次对话均正常。
- dsh-desktop-website：版本展示为 0.2.1，无残留 0.2.0 版本串。
- 全部 git 提交遵循 `git-commit` 技能 Conventional Commits 前缀。

## 6. 风险与缓解

- **补丁重锚/重写风险**（native-picker、welcome-notice、rebrand、hardlink-to-rename、allow-all-interfaces、lefthook）：每个补丁应用后用 `git apply --check` 校验；若某补丁意图已被上游覆盖（如 symlink-to-copy 先例），移除而非硬套，并在 patches 目录留 `REMOVE-*.md` 说明。
- **真机依赖**（hos）：hdc 已就绪但当前 `hdc list targets` 为空；构建完成后若设备断连则报告并暂停部署测试。
- **API KEY 安全**：仅用于本地功能测试，严禁写入任何提交/补丁/文档。
