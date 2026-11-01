# 201 · dsh-market 插件市场（技术方案 · 父工程）

> HOW：跨四壳的统一做法 + 各壳落点。WHAT 见 [spec.md](./spec.md)（`FR-201-*`）。
> 设计细节：ark `docs/pnpm+dsh-market.md`、hos `docs/110-pnpm+dsh-market.md`、各壳 `201-dsh-market/{spec,plan}.md`。

## 1. 统一装配流水线（各壳同构）

```
overlay(stage)
  → 物化本工程包（@dsh-*/…）
  → 物化 dshmarket（FR-201-001~004）
  → 物化进程内 pnpm 引擎 dsh-market-pnpm（110）
  → 注入 profile（bundle 声明 + 运行配置，FR-201-006~008）
  → 垫片/补丁
assertStage
  → dshmarket 入口/闭包可解析、无 @deepseek-ai 副本
  → 产物新鲜度：market 的 src 不新于 lib（FR-201-020/021）
```

## 2. 各壳落点映射

| FR | `desktop` | `hos` | `ark` | `android` |
|---|---|---|---|---|
| 物化 | `build/collect-dsh`（Electron 打包内） | `scripts/collect-dsh.mjs::collectDshMarket()` | `scripts/assemble-payload.mjs::collectDshMarket()` | `scripts/assemble-payload.mjs::collectDshMarket()` |
| 构建市场 | 构建期构建 | `scripts/build-dsh.mjs`（**无条件 `npm run build`**） | 构建期构建 + `assertMarketBuilt` | 构建期构建 + `assertMarketBuilt`（本次补） |
| 版本断言 | 构建期 | `collect-dsh.mjs::assertDshMarketVersion()`（1.66.7） | `assertDshMarketArtifact`（1.66.42） | 装配期（1.67.0） |
| profile/bundle | desktop profile `dependencies`+`bundles`+`config{profile,allowRestart:false}` | `$DSH_HOME/profiles` 复制 + `cordis.patch.yml` 注入 `config{profile:desktop,allowRestart:false}` | `profiles/ohos` `dsh.profile.bundles` 追加 `dshmarket`（无 dependencies） | `profiles/android` 同 ark |
| 装卸通道 | 便携 Node+pnpm+dsh CLI | 进程内 pnpm（`packages/market-hos` 契约） | 进程内 pnpm（`packages/market-ohos`，含 `DSH_OHOS_JITLESS_ENTRY`） | 进程内 pnpm（`packages/market-android`） |
| 运行期接线 | PATH 注入便携 Node | 复制到 profile（禁 symlink） | `payload/host.js`：`DSH_PNPM_ENGINE`+`pnpm-workspace.yaml`；`bootstrap.js`：`DSH_OHOS_JITLESS_ENTRY` | `DshRuntime` env `DSH_PNPM_ENGINE` + profile pnpm 配置 |

## 3. 产物新鲜度门禁（405 修复的技术实现）

各壳在「把市场复制进载荷」前调用等价断言（ark/android 为 `assertMarketBuilt()`）：

```js
// 伪代码（各壳实现见 scripts/*.mjs）
const pairs = [['src/index.ts','lib/index.js'],
               ['src/routes.ts','lib/routes.js'],
               ['src/client/index.ts','client/client.js']];
for (const [src, lib] of pairs)
  if (mtime(src) > mtime(lib) + 1000ms) FAIL('dsh-market 产物落后于源码，先 npm run build');
```

- `android`：`collectDshMarket()` 内 `assertMarketBuilt(MARKET_ROOT)`（新增，2026-11-01）。
- `ark`：同名断言（本模块 FR 的原始来源）。
- `hos`：`build-dsh.mjs` 构建期**无条件** `npm run build`，产物天然新鲜；`collect-dsh.mjs` 再断言版本。
- `desktop`：构建期构建。

> 若壳选择「复制副本」，还需比对**源目录 vs 已物化副本**的 mtime，落后即重拷（FR-201-021）。

## 4. 进程内 pnpm 契约（hos/ark/android）

- 宿主在 market bundle mount 前提供 `desktopProfiles`；市场 `ctx.get('desktopProfiles')` 命中即用进程内包管理器。
- `desktopPnpm.runPlugin(args, dir, signal)` 返回句柄（`stdout`/`stderr`/`done`/`cancel`）；市场据此执行 `add`/`remove`。
- 引擎 `dsh-market-pnpm`（`110`）随载荷物化；`DSH_PNPM_ENGINE` 指向其 `bin/pnpm.cjs`。
- ark `--jitless`：额外下发 `DSH_OHOS_JITLESS_ENTRY`，供 worker 新 realm 安装兼容层。

## 5. 验证

| 层 | 手段 | 判据 |
|---|---|---|
| 构建期 | 各壳装配脚本 | 版本/入口/闭包断言通过；产物新鲜度断言通过；`dshmarket/node_modules` 无 `@deepseek-ai` |
| 运行期 | 设备/模拟器 | 设置页出现「插件市场」；日志有「契约已提供」；安装走进程内、无 pnpm 子进程 |
| 回归（405） | 设备 | 设置→插件市场→「显示」不再 `HTTP 405`；`lib/routes.js` 含 `sidebar-entry` |

## 6. 风险

- 网络（registry / 目录源）不可达 → 如实记录，不伪造成功（C3）。
- 市场版本漂移（各壳 pin 不一致）→ 以 FR-201-005 对齐；drift 已在 §4 矩阵登记。
- `ignoreScripts`：需构建脚本的插件不可安装（已披露，FR-201-023）。
- 载荷分代：换包后已装插件不保留（FR-201-024）。
