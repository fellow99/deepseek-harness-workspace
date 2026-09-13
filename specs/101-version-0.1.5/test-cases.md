# 101-version-0.1.5 — 测试用例

## TC-1 Patch 兼容性
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-1.1 | 对 `patches/dsh-v0.1.5-rc.2/` 7 个补丁执行 `git apply --check` | 全部 exit 0 |
| TC-1.2 | `npm run build:dsh` 重复执行 | 幂等：已应用则跳过，不报错 |

## TC-2 版本号一致性
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-2.1 | grep `0.1.2` / `dsh-v0.1.2-rc.1`（排除 node_modules/历史 spec） | 无遗留有效引用 |
| TC-2.2 | desktop `package.json` | `version = 0.1.5` |
| TC-2.3 | harmony `AppScope/app.json5` | `versionCode=1005`, `versionName=0.1.5` |

## TC-3 Desktop 基本功能
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-3.1 | 启动打包产物 | 窗口打开并加载 dsh Web UI |
| TC-3.2 | 设置 DeepSeek API KEY | 保存成功 |
| TC-3.3 | 创建工作区 | 工作区创建成功 |
| TC-3.4 | 发起一次对话 | 收到模型回复 |
| TC-3.5 | 窗口为常规 Windows 标题栏（系统 min/max/close） | frame:true，无自绘控制 |
| TC-3.6 | 最大化后重启应用 | 恢复为最大化状态 |
| TC-3.7 | 调整到自定义位置/尺寸后重启 | 恢复原位置与尺寸 |
| TC-3.8 | 按 F11 | 进入全屏；再按 F11 退出全屏 |
| TC-3.9 | 观察窗口顶部 | 无 Electron 菜单栏 |

## TC-4 Harmony 真机基本功能
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-4.1 | hdc 安装 + 启动 | 应用在真机启动，加载 dsh Web UI |
| TC-4.2 | 设置 DeepSeek API KEY | 保存成功 |
| TC-4.3 | 创建工作区 | 工作区创建成功 |
| TC-4.4 | 发起一次对话 | 收到模型回复 |
| TC-4.5 | 最大化后重启应用 | 恢复为最大化状态 |
| TC-4.6 | 调整到自定义位置/尺寸后重启 | 恢复原位置与尺寸 |
| TC-4.7 | 按 F11 | 进入全屏；再按 F11 退出全屏 |
| TC-4.8 | 观察窗口顶部 | 无 Electron 菜单栏 |

## TC-5 安全
| 用例 | 步骤 | 预期 |
|---|---|---|
| TC-5.1 | `git status` / `git diff --cached` 扫描 API KEY | 无命中 |
