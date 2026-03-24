我来给你讲清楚，这些包在 Linux 下编译 Qt 时，每个都有明确作用，不安装就会报错或缺功能。

------

# 一、编译工具类（必装）

| 包名              | 作用                                                   |
| ----------------- | ------------------------------------------------------ |
| `build-essential` | 提供基本编译工具（gcc、g++、make 等），Qt 编译核心依赖 |
| `perl`            | Qt 的 configure 脚本和一些工具依赖 Perl 脚本           |
| `python3`         | Qt 5 的某些工具和脚本（尤其是 QtWebEngine）需要 Python |

> 没这些，根本无法编译源码。

------

# 二、核心库和开发头文件（头文件 + 链接库）

| 包名                 | 作用                                                         |
| -------------------- | ------------------------------------------------------------ |
| `libglib2.0-dev`     | 提供 GLib 基础库，Qt 依赖一些 GLib 功能                      |
| `libx11-dev`         | X11 核心库头文件，Qt GUI 需要                                |
| `libxcb1-dev`        | XCB（X protocol C-language Binding）库，用于 Qt GUI、窗口管理 |
| `libx11-xcb-dev`     | X11 与 XCB 连接桥接，确保 Qt 可以正确使用 X11                |
| `libxext-dev`        | X11 扩展库，常用功能                                         |
| `libxfixes-dev`      | 修复和扩展 X11 功能，用于高级 GUI                            |
| `libxi-dev`          | X Input extension，鼠标键盘事件支持                          |
| `libxrender-dev`     | X Render 扩展，绘图和字体渲染用                              |
| `libxrandr-dev`      | X Resize, Rotate & Reflect 扩展，Qt 窗口旋转、缩放用         |
| `libfontconfig1-dev` | 字体管理库，Qt 渲染文本必备                                  |

------

# 三、总结一句话

这些东西分两类：

1. **编译工具** → gcc、make、perl、python
2. **GUI/系统库** → X11/XCB、字体、GL、窗口扩展

> 少装一个，你就会在 `./configure` 或 `make` 阶段报错：“找不到 xxx 库” 或 “header file missing”。

------

# 检查安装

# 一、最直接：逐个检查（推荐）

用 `dpkg`：

```
dpkg -s build-essential
```

如果已安装，会看到：

```
Status: install ok installed
```

------

## 批量检查（更实用）

```
dpkg -l | grep -E "build-essential|perl|python3|libglib2.0-dev|libx11-dev|libxcb1-dev|libx11-xcb-dev|libxext-dev|libxfixes-dev|libxi-dev|libxrender-dev|libxrandr-dev|libfontconfig1-dev"
```

------

# 二、看状态标志（重点）

输出类似：

```
ii  libx11-dev     ...
```

👉 含义：

| 标志 | 含义             |
| ---- | ---------------- |
| ii   | 已安装 ✅         |
| rc   | 已删除但残留配置 |
| un   | 未安装           |

# 核心流程

你现在已经走到**正确路径**了：用 `Qt-everywhere-src-5.12.2` 自己编译 Qt。
下面直接给你一套**可落地的生成流程**（按嵌入式/控制器场景，尽量避坑）。

------

# 一、解压源码

```bash
 -
cd qt-everywhere-src-5.12.2
```

------

# 二、安装依赖（必须）

在 Debian 下：

```bash
sudo apt update
sudo apt install -y \
build-essential \
perl \
python3 \
git \
libglib2.0-dev \
libsm-dev \
libxrender-dev \
libfontconfig1-dev \
libx11-dev \
libxcb1-dev \
libx11-xcb-dev \
libxext-dev \
libxfixes-dev \
libxi-dev \
libxrandr-dev
```

------

# 三、配置（configure）【核心步骤】

⚠️ 这一步决定你后面是否成功

------

## 1. 最小化配置（推荐你先用这个）

```bash
./configure \
-prefix /opt/qt5.12.2 \
-release \
-opensource \
-confirm-license \
-nomake examples \
-nomake tests \
-skip qtwebengine \
-skip qt3d \
-skip qtgamepad \
-skip qtlocation \
-skip qtscript \
-skip qtwayland
```

------

或者

```
./configure \
-prefix /opt/qt5.12.2 \
-release \
-opensource \
-confirm-license \
-xcb \                  # 使用 X11
-opengl es2 \            # ARM 上常用 OpenGL ES2
-nomake examples \       # 不编译示例，节省时间
-nomake tests \          # 不编译测试
-skip qtwebengine \      # 跳过 webengine
-skip qt3d \             # 跳过 3D
-skip qtgamepad          # 跳过游戏手柄模块
```



## 2. 如果你是“无GUI设备”（很常见）

👉 用 framebuffer：

```bash
./configure \
-prefix /opt/qt5.12.2 \
-release \
-opensource \
-confirm-license \
-nomake examples \
-nomake tests \
-skip qtwebengine \
-skip qt3d \
-no-opengl \
-linuxfb
```

------

## 3. 如果你有X11桌面

X11 桌面启动时，会设置环境变量 `DISPLAY`：

```
echo $DISPLAY
```

- 如果输出类似 `:0` 或 `:0.0` → 有 X11
- 如果没有输出 → 很可能是无头（headless）环境，没有 X11

------

### 二、检查 X11 相关进程

```
ps aux | grep X
```

- 正常桌面环境会有类似 `Xorg` 或 `Xwayland` 进程
- 没有这些 → 说明没有 X11

------

### 三、检查 X11 库是否安装

```
dpkg -l | grep libx11
```

- 如果看到 `libx11-dev` 等 → 已安装 X11 库
- 但是有库不等于运行环境，有可能只是头文件存在

如果已经有了 加上：

```bash
-xcb
```

------

# 四、开始编译

```bash
make -j4
```

👉 `-j4` 根据你CPU核数调整：

- 2核 → `-j2`
- 4核 → `-j4`

------

# 五、安装

```bash
sudo make install
```

默认会安装到：

```bash
/opt/qt5.12.2
```

------

# 六、配置环境变量

```bash
vim ~/.bashrc
```

添加：

```bash
export QTDIR=/opt/qt5.12.2
export PATH=$QTDIR/bin:$PATH
export LD_LIBRARY_PATH=$QTDIR/lib:$LD_LIBRARY_PATH
```

生效：

```bash
source ~/.bashrc
```

------

# 七、验证

```bash
qmake -v
```

正常输出：

```
QMake version 3.x
Using Qt version 5.12.2
```

------

# 八、你这个阶段最容易踩的坑

------

## 1. configure 失败

常见报错：

```
ERROR: XCB not found
```

👉 解决：

```bash
sudo apt install libxcb-xinerama0
```

------

## 2. 编译卡死 / 超慢

👉 原因：

- qtwebengine 没关（非常大）

✔️ 已帮你在配置里 `-skip` 掉了

------

## 3. 内存不够

👉 表现：

```
Killed
```

解决：

```bash
make -j1
```

------

## 4. 权限问题

```bash
Permission denied
```

👉 用：

```bash
sudo make install
```

------

# 九、进阶（你后面一定会用到）

------

## 1. 精简 Qt（工业项目推荐）

你其实常用：

- QtCore
- QtGui
- QtWidgets

👉 可以进一步裁剪（减少体积）

------

## 2. 结合 OpenCV

后面你可以这样用：

```cpp
#include <opencv2/opencv.hpp>
#include <QImage>
```

👉 Qt + OpenCV 在这个环境是完全OK的

------

## 十、给你一个更工程化建议（重点）

你现在是：

- 工控 / 控制器
- Qt + C++

👉 推荐流程：

1. PC（x86）开发
2. 控制器上部署 Qt（你现在做的）
3. 用 scp / rsync 部署程序

------

# 总结一句

你现在只需要执行三步：

```bash
./configure ...
make -j4
sudo make install
```

------

如果你下一步想优化（建议你做）：

我可以帮你：

- 定制一份 **最小 Qt 编译参数（专门适配你板卡）**
- 或者帮你做 **交叉编译方案（比现在快10倍）**

你可以把这个发我：

```bash
uname -m
free -h
```

我可以帮你把编译参数“收敛到最优”。