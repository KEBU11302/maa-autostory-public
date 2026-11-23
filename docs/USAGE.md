# 使用与开发指南

[返回项目首页](../README.md) · [贡献说明](../CONTRIBUTING.md)

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

下列命令均在仓库根目录（`AutoStory`）执行。

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

`data/catalogue.json` 未随源码提交；缺少它时，控制台没有完整关卡目录。生成脚本 [`build_catalogue.py`](../build_catalogue.py) 需要：

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

测试使用 unittest；部分模块直接读取 `release/evidence/flow093x/...`、`tests/FIXTURES_MANIFEST.json`、`bad_assets` 等未随仓库提供的夹具，因此完整套件在当前 checkout 不能视为全绿。[历史开发与验证记录](../CHANGELOG.md) 等是历史本地验证记录，其中的测试数字不是本 checkout 的当前结果，也不代表完整无人值守验收。提交前至少运行与改动相关的测试，并在 PR 中写明未运行的项。
