# 上游贡献推进方案：5 个通用 patch

> 编制日期：2026-10-17
> 对象：`dsh-desktop-hos/patches/dsh-v0.2.0-rc.2/` 里的 5 个通用型 patch
> 目标仓库：`deepseek-ai/deepseek-harness`（251,254★）

---

## 0. 结论先行：**不能提 PR，只能走 Discussions + 社区 patch layer**

上游 `CONTRIBUTING.md` 原文（2026-10-17 核实）：

> DeepSeek Harness is still at an early stage and under active development.
> **We are sorry that we cannot accept external pull requests at the moment.**
> However, contributing code to this repository is far from the only way to help.

配套事实：

| 事实 | 值 | 含义 |
|---|---|---|
| Issues | **已关闭**（`has_issues: false`） | 不能开 issue |
| Discussions | **开启**（`has_discussions: true`） | 唯一官方书面通道 |
| 默认分支 | `master` | 不是 `main` |
| 最新发布 | `dsh-v0.2.1-alpha.5`（2026-10-16） | 你的 patch 基于 `dsh-v0.2.0-rc.2`，**落后一个 minor** |
| 开放 issue 数 | 0 | 没有公开的 bug 队列可蹭 |

所以「整理成上游 PR」这个动作要改造成两件事：

1. **3 条 GitHub Discussion**（不是 5 条）：报问题 + 附完整 diff + 明确「不需要署名，直接取用」——维护者可以内部 cherry-pick。
2. **1 个社区 patch layer 仓库**：让生态今天就能用上；同时它也是你在没有 PR 通道时唯一的「使用数据 + 背书」来源。

---

## 1. 先做事实核查：这 5 个缺口还在不在？

已对 `master`（= 0.2.1-alpha.5）逐项核实，**全部仍然存在**：

| # | 缺口 | 核实结果 |
|---|---|---|
| 1 | `ctx.fs` 无 `remove` | `FileSystem` 抽象成员只有 resolve / processPath / fileUrl / contains / stat / lstat / readText / streamText / readBytes / readByteRange / listDir / writeText / editText |
| 2 | `ctx.fs` 无 `writeBytes` | 同上；且 `tool-fs/` 只暴露 `read` / `read_image` / `write` / `edit`，**没有任何删除或字节写入工具** |
| 3 | `ctx.fs` 无 `chmod` | 同上；写入侧只有 `writeText` 的 `mode` 参数 |
| 4 | `writableRoots()` 不可扩展 | `packages/sandbox/sandbox/src/roots.ts` 仍只有 `[workspaceRoot, '/tmp', tmpdir()]`，不读任何环境变量 |
| 5 | 无硬链接文件系统上 guarded-create 失败 | `writeFileAtomic` 的 `createIfAbsent` 分支仍 `await linkFile(...)`，失败即 `throwGuardedCreateFailure`，没有 EACCES/EPERM/ENOSYS/EXDEV 的 rename 回退 |

**这是好消息**：缺口真实、没有被上游顺手补掉，值得发。

**但有个坏消息**：`packages/fs/README.md` 明确写了两条设计哲学：

> "backends swap without touching the tools or the policy"
> "`tool-fs-search` deliberately does not extend the provider contract — search is a process-backed ripgrep workflow, **so filesystem backends stay free of a universal search API**"

这几乎预告了维护者对 #1–#3 的第一反应：**「写个工具插件就行，别动 seam」**。所以这三条必须主动把反驳写进正文（见 §3.3 的「为什么不走工具插件」）。

---

## 2. 推进顺序：从小到大，先赢一次

| 顺序 | 内容 | 类型 | 被接受概率 | 理由 |
|---|---|---|---|---|
| ① | **#5 硬链接回退** | bug | 高 | 真 bug，不涉及 API 形状；上游已有 `isENOENT`/`isEEXIST`/`isPermissionError` 同风格 helper，改动 60 行 |
| ② | **#4 可扩展可写根** | 配置 | 中高 | 不改 API，只加一个部署态环境变量；`SandboxExecutionPolicy` 只有一个 `workspaceRoot`，这是真实表达力缺口 |
| ③ | **#1+#2+#3 合并为一条 API 提案** | 设计 | 中低 | 触及 `ctx.fs` 契约，需要维护者表态；三条是同一个问题「seam 只有读改写，没有变更」 |

**不要一次全发。** 上游是「很小的团队、不保证回复」（CONTRIBUTING 原话），一次发 5 条会被当噪音。每发一条，等 2 周。

---

## 3. 每条怎么写

### 3.1 硬性门槛（缺一条就别发）

- [ ] **rebase 到 `dsh-v0.2.1-alpha.5`**：现在 5 个 patch 都基于 rc.2，直接贴 diff 会 apply 失败。方法见 §4。
- [ ] **带测试**：现有 5 个 patch **一个测试文件都没有**（已核对 `diff --git` 清单）。上游代码里 `/* v8 ignore */` 注释满天飞，说明覆盖率是硬门禁。每条至少要带一个 spec。
- [ ] **英文正文**：中文注释、中文正文会直接被跳过。
- [ ] **贴 diff 而不是贴仓库链接**：外部链接对维护者是负担；附 `git diff` 或 `.patch` 附件。
- [ ] **明确「无署名诉求」**：一句话消除法务/归属顾虑。

### 3.2 Discussion 的结构（照抄）

```
标题：<一句话现象，不是「建议增加 XXX」>

1. What happens（现象 + 最小复现，带平台/版本）
2. Why it happens（根因，指到文件与函数）
3. What I changed（diff 摘要，几行关键代码，不贴全文）
4. Why not a plugin（预判反驳，见下）
5. Tests I ran（或：tests included in the attached diff）
6. Offer（"Happy to hand this over as-is, no attribution needed."）
```

### 3.3 「为什么不走插件」——三条原语必须回答的问题

维护者会说：`tool-fs-search` 就是工具，你为什么不写个 `tool-fs-delete`？

答案（必须写进去）：

> 工具层直接调 `node:fs` 会**绕过 `fs-sandbox` 的 per-call fence**。`ctx.fs` 存在的意义就是所有变更走同一条围栏；`write`/`edit` 走 seam，删除和字节写入走裸 `node:fs`，等于给这条围栏开一个后门。`remove`/`writeBytes`/`chmod` 不是"通用搜索 API"那种可以外置的能力，它们是**和 writeText 同级的变更原语**，天然属于 seam。

这条论点是整个提案的支点。写不进去就别发。

---

## 4. 落地步骤（两周一轮）

### 第 1 步：把 patch 迁到最新 tag（半天）

```bash
# 在 dsh-desktop-hos 里新增 patches/dsh-v0.2.1-alpha.5/
git -C ../deepseek-harness fetch --tags
git -C ../deepseek-harness checkout dsh-v0.2.1-alpha.5
cd ../deepseek-harness
# 逐个试应用，记冲突
for p in ../dsh-desktop-hos/patches/dsh-v0.2.0-rc.2/*.patch; do
  git apply --check "$p" && echo "OK   $p" || echo "FAIL $p"
done
```

- 能干净应用的：直接换目录、改 `build-dsh.mjs` 的钉点。
- 冲突的：`git apply --3way` 手工解，解完 `git diff > 新 patch`。
- **注意**：这一步同时就是「把外壳升到 0.2.1-alpha.5」的前置工作，属于升级任务，不要混进贡献任务里。

### 第 2 步：补测试（1 天/条）

参考上游现有 spec 的写法（`packages/fs/fs-local/tests/`、`packages/sandbox/sandbox/tests/`）。最少覆盖：
- 正常路径
- 目标不存在（`FS_NOT_FOUND`）
- 围栏拒绝（sandbox 模式）
- 存储层"假成功"（chmod 回读不一致）

### 第 3 步：发第一条（#5 硬链接回退）

用 `01-hardlink-fallback.md` 的正文。发完**不要跟进催**；2 周后再看。

### 第 4 步：发第二条（#4 可写根）

用 `02-extra-writable-roots.md`。

### 第 5 步：发第三条（#1+#2+#3 合并提案）

用 `03-fs-mutation-primitives.md`。**这条要等到前两条有回应之后再发**——如果前两条都被采纳，第三条的可信度会显著提高。

### 第 6 步：无论回应如何，发 patch layer（与 1–5 并行）

见 `04-patch-layer.md`。这是唯一不受上游态度影响的收益路径。

---

## 5. 兜底：社区 patch layer（不受上游态度的收益）

上游不接受 PR 的官方立场是「生态自己长」（CONTRIBUTING 后面整段都在讲这个）。生态里已经有对应形态：策展列表 `Dominic789654/awesome-deepseek-harness` 有 **「Profiles & Patch Layers」** 专区。

所以把 5 个 patch 打包成一个**可复用的 patch layer**：

```
dsh-patch-fs-primitives/          # 仓库名即 npm 包名
├── README.md                     # 支持的 dsh tag 矩阵 + 一键应用
├── patches/
│   ├── dsh-v0.2.0-rc.2/
│   └── dsh-v0.2.1-alpha.5/
├── apply.mjs                     # 幂等：--reverse --check 检测已应用
└── verify.mjs                    # 应用后跑 smoke：remove/writeBytes/chmod
```

分发动作（与推广方案 §4 线 B/C 合并）：
- GitHub topics：`dsh` `dsh-plugin` `dsh-patch-layer`
- 提 PR 到两个 awesome 列表的 **Profiles & Patch Layers** 段
- README 顶部一句「为不支持 external PRs 的上游准备的补丁层，随 tag 更新」

**收益**：即使上游一条都不采纳，你也有一个被人 star / fork / 引用的产物，而不是一堆躺在自己仓库里的 diff。

---

## 6. 验收标准

| 时间 | 达标 | 不达标就调整 |
|---|---|---|
| 2 周 | #5 的 Discussion 有维护者回应或至少被 upvote | 改标题为纯 bug 复现（去掉任何"我做了个补丁"的表述） |
| 4 周 | #4 发出且有回应 | 合并进 #5 的帖子，别单开 |
| 8 周 | 至少 1 条被内部采纳 / 有人引用你的 patch | 停止投递，资源全部转到 patch layer + 鸿蒙内容线 |
| 持续 | patch layer 有 ≥ 3 个外部使用者 | 放弃 patch layer，只保留 13 个 patch 作为外壳的内部资产 |

**注意**：上游 251k★ 但 0 个 open issue、Discussions 回复率未知。**「没回应」是默认预期，不是失败**。真正的失败是把两个月花在一个不回话的仓库上，而 patch layer 和鸿蒙内容线停摆。

---

## 7. 文件索引

| 文件 | 内容 |
|---|---|
| `01-hardlink-fallback.md` | #5 的完整 Discussion 正文（可直接粘贴，英文） |
| `02-extra-writable-roots.md` | #4 的完整 Discussion 正文 |
| `03-fs-mutation-primitives.md` | #1+#2+#3 合并提案（含三条各自的 diff 摘要与逐项反驳） |
| `04-patch-layer.md` | 社区 patch layer 仓库的方案与 README 草稿 |
