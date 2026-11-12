# MotoNav 1.85B 桌面模拟器（Windows x64）

在 Windows 桌面上运行 1.85B 圆屏固件的**界面代码**：鼠标操作、固定时钟复现动画、导出
360×360 面板 PNG。改完界面先在电脑上看结果，不必每次烧录。本包为免安装绿色版。

## 0. 下载

- [MotoNav-Simulator-1.85B-win64.zip](https://github.com/Byte-17/MotoNav-Simulator/releases/latest/download/MotoNav-Simulator-1.85B-win64.zip)
  （约 273 KB，含 exe、启动脚本、本说明、许可与构建信息）
- 校验：包内 `SHA256SUMS.txt` 逐文件核对；本页下载的 ZIP 的 SHA-256 为
  `a5cd2fcc3139804a049335af484d364c8877a5e844b1a6195e71013937b8a221`
- 环境：Windows 10 / 11 x64，**不需要**装 Python、Arduino、Visual Studio、MSYS2 或任何 DLL
- 历史版本与校验文件：[Releases](https://github.com/Byte-17/MotoNav-Simulator/releases)

下载后**解压整个 ZIP**，双击 `启动模拟器.bat`（或直接双击 `motonav-sim.exe`）。


## 1. 它能做什么，不能做什么

它链接的是固件仓库里的同一份 `libraries/GFX_Library_for_Arduino`（字体、抗锯齿、RGB565
转换一致）和同一份 `boards/185b/MotoNav/ui/` 生产绘图代码，所以布局、配色、手势、换页
动画与设备端一致。

它**不是**电路或指令集模拟器：不模拟 QSPI 时序、PSRAM、面板初始化、功耗和真实蓝牙射频，
不连接真实手机或地图服务。真实屏幕色彩、帧率、触摸芯片和手机蓝牙行为仍需实机验证。

## 2. 运行

解压整个 ZIP（不要只把 exe 拖出来），双击 `启动模拟器.bat`，或直接双击 `motonav-sim.exe`。

- 系统要求：Windows 10 / 11 x64。exe 已静态链接 C++ 运行库（libstdc++/libgcc/winpthread），
  运行时只依赖 Windows 自带的 UCRT、KERNEL32、USER32、GDI32，**不需要装 Visual Studio、
  MSYS2、clang、CMake 或 Python**。
- 中文文字（路名、天气地名、歌名、来电人）用本机 **Microsoft YaHei** 栅格化成 4 位灰度图，
  再走固件的 `handleBitmap()` 分包路径，与手机渲染文字图片发过来的方式一致。若系统缺这个
  字体，中文区域会退化为空图，换一台有该字体的 Windows 即可。
- 杀软首次运行时可能对未签名 exe 提示，属正常现象；可用包内 `SHA256SUMS.txt` 核对文件。

窗口左侧是圆屏画面（物理 360×360 像素，整数倍放大、不插值），右侧是模拟控制面板：

| 控件 | 作用 |
| --- | --- |
| `< scene` / `scene >` | 切换预设场景 |
| `reset scene` | 回到该场景的初始状态 |
| `pause` / `resume` | 冻结或恢复固定时钟（动画、长按、过期都停/走） |
| `live clock` | 改成跟随真实时间 |
| `+100 ms` / `+1000 ms` | 手动推进时间；动画和长按在相同时间点可复现 |
| `rotate 90` | 依次切 0°/90°/180°/270°；面板按生产 `PortScale` 重新采样 |
| `connect` / `disconnect` | 切换“手机已连接并验证”状态 |
| `zoom +` / `zoom -` | 显示放大 1×～4×（只影响窗口，不影响导出） |
| `screenshot` | 导出当前面板为 PNG |

快捷键：`space` 暂停，`n` 下一场景，`r` 重置，`s` 截图，`q` 退出。

点击前轮 / 后轮查看压力和温度详情，再点返回；从仪表向下拖动进入动力电池详情，点右侧箭头
查看电芯并翻页；左滑到天气、右滑到音乐，来电长按接听——走的都是固件同一套手势代码。

## 3. 命令行 / 无窗口

在包目录打开 cmd 或 PowerShell：

```bat
motonav-sim --list-scenes
motonav-sim --scene cells-demo --headless --frames 3 --screenshot cells.png --report
motonav-sim --scene nav-cruise --headless --frames 30 --time 4000 --screenshot out.png
motonav-sim --headless --frames 3 --drag 300,180,30,180 --report
```

| 参数 | 作用 |
| --- | --- |
| `--scene NAME` | 载入场景，默认 `dashboard-demo` |
| `--list-scenes` | 列出全部场景后退出 |
| `--screenshot FILE` | 导出 360×360 面板 PNG（无窗口边框、无桌面缩放） |
| `--headless` | 不建窗口；服务器 / 脚本里也能跑 |
| `--frames N` | 截图前先跑 N 帧 |
| `--time MS` | 固定时钟的起始毫秒 |
| `--live` | 跟随墙上时钟 |
| `--run` | 窗口打开期间让固定时钟继续走（每帧 33 ms） |
| `--scale N` | 窗口放大 1..4 倍，默认 2 |
| `--canvas FILE` | 另存 466×466 逻辑画布 PNG（与 360×360 面板分开标注） |
| `--colour-check` | 把黑/白/红/绿/蓝经 `PortScale` 后取回逐色核对，验证通道顺序 |
| `--rotation DEG` | 面板旋转 0..359；非 90° 整数倍走双线性路径 |
| `--tap X,Y` | 截图前注入一次触摸 |
| `--drag X0,Y0,X1,Y1` | 截图前注入一次拖动 |
| `--disconnect` | 无窗口时在点击后、拖动前模拟手机断连 |
| `--report` | 打印页面、数据来源与当前模型值（`key=value`），供脚本核对 |
| `--verbose` | 打印进度细节 |

坐标约定：命令行与鼠标输入都是**物理面板坐标**（0..359），内部用生产的
`PortScale::touch()` 映射到 466×466 逻辑坐标，不做第二次换算。

## 4. 场景

`dashboard-demo`、`battery-demo`、`cells-demo`、`home-connected`、`home-disconnected`、
`home-link-lost`、`home-pairing`、`weather-hourly`、`weather-daily`、`weather-alert`、
`weather-empty`、`music-playing`、`music-paused`、`music-no-phone`、`call-incoming`、
`arrive-destination`、`arrive-waypoint`、`nav-arrow`、`nav-cruise`、`nav-stale`。

场景把数据写入共享状态再调用生产页面绘制函数。电芯演示用合成帧走 BMS 解码与分页，不与
真实设备通信。演示数据一律标有 `SIM DATA`，不包含真实设备数据、实测参数或真实 BMS 协议。

## 5. 已知边界

- **导航页尚未迁移**：地图渲染、箭头导航页、手机协议解析仍在 `MotoNav.ino` 里，
  `nav-arrow` / `nav-cruise` / `nav-stale` 会在画面上直接写 `NAV PAGE NOT PORTED YET`。
  宁可写明缺口，也不显示空白帧让人误判成渲染 bug。
- 只运行 1.85B 页面，不模拟 1.75 板型。
- 桌面帧率与内存占用不代表 ESP32 的表现。
- “暂停”冻结固定时钟；固件里有阻塞式动画循环（如换页 `slideAnimate()`）会轮询
  `millis()`，为避免死循环，连续 16 次无推进的读取之后按忙等处理、每次前进 1 ms。
  正常静止画面不会触发，但严格意义上“暂停”不是绝对的零流逝。
- 告警阈值、持久化存储、真实传感器输入和手机联调仍受外部关卡限制，演示数据不代表
  这些功能已在硬件上配置或验证。

## 6. 源码与移植

本仓库只发布可执行包，**不含源码**。源码在 MotoNav 项目主仓库的
`motonav/firmware/simulator/`，与固件共用 `motonav/firmware/boards/185b/MotoNav/ui/` 和
`motonav/firmware/libraries/GFX_Library_for_Arduino/`；对应的提交号与编译器版本记在
`BUILD-INFO.txt`。构建由 Python 直接驱动原生 C++17 编译器（与固件宿主测试同一约定，
不需要 CMake/make/ninja/SDL2）：

```sh
python motonav/firmware/simulator/build.py          # 增量构建
python motonav/firmware/simulator/build.py --clean  # 全量重建
python motonav/firmware/simulator/build.py --run    # 构建后直接开窗口
```

编译器按顺序探测 `MOTONAV_CXX` → 仓库自带 `.tools/msys64/ucrt64/bin/clang++` →
`PATH` 上的 `clang++` / `g++`；也可 `--compiler <路径>` 指定。链接时 `-static` 把
libstdc++/libgcc/winpthread 静态链进去，产物即可脱离工具链运行。

平台接缝在 `host/`：`Arduino.h` / `Print.h` / `WString.h` / `HostRuntime.*` 提供桌面端
Arduino API 与可注入时钟，`SimUi.*` 实现生产代码的窄接口（画布、刷屏、触摸、文字图），
`SimDevice.*` 是场景层，`SimWindow.cpp` 是 Win32 窗口与控制面板，`Png.*` 自带 stored-deflate
PNG 导出（无第三方库）。要移植到别的固件，替换 `ui/` 那一层即可，`host/` 不需要动。

## 7. 许可

随包 `LICENSE.md` 为项目非商业使用条款：允许个人非商业用途的查看、编译、修改、自用与
免费分享（须保留条款与来源、标注修改），禁止任何商业使用与变相获利。第三方组件、字体、
商标、地图与天气数据不适用该条款，仍受各自许可约束。

构建来源、编译器版本与各文件 SHA-256 见 `BUILD-INFO.txt` 与 `SHA256SUMS.txt`。
