# 101-version-0.1.5 — 版本更新：0.1.5

## 需求编号
101-version-0.1.5

## 需求名称
版本更新：0.1.5

## Git 分支
不改动（各工程沿用当前分支：父工程/desktop/harmony/website = `main`；dsh = `Branch_dsh-v0.1.5-rc.2`）

## 涉及工程
- 父工程（deepseek-harness-workspace）
- deepseek-harness
- deepseek-harness-desktop
- deepseek-harness-harmony
- deepseek-harness-desktop-website

## 背景
上游 `deepseek-harness` 已发布 `dsh-v0.1.5-rc.2`（本地分支 `Branch_dsh-v0.1.5-rc.2` 已就绪）。
两个封装工程此前停留在 `0.1.2` / `dsh-v0.1.2-rc.1`，需升级到 `0.1.5` / `dsh-v0.1.5-rc.2`，
完成 patch 适配、重新打包、真机功能验证，并同步官网与父工程文档。

## 功能需求

### FR-1 deepseek-harness 子工程
- 使用本地已存在的分支 `Branch_dsh-v0.1.5-rc.2`，对应 tag `dsh-v0.1.5-rc.2`。
- 不推新分支、不改动分支。

### FR-2 deepseek-harness-desktop 子工程
- FR-2.1 清理所有中间文件、临时文件。
- FR-2.2 基于 `dsh-v0.1.5-rc.2` 制作 patch（新增 `patches/dsh-v0.1.5-rc.2/` 并更新 `scripts/build-dsh.mjs` pin）。
- FR-2.3 版本号设为 `0.1.5`。
- FR-2.4 重新打包：`pnpm i && npm run package`（等价链路：主工程依赖安装 → `npm run build:dsh` → `npm run package`）。
- FR-2.5 打包后测试基本功能：设置 DeepSeek API KEY → 创建工作区 → 进行一次对话。
- FR-2.6 窗口外观：`frame:true` 常规 Windows 标题栏，移除无边框/自绘标题栏与自定义窗口控制（min/max/close）。
- FR-2.7 主窗口状态持久化：记录主窗口最后状态（最大化 / 普通；普通时记录位置与尺寸），下次启动时恢复。
- FR-2.8 F11 切换全屏模式。
- FR-2.9 去掉 Electron 默认菜单栏。

### FR-3 deepseek-harness-harmony 子工程
- FR-3.1 清理所有中间文件、临时文件。
- FR-3.2 基于 `dsh-v0.1.5-rc.2` 制作 patch（新增 `patches/dsh-v0.1.5-rc.2/` 并更新 `scripts/build-dsh.mjs` pin）。
- FR-3.3 版本号设为 `0.1.5`（`versionName=0.1.5`、`versionCode=1005`）。
- FR-3.4 重新编译构建，签名产出 `.hap` 并部署到真机。
- FR-3.5 真机测试基本功能：设置 DeepSeek API KEY → 创建工作区 → 进行一次对话。
- FR-3.6 主窗口状态持久化：记录主窗口最后状态（最大化 / 普通；普通时记录位置与尺寸），下次启动时恢复。
- FR-3.7 F11 切换全屏模式。
- FR-3.8 去掉 Electron 默认菜单栏。

### FR-4 deepseek-harness-desktop-website
- 版本文案由 `v0.1.2` / `dsh-v0.1.2-rc.1` 更新为 `v0.1.5` / `dsh-v0.1.5-rc.2`。

### FR-5 父工程
- README 版本表更新到 `0.1.5` / `dsh-v0.1.5-rc.2`，并推进 desktop/harmony/website 子模块指针。

## 非功能需求
- NFR-1 提交注释必须且只能使用 `git-commit` 技能规定的 Conventional Commits 前缀。
- NFR-2 API KEY 等敏感信息严禁写入任何被提交的文件。
- NFR-3 patch 必须对新 tag 幂等且可干净应用。

## 关键技术风险（已定位）
- `dsh-disable-native-picker.patch` 与 rc.2 冲突：上游把 `facts.env.SSH_CONNECTION/SSH_TTY` 重构为 `facts.ssh`。
  必须针对新 `resolve.ts` 重新生成补丁。
- 其余 6 个补丁可应用但存在行号偏移，需按 rc.2 基准重新生成以保证稳定。

## 验收标准
- AC-1 7 个新 patch 对 `dsh-v0.1.5-rc.2` 全部 `git apply --check` 通过。
- AC-2 desktop 打包产物可启动，三步基本功能（API KEY/工作区/对话）通过。
- AC-3 harmony 真机部署成功，三步基本功能通过。
- AC-4 全部版本引用一致为 `0.1.5` / `dsh-v0.1.5-rc.2`。
- AC-5 无 Critical 代码审查遗留问题。
- AC-6 提交前缀合规；desktop/harmony/website 打 `v0.1.5` 注解 tag。
- AC-7 desktop/harmony 窗口外观为常规标题栏；主窗口状态（最大化/普通 + 位置尺寸）可跨重启恢复；F11 可切换全屏。
