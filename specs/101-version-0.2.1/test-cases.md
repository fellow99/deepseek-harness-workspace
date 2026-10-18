# 测试用例 — 版本更新 0.2.1

需求编号：101-version-0.2.1

## 一、dsh-desktop（本地功能测试）

| ID | 用例 | 步骤 | 预期 |
|---|---|---|---|
| D1 | 打包成功 | `pnpm i && npm run package` | exit 0，产出 out/DSH Desktop-win32-x64/ |
| D2 | 补丁全部应用 | build:dsh 阶段 git apply | 3 补丁均 apply，无冲突 |
| D3 | 版本正确 | 打包产物内 dsh 版本/包版本 | package.json version=0.2.1 |
| D4 | 设置 API KEY | 启动 exe → 设置 → DeepSeek API KEY 填入 | 保存成功，无报错 |
| D5 | 创建工作区 | 新建一个工作区 | 工作区创建成功，列表可见 |
| D6 | 一次对话 | 在工作区发起一次对话并发送消息 | 收到模型回复，对话正常渲染 |

## 二、dsh-desktop-hos（真机测试）

| ID | 用例 | 步骤 | 预期 |
|---|---|---|---|
| H1 | 编译构建成功 | collect-runtime → build-dsh → collect-dsh → HAP build | exit 0，产出 .hap |
| H2 | 补丁全部应用 | build-dsh 阶段 git apply | 14 补丁均 apply（不含 symlink-to-copy），无冲突 |
| H3 | 版本正确 | HAP 元信息 / app.json5 | versionName=0.2.1, versionCode=1009 |
| H4 | 部署真机 | hdc install .hap + aa start | 安装并启动成功 |
| H5 | 设置 API KEY | 真机设置 DeepSeek API KEY | 保存成功 |
| H6 | 创建工作区 | 新建工作区 | 创建成功 |
| H7 | 一次对话 | 发起一次对话 | 收到模型回复 |

## 三、dsh-desktop-website

| ID | 用例 | 预期 |
|---|---|---|
| W1 | 版本展示 | index.html/index_zh.html/README.md 均为 0.2.1/dsh-v0.2.1-rc.2，无 0.2.0 残留 |

## 四、回归（收尾）

| ID | 用例 | 预期 |
|---|---|---|
| R1 | dsh-desktop 产物版本 | 0.2.1 |
| R2 | hos HAP 版本 | 0.2.1 / 1009 |
| R3 | 父 README 版本表 | dsh-desktop 0.2.1/dsh-v0.2.1-rc.2，hos 0.2.1/dsh-v0.2.1-rc.2 |
| R4 | git 提交前缀 | 全部符合 Conventional Commits（git-commit 技能） |
| R5 | 无敏感信息 | 补丁/提交/文档无 API KEY |

## 五、执行结论（2026-10-18）

| 组 | 结论 |
|---|---|
| D1–D6（dsh-desktop） | ✅ 全部通过：打包成功、3 补丁应用、版本 0.2.1、API KEY 保存成功、工作区 `ws-021` 创建、对话回复「收到。」 |
| H1–H3（dsh-desktop-hos） | ✅ 通过：四阶段构建 `BUILD SUCCESSFUL`、13 补丁顺序应用、HAP `versionName 0.2.1 / versionCode 1009` |
| H4–H7（dsh-desktop-hos） | ⏭️ 按用户决定跳过（设备不可见；不执行部署测试、不打 tag） |
| W1–W4（dsh-desktop-website） | ✅ 通过：中英两页 4 处版本行均为 0.2.1，实际渲染无破图，验证声明如实 |
| R1–R3（回归） | ✅ 通过：desktop 0.2.1 / HOS HAP 0.2.1·1009 / 父 README 版本表已同步 |
| R4（提交前缀） | ✅ 通过：全部 Conventional Commits |
| R5（无敏感信息） | ✅ 通过：补丁、提交、文档均无 API KEY |
