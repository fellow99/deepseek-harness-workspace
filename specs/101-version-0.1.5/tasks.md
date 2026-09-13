# 101-version-0.1.5 — 任务分解

> 执行模式：精简流程 + 连续执行（主链不停顿；仅 Critical/阻塞时暂停）。

## T0 准备
- [ ] T0.1 校验 dsh 工作树干净且 HEAD = `dsh-v0.1.5-rc.2`
- [ ] T0.2 建立各工程 `logs/20260913-1/` 日志目录
- [ ] T0.3 确认环境：node / npm / pnpm / hdc / 真机连接

## T1 Patch 重建
- [ ] T1.1 desktop：`patches/dsh-v0.1.5-rc.2/dsh-disable-hmr.patch`（按 rc.2 重生成）
- [ ] T1.2 desktop：`patches/dsh-v0.1.5-rc.2/dsh-disable-native-picker.patch`（针对新 `resolve.ts` 重写）
- [ ] T1.3 harmony：`patches/dsh-v0.1.5-rc.2/dsh-symlink-to-copy.patch`
- [ ] T1.4 harmony：`patches/dsh-v0.1.5-rc.2/dsh-allow-all-interfaces.patch`
- [ ] T1.5 harmony：`patches/dsh-v0.1.5-rc.2/dsh-disable-hmr.patch`
- [ ] T1.6 harmony：`patches/dsh-v0.1.5-rc.2/dsh-disable-native-picker.patch`
- [ ] T1.7 更新两个 `scripts/build-dsh.mjs` 的 patch pin
- [ ] T1.7b harmony：新增 `patches/dsh-v0.1.5-rc.2/dsh-flock-openharmony.patch`（openharmony 单进程放行 flock）
- [ ] T1.8 7 个补丁 `git apply --check` 全部通过

## T2 版本号变更
- [ ] T2.1 desktop：`package.json` + `package-lock.json` → 0.1.5
- [ ] T2.2 desktop：`README.md` / `README_zh.md` 版本引用
- [ ] T2.3 harmony：`AppScope/app.json5` versionCode=1005 / versionName=0.1.5
- [ ] T2.4 harmony：`README.md` / `README_zh.md` 版本引用
- [ ] T2.5 website：`index.html` / `index_zh.html` / `README.md` 版本文案
- [ ] T2.6 父工程：`README.md` / `README_zh.md` 版本表

## T2b 新增窗口功能（desktop + harmony）
- [ ] T2b.1 desktop：`windows.ts` 改 `frame:true` 常规标题栏，移除自定义窗口控制（min/max/close IPC + preload）
- [ ] T2b.2 desktop：主窗口状态持久化（最大化/普通 + 位置尺寸，跨重启恢复）
- [ ] T2b.3 desktop：F11 切换全屏
- [ ] T2b.4 harmony：主窗口状态持久化（最大化/普通 + 位置尺寸，跨重启恢复）
- [ ] T2b.5 harmony：F11 切换全屏
- [ ] T2b.5b harmony：profile 禁用 `open-in-app`/`ui-open-in-app`（0.1.5 host-open-in-app 依赖 subprocess）
- [ ] T2b.6 两端 `typecheck` / 语法校验通过

## T3 Desktop 构建与测试
- [ ] T3.1 清理中间产物（dsh-dist / out / .vite）
- [ ] T3.2 `npm install`
- [ ] T3.3 `npm run build:dsh`
- [ ] T3.4 `npm run package`
- [ ] T3.5 启动实测：API KEY → 工作区 → 对话（截图取证）

## T4 Harmony 构建与测试
- [ ] T4.1 `collect-runtime` → `build-dsh` → `collect-dsh`
- [ ] T4.2 `dsh-dist.tar.gz` 打包
- [ ] T4.3 `build-hap.ps1` 签名构建
- [ ] T4.4 hdc 安装 + 启动
- [ ] T4.5 真机实测：API KEY → 工作区 → 对话（截图取证）

## T5 代码审查
- [ ] T5.1 `requesting-code-review` 派发审查，产出 `REVIEW_REPORT.md`
- [ ] T5.2 `receiving-code-review` 处理反馈（Critical/Important 优先）

## T6 修复与回归
- [ ] T6.1 修复测试/审查暴露的问题
- [ ] T6.2 回归验证，更新 `TEST_REPORT.md`

## T7 提交与 Tag
- [ ] T7.1 各工程 Conventional Commits 提交
- [ ] T7.2 父工程推进子模块指针并提交
- [ ] T7.3 desktop / harmony / website 创建 `v0.1.5` 注解 tag
