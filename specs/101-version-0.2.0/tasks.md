# 任务清单 — 版本更新 0.2.0

需求编号：101-version-0.2.0

## dsh-desktop
- [ ] T1 清理 out/ dsh-dist/ runtime/ .vite/ 中间临时文件
- [ ] T2 新建 patches/dsh-v0.2.0-rc.2/：复制 hmr + native-picker，重锚 welcome-notice
- [ ] T3 build-dsh.mjs patchDir pin → dsh-v0.2.0-rc.2
- [ ] T4 package.json version → 0.2.0
- [ ] T5 git apply --check 校验 3 补丁
- [ ] T6 pnpm i && npm run package 打包
- [ ] T7 功能测试：API KEY + 工作区 + 对话
- [ ] T8 测试通过打 tag v0.2.0

## dsh-desktop-hos
- [ ] T9 清理中间/临时构建文件
- [ ] T10 新建 patches/dsh-v0.2.0-rc.2/（14 个：8 原样 + 6 重写/重锚，去 hmr）
- [ ] T11 build-dsh.mjs patchDir + patchFiles（移除 hmr）
- [ ] T12 app.json5 versionName 0.2.0 / versionCode 1007
- [ ] T13 git apply --check 校验 14 补丁
- [ ] T14 编译构建（collect-runtime → build-dsh → collect-dsh → HAP）
- [ ] T15 部署真机 + 功能测试：API KEY + 工作区 + 对话

## dsh-desktop-website
- [ ] T16 index.html / index_zh.html / README.md 版本串 → 0.2.0

## 审查与收尾
- [ ] T17 requesting-code-review 审查三处变更
- [ ] T18 receiving-code-review 处理反馈（修 Critical/Important）
- [ ] T19 父 README/README_zh 版本表更新
- [ ] T20 git 提交（git-commit 前缀）+ 回归确认
