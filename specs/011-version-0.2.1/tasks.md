# 任务清单 — 版本更新 0.2.1（rc.2 → rc.5 升级）

需求编号：011-version-0.2.1（沿用模块，不新建；历史 101-version-0.2.1 指向同一模块）
日期：2026-11-01/02 ｜ 分支：不改动（deepseek-harness 用 `Branch_dsh-v0.2.1-rc.5`；各壳打 tag 除外）

## deepseek-harness（上游基线）
- [x] D0 本地分支 `Branch_dsh-v0.2.1-rc.5`（tag `dsh-v0.2.1-rc.5`，HEAD b830ec950f）已就位
- [x] D0b 依赖重装对齐 rc.5 lockfile（pnpm 11.28.5，367 workspace 包）

## dsh-desktop
- [x] D1 清理中间/临时文件（out/dsh-dist/runtime/.vite）
- [x] D2 建 patches/dsh-v0.2.1-rc.5/（3 补丁；rc.2→rc.5 目标文件零漂移，--check 全过）
- [x] D3 build-dsh.mjs patchDir → dsh-v0.2.1-rc.5
- [x] D4 package.json version=0.2.1（沿用）
- [x] D5 pnpm i 对齐 + build:dsh（3 补丁 + host/client/web + dsh-market **1.73.7**） + package（collect-dsh/fetch-runtime/forge）
- [x] D6 产物核对：app 0.2.1 / dsh-dist 0.2.1-rc.5 / HMR 补丁在包
- [x] D7 本机功能测试：API KEY ✔ 工作区 ws-rc5a ✔ 对话「介绍一下你自己」✔ 图片分析 ✔
- [x] D8 端侧能力测试（对照 docs/能力对照清单.md desktop 列）：fs/网络/skill/围栏 ✔；**bash/glob/rg/后台任务 ❌（P2）**
- [ ] D9 tag v0.2.1、提交（待 P2 裁决后随最终提交）

## dsh-desktop-hos
- [x] H1 清理中间/临时文件（build/.hvigor/oh_modules/dsh-dist）
- [x] H2 建 patches/dsh-v0.2.1-rc.5/（13 补丁，--check 全过）
- [x] H3 build-dsh.mjs patchDir → dsh-v0.2.1-rc.5
- [x] H4 app.json5 versionName 0.2.1 / versionCode 1009（沿用）
- [x] H5 依赖恢复：ohpm install（清理 oh_modules 后 inversify 等链接重建）+ pnpm install
- [x] H6 build-dsh（13 补丁 + host/client/web + dsh-market 1.73.7）
- [x] H7 collect-dsh（DSH_MARKET_VERSION 1.66.21→1.73.7；profiles/desktop/package.json dshmarket 同步）
- [x] H8 打 tar（dsh-dist.tar.gz 替换 rc.2 旧包；49094 条目）
- [x] H9 HAP 构建（deveco-studio-6 工具链；BUILD SUCCESSFUL 20s；394.6MB；/app 0.2.1/1009；dsh-dist rc.5）
- [ ] H10 真机测试 ⏭️（无鸿蒙真机；按用户口径跳过）／tag v0.2.1 待提交

## dsh-desktop-ark
- [x] A1 清理中间/临时文件（build/oh_modules/.hvigor/payload-stage）
- [x] A2 上游 pin：verify-upstream.mjs EXPECTED_TAG → rc.5；assemble-payload.mjs 提示信息 → rc.5
- [x] A3 assemble-payload --deploy --apply（16711 文件；DSH_MARKET_VERSION 1.66.42→**1.73.7**）
- [x] A4 package-payload（dsh.tar.gz 45.9MB→resfile）
- [x] A5 build-hap.sh（BUILD SUCCESSFUL 34s；HAP 91.9MB；versionName 0.2.1/versionCode 201；载荷 @deepseek-ai/dsh 0.2.1-rc.5）
- [ ] A6 真机测试 ⏭️（无鸿蒙真机）／tag v0.2.1 待提交

## dsh-desktop-android
- [x] AN1 清理中间/临时文件（build/.gradle/.kotlin/app/build）
- [x] AN2 上游/工程 pin：assemble/package-payload rc.2→rc.5；workflow DSH_HARNESS_REF→rc.5、DSH_MARKET_REF→**v1.73.7**
- [x] AN3 引擎投放：provision-engine → app jniLibs（libnode.so 87.7MB；**node 26.13.0**，见漂移项）
- [x] AN4 assemble-payload --deploy --apply（15789 文件；dshmarket 1.73.7；node-pty 1.1.0 配对）
- [x] AN5 package-payload（dsh.tar 243MB→assets）
- [x] AN6 gradlew :app:assembleFullRelease -PappVersionName=0.2.1 -PappVersionCode=201（BUILD SUCCESSFUL 1m19s；APK 79.4MB；versionName 0.2.1/versionCode 201）
- [ ] AN7 真机测试 ⛔ **设备安装被拒**（INSTALL_FAILED_USER_RESTRICTED，待处理）／tag v0.2.1 待提交

## dsh-desktop-website
- [x] W1 版本串 rc.2→rc.5（index.html/index_zh.html/README.md，各 6/6/4 处）+ Android Edition 0.1.0→0.2.1
- [x] W2 提交（docs: 前缀，commit 8848bce）+ 打 tag v0.2.1

## 父工程
- [x] P1 父 README.md / README_zh.md 版本表：rc.2→rc.5、android 0.1.0→0.2.1、状态列同步
- [x] P2 规格模块 specs/011-version-0.2.1/{spec,plan,tasks,test-cases}.md 本轮更新
- [ ] P3 子模块指针提交（deepseek-harness rc.5、dsh-market 1.73.7）+ 收尾提交

## 审查与收尾
- [ ] R1 requesting-code-review 审查本轮代码变更（各壳 pin/补丁/市场版本）
- [ ] R2 receiving-code-review 处理反馈
- [ ] R3 回归确认：产物版本/无残留/无敏感信息（API KEY 不落盘）
- [ ] R4 tag：desktop v0.2.1 / ark v0.2.1 / android v0.2.1（hos 不打，见口径）

## 待用户裁决
1. **desktop P2**：Windows 子进程全挂（Runner 用 process.execPath 被 RunAsNode=false 拦截）是否加补丁修（随包便携 Node）或只文档化。
2. **android 真机**：设备安装 USER_RESTRICTED，需设备授权或跳过。
3. **引擎 node**：本地 26.13.0 vs 文档 24.22.1 的对齐口径。
