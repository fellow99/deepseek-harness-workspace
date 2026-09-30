# 实施计划 — 版本更新 0.2.0

需求编号：101-version-0.2.0 ｜ 分支：不改动（dsh-desktop 打 tag 除外）

## 执行顺序（严格串行，依赖关系）

```
[dsh-desktop] 清理 → 建补丁 → 改版本pin → 打包 → 本地功能测试 → tag v0.2.0
        │  测试通过
        ▼
[dsh-desktop-hos] 清理 → 建补丁 → 改版本/pin → 编译构建 → 部署真机功能测试
        │  测试通过
        ▼
[dsh-desktop-website] 版本展示更新
        ▼
[代码审查] requesting-code-review + receiving-code-review（三处变更）
        ▼
[收尾] 父 README 版本表 + git 提交（git-commit 前缀）+ 回归确认
```

> 子 agent 约束：同一时间最多并发 1 个。补丁重写委托 1 个 deep agent 执行，主进程等待结果后再做版本/构建/测试。

## 阶段 1 — dsh-desktop

1. 清理中间/临时文件：`out/`(1.8GB)、`dsh-dist/`(900MB)、`runtime/`(150MB)、`.vite/`（均为 .gitignore 构建产物，安全删除）。
2. 新建 `patches/dsh-v0.2.0-rc.2/`：
   - `dsh-disable-hmr.patch` ← 复制自 `dsh-v0.1.7-rc.2/`
   - `dsh-disable-native-picker.patch` ← 复制
   - `dsh-disable-welcome-notice.patch` ← 重新锚定（见 spec §3.1）
3. `scripts/build-dsh.mjs`：`patchDir` → `patches/dsh-v0.2.0-rc.2`（patchFiles 列表不变，3 个）。
4. `package.json`：`version` → `0.2.0`。
5. 校验补丁：`git apply --check`（在 deepseek-harness 上）逐一通过。
6. 打包：`pnpm i && npm run package`（含 build:dsh 自动 apply 补丁 + collect + fetch-runtime）。
7. 功能测试（Playwright / 直接启动 exe）：设 API KEY → 建工作区 → 一次对话。
8. 通过后：`git tag v0.2.0`（在 dsh-desktop 子仓库）。

## 阶段 2 — dsh-desktop-hos

1. 清理中间/临时文件（`.hvigor/`、`dsh-dist/`、`oh_modules/` 缓存等构建产物，保留源码与配置）。
2. 新建 `patches/dsh-v0.2.0-rc.2/`（14 个，见 spec §3.2）：8 原样 + 6 重写/重锚，**不含** dsh-disable-hmr。
3. `scripts/build-dsh.mjs`：`patchDir` → `patches/dsh-v0.2.0-rc.2`，`patchFiles` 移除 `dsh-disable-hmr.patch`。
4. `AppScope/app.json5`：`versionName` → `0.2.0`，`versionCode` → `1007`。
5. 校验补丁：`git apply --check` 逐一通过。
6. 编译构建：collect-runtime → build-dsh → collect-dsh → HAP build（hvigor / DevEco）。
7. 部署真机（hdc install + aa start）：设 API KEY → 建工作区 → 一次对话。

## 阶段 3 — dsh-desktop-website

更新 `index.html`、`index_zh.html`、`README.md` 中所有 0.1.7/0.1.5/dsh-v0.1.x 版本串 → 0.2.0/dsh-v0.2.0-rc.2。

## 阶段 4 — 代码审查

- `requesting-code-review`：对三处变更的 diff 发起审查（Critical/Important/Minor 分级）。
- `receiving-code-review`：逐条验证，修复 Critical/Important，技术性质疑无效项。

## 阶段 5 — 收尾

- 父工程 `README.md` / `README_zh.md` 版本表更新。
- 各子仓库按 `git-commit` 技能 Conventional Commits 前缀提交。
- 回归确认：dsh-desktop 打包产物版本=0.2.0、hos HAP versionName=0.2.0、website 无残留旧版本。

## 工件

- 规格：`specs/101-version-0.2.0/{spec,plan,tasks,test-cases}.md`
- 日志：`dsh-desktop/logs/20260930-1/`（DEV_CHECKLIST、REVIEW_REPORT、TEST_REPORT）
- 日志：`dsh-desktop-hos/logs/20260930-1/`
