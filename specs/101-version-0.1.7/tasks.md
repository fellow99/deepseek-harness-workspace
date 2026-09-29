# 101-version-0.1.7 — 任务分解

> 执行模式：精简流程 + 连续执行（主链不停顿；仅 Critical/阻塞时暂停）。
> 子 agent：需要时创建 `auto` 子 agent。

## T0 准备
- [ ] T0.1 校验 dsh 工作树干净，分支 `Branch_dsh-v0.1.7-rc.2`，HEAD = tag `dsh-v0.1.7-rc.2`（477b4f4）
- [ ] T0.2 预置 pnpm shim（corepack → pnpm@11.7.0，PATH 最前），验证 `pnpm --version`
- [ ] T0.3 建立各工程 `logs/20260929-1/` 日志目录

## T1 Patch 重建（dsh-desktop）
- [ ] T1.1 新增 `patches/dsh-v0.1.7-rc.2/dsh-disable-hmr.patch`（基于 rc.2 profile-boot.ts 重新生成）
- [ ] T1.2 新增 `patches/dsh-v0.1.7-rc.2/dsh-disable-native-picker.patch`（旧补丁可应用，复制固化）
- [ ] T1.3 新增 `patches/dsh-v0.1.7-rc.2/dsh-disable-welcome-notice.patch`（基于 rc.2 ui-settings-models 两处重新生成）
- [ ] T1.4 更新 `scripts/build-dsh.mjs` 的 patchDir pin 到 dsh-v0.1.7-rc.2
- [ ] T1.5 更新 `.github/workflows/ci.yml`、`release.yml` 的 dsh tag 引用
- [ ] T1.6 3 个补丁 `git apply --check` 全部通过，reverse-check 幂等验证

## T2 版本号与文档
- [ ] T2.1 desktop：`npm version --no-git-tag-version 0.1.7`（同步 package.json + package-lock.json）
- [ ] T2.2 desktop：README.md / README_zh.md dsh tag 引用更新
- [ ] T2.3 website：index.html / index_zh.html 首行拆分版本口径
- [ ] T2.4 website：两处状态区 desktop 状态更新为 v0.1.7
- [ ] T2.5 website：README.md 版本段更新
- [ ] T2.6 父工程：README.md / README_zh.md 版本表（desktop=0.1.7，hos 保留 0.1.5）

## T3 Desktop 构建
- [ ] T3.1 清理中间产物：dsh-dist / out / .vite
- [ ] T3.2 `pnpm i`
- [ ] T3.3 `npm run build:dsh`（apply patch + 构建 dsh host/client/web + dsh-market）
- [ ] T3.4 `npm run package`（prepackage: collect:dsh + fetch-runtime；必要时 ELECTRON_MIRROR）
- [ ] T3.5 确认产物 `out/DSH Desktop-win32-x64/dsh-desktop.exe` 存在

## T4 打包产物实测（CDP）
- [ ] T4.1 以 `--remote-debugging-port=9222` 启动打包 exe，CDP 连接
- [ ] T4.2 设置 DeepSeek API KEY，确认保存成功（截图）
- [ ] T4.3 创建一个工作区（截图）
- [ ] T4.4 发起一次对话，确认收到模型回复（截图）
- [ ] T4.5 产出 TEST_REPORT.md 于 logs/20260929-1/

## T5 代码审查
- [ ] T5.1 加载 `requesting-code-review`，派发审查（desktop 全部变更 diff + spec 引用），产出 REVIEW_REPORT.md
- [ ] T5.2 加载 `receiving-code-review`，逐条验证并处理 Critical/Important
- [ ] T5.3 有 Critical 修复后复审，直至无 Critical 遗留

## T6 提交与 Tag
- [ ] T6.1 dsh-desktop 按 git-commit 技能规范提交（build/chore 等合规前缀）
- [ ] T6.2 website 提交（feat/docs 合规前缀），desktop/website 分别打注解 tag `v0.1.7`
- [ ] T6.3 父工程推进全部子模块指针并提交（chore）
- [ ] T6.4 各工程 `git push` + `git push --tags`（含父工程）

## T7 回归与收尾
- [ ] T7.1 回归确认：远端 tag 可见、父工程指针正确、打 tag 产物与实测一致
- [ ] T7.2 最终报告
