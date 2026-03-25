可以，我帮你把之前在 Linux 下挂载 `/dev/sda` SSD 的操作整理成一个完整文档，按步骤清楚说明，包括分区、格式化、挂载、权限调整和开机自动挂载。这个文档适合直接保存到 Typora 或 Markdown 文件。

------

# 在 Linux 下挂载 SSD 并使用

本文档记录在 Linux (aarch64 / Ubuntu) 下，将一块新 SSD `/dev/sda` 挂载为 `/mnt/ssd` 的完整操作流程，包括分区、格式化、挂载、权限调整以及开机自动挂载。

------

## 1. 查看磁盘信息

查看系统识别到的磁盘：

```bash
sudo fdisk -l
lsblk -f
```

示例输出：

```
Disk /dev/sda: 119.24 GiB, 128035676160 bytes
Disk model: 128GB SATA SSD
...
```

- `/dev/sda` 是目标 SSD
- 检查是否已有分区或数据

------

## 2. 创建分区（如果磁盘为空）

使用 `fdisk` 创建新的主分区：

```bash
sudo fdisk /dev/sda
```

在 fdisk 交互界面：

1. 输入 `n` → 新建分区
2. 输入 `p` → 主分区
3. 分区号默认 `1`
4. 起始扇区默认，回车
5. 结束扇区默认，回车（使用整块磁盘）
6. 输入 `w` → 写入并退出

- 创建完成后生成 `/dev/sda1`

------

## 3. 格式化分区

将分区格式化为 `ext4` 文件系统：

```bash
sudo mkfs.ext4 /dev/sda1
```

可选：添加分区标签 `SSD128`：

```bash
sudo mkfs.ext4 -L SSD128 /dev/sda1
```

------

## 4. 创建挂载点

创建挂载目录 `/mnt/ssd`：

```bash
sudo mkdir -p /mnt/ssd
```

------

## 5. 挂载分区

挂载 `/dev/sda1` 到 `/mnt/ssd`：

```bash
sudo mount /dev/sda1 /mnt/ssd
```

检查挂载状态：

```bash
df -h /mnt/ssd
mount | grep /mnt/ssd
```

- 挂载成功后可以直接访问 `/mnt/ssd`
- 如果提示 `already mounted`，说明分区已经挂载

------

## 6. 设置权限让普通用户可操作

方法 1：修改挂载目录所有权（推荐）：

```bash
sudo chown -R $USER:$USER /mnt/ssd
```

方法 2：修改权限（所有用户可读写执行）：

```bash
sudo chmod 777 /mnt/ssd
```

- 修改后，普通用户可以复制粘贴、读写文件

------

## 7. 开机自动挂载

编辑 `/etc/fstab`：

```bash
sudo nano /etc/fstab
```

添加一行：

```text
/dev/sda1    /mnt/ssd    ext4    defaults,uid=1000,gid=1000    0  2
```

- `uid=1000,gid=1000` 替换为你的用户 ID，可通过 `id` 命令查看
- 保存并退出编辑器

重新加载 systemd 配置并挂载：

```bash
sudo systemctl daemon-reload
sudo mount -a
```

- 下次开机 `/dev/sda1` 会自动挂载到 `/mnt/ssd`

------

## 8. 使用 SSD

挂载完成后即可使用 `/mnt/ssd` 存储文件，例如：

```bash
cd /mnt/ssd
touch test_file.txt
cp ~/Documents/example.txt .
```

- 挂载点可直接读写文件
- 空间大小可用 `df -h /mnt/ssd` 查看

------

这个文档覆盖了 **从分区、格式化、挂载到权限配置、开机自动挂载的完整流程**，保证 SSD 能被普通用户直接使用。

------

如果你希望，我可以再帮你整理 **一条命令或小脚本**，完成从分区到挂载并设置权限、写入 fstab 的一键操作，插盘即用。

你希望我做吗？