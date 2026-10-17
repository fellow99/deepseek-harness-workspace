# 社区 patch layer：让 5 个 patch 在「没有 PR 通道」时也能被用上

> 定位：上游明确不接受外部 PR，所以这 5 个 patch 的最终归宿不是 PR，而是**一个可被引用的补丁层**。
> 它同时是 #5/#4/#1-3 三条 Discussion 的「证据」：贴出去的不是一段代码，而是一个有人用、有版本矩阵、有 CI 的产物。

---

## 1. 为什么是 patch layer，不是插件

| 方案 | 可行性 | 问题 |
|---|---|---|
| 上游 PR | ❌ | CONTRIBUTING 明说不接受外部 PR |
| 插件（`dsh plugin add`） | ⚠️ 部分 | `remove`/`writeBytes`/`chmod` 要挂在 `ctx.fs` seam 上，插件只能装饰实例，无法让 `fs-sandbox` 的 `checkedTarget` 生效——绕围栏的事不能干 |
| **patch layer** | ✅ | 与上游「生态自己长」的立场一致；生态里已有专门分类（策展列表的 **Profiles & Patch Layers**） |

一句话卖点：

> A patch layer for the parts of `ctx.fs` that upstream cannot take as a PR yet.

---

## 2. 仓库结构

```
dsh-patch-fs-primitives/
├── README.md                  # 支持的 tag 矩阵 + 一键应用 + Known Limitations（英文）
├── LICENSE                    # MIT
├── patches/
│   ├── dsh-v0.2.0-rc.2/       # 从 dsh-desktop-hos 直接搬
│   │   ├── dsh-fs-remove-primitive.patch
│   │   ├── dsh-fs-write-bytes.patch
│   │   ├── dsh-fs-chmod-primitive.patch
│   │   ├── dsh-extra-writable-roots.patch
│   │   └── dsh-fs-hardlink-fallback.patch
│   └── dsh-v0.2.1-alpha.5/    # rebase 后新增
├── apply.mjs                  # 幂等应用：--reverse --check 已应用则跳过
├── verify.mjs                 # 应用后 smoke：三个原语各跑一次
└── .github/workflows/matrix.yml   # 对每个支持的 tag：checkout → apply → build → test
```

**关键设计**：`patches/<tag>/` 的目录名就是 dsh 的 tag，和两个外壳工程用的约定完全一致——这样同一个 patch 集既能被外壳消费，也能被社区单独消费。

---

## 3. `apply.mjs` 的行为约定（写进 README）

```
node apply.mjs --dsh ../deepseek-harness      # 自动探测 tag，选 patches/<tag>/
node apply.mjs --dsh ... --tag dsh-v0.2.0-rc.2
node apply.mjs --check                        # 只检查，不写
node apply.mjs --reverse                      # 回滚
```

- **幂等**：先 `git apply --reverse --check`，成功即视为已应用，跳过。
- **硬失败**：目录不存在 / tag 不在矩阵里 / patch 应用失败 → 非零退出并打印缺失项，**绝不静默跳过**（这条是从两个外壳工程踩坑后学到的）。
- **不碰 git 历史**：只改工作区，不 commit。

---

## 4. README 草稿（英文，可直接用）

```md
# dsh-patch-fs-primitives

A patch layer that adds the `ctx.fs` mutation primitives DeepSeek Harness does not ship yet:

- `remove` — delete a file, or a directory with `recursive`
- `writeBytes` — write raw bytes through the sandbox fence
- `chmod` — apply permission bits and verify they took effect
- plus two smaller fixes: deployment-level extra writable roots, and a hard-link fallback
  for filesystems that refuse `link()`

> **Why a patch layer?** Upstream does not accept external pull requests at the moment
> (see its CONTRIBUTING). This repo exists so the parts that need to live *on the
> `ctx.fs` seam* are still available to the ecosystem. Everything that can be a plugin
> should be a plugin — these cannot, because a tool that calls `node:fs` directly bypasses
> the `fs-sandbox` per-call fence.

## Supported tags

| dsh tag | status |
|---|---|
| `dsh-v0.2.0-rc.2` | verified |
| `dsh-v0.2.1-alpha.5` | verified |

Nothing else is supported. A tag outside this table gets a hard failure, not a best-effort
apply.

## Use

    git clone https://github.com/deepseek-ai/deepseek-harness ../deepseek-harness
    git -C ../deepseek-harness checkout dsh-v0.2.1-alpha.5
    node apply.mjs --dsh ../deepseek-harness
    node verify.mjs --dsh ../deepseek-harness

## Known limitations

- These patches are **not upstream**. Upstream may change the same files; the matrix is the
  contract.
- `remove` does not participate in `fs-observation-policy`'s observed-state guard (the
  read-before-overwrite rule). Deleting a file the session never read is allowed.
- The hard-link fallback publishes by same-directory `rename` when the filesystem refuses
  `link()`. That trades an atomic no-overwrite guarantee for "the write can happen at all";
  the fallback re-probes the target immediately before renaming.
- `chmod` fails loudly when the storage layer accepts the call without applying the mode
  (observed on HarmonyOS hmdfs). That is deliberate.

## Status

These are proposed upstream as Discussions, not PRs (upstream cannot accept PRs today).
If any of this lands upstream, the corresponding patch is deleted from this repo.
```

---

## 5. 分发动作（与推广方案 §4 合并执行）

1. **GitHub topics**：`dsh` `dsh-plugin` `dsh-patch-layer` `deepseek-harness`
2. **策展列表 PR**：提到 `Dominic789654/awesome-deepseek-harness` 的 **Profiles & Patch Layers** 段；同时提 `Zhiyuan-Fan/Awesome-DeepSeek-Harness-Plugins` 的 Runtime & Sandboxes 段
3. **两个外壳的 README 交叉引用**：`dsh-desktop-hos` 的「Limits, in detail」段加一句「这些补丁也可单独取用 → 仓库链接」
4. **Discussion 附件**：三条 Discussion 的 diff 附件都从本仓库的 `patches/<tag>/` 生成，形成闭环

---

## 6. 维护成本与止损线

| 项 | 成本 |
|---|---|
| 每个新 dsh tag rebase | 0.5–1 天（`git apply --3way` + 跑 matrix） |
| CI | 一条 workflow，矩阵 2 个 tag |
| 止损线 | **连续 3 个 tag 无人使用（无 issue、无 star、无下载）→ 停止支持新 tag，仓库转 archived 并在 README 写明「只保留到 X 版本」** |

> 不要让它变成第二个需要维护的产品线。它的价值是「让 Discussion 有据可依」+「让外壳的补丁有独立出处」，不是收入。
