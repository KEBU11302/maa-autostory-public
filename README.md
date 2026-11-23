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

## 限制与未验证范围

这不是“全章节无人值守通关”的保证：

- 仅在 Windows 10/11 x64 上开发；依赖 ADB、一个 16:9 的 Android 模拟器（截图归一化为 1280×720）和已登录的明日方舟。
- 固定编队、无演习入口、特殊难度（easy/tough/six_star）、多区域地图、剧情解锁、缺失地图资源等仍有适配缺口；目录规模不等于已验证通关数。
- 账号进度只来自模拟器截图；没扫描到不等于未通关，工具会保留 `unknown`/`unseen`。
- 设备离线、ADB 权限、模式或结算不明确、正式作战失败等会停止整轮；不保证自动恢复。
- MAA 定时任务必须关闭，也不能与 MAA 日常任务同时控制同一模拟器；工具会读取 MAA 配置并主动拒绝并发。
- 历史文档涉及不同 MAA 版本，不能据此声称新版 MAA 已验证。源码**没有全局 6.20.1 硬门槛**，只有少数分支做版本判断，见下节。

### 关于 MAA 版本

代码只在少数路径检查版本：作业文档 `minimum_required > 6.20.1` 会被拒绝（`autostory/jobs.py`）；技能升级和干员档案定位在非 6.20.1 上会跳过或报错（`autostory/skill_training.py`、`autostory/operator_details.py`）；旧失败记录迁移也只在 6.20.1 执行（`autostory/controller.py`）。其余主流程没有全局版本断言，但这不等于已经适配其他版本。

## 从源码运行

### 1. 目录布局

`autostory/common.py` 把 MAA 根目录解析为仓库目录的**父目录**（或 `AUTOSTORY_ROOT` 的父目录），再从 `MAA/Python` 导入 `asst`、从 MAA 根目录加载 DLL 和资源。最稳妥的布局如下；也可以设置 `AUTOSTORY_ROOT` 指向本仓库，但它的父目录仍必须是 MAA 根目录。

```text
<MAA 根目录>\
  MAA.exe
  Python\            # 含 asst 包
  resource\          # MAA 任务与模板资源
  AutoStory\         # 本仓库
```

### 2. 依赖

- Windows 10/11 x64、Python 3（历史构建使用 3.12；仓库未声明完整支持矩阵）。
- `python -m pip install -r requirements.txt`：`numpy`、`opencv-python`、`rapidocr-onnxruntime`。
- MAA 完整运行时：`Python/asst`、`MaaCore.dll`、`resource/`；版本注意事项见上节。
- ADB、Android 模拟器和已登录的明日方舟。
- 发行版WebView2 桌面窗口需要 Microsoft Edge WebView2 Runtime；Tk 回退窗口需要 Python 的 `tkinter`。构建 WebView host 另需 .NET Framework `csc.exe` 和 WebView2 SDK。

可在仓库目录创建隔离环境：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\Activate.ps1
```

### 3. 配置

```powershell
Copy-Item settings.example.json settings.json
```

按需修改 `device`（默认 `127.0.0.1:16384`）、`adb_path`（留空时读取 MAA 的 `config/gui.new.json`）、`port`（默认 `18741`）、每轮理智/次数上限和 OCR 后端。`settings.json` 已被 `.gitignore` 忽略；不要提交真实配置、ADB 密钥、截图或日志。`settings.example.json` 里的 `ocr_backend` 默认是 `directml`；没有 DML provider 时会回退 CPU，并在运行记录中写明原因，不会声称已使用 GPU。

### 4. 准备关卡目录数据

`data/catalogue.json` 未随源码提交；缺少它时，控制台没有完整关卡目录。生成脚本 [`build_catalogue.py`](build_catalogue.py) 需要：

- `research/gamedata/stage_table.json`
- `research/gamedata/zone_table.json`
- `research/gamedata/retro_table.json`
- MAA 运行时中的 `resource/Arknights-Tile-Pos/overview.json`

前三个文件来自脚本注明的 ArknightsGameData 数据源，需自行准备与目标客户端匹配的数据，并遵守来源条款。在仓库目录运行 `python build_catalogue.py` 后生成本地目录数据。脚本不会自动下载这些输入。

### 5. 启动

先运行本地界面；界面上的任务操作及 worker 命令会操作模拟器并可能消耗游戏资源。

```powershell
python server.py
```

用浏览器打开 `http://127.0.0.1:18741`（修改端口后使用对应地址）。也可运行 `python desktop.py`：有 WebView2 host 时使用该窗口，否则回退到 Tk 窗口。

准备好 MAA、关卡目录和设备后，也可通过命令行运行：

```powershell
python worker.py scan --chapter CW-EX
python worker.py run --chapter CW-EX
python worker.py run-all
```

根目录的 PowerShell/VBS 启动器面向历史打包安装布局，不能代替上述源码环境准备。

## 构建与打包

`python server.py` 只启动回环 HTTP 服务；页面和 API 不依赖桌面窗口或 WebView。实际执行任务时，它启动的 worker 仍需要 MAA 运行时、ADB 和设备。

### Python 桌面包

```powershell
python -m pip install pyinstaller
powershell -ExecutionPolicy Bypass -File packaging/build_app.ps1 -AppName App-0.9.44
```

`build_app.ps1` 调用 PyInstaller 和 `packaging/AutoStory.spec`。注意：spec 的版本资源来自 `packaging/version.txt`，该文件当前仍写着 0.5.0；历史 0.9.44 构建脚本使用 `packaging/build_fullcounter0944.py` 在 0.9.43 基础上替换模块并复制 `_internal`/`webview`，且依赖未随仓库提供的 `App-0.9.43` 基线。因此，干净 checkout 仅靠 `build_app.ps1` 不能保证复现 0.9.44 发行包；构建前需要自行核对版本元数据和随包资源。另外，frozen 模式把项目根解析为可执行文件父目录的父目录，`web/`、`assets/` 和 MAA 根仍在外层，构建产物不能当作可单独移动的单文件包。

### WebView2 host

```powershell
powershell -ExecutionPolicy Bypass -File packaging/build_webhost.ps1
```

`build_webhost.ps1` 直接调用 `csc.exe`，并硬编码本机 `C:\Program Files\Microsoft OfficePLUS\3.16.0.46159\addin` 下的 WebView2 DLL；该路径不存在时构建失败。脚本会校验 Microsoft 签名并输出到 `_tmp/webhost-090`。仓库不包含预编译的 `AutoStory.WebHost.exe`、`Microsoft.Web.WebView2.*.dll`、`WebView2Loader.dll`，详见 `webview/README.md`。

## 测试

```powershell
python -m unittest discover -s tests -v
```

测试使用 unittest；部分模块直接读取 `release/evidence/flow093x/...`、`tests/FIXTURES_MANIFEST.json`、`bad_assets` 等未随仓库提供的夹具，因此完整套件在当前 checkout 不能视为全绿。`FULLCOUNTER_0944.md`、`VALIDATION.md` 等是历史本地验证记录，其中的测试数字不是本 checkout 的当前结果，也不代表完整无人值守验收。提交前至少运行与改动相关的测试，并在 PR 中写明未运行的项。

## 问题反馈与贡献

通过 [Issues](https://github.com/VerticalSearch/maa-autostory-public/issues) 提交问题或功能建议。请附版本、环境、复现步骤和脱敏日志；提交改动前阅读 [贡献说明](CONTRIBUTING.md) 与 [隐私说明](PRIVACY.md)。

历史应用说明见 [README_APP.md](README_APP.md)，0.9.44 开发记录见 [FULLCOUNTER_0944.md](FULLCOUNTER_0944.md)。这些记录描述当时的本地环境，不是本次代码的重新验收结果。

## 许可与第三方权利

本仓库目前公开展示源码，**尚未添加开源许可证**。修改、再分发及商业使用的授权方式待维护者明确。

MAA、游戏素材、OCR 模型、WebView2 SDK 和作业数据等第三方内容遵循各自的许可与使用条款，本仓库不改变其授权条件。
