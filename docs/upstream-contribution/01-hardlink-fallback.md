# #5 硬链接回退 · Discussion 正文

> 目标：`deepseek-ai/deepseek-harness` → Discussions（**不是 PR**，上游不接受外部 PR）
> 优先级：**第 1 条发**（真 bug，不涉 API 形状）
> 对应补丁：`dsh-desktop-hos/patches/dsh-v0.2.0-rc.2/dsh-fs-hardlink-fallback.patch`（62 行，1 个文件）

---

## 发之前必须做的三件事

1. **rebase 到 `dsh-v0.2.1-alpha.5`**：`git apply --check` 通过后再贴 diff。
2. **补一个 spec**：现在补丁里没有测试（`packages/fs/fs-local/tests/`）。最少三个用例（见文末）。
3. **标题不要出现 "feature request"**：这是 bug 报告。

**建议标题**：

> `writeFileAtomic({ createIfAbsent })` fails outright on filesystems that refuse hard links

---

## 正文（英文，可直接粘贴）

```md
**What happens**

`writeFileAtomic()` with `createIfAbsent` publishes by `link(tempPath, absolutePath)`.
On a filesystem that does not implement hard links, that call never succeeds, so every
such write fails with `FS_IO_ERROR` even though the target is absent and the staging
file is already in the destination directory.

Observed on HarmonyOS hmdfs user mounts: `link()` is refused with `EPERM` because the
directory inode operations export no `.link` handler. The same shape shows up on FAT/
exFAT and some FUSE mounts (`ENOSYS`, `EOPNOTSUPP`, `EXDEV`).

Repro (POSIX, with any filesystem whose `link()` fails):

    const info = await fs.writeText(target, 'hello', { createIfAbsent: true })
    // → FsError: cannot write "…": EPERM … , code FS_IO_ERROR

**Why it happens**

`packages/fs/fs-local/src/fsio.ts`, in `writeFileAtomic`:

    if (createIfAbsent !== undefined) {
      try {
        await linkFile(tempPath, absolutePath)
      } catch (error: unknown) {
        await throwGuardedCreateFailure(error, absolutePath, createIfAbsent.displayPath, inspectPublicationTarget)
      }
    }

`throwGuardedCreateFailure` classifies the error as `EEXIST`/existing-target → structured
`FS_NOT_OBSERVED`, and everything else → `FS_IO_ERROR`. A link-refusing filesystem is
"everything else", so the write cannot succeed at all.

**What I changed**

Treat the specific errno set that means "this filesystem does not do hard links" as a
publish strategy switch rather than a failure. The staging file is already created in the
destination directory, so a same-directory `rename()` publishes the same bytes atomically:

    function isHardLinkRefused(error: unknown): boolean {
      const code = (error as NodeJS.ErrnoException).code
      return code === 'EACCES' || code === 'EPERM' || code === 'ENOSYS'
        || code === 'EOPNOTSUPP' || code === 'ENOTSUP' || code === 'EXDEV'
    }

    if (createIfAbsent !== undefined) {
      try {
        await linkFile(tempPath, absolutePath)
      } catch (error: unknown) {
        if (isHardLinkRefused(error) && await isTargetAbsent(absolutePath, inspectPublicationTarget)) {
          await rename(tempPath, absolutePath)
        } else {
          await throwGuardedCreateFailure(error, absolutePath, createIfAbsent.displayPath, inspectPublicationTarget)
        }
      }
    }

**Trade-off I want to flag, not hide**

`link()` gives an atomic no-overwrite guarantee; `rename()` does not. The fallback
therefore re-probes the target with the same `inspectPublicationTarget` the guard already
uses and only renames when it is verified absent. That leaves a narrow TOCTOU window in
which a concurrent creator could be overwritten — but on these filesystems the alternative
is not "a safer write", it is "no write at all", and for a `createIfAbsent` write whose
caller already probed absence the window matches the caller's own probe.

If you would rather keep the guarantee strict, the fallback could instead claim the target
with `open(absolutePath, 'wx')` and then copy the staged bytes in, at the cost of losing
atomic publication. I picked rename because it preserves "the file appears complete or not
at all", which is what the staging directory exists for. Happy to switch if you prefer.

**Tests**

Added to `packages/fs/fs-local/tests/` (three cases):
1. `link` throws `EPERM`, target absent → rename publishes; content and mode match.
2. `link` throws `EPERM`, target present → still `FS_NOT_OBSERVED` (no overwrite).
3. `link` throws `EEXIST` → unchanged behaviour (guard path, no rename).

**Offer**

The full diff is attached (60 lines, one file). Take it as-is if useful — no attribution
needed, and no expectation of a reply.
```

---

## 附件

- `dsh-fs-hardlink-fallback.patch`（rebase 后重新生成）
- 建议同时贴出：在 hmdfs 上的 `hilog`/errno 证据（一句 `link() EPERM` 比任何描述都有说服力）

---

## 预期反驳与预备答案

| 反驳 | 回答 |
|---|---|
| "这只是你的文件系统的问题" | errno 集合覆盖 FAT/exFAT/FUSE/NFS 只读场景；`EXDEV` 已包含跨设备 |
| "rename 破坏 no-overwrite" | 已在正文主动披露并给替代方案（`open(wx)` + copy） |
| "为什么不用 copyFile" | copy 不原子：中途失败会留下半截文件，正是 staging dir 要避免的 |
| "应该由宿主决定发布策略" | 这属于后端实现细节，不该让每个宿主重复实现一遍 |

---

## 如果两周没回应

把这条降级为 patch layer 的一部分（`04-patch-layer.md`），不再跟进。**它是五条里唯一"不发也说得通"的**——因为它在鸿蒙外壳里已经跑通了。
