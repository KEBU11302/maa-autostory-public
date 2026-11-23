# MAA 自动推图助手（MAA AutoStory）

> 公开源码快照 · v0.9.44 · 非 MAA 官方项目 · 仓库：<https://github.com/VerticalSearch/maa-autostory-public>

MAA 自动推图助手是面向《明日方舟》的 Windows 本地辅助工具，通过 MAA 的 Python API 和 ADB 实现关卡扫描、作业匹配、演习及作战结果核验。这是独立社区项目，与 MAA 官方团队无隶属关系。

**源码已包含在本仓库中**：核心逻辑在 [`autostory/`](autostory/)，任务入口为 [`worker.py`](worker.py)。可点击 GitHub 页面上的 **Code → Download ZIP** 下载，或执行：

```powershell
git clone https://github.com/VerticalSearch/maa-autostory-public.git AutoStory
```

当前源码版本为 **0.9.44**。仓库提供源码与开发文档；运行需要另行准备 MAA、模拟器和本地数据。许可状态见文末。

## 源码导航

| 路径 | 作用 |
|---|---|
| [`autostory/`](autostory/) | 核心包：控制器、导航、OCR/视觉、作业、恢复、状态；`autostory/core.py` 封装 MaaCore 任务 |
| [`worker.py`](worker.py) | 命令行任务入口；获取任务锁、写状态、分发 scan/run 等动作 |
| [`server.py`](server.py) | 仅监听 `127.0.0.1` 的控制面；提供页面 API 并启动 worker |
| [`desktop.py`](desktop.py) | 统一启动入口：desktop / `--server` / `--worker` / 自检 |
| [`web/index.html`](web/index.html) | 本地控制台单页；由 `server.py` 注入进程级随机 token |
| [`tests/`](tests/) | unittest 回归；部分用例依赖未随仓库提供的夹具 |
| [`packaging/`](packaging/) | PyInstaller spec、构建/安装/审计脚本、C# WebView2 host |

最短阅读路径：`worker.py` → `autostory/controller.py` → `autostory/navigation.py` → `autostory/core.py`；控制面从 `server.py` 和 `desktop.py` 进入。

## 能做什么

- **关卡扫描与断点续跑**：读取普通、突袭等难度进度并逐关保存；中断后可从检查点继续，未识别状态保留为未知。
- **作业匹配与执行**：从公开作业源获取作业，按关卡、难度、干员与练度筛选；按配置先演习、再正式作战，并回读结算和地图标记；0 理智且无演习入口的关卡走单独分支。
- **干员与练度核对**：通过 MAA `OperBox` 刷新干员并写回 MAA 快照；对候选干员补读技能/专精信息。
- **失败记录与换作业**：按作业内容与实际队伍记录失败；续跑时跳过相同条件，预算或候选耗尽时停止/跳过。
- **本地控制面**：网页或桌面窗口显示进度、日志和停止按钮；服务只绑定回环地址。
- **可选 GPU OCR**：配置可请求 DirectML；运行库缺少 DML provider 时会回退 CPU 并记录原因。

## 使用与文档

| 需要做什么 | 入口 |
| --- | --- |
| 准备环境、配置设备、生成关卡目录、从源码启动 | [使用指南](docs/USAGE.md) |
| 查看构建方法、兼容性限制和测试要求 | [使用与开发指南](docs/USAGE.md) |
| 反馈问题、提交改动、处理日志与隐私 | [贡献说明](CONTRIBUTING.md) |
| 查阅旧版本开发与实机验证记录 | [历史记录](CHANGELOG.md)（按原文件名折叠） |

程序依赖 Windows、MAA 运行时、ADB 和模拟器，关卡数据需自行准备。当前源码不等于完整安装包，也不保证全章节无人值守通关；具体限制见使用指南。

通过 [Issues](https://github.com/VerticalSearch/maa-autostory-public/issues) 反馈问题或建议，请附版本、环境、复现步骤和脱敏日志。

## 许可与第三方权利

本仓库目前公开展示源码，**尚未添加开源许可证**。修改、再分发及商业使用的授权方式待维护者明确。

MAA、游戏素材、OCR 模型、WebView2 SDK 和作业数据等第三方内容遵循各自的许可与使用条款，本仓库不改变其授权条件。
