# 构建工作流 功能规格（父工程）

> Module: 012-building-workflow
> 范围：父工程 + 有 CI 的子工程（`dsh-desktop`、`dsh-desktop-android`）
> Status: Implemented（as-built）
> Last Updated: 2026-11-01

## 1. 模块概述

### 1.1 目的（Why this module exists）

本模块定义整个工作区的**构建工作流**：在代码托管平台（GitHub Actions）上自动完成“准备同级源码 → 应用 patch/装配 → 构建产物 → 上传/发布”的流水线，保证四个交付版本中**有 CI 的版本**（桌面版、Android 版）可重复构建与发布。

### 1.2 解决的问题

- 四个壳依赖同一批上游/兄弟工程（dsh、dsh-market、运行时），版本必须与 patch/载荷锁定一致；手工构建容易漂移。
- 桌面版需要 Windows / Linux / macOS 三平台产物，本地无法一次覆盖。
- Android 版需要引擎投放、载荷装配、签名注入、版本注入与 Release 发布，步骤多且易错。
- 鸿蒙工程（hos / ark）的构建/签名依赖 DevEco Studio / hvigor 与本地签名材料，当前无 GitHub Actions 流水线。

### 1.3 范围

**包含：**

- 父工程：submodule 指针推进、版本表同步、跨工程构建约定。
- `dsh-desktop`：CI（push/PR 三平台构建）与 Release（`v*` tag 三平台产物 + GitHub Release）。
- `dsh-desktop-android`：tag 触发的云端构建（引擎投放 → 载荷装配 → APK → Release）。
- 构建工作流的版本/上游锁定原则、签名材料零入库、制品命名与证据要求。

**不包含：**

- 鸿蒙工程（`dsh-desktop-hos`、`dsh-desktop-ark`）的构建流水线；其构建/签名见各自 `specs/111-build-pipeline`、`specs/108-compliance-release` 等模块。
- 代码签名/公证的具体密钥管理（只规定“不入库、经 Secrets/本地文件注入”）。
- 商店上架审核流程。

## 2. 用户故事

- 作为维护者，我希望 push/PR 自动验证桌面版三平台可构建，避免合并后才发现跨平台问题。
- 作为发布负责人，我希望打 tag 后自动得到桌面版三平台安装包与 Android APK，并生成 Release。
- 作为构建排障者，我希望流水线显式锁定上游 tag、运行时版本与签名来源，失败时能定位是上游漂移还是壳代码问题。
- 作为安全负责人，我希望签名材料与凭据不出现在仓库、日志与制品中。

## 3. 功能需求

### 3.1 通用构建约定

- FR-012-001：所有 CI 构建 MUST 显式锁定上游 `deepseek-harness` tag、`dsh-market` 版本与运行时版本；MUST NOT 使用浮动的 `latest`。
- FR-012-002：构建 MUST 可重复；相同输入 SHOULD 产生相同版本/哈希（允许平台时间戳差异）。
- FR-012-003：构建 MUST NOT 依赖开发者本机缓存/未入库文件；所需二进制 MUST 由脚本下载/投放并校验哈希。
- FR-012-004：构建日志 MUST 输出关键版本与哈希（上游 tag、引擎 sha256、载荷 sha256、APK/安装包 sha256）。
- FR-012-005：签名材料、keystore、口令 MUST NOT 入库；CI MUST 通过 Secrets 注入，缺失时 MUST 失败而不是回落 debug。
- FR-012-006：构建失败 MUST 使流水线失败；MUST NOT 以“跳过/警告”掩盖失败。

### 3.2 桌面版（dsh-desktop）

- FR-012-007：CI MUST 在 `push` main 与 `pull_request` 触发，并使用 Windows / Linux / macOS 三平台矩阵。
- FR-012-008：CI MUST 以同级目录 checkout dsh 与 dsh-market，准备 Node/corepack 与 Linux 打包依赖，执行 `npm ci` → `npm run build:dsh` → `npm run make`。
- FR-012-009：CI MUST 上传 `out/make/` 制品；无制品 MUST 失败。
- FR-012-010：Release MUST 由 `v*` tag 或手动 `workflow_dispatch` 触发，汇总三平台制品并创建 GitHub Release。
- FR-012-011：Release MUST 允许手动覆盖 dsh / dsh-market ref，缺省使用锁定版本。

### 3.3 Android 版（dsh-desktop-android）

- FR-012-012：Android 构建 MUST 由 tag `v*` 触发（另支持手动彩排），并从 tag 派生 `versionName` / `versionCode`。
- FR-012-013：云端构建 MUST 完成引擎投放（下载归档 → SHA-256 对账 → ELF 门禁 → jniLibs）、载荷装配（dsh + dsh-market → `assemble-payload` → `package-payload`）与 APK 构建。
- FR-012-014：Release 构建 MUST 使用固定 release 密钥签名（Secrets 注入），并验证证书指纹；彩排 MAY 使用 debug 签名。
- FR-012-015：Release MUST 发布 APK、载荷/版本摘要与四类哈希（上游 ref、引擎归档、载荷、APK）。
- FR-012-016：构建 MUST 遵循 Android 的二进制不入库与 16 KB 对齐要求。

### 3.4 父工程协同

- FR-012-017：版本更新（`011-version-*`）MUST 同步更新各工程 CI 中的上游 tag/market 版本引用。
- FR-012-018：父工程 MUST 在版本表中记录各壳版本与 dsh 版本；submodule 指针 MUST 与发布版本一致。
- FR-012-019：构建工作流的变更（新增/删除步骤、版本锁定变更）MUST 在对应子工程 `012-building-workflow/plan.md` 与父工程本模块登记。

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|------|------|----------|
| Workflow Run | 一次 CI/Release 运行 | 触发、矩阵、状态、日志 |
| 上游锁定 | dsh / market / runtime | tag、commit、sha256 |
| 制品 | 安装包/APK | 平台、格式、sha256 |
| Release | GitHub Release | tag、附件、摘要 |
| 签名材料 | keystore / Secrets | 来源、是否入库（必须否） |

## 5. 验收场景

### 场景：桌面版主分支 CI

- Given 代码 push 到 main
- When CI 完成
- Then 三平台矩阵均构建成功，且 `dist-<os>` 制品可下载

### 场景：桌面版发布

- Given 推送 `v*` tag
- When Release 完成
- Then Release 含三平台安装包与自动生成的 release notes

### 场景：Android tag 构建

- Given 推送 `v0.2.1` tag
- When 云端构建完成
- Then APK 的 versionName/versionCode 正确、签名证书固定、Release 附件与哈希摘要齐全

### 场景：签名缺失

- Given tag 构建但缺少签名 Secrets
- When 执行 release 构建
- Then 构建失败并提示缺失项，不产出未签名包冒充正式包

## 6. 非功能需求

- **可重复性**：版本与哈希显式锁定；构建脚本幂等。
- **安全性**：签名材料零入库；日志不打印 token/口令。
- **可观测性**：失败步骤与原因清晰；关键哈希可追溯。
- **成本**：矩阵并行；并发组取消旧运行；缓存 npm/pnpm/Gradle。
- **合规**：Android release 必须正式签名；HarmonyOS 构建不在本模块范围。

## 7. 假设与约束

- 假设 GitHub Actions 可访问 GitHub 上的 dsh / dsh-market / runtime 仓库。
- 约束：桌面安装包当前未签名/未公证；不满足商店分发。
- 约束：鸿蒙工程无 GitHub Actions，构建经 DevEco/hvigor 本地完成。
- 约束：CI 不替代完整 E2E；真机/功能回归仍需人工或平台自动化。

## 8. 依赖

- 上游：`deepseek-harness`、`dsh-market`、`aosp-node-runtime`、`node-bridge`。
- 平台：GitHub Actions、各平台打包工具、Android SDK/JDK、签名 Secrets。
- 下游：Release 下载页、产品官网版本展示、`011-version-*` 版本更新。
