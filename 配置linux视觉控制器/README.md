# Linux Qt 5.15.5 aarch64 编译指南

明白，我帮你整理一张 **针对 Qt 5.15.5 在 aarch64 Linux 上部署、可运行 X11 + xcb 的最小依赖开发包表格**。表格列出了包名、用途/功能，以及是否你已经安装（根据你提供的信息）。这样你在编译 Qt Everywhere 前就可以保证所有依赖到位。

| 序号 | 包名                      | 功能 / 作用                             | 安装状态 |
| ---- | ------------------------- | --------------------------------------- | -------- |
| 1    | `libxcb1-dev`             | X C Binding 核心开发库，Qt xcb 插件依赖 | 已安装   |
| 2    | `libx11-dev`              | X11 核心开发库，Qt GUI 基础             | 已安装   |
| 3    | `libx11-xcb-dev`          | 让 X11 与 XCB 通信的接口开发库          | 已安装   |
| 4    | `libxcb-render0-dev`      | XCB 渲染扩展库开发文件                  | 已安装   |
| 5    | `libxkbcommon-dev`        | xkbcommon 核心开发库，键盘映射          | 已安装   |
| 6    | `libxcb-xinerama0-dev`    | Xinerama 扩展，用于多显示器支持         | 未安装   |
| 7    | `libxcb-icccm4-dev`       | ICCCM 协议支持（窗口管理器）            | 未安装   |
| 8    | `libxcb-image0-dev`       | XCB 图像扩展开发库                      | 未安装   |
| 9    | `libxcb-keysyms1-dev`     | 键盘符号映射支持                        | 未安装   |
| 10   | `libxcb-randr0-dev`       | RandR 扩展，用于动态屏幕尺寸            | 未安装   |
| 11   | `libxcb-render-util0-dev` | 渲染实用函数开发库                      | 未安装   |
| 12   | `libxcb-shape0-dev`       | 窗口形状扩展支持                        | 未安装   |
| 13   | `libxcb-shm0-dev`         | 共享内存扩展开发库                      | 未安装   |
| 14   | `libxcb-sync0-dev`        | 同步扩展支持                            | 未安装   |
| 15   | `libxcb-xfixes0-dev`      | XFixes 扩展，用于窗口修正               | 未安装   |
| 16   | `libxcb-xkb-dev`          | XKB 扩展接口                            | 未安装   |
| 17   | `libxkbcommon-x11-dev`    | xkbcommon 与 X11 绑定头文件             | 未安装   |
| 18   | `libfontconfig1-dev`      | 字体配置库，用于渲染文本                | 建议安装 |
| 19   | `libfreetype6-dev`        | 字体渲染库                              | 建议安装 |
| 20   | `libglu1-mesa-dev`        | OpenGL ES2 / GL 支持                    | 建议安装 |
| 21   | `libgl1-mesa-dev`         | OpenGL 核心开发库                       | 建议安装 |
| 22   | `libxext-dev`             | X11 扩展库，Qt GUI 常用                 | 建议安装 |
| 23   | `libxrender-dev`          | X11 渲染扩展                            | 建议安装 |
| 24   | `libxcb-glx0-dev`         | XCB GL 扩展，用于 OpenGL 渲染           | 建议安装 |

------

### 说明

1. **前 17 个包**是编译 Qt xcb 插件的 **绝对必须开发包**，否则 configure 会报错。
2. **18-24 包**是 Qt GUI 最小功能运行所需的支持库，用于字体、OpenGL、渲染等，强烈建议安装。
3. 已安装/未安装状态是基于你之前的 `dpkg -s` 和 `ls` 信息标注的。
4. 安装命令示例：

```bash
sudo apt update
sudo apt install -y \
libxcb1-dev libx11-dev libx11-xcb-dev libxcb-render0-dev libxkbcommon-dev \
libxcb-xinerama0-dev libxcb-icccm4-dev libxcb-image0-dev \
libxcb-keysyms1-dev libxcb-randr0-dev libxcb-render-util0-dev \
libxcb-shape0-dev libxcb-shm0-dev libxcb-sync0-dev libxcb-xfixes0-dev \
libxcb-xkb-dev libxkbcommon-x11-dev \
libfontconfig1-dev libfreetype6-dev libglu1-mesa-dev libgl1-mesa-dev \
libxext-dev libxrender-dev libxcb-glx0-dev
```

------

如果你愿意，我可以帮你 **在这个表格基础上，写一个“一条命令安装 + Qt configure 最小化部署命令”完整脚本**，保证可以直接编译出带 xcb 的可部署 Qt。

你希望我直接写吗？

## 二、Qt 5.15.5 aarch64 最小化编译

### 1. 核心模块

- QtCore、QtGui、QtWidgets（基础库和 GUI）
- QtNetwork（网络模块，可选）
- qtserialbus / qtserialport（工业串口/CAN 功能）
- QtTools（moc、uic、rcc 等工具）
- QtTranslations（国际化支持）

### 2. 可跳过模块（节省时间和空间）

- QtQuick/QML 模块：qtdeclarative、qtquickcontrols、qtquickcontrols2、qtquick3d
- Web 相关：qtwebengine、qtwebview、qtwebchannel、qtwebsockets、qtwebglplugin
- 多媒体：qtmultimedia、qtspeech
- 其他：qt3d、qtcharts、qtconnectivity、qtdatavis3d、qtgamepad、qtgraphicaleffects、qtvirtualkeyboard、qtwinextras、qtx11extras、qtlottie、qtremotobjects、qtxmlpatterns

------

### 3. PCH（预编译头文件）

- 用途：加速编译，将公共头文件预编译成二进制缓存
- 优点：编译速度快
- 缺点：占用磁盘空间大，交叉编译或小磁盘容易报错
- Qt 交叉编译建议用 `-no-pch` 禁用

------

### 4. configure 命令

你的这条 `configure` 命令基本上是针对 **最小化、可部署 Qt GUI** 的合理配置，但有几个细节需要注意：

------

#### 分析你的配置

```text
./configure -release \
-prefix /mnt/ssd/opt/qt5.15.5 \
-opensource -confirm-license \
-nomake examples -nomake tests \
-xcb -xcb-xlib -bundled-xcb-xinput \
-opengl es2 \
-dbus-runtime \
-skip qtwebengine -skip qt3d -skip qtcanvas3d -skip qtwayland -skip qtgamepad -skip qtvirtualkeyboard
```

1. **-release**：只编译 release 版本，占用空间小 ✅
2. **-prefix /mnt/ssd/opt/qt5.15.5**：安装路径在你的 SSD 上 ✅
3. **-opensource -confirm-license**：开源版本自动确认 ✅
4. **-nomake examples -nomake tests**：不编译示例和测试，节省空间 ✅
5. **-xcb -xcb-xlib -bundled-xcb-xinput**：启用 xcb 插件，支持 X11 GUI ✅
   - 前提是 **你系统必须安装完整 xcb / X11 / xkbcommon 开发包**
6. **-opengl es2**：启用 OpenGL ES2 支持，适合嵌入式 ARM / aarch64 ✅
7. **-dbus-runtime**：启用运行时 D-Bus 支持 ✅
8. **-skip qtwebengine -skip qt3d -skip qtcanvas3d -skip qtwayland -skip qtgamepad -skip qtvirtualkeyboard**
   - 这些模块体积大，如果不需要可以跳过 ✅

------

#### 潜在问题或优化建议

1. **xkbcommon-x11**：确保已经安装 `libxkbcommon-x11-dev`，否则 configure 还是会报 `xcb precondition failed`。
2. **字体 & 渲染支持**：建议至少安装 `libfontconfig1-dev`、`libfreetype6-dev`、`libxrender-dev`，否则 Qt GUI 可能无法正确显示文字或渲染。
3. **GL / Mesa**：如果要使用 OpenGL 功能，需要 `libglu1-mesa-dev`、`libgl1-mesa-dev`。
4. **sysroot**：你没有指定 `-sysroot`，如果交叉编译或者使用 chroot/rootfs，可能需要，否则本机编译可以省略。

------

#### 总结

- 如果你已经安装了前面列出的 xcb / X11 / xkbcommon / font / OpenGL 开发包，这条命令可以直接用来编译 **最小化 Qt GUI + xcb 插件**，并部署到 `/mnt/ssd/opt/qt5.15.5`。
- 跳过模块和 `-nomake` 设置可以节省大量空间。
- 编译完成后，使用 `linuxdeployqt` 或者手动复制 `/mnt/ssd/opt/qt5.15.5/lib` 下的 `.so` 和 `platforms/libqxcb.so` 就可以部署运行。

------

### 5. 编译与安装

```bash
gmake -j$(nproc)
sudo gmake install
```

- 安装路径：`/opt/aarch64/qt5.15.5`
- 配置环境变量：

```bash
export PATH=/opt/aarch64/qt5.15.5/bin:$PATH
export LD_LIBRARY_PATH=/opt/aarch64/qt5.15.5/lib:$LD_LIBRARY_PATH
```

- 验证安装：

```bash
qmake -v
```

------

### 6. 交叉编译器

- RK3588 CPU 是 aarch64，通用 Linaro 工具链即可
- 官方 BSP 也自带交叉编译器，适合驱动/板卡开发
- Ubuntu/Debian 安装示例：

```bash
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

- 迁移工具链时需完整目录，并设置 PATH / LD_LIBRARY_PATH

------

### 7. 常见报错

| 报错                                             | 原因               | 解决                         |
| ------------------------------------------------ | ------------------ | ---------------------------- |
| `-skip qtbluetooth` / `qtremotobjects`           | 拼写或大小写错误   | 改为源码目录实际名称         |
| `cannot write PCH file: No space left on device` | 磁盘空间不足       | 删除 `.pch` 或使用 `-no-pch` |
| `Invalid command line parameter '-prefix'`       | 行续行符或空格错误 | 删除行尾空格或写成一行命令   |

------

### 8. 清理与重新编译

```bash
cd ~/qt-build-aarch64
gmake clean
rm -rf .pch
sudo rm -rf /opt/aarch64/qt5.15.5
sudo mkdir -p /opt/aarch64/qt5.15.5
sudo chown $USER:$USER /opt/aarch64/qt5.15.5
```

然后重新执行 configure、gmake 和 gmake install。

------

### 9. 最小化编译策略

- 只保留核心模块和工业相关模块
- 禁用 PCH
- 跳过示例、测试和不需要模块
- 减少编译时间和磁盘占用

------

### 10. 总结

1. 检查 Linux 编译依赖
2. 清理旧编译和安装目录
3. configure 时只保留需要模块，禁用 PCH
4. gmake 编译并 sudo gmake install 安装
5. 配置环境变量，验证安装
6. 交叉编译器可用通用 aarch64 工具链

> 这样就能在 aarch64 Linux 下干净、高效地编译 Qt 5.15.5，并避免常见报错。

# 检查qmake是不是aarc64的

可以用几种方法验证你生成的 `qmake` 是否针对 **Linux aarch64 g++ 交叉编译器**。核心思路就是查看 `qmake` 内置的 mkspec 和编译器信息。

------

### 方法 1：使用 `qmake -query`

```bash
/path/to/qmake -query
```

示例输出（关键部分）：

```text
QT_SYSROOT:
QT_INSTALL_PREFIX:/opt/aarch64/qt5.15.5
QMAKE_SPEC:linux-aarch64-gnu-g++
QMAKE_CC:aarch64-linux-gnu-gcc
QMAKE_CXX:aarch64-linux-gnu-g++
```

- `QMAKE_SPEC` → 当前使用的 mkspec（编译平台配置）
- `QMAKE_CC` / `QMAKE_CXX` → 指向 aarch64 交叉编译器
- `QT_INSTALL_PREFIX` → 安装路径（确认 configure 时指定的 `-prefix`）

如果 `QMAKE_SPEC` 是 `linux-aarch64-gnu-g++`，就说明这个 qmake **是 aarch64 Linux 交叉编译版本**。

------

### 方法 2：查看 qmake 的内置信息

```bash
/path/to/qmake -v
```

示例输出：

```text
QMake version 3.1
Using Qt version 5.15.5 in /opt/aarch64/qt5.15.5/lib
```

- 不能直接显示 target 架构，但结合 `-query QMAKE_SPEC` 可以判断。

------

### 方法 3：生成测试 Makefile 验证

1. 新建一个简单 Qt 项目目录：

```bash
mkdir ~/test_qmake
cd ~/test_qmake
```

1. 创建 `test.pro`：

```text
QT       += core
CONFIG   += console
TARGET   = test
SOURCES  = main.cpp
```

1. 创建 `main.cpp`：

```cpp
#include <QCoreApplication>
int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);
    return 0;
}
```

1. 生成 Makefile：

```bash
/path/to/qmake test.pro
cat Makefile | grep -E "CXX|CC"
```

- 你会看到 Makefile 里 `CXX = aarch64-linux-gnu-g++`
- 说明 qmake 生成的 Makefile **针对 aarch64 交叉编译器**

------

### 总结

- **最简单**：执行

```bash
/path/to/qmake -query QMAKE_SPEC
/path/to/qmake -query QMAKE_CC
/path/to/qmake -query QMAKE_CXX
```

- 如果输出 `linux-aarch64-gnu-g++` 和 `/usr/bin/aarch64-linux-gnu-g++` → qmake 正确
- 也可以通过生成一个简单项目 Makefile 再验证一次

------

我可以帮你写一个 **一条命令就能验证 qmake 是否是 aarch64 交叉版本** 的脚本，直接打印结果，方便确认。

你希望我写吗？

# 验证库文件是否是aarc64

基本可以这么理解，但要注意区分 **qmake 本身的 target 配置** 和 **生成的库文件**：

------

### 1. qmake 的作用

- `qmake` 是 Qt 的构建工具，它 **生成 Makefile**，Makefile 里指定了：
  - 使用的 C/C++ 编译器（`QMAKE_CC` / `QMAKE_CXX`）
  - 编译器选项（target 架构、ABI）
  - 链接选项、库路径等
- 所以，如果 qmake 是针对 `linux-aarch64-gnu-g++` 的，那么用它生成 Makefile 编译出来的 Qt 库 **也会是 aarch64 架构的**。

------

### 2. 验证库文件本身

即使 qmake 配置正确，你也可以额外验证编译后的 `.so` 文件：

```bash
file /opt/aarch64/qt5.15.5/lib/libQt5Core.so
```

输出示例：

```text
libQt5Core.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, ...
```

- 看到 `ARM aarch64` → 确认是 aarch64 架构
- 这比单纯看 qmake 更直接可靠

------

### 3. 总结

- **qmake 配置**：告诉你生成 Makefile 的交叉编译目标（aarch64）
- **库文件验证**：确认最终生成的 `.so` 文件确实是 aarch64
- 两者结合 → 可以完全确认你的 Qt 库是 aarch64 的

------

如果你愿意，我可以帮你写一个 **快速命令脚本**，一次性检查 qmake 配置 + 库文件架构，直接告诉你这个 Qt 是否是 aarch64。

你希望我写吗？

# gmake的文件在哪里 

明白了，你现在有三个目录，我们来理清它们各自的作用：

------

## 1. `qt-everywhere-src-5.15.5`

- **这是 Qt 的源码目录**，你解压的原始源码
- 里面包含所有模块文件夹（qtbase、qtcharts、qtdeclarative…）
- 你**不应该直接在这里执行 gmake install**，因为这里是源码，只编译生成中间文件

------

## 2. `~/qt-build-aarch64`

- **这是你的构建目录**（build directory）
- 你在这里执行 configure + gmake 的地方
- 编译生成的 **中间文件**（`.o`、`.gch`、临时工具）都在这里
- **还没有安装到系统路径**，需要执行 `sudo gmake install` 才会安装
- 这是你主要操作的目录，进行编译、清理（gmake clean）、配置参数都在这里

------

## 3. `/opt/aarch64/qt5.15.5`

- **这是安装目录**，你 configure 里的 `-prefix` 指向它
- 执行 `sudo gmake install` 后，Qt 的库文件、头文件、工具会被安装到这里
  - `/opt/aarch64/qt5.15.5/bin` → 可执行工具（qmake、moc、uic）
  - `/opt/aarch64/qt5.15.5/lib` → 库文件
  - `/opt/aarch64/qt5.15.5/include` → 头文件

------

### 总结：

| 目录                       | 用途                                           |
| -------------------------- | ---------------------------------------------- |
| `qt-everywhere-src-5.15.5` | Qt 源码，不直接编译安装                        |
| `~/qt-build-aarch64`       | 构建目录，执行 configure + gmake 生成中间文件  |
| `/opt/aarch64/qt5.15.5`    | 安装目录，执行 `sudo gmake install` 后正式使用 |

------

所以**你当前编译、生成文件都在 `~/qt-build-aarch64`**，最终可用 Qt 是在 `/opt/aarch64/qt5.15.5`。

# 部署

在 Linux 下确实没有 Windows 上的 **`windeployqt`** 那种自动生成可执行文件所需依赖的工具，但可以通过几种方式来实现类似效果。核心目标是：把你的 Qt 应用程序运行所需的 **共享库 (.so 文件)** 和资源一起打包。

------

### 1️⃣ Linux 下的做法概述

1. **确认 Qt 依赖**

使用 `ldd` 检查你的可执行文件依赖的库：

```bash
ldd ./InomvDemo
```

- 会列出所有动态库及路径，例如：

```
libQt5Widgets.so.5 => /opt/aarch64/qt5.15.5/lib/libQt5Widgets.so.5
libQt5Gui.so.5     => /opt/aarch64/qt5.15.5/lib/libQt5Gui.so.5
libQt5Core.so.5    => /opt/aarch64/qt5.15.5/lib/libQt5Core.so.5
...
```

- 你需要把这些 `.so` 文件和你的可执行文件一起打包。

------

### 2️⃣ 手动复制 Qt 库

```bash
mkdir -p bundle/lib
cp /opt/aarch64/qt5.15.5/lib/libQt5Widgets.so.5 bundle/lib/
cp /opt/aarch64/qt5.15.5/lib/libQt5Gui.so.5 bundle/lib/
cp /opt/aarch64/qt5.15.5/lib/libQt5Core.so.5 bundle/lib/
```

- 如果你的应用用到其他 Qt 模块（如 `Qt5Network`、`Qt5PrintSupport`），也要复制对应 `.so`。
- 把资源文件（如 `.qrc` 编译后的 `.rcc`）也放进 bundle。

------

### 3️⃣ 修改运行时库搜索路径

- Linux 运行时查找共享库顺序：**RPATH > LD_LIBRARY_PATH > 系统默认路径**。
- 可以在可执行文件中指定 RPATH，让它先在 `./lib` 下找 Qt 库：

```bash
# 假设 bundle/bin/InomvDemo，库在 bundle/lib
patchelf --set-rpath '$ORIGIN/../lib' bundle/bin/InomvDemo
```

- `$ORIGIN` 是可执行文件所在目录，`../lib` 指向 lib 子目录。

------

### 4️⃣ 自动工具替代 windeployqt

- Qt 官方在 Linux 没有直接的 `linuxdeployqt`（除非你下载 AppImage 工具）。
- 可以使用第三方脚本 `linuxdeployqt`：

```bash
# 例如下载 linuxdeployqt
wget https://github.com/probonopd/linuxdeployqt/releases/download/continuous/linuxdeployqt-continuous-x86_64.AppImage
chmod +x linuxdeployqt-continuous-x86_64.AppImage

# 打包
./linuxdeployqt-continuous-x86_64.AppImage ./InomvDemo -appimage
```

- 生成一个 **AppImage**，类似 Windows 上的独立 exe，包含 Qt 库。
- 支持 ARM/64 需要对应架构的 AppImage 版本。

------

### 5️⃣ 总结

| 方法                          | 优缺点                                                 |
| ----------------------------- | ------------------------------------------------------ |
| 手动复制 Qt 库 + 设置 RPATH   | 简单，可控，但麻烦，容易漏库                           |
| 使用 linuxdeployqt / AppImage | 自动化打包，生成独立包，但要匹配架构，可能需要额外调试 |

------

# 一个样板cmakelist

```cmake
cmake_minimum_required(VERSION 3.5)  # CMake 最低版本要求
project(InomvDemo)                    # 工程名

# 指定 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 开启 Qt5 的自动 moc, ui, rcc 功能
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

# 查找 Qt5 Widgets 模块
find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets)

# 添加源文件
set(SRC_File
    source/main.cpp
    source/mywidget.cpp
)

# 添加头文件（为了 AUTOMOC 正确处理 Q_OBJECT 宏）
set(HEADERS
    headers/mywidget.h
)

# 创建可执行文件
add_executable(${PROJECT_NAME} ${SRC_File} ${HEADERS})

# 链接 Qt5 库
# 链接时把 Core Gui Widgets 全部加上
target_link_libraries(${PROJECT_NAME} Qt5::Core Qt5::Gui Qt5::Widgets)
target_include_directories(${PROJECT_NAME} PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/headers   # ✅ 正斜杠
)
# 指定 exe 输出目录为工程根目录下的 bin 文件夹
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin/Release)
```

# 父子目录的build

明白了，你问的是 **多目录、多子工程的情况**，CMake 是怎么管理 build 目录的。回答如下：

------

### 1. CMake 的 build 目录原则

- **顶层 build 目录**：由你在命令行或 IDE 指定。例如：

```bash
mkdir build
cd build
cmake ..
```

- **所有子目录（add_subdirectory）默认使用父 build 目录**：
  - 当你在顶层 `CMakeLists.txt` 中写：

```cmake
add_subdirectory(sub1)
add_subdirectory(sub2)
```

- CMake 会在 **同一个 build 目录** 下为每个子工程生成中间文件（obj、CMakeCache.txt、Makefile 等），不会在源目录生成，也不会自动在子目录创建独立 build。
- 生成的目录结构大致如下：

```
project_root/
├─ CMakeLists.txt
├─ sub1/
│   └─ CMakeLists.txt
├─ sub2/
│   └─ CMakeLists.txt
└─ build/
   ├─ CMakeCache.txt
   ├─ Makefile
   ├─ sub1/
   │   └─ *.o / *.obj / etc.
   ├─ sub2/
   │   └─ *.o / *.obj / etc.
   └─ bin/  (可执行文件，如果你在 CMakeLists.txt 设置输出)
```

> ⚠️ 所以子目录不会各自创建独立 build，**都在父 build 目录里**，只是每个子工程的输出和中间文件放在 build 的子目录里。

------

### 2. 如果想每个子工程有独立 build

- 必须单独调用 cmake：

```bash
cd sub1
mkdir build
cd build
cmake ..
make

cd ../../sub2
mkdir build
cd build
cmake ..
make
```

- 这样每个子工程都有自己的 build 目录，但这种方式会打乱顶层统一管理，不推荐大型项目。

------

### 3. 推荐做法（多目录统一管理）

- **顶层 build 目录**，所有子目录统一在父 build 下生成。
- 对可执行文件输出做区分：

```cmake
# 顶层 CMakeLists.txt
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# 如果多配置
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin/Debug)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin/Release)

# 添加子目录
add_subdirectory(sub1)
add_subdirectory(sub2)
```

- 子目录里 target 生成的 exe 会自动放到 `build/bin` 或 `build/bin/Debug`、`build/bin/Release`。

------

总结：

1. **默认情况**：所有子目录共享顶层 build 目录，不会各自独立 build。
2. **如果想独立 build**：必须单独为每个子目录调用 cmake。
3. **推荐方式**：统一顶层 build，CMakeLists.txt 设置输出目录（bin/lib），Debug/Release 分类即可。

------

# 为什么cmakelist生成的软件直接就在控制器里不需要依赖运行？

明白了，你是 **CMake + VSCode** 而不是 qmake。那 rpath/runpath 指向你 Qt 自定义目录的原因和 `settings.json` 关系不大，主要还是 **CMake 配置和编译器链接器参数** 决定的。

------

### **1. CMake 决定 rpath 的几个地方**

1. **CMAKE_INSTALL_RPATH / CMAKE_BUILD_RPATH**

   - CMake 默认会把 `CMAKE_INSTALL_RPATH` 写入可执行文件，用于安装后的库搜索路径。

   - `CMAKE_BUILD_RPATH` 用于构建目录下运行时加载库。

   - 在 CMakeLists.txt 里可以设置：

     ```cmake
     set(CMAKE_BUILD_RPATH "/mnt/ssd/opt/qt5.15.5/lib")
     set(CMAKE_INSTALL_RPATH "/mnt/ssd/opt/qt5.15.5/lib")
     set(CMAKE_SKIP_BUILD_RPATH FALSE)
     set(CMAKE_SKIP_INSTALL_RPATH FALSE)
     set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
     ```

   - 这样 CMake 会在 ELF 可执行文件里写入 `/mnt/ssd/opt/qt5.15.5/lib`。

2. **Qt 的 CMake 配置**

   - 你在 CMake 里用 `find_package(Qt5 COMPONENTS Widgets Gui Core REQUIRED)`

   - CMake 找到 `/mnt/ssd/opt/qt5.15.5/lib/cmake/Qt5`，会把对应库路径写入链接器命令：

     ```
     -L/mnt/ssd/opt/qt5.15.5/lib -Wl,-rpath,/mnt/ssd/opt/qt5.15.5/lib
     ```

   - 这就是为什么编译后的可执行文件自动指向你自己 Qt 目录。

3. **手动 target_link_libraries 指定绝对路径**

   ```cmake
   target_link_libraries(InomvDemo
       PRIVATE /mnt/ssd/opt/qt5.15.5/lib/libQt5Widgets.so
               /mnt/ssd/opt/qt5.15.5/lib/libQt5Core.so
               /mnt/ssd/opt/qt5.15.5/lib/libQt5Gui.so
   )
   ```

   - 链接器会直接把这些路径写入 rpath。

------

### **2. settings.json 的作用**

- VSCode 的 `settings.json` 主要影响 **编辑器智能提示/IntelliSense** 和 **CMake Tools 插件**

- 它不会直接修改 ELF 的 rpath

- 如果你在 `CMake Tools: Configure Args` 里指定了 Qt 路径，比如：

  ```
  -DCMAKE_PREFIX_PATH=/mnt/ssd/opt/qt5.15.5
  ```

  那 CMake 就会找到你的 Qt 并把路径写入链接命令，从而最终写入 rpath。

------

### **3. 总结**

- **可执行文件指向自定义 Qt 路径**：来源于 CMake 链接器参数和 rpath/runpath
- **VSCode settings.json**：只影响 CMake 配置，不直接写 rpath
- **系统 Qt 5.14 不会被加载**：因为 rpath 优先级比系统路径高

------

# rpath/runpath 原理

你这个现象说明，你的可执行文件 `InomvDemo` 在 **编译/链接阶段**就已经把 **Qt 库的绝对路径写入了 ELF 文件的 rpath/runpath**，所以运行时它会优先去你指定的 `/mnt/ssd/opt/qt5.15.5/lib/` 找库，而不是去系统默认路径（系统上有 Qt 5.14 也不会影响）。

------

### **1. rpath/runpath 原理**

在 Linux ELF 可执行文件中，有一段特殊字段：

- **rpath**：指定运行时库搜索路径，会在 LD_LIBRARY_PATH 之前查找
- **runpath**：现代 Linux 用法，和 rpath 类似，但会被 LD_LIBRARY_PATH 覆盖

你可以查看 `InomvDemo` 的 rpath/runpath：

```bash
readelf -d InomvDemo | grep PATH
```

输出示例可能是：

```
0x000000000000001d (RUNPATH)            Library runpath: [/mnt/ssd/opt/qt5.15.5/lib]
```

意思就是 **程序运行时会直接去 `/mnt/ssd/opt/qt5.15.5/lib` 查找 Qt 库**。

------

### **2. 为什么会自动指向这个目录**

可能是以下操作导致：

1. **qmake pro 文件里指定了 LIBS 或 QMAKE_RPATHDIR**

   ```pro
   LIBS += -L/mnt/ssd/opt/qt5.15.5/lib
   QMAKE_RPATHDIR += /mnt/ssd/opt/qt5.15.5/lib
   ```

   - `QMAKE_RPATHDIR` 会在编译时把路径写入 ELF 的 rpath/runpath。

2. **qmake + Qt 编译器的默认行为**

   - 当你用 qmake 构建 Qt GUI 程序，并使用非系统 Qt 安装路径（比如你自己编译的 5.15.5），qmake 默认会把 `rpath` 写入可执行文件，使它运行时优先使用这个 Qt 版本。

3. **手动用 patchelf 或 chrpath 设置过**

   - 可以在可执行文件上执行：

     ```bash
     patchelf --set-rpath /mnt/ssd/opt/qt5.15.5/lib InomvDemo
     ```

   - 这样运行时会直接去这个目录加载库。

------

### **3. 你系统有 Qt5.14 为什么没被用到**

- 因为 rpath 写入了 `/mnt/ssd/opt/qt5.15.5/lib`
- Linux 动态链接器优先 rpath/runpath，然后才是 LD_LIBRARY_PATH，再然后才是系统默认路径
- 所以即使系统有 Qt5.14，也不会被加载

------

### **4. 验证方法**

可以执行：

```bash
readelf -d InomvDemo | grep PATH
```

你会看到类似：

```
RUNPATH            /mnt/ssd/opt/qt5.15.5/lib
```

也可以用 ldd 看实际加载库：

```bash
ldd InomvDemo | grep Qt5
```

会显示 `/mnt/ssd/opt/qt5.15.5/lib/libQt5*.so.5`，说明运行时确实从 rpath 指定的路径加载。

------

