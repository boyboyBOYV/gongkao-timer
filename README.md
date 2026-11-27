# 公考计时器 · 本地版（v1.4）

纯本地运行的考公刷题计时器：分模块计时、套题命名、历史用时/得分折线、日间/夜间主题。
内置行测、申论、公安三种模板，其中公安卷不分模块、整卷一条计时。
不联网、不注册、不上传数据。

![计时页](docs/screenshots/timer-light.png)

![公安整卷计时](docs/screenshots/police-timer.png)

## 下载即用（Windows 10/11）

到 Releases 页面下载：

- `公考计时器-单文件版.exe`：一个文件，双击就跑，适合直接放桌面。
- `公考计时器-快速版.zip`：解压后运行 `公考计时器\公考计时器.exe`，启动更快；整个文件夹要一起保留。

程序不写注册表，数据只存在 `%APPDATA%\GongkaoTimer\data.json`，删掉这个文件就恢复初始状态。

## 运行

环境：Windows 10/11，Python 3.13（自带 tkinter）。

```powershell
cd <项目根>
python source\gk_timer.py
```

数据保存在 `%APPDATA%\GongkaoTimer\data.json`；删除该文件即可恢复初始状态。

## v1.4 功能

- 新增「公安」模板：单模块整卷计时（默认 120 分钟），计时页不出现「下一模块」，记录与趋势按整卷一条处理。
- 三种模板可在计时页一键切换；小时/分钟/秒目标都可逐块调整，记录页可修正实际用时。
- 记录页筛选与折线支持「全部 / 行测 / 申论 / 公安」；单模块记录只画总用时（或总分）一条曲线。
- 页面切换为新旧两页联动的横移动画（约 0.24 秒）；模块行、按钮、导航胶囊、进度条和 Toast 都有 eased 过渡。
- 页头不放自绘的「最小化」按钮；需要收进任务栏时使用 Windows 标题栏的最小化，计时继续在后台累计。
- 分模块模板固定按第 1 段开始，点「下一模块」顺序推进；模块行不支持从中间开始，也不能拖动排序。
- 记录页每块可录入得分，列表和详情显示总分；折线图可在「用时 / 得分」之间切换，得分模式统计有分套数、平均分和最高分。
- 右上角「日间 / 夜间」即时切换，不重置正在跑的计时；选择写入 `ui.theme`。
- 套题命名、分模块/总用时、CSV 导出（含总分和每块得分）、重命名/删除、单实例运行。

## 截图

| 记录页（日间） | 设置页（日间） |
| --- | --- |
| ![记录页](docs/screenshots/records-light.png) | ![设置页](docs/screenshots/settings-light.png) |

| 计时页（夜间） | 调整目标 |
| --- | --- |
| ![夜间](docs/screenshots/timer-dark.png) | ![调整目标](docs/screenshots/dialog-target.png) |

## 自测

```powershell
python tools\selftest.py
python tools\check_animations.py
python tools\check_drag_events.py
python tools\check_dialogs.py
```

`selftest.py` 覆盖数据、计时、保存、记录、主题、顺序推进、禁止拖动、得分与得分图表，以及公安整卷模板；`check_animations.py` 验证联动切页、导航动画、主题/页头 hover 和最小化/恢复；`check_drag_events.py` 验证计时页拖动不会改变顺序；`check_dialogs.py` 验证目标编辑（含公安整卷）和记录数据编辑弹窗的解析与保存；`check_preview_police.py` 生成公安模板演示截图。

## 打包（PyInstaller）

需要 PyInstaller 6.x。下面以本机 Python 3.13 为例，`$py` 换成实际解释器路径：

```powershell
$proj = "<项目根>"
$py = "<Python 3.13 路径>\python.exe"
$base = & $py -c "import sys; print(sys.base_prefix)"
$tcl = Join-Path $base "tcl\cl8.6"
$tk = Join-Path $base "tcl\tk8.6"

# 单文件版
& $py -m PyInstaller --noconfirm --clean --onefile --windowed `
  --name "公考计时器" --icon "$proj\app_icon.ico" `
  --manifest "$proj\app.manifest" `
  --add-data "$proj\app_icon.ico;." `
  --add-data "$tcl;tcl/tcl8.6" --add-data "$tk;tcl/tk8.6" `
  --distpath "$proj\dist_v14" --workpath "$proj\build_v14" `
  --specpath "$proj\build_v14" "$proj\source\gk_timer.py"

# 快速版（onedir，启动更快，推荐交付）
& $py -m PyInstaller --noconfirm --clean --onedir --windowed `
  --name "公考计时器" --icon "$proj\app_icon.ico" `
  --manifest "$proj\app.manifest" `
  --add-data "$proj\app_icon.ico;." `
  --add-data "$tcl;tcl/tcl8.6" --add-data "$tk;tcl/tk8.6" `
  --distpath "$proj\dist_fast_v14" --workpath "$proj\build_fast_v14" `
  --specpath "$proj\build_fast_v14" "$proj\source\gk_timer.py"
```

打包后的快速版必须保留整个 `公考计时器` 文件夹（含 `_internal`），不能只移动 exe。

## 结构

```text
source\
  gk_timer.py     入口、窗口、页头、页面切换、主题初始化
  timer_page.py   计时页：模块、顺序计时、公安整卷模式、目标编辑、开始/暂停/结束
  records_page.py 记录页：折线图、套题明细、用时/得分编辑、CSV
  settings_page.py 设置页：模板、模块编辑、数据工具
  widgets.py      自绘控件：按钮、输入框、卡片、折线图等
  theme.py        日/夜色板与运行时换色
  store.py        本地 JSON 读写
app.manifest      Windows DPI 感知清单（避免打包版右侧被裁）
app_icon.ico      程序图标
build_exe.ps1     一键打包单文件版 / 快速版
tools\
  selftest.py     无窗口自测
  check_animations.py 页面动效/最小化冒烟测试
  check_drag_events.py 拖拽事件冒烟测试
  check_dialogs.py 目标与记录编辑弹窗测试
  check_preview_police.py 公安模板演示数据与截图
  check_packaged.py 已打包 exe 的窗口截图冒烟测试
```

## 数据字段

`data.json` 顶层包含 `preset`、`total_target_min`、`modules`、`runs`、`ui`。
`preset` 可为 `行测` / `申论` / `公安`；公安只有 1 个模块 `公安专业科目`，且 `total_target_min` 与该模块目标一致。
每条 `runs` 记录包含 `name`、`preset`、`date`、`total`、`modules`（每段 `name/seconds/target/score`）。
`score` 可为 `None`（未录分）；`total` 在编辑实际用时后按模块用时重新求和。
`ui.theme` 为 `dark` 或 `light`。

## 说明

个人备考自用项目，代码可自由参考；转载或二次分发请注明出处（boyboyBOYV）。
