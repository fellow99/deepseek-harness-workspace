# 101-version-0.1.7 — 版本更新：0.1.7

## 需求编号
101-version-0.1.7

## 需求名称
版本更新：0.1.7

## Git 分支
不改动（各工程沿用当前分支：父工程/desktop/hos/website = `main`；dsh = `Branch_dsh-v0.1.7-rc.2`）

## 涉及工程
- 父工程（deepseek-harness-workspace）
- deepseek-harness
- dsh-desktop
- dsh-desktop-hos（本轮不更新版本）
- dsh-desktop-website

## 背景
上游 `deepseek-harness` 已发布 `dsh-v0.1.7-rc.2`（本地分支 `Branch_dsh-v0.1.7-rc.2`，HEAD 477b4f4 = tag）。
`dsh-desktop` 当前为 `0.1.5` / `dsh-v0.1.5-rc.2`，需升级到 `0.1.7` / `dsh-v0.1.7-rc.2`，
完成 patch 适配、重新打包、打包产物功能实测；官网同步拆分两版口径；父工程文档与全部子模块指针推进。
`dsh-desktop-hos` 本轮不发版，停留在 `0.1.5` / `dsh-v0.1.5-rc.2`。

## 功能需求

### FR-1 deepseek-harness 子工程
- 使用本地已存在的分支 `Branch_dsh-v0.1.7-rc.2`，验证 HEAD 对应 tag `dsh-v0.1.7-rc.2`，工作树干净。
- 不推新分支、不改动源码。

### FR-2 dsh-desktop 子工程
- FR-2.1 清理所有中间文件、临时文件（`dsh-dist/`、`out/`、`.vite/`；`node_modules/` 保留）。
- FR-2.2 基于 `dsh-v0.1.7-rc.2` 制作 patch：新增 `patches/dsh-v0.1.7-rc.2/`（3 个补丁），更新 `scripts/build-dsh.mjs` 的 pin。
- FR-2.3 版本号设为 `0.1.7`（`package.json` + `package-lock.json`）。
- FR-2.4 重新打包：`pnpm i && npm run package`（等价完整链路：pnpm i → `npm run build:dsh` → `npm run package`，prepackage 自动执行 collect:dsh + fetch-runtime）。
- FR-2.5 打包后 CDP 实测基本功能：设置 DeepSeek API KEY → 创建工作区 → 进行一次对话。
- FR-2.6 README.md / README_zh.md 中 dsh tag 引用更新为 `dsh-v0.1.7-rc.2`。
- FR-2.7 `.github/workflows/ci.yml`、`release.yml` 中的 dsh tag 引用更新为 `dsh-v0.1.7-rc.2`。

### FR-3 dsh-desktop-hos 子工程
- 本轮不更新版本、不改动版本号与文案。
- 父工程推进其 main 最新提交对应的子模块指针。

### FR-4 dsh-desktop-website
- FR-4.1 `index.html` / `index_zh.html` 首行版本口径拆分：
  Desktop v0.1.7 · built on dsh-v0.1.7-rc.2；HarmonyOS v0.1.5 · built on dsh-v0.1.5-rc.2 · HarmonyOS device-verified (6.1.0.135 / API 24)。
- FR-4.2 两处状态区 desktop 状态由 v0.1.5 更新为 v0.1.7 / dsh-v0.1.7-rc.2（HarmonyOS 状态行保留 v0.1.5）。
- FR-4.3 `README.md` 版本段更新为 desktop v0.1.7 / dsh-v0.1.7-rc.2，hos v0.1.5 / dsh-v0.1.5-rc.2。

### FR-5 父工程
- FR-5.1 README.md / README_zh.md 版本表：desktop 行更新为 0.1.7 / dsh-v0.1.7-rc.2；hos 行保留 0.1.5。
- FR-5.2 推进全部子模块指针到当前提交：dsh（rc.2）、desktop（v0.1.7）、website（v0.1.7）、hos（main 最新）、market（main 最新）。

## 非功能需求
- NFR-1 提交注释必须且只能使用 `git-commit` 技能规定的 Conventional Commits 前缀
  （feat/fix/docs/style/refactor/perf/test/build/ci/chore/revert），严禁其它前缀。
- NFR-2 API KEY 等敏感信息仅运行时输入，严禁写入任何被提交的文件。
- NFR-3 patch 必须对新 tag 幂等、可干净应用与 reverse-check。
- NFR-4 dsh 构建使用 pnpm@11.7.0（通过 corepack shim 保证）。

## 关键技术风险（已定位）
- `dsh-disable-hmr.patch` 与 rc.2 冲突：`apps/cli/src/profile-boot.ts` 上下文变化，必须基于 rc.2 重新生成。
- `dsh-disable-welcome-notice.patch` 两处冲突：`packages/client/ui-settings-models/src/client/index.ts` 与 `tests/apply.client.spec.ts`，必须基于 rc.2 重新生成。
- `dsh-disable-native-picker.patch` 可干净应用，按新版本目录复制固化。
- Electron 二进制下载可能超时：使用 npmmirror 镜像规避。

## 验收标准
- AC-1 3 个新 patch 对 `dsh-v0.1.7-rc.2` 全部 `git apply --check` 通过，重复执行幂等。
- AC-2 desktop 打包成功，产物可启动，三步基本功能（API KEY/工作区/对话）全部通过并截图取证。
- AC-3 全部版本引用一致：desktop=0.1.7 / dsh-v0.1.7-rc.2，hos=0.1.5 / dsh-v0.1.5-rc.2。
- AC-4 代码评审无 Critical 遗留。
- AC-5 提交前缀 100% 合规；desktop 与 website 打注解 tag `v0.1.7` 并推送远端；各工程提交均推送。
- AC-6 父工程全部子模块指针推进并提交推送。
