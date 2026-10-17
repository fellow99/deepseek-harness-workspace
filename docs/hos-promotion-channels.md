# dsh-desktop-hos 推广途径清单

> 编制日期：2026-10-17（同日按「已稳定投稿」的现状重排 P0）
> 与 [`社区推广方案.md`](社区推广方案.md) §5 的关系：那份讲**受众分层与选题**，这份讲**渠道、门槛与第一个动作**。
> 渠道可用性已逐条核实（见 §7）。

---

## 0. 四条硬约束（它们决定优先级，不是建议）

**① 装机被卡住，所以 6 个月内买不到"下载量"。**
debug 签名绑定设备 → 别人装不上；release 签名只能走 AppGallery；上架还要 ACL 受限权限 + AI 合规三件套。**推广的货币只能是三样**：技术声誉（被引用）、开发者采用（自己构建成功）、生态位（被收录/被转发）。

**② 主流榜单有星级门槛，你现在进不去。**
`awesome-electron` 贡献规则原文：*"The project repo should have at least 100 stars"*、*"If you just created something, wait at least 30 days"*、开源应用还要**至少一个可安装二进制 + 英文 README + README 内截图**。你是 2★、且鸿蒙版没有可分发的二进制 → **这条今天走不通**。

**③ 应用名已去商标，搜索入口必须自己造。**
上架合规把 "DeepSeek Harness" 从应用名拿掉了，用户搜"DeepSeek Harness 鸿蒙"搜不到你。**关键词得靠文章标题和仓库 description 补**。

**④ 你的瓶颈已经从「渠道」转移到「落地页」。**
你已有 10 个平台稳定分发（微信公众号、知乎、掘金、小红书长文、头条、51CTO、简书、思否、开源中国、B站），还有《系列文章规划》与《技术文章发表套路》。「内容 → 曝光」是通的，**断点全在曝光之后**：仓库搜不到、README 不回链、镜像不存在、没有收录位。所以 §3 的 P0 全是"落地端"的活，**不是再开账号**。

---

## 1. 三条并行线

| 线 | 换什么 | 当前状态 |
|---|---|---|
| **内容网** | 技术声誉、长尾搜索 | ✅ 已覆盖（10 平台）——保持节奏即可，不是本轮重点 |
| **开发者网** | 可复现的构建、issue、star | ❌ 短板：搜不到、不回链、无镜像 |
| **生态网** | 背书、转发、收录 | ❌ 短板：收录位在文末、未自荐、无上游联动 |

---

## 2. 途径总表

| 优先级 | 途径 | 门槛 | 投入 | 换来什么 |
|---|---|---|---|---|
| **P0** | 修死链 + 统一 CTA | 无 | 30 分钟 | 止住已有曝光的漏损 |
| **P0** | GitHub topics / description | 无 | 10 分钟 | 搜索可见性（当前最大漏洞） |
| **P0** | README 反向链接到文章 | 无 | 20 分钟 | 免费的站内转化闭环 |
| **P0** | AtomGit 镜像 + 上游 issue | 无 | 30 分钟 + 1 issue | 华为生态可信度 |
| **P0** | Gitee 镜像 | 无 | 30 分钟 | 国内开发者触达 |
| **P0** | 策展列表条目升级 | 无 | 1 小时 | 从文末挪到 UI & Clients |
| **P0** | HelloGitHub 自荐 | **无星级门槛** | 1 小时 | 182k★ 月刊流量 |
| **P0** | 版本/补丁数口径统一 | 无 | 30 分钟 | 止住"13 vs 14"的信任损耗 |
| P1 | B站 3 分钟真机录屏 | 无 | 1 天 | 最强"存在证明"，文章可复用 |
| P1 | 华为开发者社区 / 51CTO 投稿 | 无 | 已有节奏 | 权威背书 |
| P1 | ohpm 三方库 | 需抽包 | 3–5 天 | 从"应用"变"生态组件" |
| P1 | OpenHarmony SIG | 需技术对话 | 持续 | 上游影响力 |
| P2 | awesome-electron | **≥100★** | 1 小时（届时就绪） | 国际 Electron 受众 |
| P2 | AppGallery 上架 | ACL + 合规 | 数周 | 唯一合规分发 |
| P2 | HDC / 创新赛 / 极客马拉松 | 按当期公告 | 视活动 | 一次露出 > 十篇自荐 |
| P2 | HN / r/HarmonyOS | 无 | 1 小时 | 稀有题的国际曝光 |

---

## 3. P0 清单（全部是"落地端"）

### ① 修死链、统一 CTA（30 分钟｜最高优先）

**事实**：`deepseek-harness-desktop` / `deepseek-harness-harmony` 是 **2026-09-25 改名前的工程名，从未作为 GitHub 仓库存在**（`fellow99/...` 一律 404、无重定向）。现行仓库是 `dsh-desktop` / `dsh-desktop-hos`。更麻烦的是：这两个名字现已被**别人的真实项目**占用（`anywhere-labs/deepseek-harness-desktop` 31k★、`hairyf/deepseek-harness-desktop`），所以文章里只写仓库名而不带 owner 时，读者会被导向竞品。

- 动作：逐篇抽查已发布文章（desktop 2 篇 + harmony 已发篇）里的仓库链接；可编辑平台（知乎、掘金、简书、思否、开源中国、51CTO）统一到两个新仓库；公众号不便改就在下次发文时置顶修正。
- 验收：逐篇点开全部 200，且落地到**有截图、有 Release、有兼容矩阵**的新 README。

### ② GitHub 仓库搜索可见性（10 分钟）

`dsh-desktop-hos` 的 topics 缺上游 README 官方指定的 `dsh-plugin`，description 里也没有 "DeepSeek Harness" 全称。

- 加 topics：`dsh-plugin`、`arkts`、`2in1`、`harmonyos-next`
- description 改为：`Electron 37 on HarmonyOS — run DeepSeek Harness (DSH) as a native 2in1/tablet app. 鸿蒙上的 AI Agent 工作台`
- `dsh-desktop` 描述补上 `Linux`（它唯一独占的词）
- 验收：GitHub 搜 `dsh harmonyos`、`deepseek harness harmony`，第一页能看到仓库。

### ③ README 反向链接（20 分钟）

「文章 → 仓库」通，「仓库 → 文章」断：README 里一篇都没提到那 8 篇文章。这是唯一完全免费的转化提升。

- 两个仓库 README 末尾各加「相关文章 / 系列」：按主题列 3–5 篇（架构、保姆级复现、鸿蒙踩坑、ACL 申请），带平台名。
- 验收：README 里至少 3 个可点链接，点进去能回到同一个仓库（闭环）。

### ④ AtomGit 镜像 + 上游联动（30 分钟 + 1 个 issue）

`harmonypc-electron` 本体在 AtomGit，鸿蒙开发者会去那里看运行时。做法见 §8。

- 另加一个动作：在 `harmonypc-electron` 提 issue，说「有个下游应用把它跑起来了」，附真机截图 + 适配清单（loopback、symlink、HOME）。这是进鸿蒙 Electron 圈子最自然的一次曝光。
- 验收：AtomGit 仓库可访问；上游 issue 有人回或至少被看到。

### ⑤ Gitee 镜像（30 分钟）

同 §8。Gitee 的「项目推荐 / GVP」按平台当期表单提交；**镜像仓库的 star 不与 GitHub 同步**，别指望它凑 100★。

### ⑥ 策展列表条目升级（1 小时）

仓库已在 `Dominic789654/awesome-deepseek-harness`（5,657 条目）里，但排在文末低曝光区。

- 提 PR：条目挪到 **「UI & Clients」** 段；描述加 `HarmonyOS (2in1)`；申请预览图位；顺手补 `Zhiyuan-Fan/Awesome-DeepSeek-Harness-Plugins` 的 Runtime & Sandboxes 段。
- 验收：条目出现在 UI & Clients，描述含 HarmonyOS。

### ⑦ HelloGitHub 自荐（1 小时）

`hellogithub.com/periodical` 是自荐入口，README 明确写「欢迎推荐或自荐」，**没有星级门槛**（对比 awesome-electron 的 ≥100★）。

- 材料：3 句话定位 + 1 张真机图 + 仓库链接。他们偏好"有代表性、能跑起来"的项目。
- 验收：提交成功；若被收录，当月有一波集中流量（配合 ③ 的闭环）。

### ⑧ 版本/补丁数口径统一（30 分钟）

三处不一致：文章里写「补丁从 6 涨到 14+」，README 与 `build-dsh.mjs` 是 **13**；文章里产品名仍是旧名。

- 动作：在《技术文章发表套路》里加一行固定字段 ——「版本口径：`dsh-v0.2.0-rc.2` · 13 patch · 产品名 `DSH Desktop（鸿蒙版）`」，每次写作直接抄。
- 验收：下一篇文章不再出现 14。

---

## 4. 4 周日历（每周 2 件，不要多）

| 周 | 动作 |
|---|---|
| W1 | ① 修死链 + ② 仓库关键词；③ README 反向链接 |
| W2 | ④ AtomGit 镜像 + 上游 issue；⑤ Gitee 镜像 |
| W3 | ⑥ 策展列表 PR；⑦ HelloGitHub 自荐 |
| W4 | ⑧ 口径统一 + 存量文章一次「改链接」集中清理 |

---

## 5. 指标（别看 star）

| 指标 | 4 周目标 | 说明 |
|---|---|---|
| 旧文死链 | 0 | 纪律项 |
| README → 文章点击 | 有即可 | 装 GitHub 流量分析后看 referrer |
| 外部 issue / 构建求助 | ≥ 1 | **最高质量信号**：有人真的在构建 |
| star | 10–30 | 4 周内到 100★ 不现实，别用它判成败 |

**通向 awesome-electron 的现实路径**：存量文章 → 30–40★ → 策展列表 + HelloGitHub 二次曝光 → 100★ → 再用 `dsh-desktop` 提（它有可安装二进制，符合规则）。

---

## 6. 不要做

- **不要发"求 star"**：开源圈子对这类内容的负反馈是立即的。
- **不要再开第 11 个平台**：10 个已超过单人维护的合理上限，边际收益接近 0。
- **不要在 AppGallery 上架前做消费级营销**：用户来了装不上，比没来更糟。
- **不要只做镜像不做联动**：镜像仓库存量 = 0 曝光，一次上游 issue 比十个镜像有用。
- **不要拿"Electron"当唯一卖点对鸿蒙开发者讲**：他们更关心"我的 ArkTS 能力能不能复用"；对 Electron 开发者讲才是"鸿蒙"这个词最有冲击力。

---

## 7. 渠道与事实核实（2026-10-17）

| 项 | 核实结果 |
|---|---|
| 旧工程名 `deepseek-harness-desktop` · `deepseek-harness-harmony` | **从未作为仓库存在**（404，无重定向）；名字现被 `anywhere-labs/`、`hairyf/` 两个真实项目占用 |
| `fellow99/dsh-desktop` / `dsh-desktop-hos` | 0★ / 2★；后者 topics 缺 `dsh-plugin` |
| 华为开发者社区 | `developer.huawei.com/consumer/cn/community` 可访问 |
| 51CTO 鸿蒙社区 | `ost.51cto.com` 可访问 |
| AtomGit / Gitee | `atomgit.com` / `gitee.com` 可访问（运行时托管在 AtomGit） |
| ohpm 三方库中心仓 | `ohpm.openharmony.cn` 可访问 |
| HelloGitHub | 仓库 182,566★，README 明确写「欢迎推荐或自荐」 |
| awesome-electron | 27,316★；**≥100★ + 30 天冷静期 + 至少一个可安装二进制** |
| GitHub `topic:harmonyos` | 1,377 个仓库，最大 19,982★——有受众但头部集中 |
| 策展列表 | 5,657 条目，仓库已在列（文末），有 Profiles & Patch Layers 段 |

---

## 8. GitHub → Gitee / AtomGit 镜像怎么设

三条路。**GitHub 是唯一真源，镜像只做单向**；Gitee 有官方镜像功能，AtomGit 没有文档支撑，所以两边用的方案不同。

| 方案 | 持续同步 | 用在哪 |
|---|---|---|
| **A. Gitee「仓库镜像管理」** | ✅ 自动（webhook）或手动 | **Gitee 首选**，官方文档步骤确定 |
| **B. GitHub Actions 单向推送** | ✅ 每次 push/tag | **AtomGit 首选**；也是 Gitee 的兜底 |
| C. 平台「导入仓库」 | ❌ 一次性快照 | 只想先把仓库变出来时 |

### A. Gitee 仓库镜像管理（官方步骤）

来源：Gitee 帮助中心《仓库镜像管理（Gitee ↔ GitHub 双向同步）》。

1. Gitee 目标仓库 → **管理** → **仓库镜像管理** → **添加镜像**。
2. 首次使用先**绑定 GitHub 账号**（按弹窗引导）。
3. 镜像方向选 **Pull**（GitHub → Gitee）。**不要选 Push，也不要双向。**
4. 「镜像仓库」下拉选 `fellow99/dsh-desktop-hos`。
5. 「个人令牌」填 GitHub PAT：**必须含 `repo`**；若勾「自动从 GitHub 同步仓库」，还要 **`admin:repo_hook`**（Gitee 会去 GitHub 仓库建 webhook）。
6. 保存。之后 push GitHub 即自动同步，也可在页面手动点「更新」。

官方约束（照抄要点，容易踩）：

- 只同步 **分支 / 标签 / 提交**；**不支持 Git-LFS**。
- 触发间隔最短 **5 分钟**，单次超过 **30 分钟**算超时。
- 镜像会**覆盖**目标仓库的分支、标签与提交记录。
- **双向镜像有代码丢失风险**，官方自己提示谨慎使用。
- 该页仍挂着一句旧公告「限时开放至 2022-08-31」——若你的账号里看不到这个菜单，说明当前套餐未开放，直接走 B。

### B. GitHub Actions 单向推送（AtomGit 首选）

1. **在 Gitee / AtomGit 各建一个空仓库**。**不要**勾选「初始化 README」——否则首次推送会因历史不一致被拒。
2. **给目标仓库加部署公钥（Deploy Key，只绑单仓库，别用账号级 key）**：
   - Gitee：仓库 → 管理 → 部署公钥 → 勾选「启用推送权限」
   - AtomGit：仓库设置 → 部署公钥 → 同样要写权限
   - 没有现成 key 就 `ssh-keygen -t ed25519 -C "mirror" -f ~/.ssh/mirror`，公钥贴到平台，私钥进下一步。
3. **GitHub 仓库 → Settings → Secrets and variables → Actions**，新增两个 secret：`GITEE_SSH_KEY`、`ATOMGIT_SSH_KEY`（私钥全文，含 `BEGIN/END` 行）。
4. **加 workflow**（`dsh-desktop-hos/.github/workflows/mirror.yml`）：

```yaml
name: mirror

on:
  push:
    branches: [main]
    tags: ['v*']
  workflow_dispatch:

concurrency:
  group: mirror
  cancel-in-progress: true

jobs:
  push:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        include:
          - name: gitee
            host: gitee.com
            path: <你的 Gitee 用户名>/dsh-desktop-hos.git
            key: GITEE_SSH_KEY
          - name: atomgit
            host: atomgit.com
            path: fellow99/dsh-desktop-hos.git
            key: ATOMGIT_SSH_KEY
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Push to ${{ matrix.name }}
        env:
          KEY: ${{ secrets[matrix.key] }}
        run: |
          mkdir -p ~/.ssh
          printf '%s\n' "$KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H "${{ matrix.host }}" >> ~/.ssh/known_hosts 2>/dev/null
          git remote add mirror "git@${{ matrix.host }}:${{ matrix.path }}"
          git push --force mirror \
            'refs/heads/*:refs/heads/*' \
            'refs/tags/*:refs/tags/*'
```

5. **手动跑一次**（Actions → mirror → Run workflow），确认两边 `git log -1` 一致。

### 注意（踩坑点）

- `--force` **只作用于镜像远端**。镜像仓库应当是只读的：README 顶部写「Mirror of github.com/fellow99/dsh-desktop-hos — do not open PRs here」。
- **不要镜像 issues / PR / wiki**：平台模型不同，导不过去，也别让人误以为能提 issue。
- 目标仓库若有**受保护分支**，会拒绝 push → 在平台设置里放开默认分支，或改用机器人账号。
- **Release 附件不会同步**：`dsh-dist.tar.gz` 之类要单独处理（或干脆不同步，镜像只留源码）。
- 首次推送如果报 `non-fast-forward`，说明目标仓库不是空的（多半是勾了初始化 README）→ 删库重建。
- **star 不互通**：Gitee/AtomGit 有各自的 star，不会帮你凑 awesome-electron 的 100★。

### C. 一次性导入（把仓库先变出来）

- **Gitee**：新建仓库 → 「导入已有仓库」→ 填 GitHub 地址 → 导入（含分支与标签，**一次性**）。之后的持续同步回到方案 A。
- **AtomGit**：新建仓库时有导入入口，从 GitHub / Gitee 地址导入；同样是一次性。AtomGit 文档站（与 GitCode 共用文档）**查不到镜像或自动同步的条目**，所以不要指望它有持续同步——用方案 B。

> 取舍：**Gitee 用 A（官方支持、零维护），AtomGit 用 B（多 10 分钟，但一定跑得通）。**
