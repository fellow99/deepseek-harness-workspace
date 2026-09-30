# 规格说明 — 版本更新 0.2.0

- **需求编号**：101-version-0.2.0
- **需求名称**：版本更新：0.2.0
- **日期**：2026-09-30
- **状态**：已确认

## 1. 概述

将 deepseek-harness 上游从 `dsh-v0.1.7-rc.2`（desktop 基线）/ `dsh-v0.1.5-rc.2`（hos 基线）
升级到 `dsh-v0.2.0-rc.2`，并随之把两个封装壳 `dsh-desktop`、`dsh-desktop-hos` 的
产品版本从 0.1.7 / 0.1.5 升到 **0.2.0**，同步更新 `dsh-desktop-website` 的版本展示。

deepseek-harness 已由用户手动切换到本地分支 `Branch_dsh-v0.2.0-rc.2`（tag
`dsh-v0.2.0-rc.2`）。**Git 分支不改动**（除 dsh-desktop 按惯例打 `v0.2.0` tag 外）。

## 2. 目标 / 非目标

### 目标
- dsh-desktop：基于 `dsh-v0.2.0-rc.2` 重建补丁、版本改 0.2.0、重新打包、真机无关的本地功能测试、打 `v0.2.0` tag。
- dsh-desktop-hos：基于 `dsh-v0.2.0-rc.2` 重建补丁、版本改 0.2.0（versionCode 1007）、重新编译构建、部署真机功能测试。
- dsh-desktop-website：版本展示与下载说明更新到 0.2.0。
- 父工程 README 版本表同步。

### 非目标
- 不改动 deepseek-harness 上游源码（只通过 patch 做幂等适配）。
- 不新增功能、不重构壳工程逻辑。
- 不改动 Git 分支结构（dsh-desktop 打 tag 除外）。

## 3. 补丁适配结论（基于 dsh tag diff 分析）

补丁按 dsh 版本分目录存放（`patches/<dsh-tag>/`）。从旧 tag 到 `dsh-v0.2.0-rc.2` 的目标文件
漂移情况决定了每个补丁是「原样复制」还是「重新锚定」。

### 3.1 dsh-desktop（基线 dsh-v0.1.7-rc.2 → dsh-v0.2.0-rc.2，共 3 个补丁）

| 补丁 | 目标文件 | 漂移 | 处置 |
|---|---|---|---|
| dsh-disable-hmr | apps/cli/src/profile-boot.ts | 未变 | 原样复制（新 tag 已无 HMR 路径，补丁为无害 no-op，保留以防未来恢复） |
| dsh-disable-native-picker | packages/host/directory-picker-auto/src/resolve.ts | 未变 | 原样复制 |
| dsh-disable-welcome-notice | ui-settings-models/{src/client/index.ts, tests/apply.client.spec.ts} | 已变 | **重新锚定**：上游把首行注释 "internal-testing" 改为 "preview-notice"，新增 analytics import 与 `track:` 行；spec 新增 analytics 断言。仍需移除 DeepSeek onboarding 注册（新 tag 仅给 welcome-notice 加了 `dshDesktop` 守卫，DeepSeek 注册未加守卫）。 |

### 3.2 dsh-desktop-hos（基线 dsh-v0.1.5-rc.2 → dsh-v0.2.0-rc.2，原 15 个补丁）

| 补丁 | 处置 | 说明 |
|---|---|---|
| dsh-symlink-to-copy | **重写** | profile.ts 上游 769 行漂移，node:fs import 块全变 |
| dsh-allow-all-interfaces | 原样 | startup.ts 未变 |
| dsh-disable-hmr | **移除** | 新 tag 已删除整个 live-reload/HMR 回退路径，补丁失效且无意义 |
| dsh-disable-native-picker | 原样 | resolve.ts 未变 |
| dsh-disable-welcome-notice | **重新锚定** | 同 desktop，且漂移更大（configForms 重命名、analytics、`dshDesktop` 守卫已覆盖 welcome-notice）；仍需移除 DeepSeek 注册 |
| dsh-rebrand | **重写（4/5 文件）** | locale/zh/en、ui-conversation locales、EmptyHero 上游大幅变更；FishLogo.tsx 未变保留 |
| dsh-flock-openharmony | 原样 | flock.ts 未变 |
| dsh-hardlink-to-rename | **重锚（2 文件）** | generation.ts 新增 import；index.ts import 重构 |
| dsh-fs-hardlink-fallback | 原样 | 偏移自动吸收 |
| dsh-fs-remove-primitive | **重锚（1 文件）** | fs/src/index.ts 新增 `import { FsError }`，补上下文即可 |
| dsh-fs-write-bytes | 原样 | 偏移吸收 |
| dsh-extra-writable-roots | 原样 | roots.ts 未变 |
| dsh-disable-lefthook-postinstall | **重锚（1 行）** | package.json scripts 新增 `start:web` 行，补 1 行上下文 |
| dsh-attachment-durable-walk-sandbox | 原样 | store.ts 未变 |
| dsh-fs-chmod-primitive | 原样 | 偏移吸收 |

> 最终 HOS 补丁数：15 - 1（移除 hmr）= **14 个**。

## 4. 版本与构建产物

| 工程 | 旧版本 | 新版本 | 关键文件 |
|---|---|---|---|
| dsh-desktop | 0.1.7 | **0.2.0** | package.json `version`、build-dsh.mjs `patchDir` pin |
| dsh-desktop-hos | 0.1.5 | **0.2.0**（versionCode 1005→1007） | AppScope/app.json5、build-dsh.mjs `patchDir`+`patchFiles` |
| dsh-desktop-website | 展示 0.1.7/0.1.5 | 展示 **0.2.0** | index.html、index_zh.html、README.md |

## 5. 验收标准（详见 test-cases.md）

- dsh-desktop：`pnpm i && npm run package` 成功；启动后设 DeepSeek API KEY、建工作区、一次对话均正常；打 `v0.2.0` tag。
- dsh-desktop-hos：编译构建成功；部署真机后设 API KEY、建工作区、一次对话均正常。
- dsh-desktop-website：版本展示为 0.2.0，无残留 0.1.7/0.1.5。
- 全部 git 提交遵循 `git-commit` 技能 Conventional Commits 前缀。

## 6. 风险与缓解

- **补丁重写风险**（symlink-to-copy、rebrand、welcome-notice）：每个补丁应用后用 `git apply --check` 校验，构建阶段再跑一次 dsh 单测/构建验证；若某补丁意图已被上游覆盖（如 hmr），移除而非硬套。
- **真机依赖**（hos）：hdc 已就绪；若设备断连则报告并暂停。
- **API KEY 安全**：仅用于本地功能测试，严禁写入任何提交/补丁/文档。
