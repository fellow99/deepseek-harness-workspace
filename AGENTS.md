# AGENTS.md — DeepSeek Harness Workspace（父工程）AI Agent 工作规范

> 本文件定义了本工作区（父工程）中 AI Agent 必须遵守的规范和工作流程。
> 所有规范均基于本机实际已安装的技能（skills）编写，不含虚构内容。
> 总则见 `~/.agents/AGENTS.md`；本文件为工作区级规范，冲突时以本文件为准。
> 各子工程有其独立的项目级 `AGENTS.md`（见下文「四、子工程导航」）。

---

## 一、工作区是什么

本仓库是一个**纯子模块容器**：将所有需要一起交付 deepseek-harness（dsh）桌面应用的项目以 **git submodule** 平铺在同级目录，使每个壳工程所需的 `../deepseek-harness`、`../dsh-market`、`../harmonypc-electron` 兄弟目录引用在容器内即可解析。

| 目录 | 角色 | 类型 |
|---|---|---|
| `dsh-desktop/` | **Desktop** — Electron 外壳（Windows / Linux / macOS），Electron 43 | 子模块 |
| `dsh-desktop-hos/` | **Electron on HarmonyOS Edition** — 鸿蒙 2in1/平板，Electron 37 运行时 | 子模块 |
| `dsh-desktop-ark/` | **HarmonyOS Edition** — 系统 ArkWeb + 进程内 Node v24.2.0，无 Electron | 子模块 |
| `dsh-desktop-android/` | **Android Edition** — 系统 WebView + 进程内 Node v26.13.0（APK） | 子模块 |
| `deepseek-harness/` | 被封装的上游 Agent 主机（dsh，官方源码引用） | 子模块 |
| `dsh-market/` | 内置可视化插件市场（npm 包 `dshmarket`） | 子模块 |
| `harmonypc-electron/` | Electron-on-HarmonyOS 运行时（Electron 37 / Node 22.17） | 子模块 |
| `dsh-desktop-website/` | 四款 Edition 的产品官网（纯静态） | 子模块 |
| `dsh-plugins/` | 通用可共享 dsh 插件（`dsh-plugin-XXX`） | 普通目录 |
| `skills/` | 通用可共享 dsh 工具技能（kebab-case） | 普通目录 |
| `specs/` | 父工程 as-built 规格体系（0XX 公共 / 2XX 跨壳插件） | 普通目录 |
| `docs/` | 工作区文档（能力对照清单、推广方案等） | 普通目录 |
| `harmonypc-electron-versions/` | 运行时发布归档 + Electron 头文件指南（不入库） | 普通目录 |

**共享架构**：所有壳工程都以「进程内 Host + webserver + 同源数据面」运行官方未改动的 dsh Web UI——零 CORS、零鉴权、零自定义协议、零上游改动。

---

## 二、通用规范

### 2.1 语言
- 与用户交互、编写文档、提交注释均使用**中文**。
- 代码、变量名、函数名、技术术语使用**英文**。

### 2.2 安全
- 严禁提交密钥、Token、密码、私钥、签名凭据（`signing.*.local.json`、`keystore.properties` 等）等敏感信息。
- 严禁使用 `as any`、`@ts-ignore`、`@ts-expect-error` 压制类型错误。
- 严禁使用空的 `catch(e) {}` 错误处理。

### 2.3 代码质量
- 每次编辑文件后，必须运行 `lsp_diagnostics` 检查无新增错误。
- 修复 Bug 时，**只做最小改动**，严禁顺便重构。
- 禁止凭空猜测未读取的代码行为；关键接口必须对照源码核实。

### 2.4 任务管理
- 多步骤任务（2 步及以上）必须先创建 Todo 列表再执行。
- 同一时间只允许一个 `in_progress` 任务。
- 每完成一步立即标记 `completed`，不得批量标记。

### 2.5 子模块工作目录
- 修改某一子工程的代码时，先在**该子工程目录内**工作（其 `../` 兄弟引用在工作区内自动成立）。
- 编辑器/终端默认工作目录必须落在对应子工程内，方能用其各自独立的 `.codegraph` 索引与 `.git` 钩子。

---

## 三、代码地图（CodeGraph）规范 ——【核心规范】

### 3.1 强制使用 `codegraph-cli` 技能
已安装 **`codegraph-cli`** 技能（路径：`~/.agents/skills/codegraph-cli/SKILL.md`）。

**当任务涉及以下场景时，必须优先使用 `codegraph-cli`：**
- 理解代码库结构（"X 是如何工作的？"、"Y 在哪里定义的？"）
- 查找符号定义、调用关系、影响范围、请求流程
- 分析变更影响（"改了这个函数什么会坏？"）
- 确定哪些测试受代码变更影响

### 3.2 代码地图覆盖范围（已建立索引）

**每个有源代码的子工程**均已初始化 `.codegraph/` 代码地图（独立索引）：

| 子工程 | `.codegraph` | 索引规模（参考） |
|---|---|---|
| `deepseek-harness/` | ✅ 已建 | 8,808 文件 / 96,379 节点 |
| `dsh-market/` | ✅ 已建 | 233 文件 / 4,016 节点 |
| `dsh-desktop/` | ✅ 已建 | 24 文件 / 168 节点 |
| `dsh-desktop-android/` | ✅ 已建 | 495 文件 / 8,738 节点 |
| `dsh-desktop-hos/` | ✅ 已建 | 274 文件 / 4,527 节点 |
| `dsh-desktop-ark/` | ✅ 已建 | 118 文件 / 1,631 节点 |
| `harmonypc-electron/` | ✅ 已建 | 176 文件 / 3,173 节点 |
| `dsh-desktop-website/` | ✅ 已建 | 2 文件 / 4 节点 |

**父工程根目录**（本目录）同样已建立 `.codegraph/` **聚合索引**，包含全部子模块的代码（子模块为实体检出文件，路径以 `子目录/...` 前缀保留）。因此：
- 在某个子工程内 → 用该子工程自己的索引查询（精确、快）。
- 在父工程根 → 可用聚合索引**跨子工程查询**，例如 `codegraph query` / `codegraph context` 会同时命中 `deepseek-harness/...` 与 `dsh-market/...` 等。

### 3.3 索引自动更新（post-commit 钩子）——【强制规范】

> **本工作区把「提交后自动更新代码地图」设为强制规范。**

- 每个有源代码的子工程及父工程的 git **`post-commit` 钩子**已安装，在任何一次 commit 完成后自动执行 `codegraph sync`（增量更新），保持 `.codegraph` 索引与代码同步。
- 钩子为**非阻塞**设计：后台运行、失败只写日志、绝不阻断提交。
- 钩子日志：`<仓库根>/logs/codegraph-sync.log`。
- 钩子位于 git 命令空间（`.git/hooks/post-commit`，子模块在 `.git/modules/<名称>/hooks/post-commit`），不随仓库跟踪。

**钩子内嵌逻辑（写入各仓库 `.git/hooks/post-commit`）：**
```sh
REPO_ROOT="$(git rev-parse --show-toplevel)"
[ -z "$REPO_ROOT" ] && exit 0
if command -v codegraph >/dev/null 2>&1 && [ -d "$REPO_ROOT/.codegraph" ]; then
  mkdir -p "$REPO_ROOT/logs"
  ( cd "$REPO_ROOT" && nohup codegraph sync >> "$REPO_ROOT/logs/codegraph-sync.log" 2>&1 & )
fi
exit 0
```

### 3.4 Agent 使用规则
- **优先使用 codegraph**，回退到 grep + Read 仅用于未覆盖的细节确认。
- 索引建立后，若代码库发生大量变化（拉取、切分支、手动改文件），可先 `codegraph sync`（或依赖已装钩子）再查询。
- 常用命令速查：

| 场景 | 命令 |
|---|---|
| 探索功能区域 | `codegraph context "<任务描述>"` |
| 查找符号定义 | `codegraph query <符号名>` |
| 谁调用了 X | `codegraph callers <符号名>` |
| X 调用了谁 | `codegraph callees <符号名>` |
| 变更影响范围 | `codegraph impact <符号名>` |
| 变更影响的测试 | `codegraph affected <文件列表>` |
| 索引状态检查 | `codegraph status` |
| 增量更新索引 | `codegraph sync` |

---

## 四、子工程导航

各子工程项目级规范见其各自 `AGENTS.md`（进入该工程前先读）：

| 子工程 | AGENTS.md | 工作区中角色 |
|---|---|---|
| `dsh-desktop/` | `dsh-desktop/AGENTS.md` | Electron 桌面壳（主壳，架构参考） |
| `dsh-desktop-hos/` | `dsh-desktop-hos/AGENTS.md` | Electron on HarmonyOS Edition |
| `dsh-desktop-ark/` | `dsh-desktop-ark/AGENTS.md` | HarmonyOS Edition（ArkWeb + Node） |
| `dsh-desktop-android/` | `dsh-desktop-android/AGENTS.md` | Android Edition（WebView + Node） |
| `deepseek-harness/` | `deepseek-harness/AGENTS.md`（上游权威文档） | 被封装的上游主机 |
| `dsh-market/` | `dsh-market/AGENTS.md` | 内置插件市场 |
| `harmonypc-electron/` | `harmonypc-electron/AGENTS.md` | Electron-on-HarmonyOS 运行时 |
| `dsh-desktop-website/` | `dsh-desktop-website/AGENTS.md` | 产品官网 |

> `deepseek-harness/` 为上游源码引用，其 `AGENTS.md` 是上游权威开发规范（`CLAUDE.md` 为其符号链接），**不要覆写**；在该工程内工作时遵循上游规范，并遵守本文件第三节的 CodeGraph 更新规范。

---

## 五、Git 提交规范（强制使用 `git-commit` 技能）

### 5.1 确认
- 已安装 **`git-commit`** 技能（路径：`~/.agents/skills/git-commit/SKILL.md`）。

### 5.2 规则
1. 提交前，先加载 `git-commit` 技能获取完整规范。
2. 提交注释**必须**使用 [Conventional Commits](https://www.conventionalcommits.org/) 格式：`<type>[optional scope]: <description>`。
3. **只允许使用该技能规定的 type 前缀**（`feat` / `fix` / `docs` / `style` / `refactor` / `perf` / `test` / `build` / `ci` / `chore` / `revert`），**严禁使用其它前缀**。
4. description 用现在时、祈使句，不超过 72 字符。

### 5.3 子模块提交注意
- **父工程与子工程各自独立提交**。修改子工程代码时，先在子工程内 `git commit`（会触发该子工程 post-commit 钩子自动 `codegraph sync`），再回到父工程提交新的子模块指针。
- **严禁**使用 `--no-verify` 跳过 Git Hooks（除非用户明确要求）。

---

## 六、规格说明（Specs）规范

- 父工程聚合规格见 `specs/`（`README.md` 为索引；`constitution.md` 为编号规则）。
- 各子工程规格见其各自 `specs/`（`0XX` 公共模块、`1XX` 基础功能、`2XX` 补丁/插件）。
- 规格化流程使用 speckit 技能套件：`speckit-specify → speckit-clarify(可选) → speckit-plan → speckit-tasks → speckit-analyze(可选) → speckit-checklist(可选)`；实现阶段用 `speckit-implement`。
- 规范文档编写用 `specs-as-built`；**源码即真理，不可臆测**，行为不明时标注 `[NEEDS CLARIFICATION]`。

---

## 七、开发任务与代码审查规范

### 7.1 开发任务（`specs-based-devflow`）
**所有开发任务必须遵循完整开发生命周期**：`准备阶段 → 规格说明 → 开发 → 代码审查 → 测试 → Bug 修复 → 回归测试`。

### 7.2 代码审查
- **发起审查**用 `requesting-code-review`；**接收/处理审查反馈**用 `receiving-code-review`。
- 反馈分级：`Critical`（立即修复）/ `Important`（继续开发前修复）/ `Minor`（记录后续处理）。
- 接收反馈：先验证再实施；**禁止表演性赞同**；允许技术性反驳；遵循 YAGNI；未解决 Critical 前禁止进入测试。

### 7.3 产品/需求规划（`brainstorming`）
需求探索、方案比选、新功能概念设计时**必须使用 `brainstorming` 技能**；设计方案获批前**严禁进入实现**。

### 7.4 调试（`systematic-debugging`）
遇到任何 Bug、测试失败、异常行为时**必须遵循四阶段调试法**；**严禁在完成根因调查前提出修复方案**。

---

## 八、技能自检与缺失安装

### 8.1 启动时自检
Agent 开始执行任务前，先对照下表自检技能是否满足任务需求：

| 任务类型 | 必备技能 |
|---|---|
| Git 提交 | `git-commit` |
| 代码库探索 | `codegraph-cli` |
| 产品/需求规划 | `brainstorming` |
| 规格说明 | `speckit-specify` / `speckit-plan` / `speckit-tasks` 等 |
| 规范文档编写 | `specs-as-built` |
| 开发任务 | `specs-based-devflow` |
| 代码审查 | `requesting-code-review` / `receiving-code-review` |
| 调试 | `systematic-debugging` |
| 前端界面开发 | `frontend-design` |
| 鸿蒙开发/测试 | `harmonyos-app-dev` / `harmonyos-app-testing` |

### 8.2 缺失技能安装脚本
若自检发现任何技能缺失，在对应子工程根目录执行（`-p` 安装到当前项目，`-y` 跳过确认）：

```bash
npx skills add -p -y https://github.com/dceoy/speckit-agent-skills
npx skills add -p -y https://github.com/obra/superpowers --skill brainstorming
npx skills add -p -y https://github.com/obra/superpowers --skill requesting-code-review
npx skills add -p -y https://github.com/obra/superpowers --skill receiving-code-review
npx skills add -p -y https://github.com/github/awesome-copilot --skill git-commit
npx skills add -p -y https://github.com/anthropics/skills --skill frontend-design
npx skills add -p -y https://github.com/fellow99/fellow99-skills --skill opus-specs-as-built
npx skills add -p -y https://github.com/fellow99/fellow99-skills --skill specs-based-devflow
npx skills add -p -y https://github.com/fellow99/fellow99-skills --skill codegraph-cli
npx skills add -p -y https://github.com/fellow99/fellow99-skills --skill harmonyos-app-dev
npx skills add -p -y https://github.com/fellow99/fellow99-skills --skill harmonyos-app-testing
```

---

## 九、技能快速索引

| 任务类型 | 必须使用的技能 |
|---|---|
| Git 提交 | `git-commit` |
| 代码库探索 / 代码地图 | `codegraph-cli` |
| 产品/需求规划 | `brainstorming` |
| 规格说明 | `speckit-specify` / `speckit-plan` / `speckit-tasks` 等 |
| 规范文档编写 | `specs-as-built` |
| 开发任务（全流程） | `specs-based-devflow` |
| 发起代码审查 | `requesting-code-review` |
| 接收/处理审查反馈 | `receiving-code-review` |
| 调试 | `systematic-debugging` |
| 前端界面开发 | `frontend-design` |
| 鸿蒙应用开发 | `harmonyos-app-dev` |
| 鸿蒙应用测试/自动化 | `harmonyos-app-testing` |

---

## 十、违规处理

违反以上任何规范的行为，Agent 必须：
1. 立即停止当前操作。
2. 向用户报告违规内容。
3. 等待用户指示后再继续。

---

> **最后更新**：2026-11-02
> **维护者**：fellow99
