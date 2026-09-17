# skills/ —— 通用、可插拔的工具技能

本目录存放**通用、可插拔**的 dsh 工具技能：它们**不依赖任何特定封装壳**的补丁、运行期环境或打包路径约定，因此可被任意一个壳（`deepseek-harness-desktop`、`deepseek-harness-harmony`，或未来的其它壳）直接采纳。

> **本目录当前为空**（只有这份 README）。迄今唯一的技能是 `harmony-runtime-capabilities`，它描述的是 harmony 壳特有的运行期环境，属**壳专用技能**，位于 [`deepseek-harness-harmony/skills/`](../deepseek-harness-harmony/skills/)。

## 一、与封装工程 `skills/` 的分工

| 维度 | 本目录 `skills/` | `<封装工程>/skills/` |
|---|---|---|
| 定位 | **通用、可插拔**技能 | 该封装工程的**专用**技能 |
| 判定标准 | 不依赖任何特定壳的补丁 / 适配 / 路径，可被多个壳采纳 | 描述或依赖该壳特有的运行期环境 |
| 命名 | 功能性 kebab-case，**无前缀** | 同左（**无前缀**） |
| 目录名 = frontmatter `name` | 是（**约定，非校验**） | 是（同） |
| 何时可用 | 由采纳它的壳决定 | 随该壳的包分发，运行期作为 bundled 根提供 |

**判断一个技能该放哪边**：问一句 —— 「它描述的东西在另一个 dsh 壳上还成立吗？」

- 成立 → 通用技能，放本目录。
- 不成立（例如描述某个壳的沙箱、围栏、打包路径、能力缺口）→ 该壳的专用技能，放 `<该壳>/skills/`。

例：`harmony-runtime-capabilities` 通篇讲 HarmonyOS HAP 上的沙箱围栏、`hmdfs` 特性、缺哪些工具 —— 在桌面 Electron 壳上完全不成立 → 属壳专用技能。

## 二、技能格式（由 dsh 决定，本目录不自定义）

dsh 的技能发现规则（实现位于 `deepseek-harness` 的 `packages/skill/skill-filesystem/`）：

- **只扫描根目录的下一层**（深度 1）。嵌套的 `**/SKILL.md` **不会**被发现。
- 两种形态，二选一：
  - **目录包**：`<技能名>/SKILL.md`（推荐；技能目录内可再放 `references/` 等资源，正文按相对路径引用）
  - **单文件**：`<技能名>.md`
- **YAML frontmatter**：文件首行必须是 `---`，并有闭合的 `---`。字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | ✅ | 技能标识，须匹配 `^[a-z0-9]+(?:-[a-z0-9]+)*$`（小写字母/数字，单连字符分隔） |
| `description` | ✅ | 非空字符串。**模型可见**：它被注入技能目录，展示时截断到 500 字 |
| `whenToUse` | — | 非空字符串，说明何时该用这个技能 |
| `metadata` | — | 任意 YAML 对象 |
| `disable-model-invocation` | — | 布尔。`true` = 不进技能目录、`skill` 工具也取不到，只能由人工 `/name` 触发 |
| `user-invocable` | — | 布尔。`false` = 人工无法用 `/name` 触发 |

- ⚠️ **必须用连字符形式**。旧驼峰键（`disableModelInvocation` / `modelInvocable` / `userInvocable`）会被**直接拒绝**，整个技能被丢弃。未知的额外字段会被忽略。
- **`name` 与目录名不一致不会被报错** —— 实现不比对二者。但本约定要求二者必须一致：被发现的始终是 frontmatter 里的 `name`，不一致只会让人和模型都困惑。

## 三、约束

1. **`name` 在同一个技能根内必须唯一**。跨根同名时按优先级裁决：rank 数字**小**者优先。
2. 技能**正文不缓存**，每次加载都重读文件；目录层（`name` / `description`）的变更经文件监听失效后生效。
3. **不检查文件大小**（解析只看内容，无体积上限）；但 `description` 展示时截断到 500 字 —— 关键信息放在 `description` 与正文开头。

   > 另注：经 `ctx.fs` 读取的技能根（非 bundled）要求文件能作**文本**读取，非文本文件会被跳过并告警（`FS_NOT_TEXT`），与体积无关；bundled 根走 `trustedHost`、直接经 Node fs 读取，不受此限。
4. 正文引用同目录资源时用**相对路径**（技能的资源根即该技能目录）。

## 四、如何被采纳（通用技能不会自动生效）

**与壳专用技能不同，本目录的技能不会被任何壳自动加载。** 每个壳自行决定是否采纳。dsh 提供的标准机制是 `skill-filesystem` 的 `customSkillDirs`：

```yaml
- id: skill-filesystem
  name: '@deepseek-ai/dsh-skill-filesystem'
  config:
    customSkillDirs:
      - !!js "process.getBuiltinModule('node:url').fileURLToPath(new URL('skills/', baseUrl))"
```

这是 dsh 自带 `cordis` preset 指向自己 `skills/` 的写法（`packages/preset/agent-presets/presets/cordis/agent.cordis.yml`）；`baseUrl` 是该 preset 自身目录。

> **现状**：目前两个壳都**没有**采纳本目录 —— 它们的 bundled 根各自指向自己的 `skills/`。本目录是"通用技能"的归位点，当前为空。

## 五、新增一个技能

1. 在本目录新建 `<技能名>/SKILL.md`（顶层，不要嵌套多层目录）。
2. frontmatter 的 `name` 与目录名同名，填好 `description`。
3. 正文说清「能做什么 / 不能做什么 / 怎么做」，并区分**已验证的事实**与**未验证的推测** —— 不要把推测写成事实。
4. 若某个壳要采纳它，在该壳的构建 / 运行期配置里加 `customSkillDirs` 指向本目录（见 §四）。

## 六、相关文档

- [`deepseek-harness-harmony/skills/README.md`](../deepseek-harness-harmony/skills/README.md) —— harmony 壳的专用技能目录约定
- [`README.md`](../README.md) —— workspace 总说明的「Skill convention」节
- [`dsh-plugins/README.md`](../dsh-plugins/README.md) —— 对应的**插件**两层约定（plugins 与 skills 是同一套分工思路）
