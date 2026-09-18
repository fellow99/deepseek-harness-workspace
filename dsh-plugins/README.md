# dsh-plugins/ —— 通用、可插拔插件

本目录存放**通用、可插拔**的 dsh 插件：它们**不依赖任何特定封装壳**的补丁、profile 或运行期适配，因此可被任意一个壳（`deepseek-harness-desktop`、`deepseek-harness-harmony`，或未来的其它壳）直接消费。

> **本目录当前为空**（只有这份 README）。迄今唯一的插件是 `delete` / `move` 文件工具，它依赖 harmony 壳独有的 `dsh-fs-remove-primitive` 补丁，属**壳专用插件**，已迁至 [`deepseek-harness-harmony/plugins/harmony-plugin-fs-mutate/`](../deepseek-harness-harmony/plugins/harmony-plugin-fs-mutate/)。

## 一、与封装工程 `plugins/` 的分工

| 维度 | 本目录 `dsh-plugins/` | `<封装工程>/plugins/` |
|---|---|---|
| 定位 | **通用、可插拔**插件 | 该封装工程的**专用**插件 |
| 判定标准 | 不依赖任何特定壳的补丁/适配，可被多个壳消费 | 依赖该壳特有的补丁、profile 或运行期适配 |
| 命名 | `dsh-plugin-XXX` | 如 harmony 壳为 `harmony-plugin-XXX` |
| 目录名 = 包名 | 是 | 是 |
| 载入时机 | 由各消费壳自行决定（可选、可插拔） | 编译期打入包内，运行期全部默认加载 |

**判断一个插件该放哪边**：问一句 —— 「把它装到另一个 dsh 壳上，它还能工作吗？」

- 能 → 通用插件，放本目录。
- 不能（依赖某个壳的补丁集 / profile / 运行期适配）→ 该壳的专用插件，放 `<该壳>/plugins/`。

例：`delete` / `move` 文件工具依赖 `ctx.fs.remove`，而该原语只存在于 harmony 壳打过 [`dsh-fs-remove-primitive.patch`](../deepseek-harness-harmony/patches/dsh-v0.1.5-rc.2/dsh-fs-remove-primitive.patch) 的 dsh 上 → 离开 harmony 壳就不工作 → 属壳专用插件。

## 二、约定

1. **一个插件一个目录**。目录名 == npm 包名 == 插件的 `name` 导出，三者在同一层内必须完全相同。
2. **命名 `dsh-plugin-XXX`**，裸包名（非 scoped），`XXX` 描述功能。
3. **纯 ESM，无构建步骤** —— `lib/` 即发布源码。
4. **`private: true`**，不发布到 registry；依赖一律声明为 `peerDependencies`，由消费壳的宿主依赖闭包提供。
5. **物化由消费壳负责**：本目录不参与构建。各壳在自己的收集阶段把需要的插件物化进随包分发的 `node_modules/`，再由 agent preset 的一行挂载。本目录**不规定**任何壳的构建方式——这正是"通用"的含义。
6. **文档随插件**：每个插件自带 `README.md`（如需中文镜像另加 `README_zh.md`），说明其配置项与**已知限制**。

## 三、新增一个插件

1. 在本目录新建 `dsh-plugin-XXX/`，内含 `package.json`（`name` = `dsh-plugin-XXX`）与 `lib/`。
2. 写该插件自己的 `README.md`。
3. 在**每个**需要它的壳里配置物化与 preset 挂载（各壳方式不同，本目录不作规定）。

> ⚠️ 与壳专用插件不同，通用插件**不会**被任何壳自动"全量加载"——每个壳自行决定是否消费。若某插件只应被一个壳使用，它就不该放在本目录（见 §一 的判定）。

## 四、相关文档

- [`deepseek-harness-harmony/plugins/README.md`](../deepseek-harness-harmony/plugins/README.md) —— harmony 壳的专用插件目录约定
- [`README.md`](../README.md) —— workspace 总说明的「Plugin convention」节
- [`deepseek-harness-harmony/specs/202-plugin-fs-mutate/`](../deepseek-harness-harmony/specs/202-plugin-fs-mutate/) —— `delete` / `move` / `copy` / `chmod` 壳专用插件的规格、方案与测试用例
