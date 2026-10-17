# #1 + #2 + #3 变更原语提案 · Discussion 正文

> 目标：`deepseek-ai/deepseek-harness` → Discussions
> 优先级：**第 3 条发**（等前两条有回应）
> 对应补丁：
> - `dsh-fs-remove-primitive.patch`（198 行，5 个文件）
> - `dsh-fs-write-bytes.patch`（211 行，5 个文件）
> - `dsh-fs-chmod-primitive.patch`（197 行，5 个文件）

---

## 为什么合并成一条

三条是**同一个问题**：`ctx.fs` 有"读"和"改写文本"，没有"变更"（删除 / 写字节 / 改权限）。拆成三条发会让维护者三次面对同一个架构判断，反而更容易被整体拒掉。合并成一条，把决策点收成一句：**变更原语该不该在 seam 上？**

---

## 事实基础（已对 `master` 核实）

- `FileSystem` 抽象成员：`resolve` `processPath` `fileUrl` `contains` `stat` `lstat` `readText` `streamText` `readBytes` `readByteRange` `listDir` `writeText` `editText`。
- `packages/fs/tool-fs/` 只暴露 `read` `read_image` `write` `edit` —— **上游没有删除工具**，也没有字节写入工具。也就是说：开箱状态下 agent 能创建和改写文件，但**不能删除**（`move` 也无从实现，因为移动 = 读 + 写 + 删）。

这不是"我想要个新 API"，而是"这条能力在架构里没有落点"。

---

## 核心论点（必须放在正文最前面）

> 工具层直接调 `node:fs` 会**绕过 `fs-sandbox` 的 per-call fence**。`ctx.fs` 存在的意义就是让所有变更走同一条围栏；`write`/`edit` 走 seam，删除和字节写入走裸 `node:fs`，等于给这条围栏开一个后门。
>
> `remove` / `writeBytes` / `chmod` 不是 `tool-fs-search` 那种"可以外置的通用搜索 API"——它们是**和 `writeText` 同级的变更原语**，天然属于 seam。

`packages/fs/README.md` 已经写了 "backends swap without touching the tools or the policy"，正好可以用它自己的设计语言反向论证：**如果删除走工具，那 fs-sandbox 就再也无法保证"所有写入都过围栏"**。

---

## 发之前

- [ ] rebase 三个 patch 到 `dsh-v0.2.1-alpha.5`
- [ ] 补 spec（三个补丁目前**都没有测试**）
- [ ] 决定一个开放问题并写进正文：`remove` 是否要参与 `fs-observation-policy` 的 observed-state 守卫（见文末）
- [ ] 准备好「要不要拆开」的回答：**可以拆，但请先给一个方向判断**

**建议标题**：

> `ctx.fs` has no mutation primitive for delete / raw bytes / permissions — and a tool-level workaround would bypass the sandbox fence

---

## 正文（英文，可直接粘贴）

```md
**The gap**

`FileSystem` exposes reads (`readText`, `streamText`, `readBytes`, `readByteRange`,
`listDir`, `stat`, `lstat`) and text mutation (`writeText`, `editText`), but no way to
remove an entry, write raw bytes, or change permission bits.

`tool-fs/` correspondingly ships `read`, `read_image`, `write`, `edit` — so out of the box
an agent can create and rewrite files but cannot delete one. `move` cannot be implemented
either, since it is read + write + remove.

**Why this is not just "a missing tool"**

`tool-fs-search` is deliberately process-backed rather than a provider method, and the fs
README says backends should "stay free of a universal search API". That reasoning does not
transfer here: search is a read path, so bypassing `ctx.fs` costs nothing structurally. A
delete or a byte write that calls `node:fs` directly **bypasses the `fs-sandbox` per-call
fence** — `checkedTarget` never runs, and the guarantee "every mutation goes through one
fence" stops being true. `remove` / `writeBytes` / `chmod` are peers of `writeText`, not
optional capabilities.

**Proposed shape**

Three methods on `FileSystem`. All three are **non-abstract** with a default body that
throws `FS_IO_ERROR` ("not supported by this filesystem backend"), because six concrete
classes extend `FileSystem` and an abstract member would break compilation for every one of
them:

    remove(target, options?: { recursive?: boolean; signal?: AbortSignal }): Promise<FsRemoveOutcome>
    writeBytes(target, content: Uint8Array, expected?: FsWriteIntent, signal?: AbortSignal,
               sandboxPolicy?: SandboxExecutionPolicy): Promise<FsWriteBytesOutcome>
    chmod(target, mode: number, options?: { signal?: AbortSignal },
          sandboxPolicy?: SandboxExecutionPolicy): Promise<FsChmodOutcome>

Outcomes stay minimal and model-facing: `{ path, kind: 'file' | 'directory' }` for remove,
`{ path, operation: 'create' | 'update' }` for a byte write, `{ path, mode }` for chmod
(mode read back, `& 0o7777`).

Implementations follow the existing split:
- `fs-local`: `remove.ts` / additions to `fsio.ts` for bytes / `chmod.ts`, each returning
  the seam's structured `FsError` taxonomy (`FS_NOT_FOUND`, `FS_IO_ERROR`, `FS_ABORTED`)
  instead of raw errno.
- `fs-sandbox`: each override runs `checkedTarget(target, sandboxPolicy)` first, then
  delegates — exactly the pattern `writeText` / `editText` already use.

A byte write reuses the existing `writeFileAtomic` path unchanged apart from accepting a
`Uint8Array` instead of a string, so atomic publication, staging, DACL handling and the
version guard all still apply.

**Two details worth reviewing**

1. `remove` probes with `lstat` before `rm`, so an absent path reports `FS_NOT_FOUND`
   rather than a raw ENOENT, a symlink is removed as a link (not its target), and a
   non-empty directory without `recursive` fails loudly instead of being silently emptied.
2. `chmod` reads the bits back after applying them and throws `FS_IO_ERROR` when the
   storage layer accepted the call without applying the mode. This is not theoretical: on
   HarmonyOS hmdfs user mounts `chmod()` returns success while the mode is unchanged, and a
   silent "success" there would hand the model a false premise about the file it just
   changed. The mask is `0o7777`, so higher bits in the argument cannot widen the change.

**Open question for you**

`fs-observation-policy` guards write/edit through the `fs/*` events, which is what makes
"read before you overwrite" work. Should `remove` participate in the same observed-state
guard (refuse to delete a file the session has not observed)? My diff wires `remove` /
`writeBytes` / `chmod` into `fs-local` and `fs-sandbox` but **not** into the observation
policy, because the right semantics there is a product decision I did not want to guess. I
would take a direction and implement it.

**Tests**

For each primitive: happy path; `FS_NOT_FOUND` on absent target; sandbox fence refusal
(`read-only` and outside-workspace); abort before effect; and for `chmod`, the
"storage-layer lied" case. Roughly 4 tests per primitive, in the packages' existing
`tests/` directories.

**If you want them split**

Happy to post three separate diffs instead — I merged them only because the underlying
question ("do mutation primitives belong on the seam?") is one decision, not three. Say the
word and I will separate them.

**Offer**

Diffs attached (13 files total across the three). Take them, reshape them, or lift only the
parts that fit your roadmap — no attribution needed, and no expectation of a reply.
```

---

## 附件清单（rebase 后重新生成）

| patch | 触及文件 |
|---|---|
| `dsh-fs-remove-primitive.patch` | `fs/src/{index,types}.ts`、`fs-local/src/{index,remove}.ts`、`fs-sandbox/src/index.ts` |
| `dsh-fs-write-bytes.patch` | `fs/src/{index,types}.ts`、`fs-local/src/{fsio,index}.ts`、`fs-sandbox/src/index.ts` |
| `dsh-fs-chmod-primitive.patch` | `fs/src/{index,types}.ts`、`fs-local/src/{index,chmod}.ts`、`fs-sandbox/src/index.ts` |

---

## 预期反驳与预备答案

| 反驳 | 回答 |
|---|---|
| "写个 `tool-fs-delete` 插件就行" | 那就绕过了 `fs-sandbox` 的 `checkedTarget`，seam 的承诺不再成立。这句话要放在正文第一段。 |
| "抽象成员更干净" | 6 个具体子类会全部编译失败；非抽象 + 默认抛错是唯一不破坏兼容的形状，也是这次补丁刻意选的。 |
| "删除应该由 UI 确认，不该给模型" | 那是授权策略层的决定（`fs-observation-policy`），不该由"缺少原语"来实现；正文已把这个开放问题交回维护者。 |
| "字节写入用 writeText + base64" | 会把 base64 解码错误、体积膨胀、大文件内存翻倍带进 seam；而且 `FS_NOT_TEXT` 语义会被绕过。 |
| "chmod 是危险操作" | 掩码限制在 `0o7777`、回读校验、过 sandbox 围栏；且它替代的是"模型 shell 里执行 `chmod`"这种更糟的路径。 |
| "先等 0.3 的 fs 重构" | 直接问：重构方向里是否已经包含变更原语？如果是，这份 diff 就当作需求说明，不再跟进。 |

---

## 如果被拒

三种收场，都要写进 patch layer 的 README：

1. **"暂时不做"** → patch layer 继续维护，半年后带着使用数据再发一次。
2. **"方向不同"** → 把 `remove` 实现改成纯工具插件（接受绕过围栏），并把这个取舍写进 README 的 Known Limitations。
3. **"已排期"** → 停止维护这三个 patch，改为在上游实现落地后删除本地补丁（这其实是最好结果）。
