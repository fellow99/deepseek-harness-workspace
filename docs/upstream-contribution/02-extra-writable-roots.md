# #4 可扩展可写根 · Discussion 正文

> 目标：`deepseek-ai/deepseek-harness` → Discussions
> 优先级：**第 2 条发**（不改 API 形状，风险低）
> 对应补丁：`dsh-desktop-hos/patches/dsh-v0.2.0-rc.2/dsh-extra-writable-roots.patch`（53 行，1 个文件）

---

## 为什么这条值得发

`SandboxExecutionPolicy` 只有一个 `workspaceRoot`。而真实产品的用户体验是：用户在"桌面 / 文档 / 下载"里让 agent 改文件，每次都要走一次逐操作升级（escalation）。这不是安全设计问题，是**表达力缺口**——部署方没有办法一次性声明"这三个目录本来就该可写"。

补丁的做法是把这件事变成**部署态配置**，而不是模型可影响的状态。这个区分是整条提案的核心，也是它能被接受的原因。

---

## 发之前

- [ ] rebase 到 `dsh-v0.2.1-alpha.5`（`roots.ts` 在 `packages/sandbox/sandbox/src/`，冲突概率低）
- [ ] 补 spec：`packages/sandbox/sandbox/tests/`
- [ ] 提前决定：**env var 还是 config 字段**——正文里要给维护者选择权，不要替他们定

**建议标题**：

> Let a deployment grant extra writable roots (beyond `workspaceRoot` / `/tmp` / `tmpdir()`)

---

## 正文（英文，可直接粘贴）

```md
**Problem**

`writableRoots(policy)` currently returns exactly three roots under `workspace-write`:

    return [...new Set([policy.workspaceRoot, '/tmp', tmpdir()].map(canonicalPath))]

A product that embeds dsh usually has a workspace *plus* a few directories its users
expect the agent to touch — Desktop, Documents, Downloads on a desktop shell. Today the
only way to allow those is a per-operation escalation, which turns a one-time product
decision ("this app may write the user's Documents folder") into a prompt on every write.

**Proposal**

Add deployment-level extra roots, read once from process state and canonicalised the same
way as the others:

    function extraWritableRoots(): string[] {
      const configured = process.env.DSH_EXTRA_WRITABLE_ROOTS
      if (configured === undefined) return []
      return configured.split(delimiter).map(e => e.trim()).filter(e => e.length > 0)
    }

    export function writableRoots(policy: SandboxExecutionPolicy): string[] {
      if (policy.mode !== 'workspace-write') return []
      return [...new Set([policy.workspaceRoot, '/tmp', tmpdir(), ...extraWritableRoots()].map(canonicalPath))]
    }

The variable is **deployment state, not model input**: no capability in the composition can
set it, so granting a root stays a decision the embedding product makes at startup and the
model cannot widen on its own. `read-only` still returns `[]`, so the mode keeps its
meaning.

**Why not a policy field**

`SandboxExecutionPolicy` is composed per call by the host; a root set is a property of the
deployment, not of one call. Putting it on the policy would push the same three-line
derivation into every host, and every host would spell it slightly differently.

I have no attachment to the env-var spelling. If you would rather have a config field on
the sandbox plugin (`Config` already exists in `sandbox/sandbox`), that works for me — the
substantive ask is "a deployment-level root set that `writableRoots()` folds in", not the
mechanism.

**Security notes**

- The value is canonicalised through the existing `canonicalPath()` (realpath-based), so a
  symlink cannot smuggle a root that differs from what is matched later.
- It cannot be set by a tool, a plugin, or the model — only by the process environment the
  embedding product controls.
- `read-only` is unchanged: extra roots never resurrect a read-only policy.

**Tests**

In `packages/sandbox/sandbox/tests/`:
1. unset variable → unchanged three-root list.
2. variable with two entries → both present, deduplicated against `workspaceRoot`.
3. empty / whitespace-only entries → ignored.
4. `mode: 'read-only'` + variable set → still `[]`.

**Offer**

Diff attached (one file, ~30 lines of logic + JSDoc). Take it or reshape it — no
attribution needed.
```

---

## 附件

- `dsh-extra-writable-roots.patch`（rebase 后重新生成）

---

## 预期反驳与预备答案

| 反驳 | 回答 |
|---|---|
| "这削弱了沙箱" | 不削弱：`read-only` 不变；新根来自部署态环境变量，模型与插件都写不了；且这是**替代逐次升级**，总暴露面反而更小 |
| "为什么不让宿主传 policy" | 宿主已经在传 policy，但只能传一个 `workspaceRoot`；这条是补表达力，不是加旁路 |
| "环境变量太隐晦" | 同意，所以正文明确说"config 字段也可以"，把机制决定权交给维护者 |
| "用户目录应该走授权而非配置" | 桌面产品的用户目录是安装时就知道的；逐次授权属于把配置问题变成打扰 |

---

## 这条的真正价值

即使被拒，它也把「**部署态 vs 调用态**」这个区分摆到了台面上——而这个区分正是后面 #1–#3 提案（"变更原语属于 seam"）的同一个论证结构。**发它的目的有一半是给第三条铺路。**
