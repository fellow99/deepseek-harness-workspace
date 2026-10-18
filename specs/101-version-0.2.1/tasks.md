# 任务清单 — 版本更新 0.2.1

需求编号：101-version-0.2.1

## dsh-desktop
- [ ] T1 清理 out/ dsh-dist/ runtime/ .vite/ 中间临时文件
- [ ] T2 新建 patches/dsh-v0.2.1-rc.2/：重锚 hmr + native-picker + welcome-notice
- [ ] T3 build-dsh.mjs patchDir pin → dsh-v0.2.1-rc.2
- [ ] T4 package.json version → 0.2.1
- [ ] T5 git apply --check 校验 3 补丁
- [ ] T6 pnpm i && npm run package 打包
- [ ] T7 功能测试：API KEY + 工作区 + 对话
- [ ] T8 测试通过打 tag v0.2.1

## dsh-desktop-hos
- [x] T9 清理中间/临时构建文件（.hvigor/build/dsh-dist）
- [x] T10 新建 patches/dsh-v0.2.1-rc.2/（13 个：7 原样 + 6 重建/重锚，不含 symlink-to-copy）
- [x] T11 build-dsh.mjs patchDir pin → dsh-v0.2.1-rc.2（patchFiles 13 个不变）
- [x] T12 app.json5 versionName 0.2.1 / versionCode 1009
- [x] T13 git apply --check 校验 13 补丁（顺序应用全通过）
- [x] T14 编译构建（collect-runtime → build-dsh → collect-dsh → tar → HAP，BUILD SUCCESSFUL）
- [ ] T15 部署真机 + 功能测试：API KEY + 工作区 + 对话（阻塞：设备未连接）

## dsh-desktop-website
- [x] T16 index.html / index_zh.html / README.md 版本串 → 0.2.1

## 审查与收尾
- [ ] T17 requesting-code-review 审查三处变更
- [ ] T18 receiving-code-review 处理反馈（修 Critical/Important）
- [ ] T19 父 README/README_zh 版本表更新
- [ ] T20 git 提交（git-commit 前缀）+ 回归确认
