# Linux Qt 5.15.5 aarch64 编译指南

---

## 一、编译依赖检查

### 1. 编译工具类（必装）

| 包名              | 作用 |
| ----------------- | ------------------------------------------------------ |
| `build-essential` | 提供 gcc/g++/make 等基础工具 |
| `perl`            | Qt configure 脚本依赖 Perl |
| `python3`         | QtWebEngine 等工具依赖 Python |

### 2. GUI/系统库（头文件 + 链接库）

| 包名                 | 作用 |
| -------------------- | ------------------------------------------------------------ |
| `libglib2.0-dev`     | GLib 基础库 |
| `libx11-dev`         | X11 核心库 |
| `libxcb1-dev`        | XCB 库，用于 GUI |
| `libx11-xcb-dev`     | X11 与 XCB 桥接 |
| `libxext-dev`        | X11 扩展库 |
| `libxfixes-dev`      | X11 修复与扩展 |
| `libxi-dev`          | 鼠标键盘事件支持 |
| `libxrender-dev`     | 绘图与字体渲染 |
| `libxrandr-dev`      | 窗口旋转、缩放扩展 |
| `libfontconfig1-dev` | 字体管理 |

### 3. 检查安装状态

单个包检查：

```bash
dpkg -s build-essential
```

批量检查：

```bash
dpkg -l | grep -E "build-essential|perl|python3|libglib2.0-dev|libx11-dev|libxcb1-dev|libx11-xcb-dev|libxext-dev|libxfixes-dev|libxi-dev|libxrender-dev|libxrandr-dev|libfontconfig1-dev"
```

状态标志：

- `ii` 已安装
- `rc` 已删除但残留配置
- `un` 未安装

------

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

### 4. configure 命令（最小化、禁用 PCH）

```bash
../qt-everywhere-src-5.15.5/configure \
-prefix /opt/aarch64/qt5.15.5 \
-release \
-opensource \
-confirm-license \
-nomake examples \
-nomake tests \
-skip qt3d \
-skip qtcharts \
-skip qtconnectivity \
-skip qtdatavis3d \
-skip qtdeclarative \
-skip qtgamepad \
-skip qtgraphicaleffects \
-skip qtmultimedia \
-skip qtvirtualkeyboard \
-skip qtwebchannel \
-skip qtwebengine \
-skip qtwebview \
-skip qtwebsockets \
-skip qtwebglplugin \
-skip qtwinextras \
-skip qtx11extras \
-skip qtlottie \
-skip qtremoteobjects \
-skip qtspeech \
-skip qtxmlpatterns \
-no-pch
```

**注意事项**：

- `-skip` 必须与源码目录模块名称一致，大小写敏感
- 每行续行符 `\` 必须紧跟行尾，无多余空格
- 禁用 PCH 可避免 “No space left on device” 错误

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