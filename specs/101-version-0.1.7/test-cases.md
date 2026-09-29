# 101-version-0.1.7 — 测试用例

## TC-1 Patch 兼容性
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-1.1 | 对 `patches/dsh-v0.1.7-rc.2/` 3 个补丁执行 `git apply --check`（在干净 rc.2 上） | 全部 exit 0 |
| TC-1.2 | `npm run build:dsh` 重复执行 | 幂等：已应用则跳过，不报错 |
| TC-1.3 | `git apply --reverse --check` 3 个补丁 | 应用后 reverse-check 全部 exit 0 |

## TC-2 版本号一致性
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-2.1 | desktop `package.json` / `package-lock.json` | version = 0.1.7 |
| TC-2.2 | grep `0.1.5` / `dsh-v0.1.5-rc.2`（desktop，排除 node_modules/历史 spec/out） | hos 相关以外无遗留有效引用 |
| TC-2.3 | website index/index_zh/README | Desktop v0.1.7 / dsh-v0.1.7-rc.2；HarmonyOS 保留 v0.1.5 |
| TC-2.4 | 父工程 README 版本表 | desktop=0.1.7，hos=0.1.5 |
| TC-2.5 | workflows ci.yml / release.yml | dsh-v0.1.7-rc.2 |

## TC-3 Desktop 打包产物基本功能
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-3.1 | `npm run package` 完成 | `out/DSH Desktop-win32-x64/dsh-desktop.exe` 生成 |
| TC-3.2 | 启动打包 exe（--remote-debugging-port=9222） | 窗口打开并加载 dsh Web UI |
| TC-3.3 | 设置 DeepSeek API KEY | 保存成功（截图） |
| TC-3.4 | 创建工作区 | 工作区创建成功（截图） |
| TC-3.5 | 发起一次对话 | 收到模型回复（截图） |

## TC-4 代码评审
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-4.1 | requesting-code-review 派发审查 | REVIEW_REPORT.md 产出，分级完整 |
| TC-4.2 | receiving-code-review 处理反馈 | 无 Critical 遗留 |

## TC-5 安全
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-5.1 | `git diff` / `git diff --cached` 扫描 API KEY（sk- 前缀） | 无命中 |

## TC-6 Tag 与推送
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-6.1 | desktop / website `git tag` | 注解 tag v0.1.7 存在 |
| TC-6.2 | `git ls-remote --tags` | 远端可见 v0.1.7 |
| TC-6.3 | 父工程子模块指针 | dsh=rc.2 / desktop=v0.1.7 / website=v0.1.7 / hos、market=main 最新 |
