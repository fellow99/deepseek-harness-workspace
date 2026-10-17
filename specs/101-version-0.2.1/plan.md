# 实施计划 — 版本更新 0.2.1

需求编号：101-version-0.2.1 ｜ 分支：不改动（dsh-desktop 打 tag 除外）

## 执行顺序（严格串行，依赖关系）

```
[dsh-desktop] 清理 → 建补丁(dsh-v0.2.1-rc.2) → 改版本/pin → 打包 → 本地功能测试 → tag v0.2.1
        │  测试通过
        ▼
[dsh-desktop-hos] 清理 → 建补丁(dsh-v0.2.1-rc.2) → 改版本/pin → 编译构建 → 部署真机功能测试
        │  测试通过
        ▼
[dsh-desktop-website] 版本展示更新
        ▼
[代码审查] requesting-code-review + receiving-code-review（三处变更）
        ▼
[收尾] 父 README 版本表 + git 提交（git-commit 前缀）+ 回归确认
```

> 子 agent 约束：同一时间最多并发 1 个。补丁重锚/重写委托 1 个子 agent 执行，主进程等待结果后再做版本/构建/测试。

## 阶段 1 — dsh-desktop

1. 清理中间/临时文件：`out/`、`dsh-dist/`、`runtime/`、`.vite/`（均为 .gitignore 构建产物，安全删除）。
2. 新建 `patches/dsh-v0.2.1-rc.2/`：
   - `dsh-disable-hmr.patch` ← 重锚（profile-boot.ts overlays 锚点行号偏移）
   - `dsh-disable-native-picker.patch` ← 重锚（resolve.ts 函数体改为 allowsRemoteAuthorities/isLoopbackHost）
   - `dsh-disable-welcome-notice.patch` ← 重锚（index.ts + spec 移除 onboarding 注册/断言，保留 ModelReasoningEfforts 新增）
3. `scripts/build-dsh.mjs`：`patchDir` → `patches/dsh-v0.2.1-rc.2`（patchFiles 列表不变，3 个）。
4. `package.json`：`version` → `0.2.1`。
5. 校验补丁：`git -C deepseek-harness apply --check` 逐一通过。
6. 打包：`pnpm i && npm run package`（含 build:dsh 自动 apply 补丁 + collect + fetch-runtime）。
7. 功能测试（启动 exe / Playwright）：设 API KEY → 建工作区 → 一次对话。
8. 通过后：`git tag v0.2.1`（在 dsh-desktop 子仓库）。

## 阶段 2 — dsh-desktop-hos

1. 清理中间/临时文件（`.hvigor/`、`dsh-dist/`、`oh_modules/` 缓存等构建产物，保留源码与配置）。
2. 新建 `patches/dsh-v0.2.1-rc.2/`（14 个，见 spec §3.2）：8 原样复制 + 6 重锚/重写（allow-all-interfaces、native-picker、welcome-notice、rebrand、hardlink-to-rename、lefthook）。**不含** dsh-symlink-to-copy（已移除）。
3. `scripts/build-dsh.mjs`：`patchDir` → `patches/dsh-v0.2.1-rc.2`（patchFiles 列表不变，14 个）。
4. `AppScope/app.json5`：`versionName` → `0.2.1`，`versionCode` → `1009`。
5. 校验补丁：`git -C deepseek-harness apply --check` 逐一通过。
6. 编译构建：collect-runtime → build-dsh → collect-dsh → HAP build（hvigor）。
7. 部署真机（hdc install + aa start）：设 API KEY → 建工作区 → 一次对话。

## 阶段 3 — dsh-desktop-website

更新 `index.html`、`index_zh.html`、`README.md` 中所有 0.2.0/dsh-v0.2.0-rc.2 版本串 → 0.2.1/dsh-v0.2.1-rc.2。

## 阶段 4 — 代码审查

- `requesting-code-review`：对三处变更的 diff 发起审查（Critical/Important/Minor 分级）。
- `receiving-code-review`：逐条验证，修复 Critical/Important，技术性质疑无效项。

## 阶段 5 — 收尾

- 父工程 `README.md` / `README_zh.md` 版本表更新。
- 各子仓库按 `git-commit` 技能 Conventional Commits 前缀提交。
- 回归确认：dsh-desktop 打包产物版本=0.2.1、hos HAP versionName=0.2.1/versionCode=1009、website 无残留 0.2.0。

## 工件

- 规格：`specs/101-version-0.2.1/{spec,plan,tasks,test-cases}.md`
- 日志：`dsh-desktop/logs/<YYYYMMDD>-<N>/`（DEV_CHECKLIST、REVIEW_REPORT、TEST_REPORT）
- 日志：`dsh-desktop-hos/logs/<YYYYMMDD>-<N>/`
