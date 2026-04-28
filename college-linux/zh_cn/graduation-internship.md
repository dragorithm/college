# 网络管理与维护综合实训报告

## 实训目标

**标题**：Linux Arch 系统安装

**目的**:

1. 完成 Linux 安装
2. 应用 CPU 指令集优化
3. 使用 ZFS 全盘加密
4. 在 ZFS 上启动 root 分区

## 实训环境

### 物理环境

- CPU: Intel Core i5‑1135G7
- Hypervisor: VMware Workstation Pro 25H2u1 25.0.1.25219725

![CPU](../statics/graduation-internship/CPU.png)

### 逻辑环境

- CPU(Cores): 3
- DRAM(GiB): 6
- VRAM(MiB): 256
- SCSI(GiB): 20 (vmdk)
- Guest OS: cachyos-desktop-linux-260308.iso
- Guest OS source: <https://cdn77.cachyos.org/ISO/desktop/260308/cachyos-desktop-linux-260308.iso>
- Firmware: EFI
- 3D Acceleration: Enabled
- USB: Disabled
- Sound: Disabled
- Network: NAT

## 实训内容

### 基本配置

#### 操作步骤

```bash
# 配置根名称变量
source /etc/os-release
export ID

# 配置启动磁盘变量
export DISK='/dev/sda'
export BOOT_PART='1'
export BOOT_DEVICE="${DISK}${BOOT_PART}"

# 配置 zpool 磁盘变量
export POOL_PART='2'
export POOL_DEVICE="${DISK}${POOL_PART}"

# 配置架构变量
export ARCH='v4'

# 生成 hostid
zgenhostid -f

# 擦除分区
zpool labelclear -f "$DISK"
wipefs -a "$DISK"
sgdisk --zap-all "$DISK"
dd if=/dev/zero of="$DISK" bs=512 count=2048
```

#### 示例

![01-01.png](../statics/graduation-internship/01-01.png)

#### 说明

读取系统发行版信息：

- `/etc/os-release`：标准 Linux 发行版信息文件
- `export ID`：将发行版 ID 导出为环境变量

设置启动磁盘变量：

- `DISK='/dev/sda'`：目标磁盘
- `BOOT_PART='1'`：启动分区编号

设置 ZFS 存储池分区：

- `POOL_PART='2'`：ZFS 存储池使用磁盘的第 2 个分区

设置架构变量：

- `export ARCH='v4'`：V4 指 x86_v4，针对支持完整 AVX‑512 指令集的 CPU 进行了优化

生成 hostid：

- `zgenhostid -f`：ZFS 依赖 hostid 来识别系统

擦除磁盘：彻底擦除磁盘以准备重新分区并创建 ZFS 池

- `zpool labelclear`：清除 ZFS 标签
- `wipefs -a`：移除所有文件系统签名
- `sgdisk --zap-all`：清除 GPT 分区表
- `dd if=/dev/zero of="$DISK" bs=512 count=2048`：将前 2048 个扇区清零，以确保 GPT 被擦除

### 分区

#### 操作步骤

```bash
# 创建 ESP 分区
sgdisk -n "${BOOT_PART}:0:+260m" -t "${BOOT_PART}:ef00" "$DISK"

# 创建 zpool 分区
sgdisk -n "${POOL_PART}:0:0" -t "${POOL_PART}:bf00" "$DISK"
```

#### 示例

![02-01](../statics/graduation-internship/02-01.png)

#### 说明

创建 EFI 系统分区：

- `sgdisk -n “${BOOT_PART}:0:+260m” -t “${BOOT_PART}:ef00” “$DISK”`：创建一个 260 MiB 的 EFI 系统分区
- 此分区是 UEFI 启动所必需的

创建 ZFS 存储池分区：

- `sgdisk -n “${POOL_PART}:0:0” -t “${POOL_PART}:bf00” “$DISK”`：创建一个占用剩余磁盘空间的 ZFS 分区
- `bf00` 是 ZFS 的标准 GPT 类型代码

### 创建存储池

#### 操作步骤

```bash
# 创建 zpool
zpool create -f -o ashift=12 \
 -O acltype=posix \
 -O xattr=sa \
 -O atime=off \
 -O normalization=formD \
 -O dnodesize=auto \
 -O encryption=on \
 -O keyformat=passphrase \
 -O keylocation=prompt \
 -o autotrim=on \
 -m none tank "$POOL_DEVICE"
```

#### 示例

![03-01](../statics/graduation-internship/03-01.png)

密码: dragon12

#### 说明

- `zpool create -f`：确保即使在未完全清理的磁盘上也能创建存储池
- `-o ashift=12`：强制进行 4K 扇区对齐，以获得最佳的 SSD/HDD 性能
- `-O acltype=posix`：启用精细的权限控制
- `-O xattr=sa`：将扩展属性存储在系统属性区而非文件系统中
- `-O atime=off`：禁用访问时间更新以提升性能
- `-O normalization=formD`：防止跨系统文件名编码冲突
- `-O dnodesize=auto`：提升元数据密集型工作负载的性能
- `-O encryption=on`：启用原生 ZFS 加密
- `-O keyformat=passphrase`：启动时要求输入用户密码短语
- `-O keylocation=prompt`：启动时提示输入密钥
- `-O autotrim=on`：保持 SSD 性能并减少写入放大
- `-m none`：避免与后续数据集布局发生冲突
- `tank “$POOL_DEVICE”`：其中“tank”是存储池名称，后跟设备名称

### 创建数据集

#### 操作步骤

```bash
zfs create -o mountpoint=none tank/ROOT -o encryption=on
zfs create -o mountpoint=/ -o canmount=noauto tank/ROOT/${ID} -o compression=zstd -o encryption=on
zfs create -o mountpoint=none tank/USERDATA -o encryption=on
zfs create -o mountpoint=/home tank/USERDATA/home -o compression=lz4 -o encryption=on
zfs create -o mountpoint=/root tank/USERDATA/root -o compression=lz4 -o encryption=on
zfs create -o mountpoint=none tank/var -o encryption=on
zfs create -o mountpoint=/var/log tank/var/log -o exec=off -o setuid=off -o compression=off -o encryption=on
zfs create -o mountpoint=/var/cache tank/var/cache -o compression=off -o encryption=on
zfs create -o mountpoint=/var/lib tank/var/lib -o compression=off -o encryption=on
zfs create -o mountpoint=/var/tmp tank/var/tmp -o exec=off -o setuid=off -o compression=off -o encryption=on
```

#### 示例

![04-01](../statics/graduation-internship/04-01.png)

#### 说明

- `-o mountpoint`：挂载点，当用于父数据集时，挂载点应设为 `none`
- `tank/ROOT`: ZFS 路径，位于命令挂载点之后
- `-o exec=off`: 禁止执行程序
- `-o setuid=off`: 禁止 SUID
- `-o compression`: 磁盘压缩
- `-o encryption=on`: 启用加密

### 设置启动根

#### 操作步骤

```bash
# 设置 BootFS
zpool set bootfs=tank/ROOT/${ID} tank
```

#### 示例

![05-01](../statics/graduation-internship/05-01.png)

#### 说明

BootFS = ZFS 存储池的缺省根文件系统指针

### 迁移存储池挂载点

#### 操作步骤

```bash
# 导出名为 tank 的 ZFS 存储池
zpool export tank

# 导入名为 tank 的 ZFS 存储池
zpool import -N -R /mnt tank

# 解锁 tank 的所有数据集
zfs load-key -r tank
```

#### 示例

![06-01](../statics/graduation-internship/06-01.png)

#### 说明

将池根挂载点从实时环境重新导入到 `/mnt`，以便在 `chroot` 期间能够访问它

### 挂载

#### 操作步骤

```bash
# 挂载 ZFS 文件系统
zfs mount tank/ROOT/${ID}
zfs mount tank/USERDATA/home
zfs mount tank/USERDATA/root
zfs mount tank/var/cache
zfs mount tank/var/lib
zfs mount tank/var/log
zfs mount tank/var/tmp
```

#### 示例

![07-01](../statics/graduation-internship/07-01.png)

#### 说明

这些命令将手动挂载所有 ZFS 数据集。

如果不进行这些挂载操作，后续的 `chroot` 和引导加载程序配置步骤将无法正常进行。

### 更新设备节点

#### 操作步骤

```bash
# 更新设备符号链接
udevadm trigger
```

#### 示例

![08-01](../statics/graduation-internship/08-01.png)

#### 说明

强制系统重新扫描并更新所有设备节点和符号链接，从而减少意外问题，并确保实训过程的可重复性。

### 安装系统

#### 操作步骤

```bash
# 安装 Arch 系统
echo 'Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch' > /etc/pacman.d/mirrorlist
pacman -Sy aria2 --noconfirm
pacman-key --keyserver hkps://keyserver.ubuntu.com --recv-keys F3B607488DB35A47
pacman-key --lsign-key F3B607488DB35A47

tee /etc/pacman.d/cachyos-${ARCH}-mirrorlist > /dev/null <<EOF
## Cloudflare R2
Server = https://cdn.cachyos.org/repo/\$arch_$ARCH/\$repo

## 德国
Server = https://aur.cachyos.org/repo/\$arch_$ARCH/\$repo
Server = https://mirror.cachyos.org/repo/\$arch_$ARCH/\$repo
Server = https://build.cachyos.org/repo/\$arch_$ARCH/\$repo
EOF
tee /etc/pacman.d/cachyos-mirrorlist > /dev/null <<EOF
## Cloudflare R2
Server = https://cdn.cachyos.org/repo/\$arch/\$repo

## 德国
Server = https://aur.cachyos.org/repo/\$arch/\$repo
Server = https://mirror.cachyos.org/repo/\$arch/\$repo
Server = https://build.cachyos.org/repo/\$arch/\$repo
EOF
tee /etc/pacman.conf > /dev/null <<EOF
[options]
HoldPkg     = pacman glibc
Architecture = auto
Color
ILoveCandy
VerbosePkgLists
DisableDownloadTimeout
ParallelDownloads = 16
DownloadUser = alpm
SigLevel = Required DatabaseOptional
LocalFileSigLevel = PackageRequired

[cachyos-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos-core-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos-extra-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos]
Include = /etc/pacman.d/cachyos-mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist
EOF

mkdir -p /root/aria2-download/
tee /root/aria2-download/updateURLs.txt > /dev/null <<EOF
https://cdn.cachyos.org/repo/x86_64/cachyos/cachyos.db
https://cdn.cachyos.org/repo/x86_64/cachyos/cachyos.db.sig
https://mirrors.aliyun.com/archlinux/core/os/x86_64/core.db
https://mirrors.aliyun.com/archlinux/extra/os/x86_64/extra.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-$ARCH/cachyos-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-$ARCH/cachyos-$ARCH.db.sig
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-core-$ARCH/cachyos-core-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-core-$ARCH/cachyos-core-$ARCH.db.sig
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-extra-$ARCH/cachyos-extra-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-extra-$ARCH/cachyos-extra-$ARCH.db.sig
EOF

aria2c -j10 -x4 -s4 -i /root/aria2-download/updateURLs.txt -d /var/lib/pacman/sync/ --retry-wait=5 --max-tries=5 --user-agent='Mozilla/5.0' --allow-overwrite=true

pacman --root /mnt -Sp linux-firmware intel-ucode linux-cachyos-hardened-zfs base zfs-dkms zfs-utils networkmanager linux-cachyos-hardened-headers \
| grep -v '^file://' \
| awk '{ print; print $0".sig" }' \
| aria2c -j16 -x16 -s16 -d /mnt/var/cache/pacman/pkg --retry-wait=5 --max-tries=5 --user-agent='Mozilla/5.0' --allow-overwrite=true --min-split-size=1M -i -

pacstrap /mnt linux-cachyos-hardened-zfs linux-cachyos-hardened-headers base zfs-dkms zfs-utils linux-firmware intel-ucode networkmanager zfsbootmenu --noconfirm

cp /etc/hostid /mnt/etc
arch-chroot /mnt
```

#### 示例

配置基础镜像、下载 aria2 并信任公钥
![09-01](../statics/graduation-internship/09-01.png)

配置 V4 镜像
![09-02](../statics/graduation-internship/09-02.png)

配置软件包数据库 URL
![09-03](../statics/graduation-internship/09-03.png)

下载软件包数据库
![09-04](../statics/graduation-internship/09-04.png)

复制缓存
![09-05](../statics/graduation-internship/09-05.png)

下载所需软件包
![09-06](../statics/graduation-internship/09-06.png)
![09-07](../statics/graduation-internship/09-07.png)

安装系统
![09-08](../statics/graduation-internship/09-08.png)
![09-09](../statics/graduation-internship/09-09.png)
![09-10](../statics/graduation-internship/09-10.png)
![09-11](../statics/graduation-internship/09-11.png)

复制 hostid 并进入/退出 chroot
![09-12](../statics/graduation-internship/09-12.png)

#### 说明

配置基础镜像、下载 aria2 并信任公钥

- 设置 Arch 镜像（阿里云）
- 安装 aria2 以加速下载
- 导入并信任 CachyOS 仓库签名密钥

配置 V4 镜像

- 这些镜像用于下载针对 CachyOS v4 优化的软件包（AVX‑512）

配置软件包数据库 URL

- 为 aria2 批量下载数据库文件准备的 URL

下载软件包数据库

- 多线程下载 `.db` 和 `.db.sig` 文件
- 绕过 pacman 的内置下载器以提高速度

复制缓存

- 将运行中系统的数据库复制到目标系统
- 使 `/mnt` 环境中的 pacman 无需重新下载即可使用数据库

下载所需软件包

- 为内核、头文件、ZFS 模块、基础组、固件/微代码及网络工具生成下载 URL
- 使用 16 个线程，通过高并发加速下载

安装系统

- 安装基于 Arch 系统的核心步骤

复制 hostid 并进入/退出 chroot

- ZFS 需要 hostid 来识别存储池 → `cp /etc/hostid /mnt/etc`
- 使用 `arch-chroot /mnt` 进入 chroot 环境以进行后续配置

### 配置 ESP 分区

#### 操作步骤

```bash
# 配置 ESP（EFI 系统分区）
zfs set org.zfsbootmenu:commandline="systemd.show_status=1 systemd.log_level=info" tank/ROOT
pacman -S dosfstools --noconfirm
mkfs --type vfat -F 32 -I "$BOOT_DEVICE"
mkdir -p /efi
mount "$BOOT_DEVICE" /efi
```

#### 示例

![10-01](../statics/graduation-internship/10-01.png)

#### 说明

此步骤将创建并挂载 UEFI 所需的 FAT32 ESP 分区，并设置 ZFSBootMenu 引导参数。

### 配置新系统的环境

#### 操作步骤

```bash
# 配置新系统
echo 'Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch' > /etc/pacman.d/mirrorlist
pacman -Sy aria2 --noconfirm
pacman-key --recv-keys F3B607488DB35A47 --keyserver keyserver.ubuntu.com
pacman-key --lsign-key F3B607488DB35A47
tee /etc/pacman.d/cachyos-${ARCH}-mirrorlist > /dev/null <<EOF
## Cloudflare R2
Server = https://cdn.cachyos.org/repo/\$arch_$ARCH/\$repo

## 德国
Server = https://aur.cachyos.org/repo/\$arch_$ARCH/\$repo
Server = https://mirror.cachyos.org/repo/\$arch_$ARCH/\$repo
Server = https://build.cachyos.org/repo/\$arch_$ARCH/\$repo
EOF
tee /etc/pacman.d/cachyos-mirrorlist > /dev/null <<EOF
## Cloudflare R2
Server = https://cdn.cachyos.org/repo/\$arch/\$repo

## 德国
Server = https://aur.cachyos.org/repo/\$arch/\$repo
Server = https://mirror.cachyos.org/repo/\$arch/\$repo
Server = https://build.cachyos.org/repo/\$arch/\$repo
EOF
tee /etc/pacman.conf > /dev/null <<EOF
[options]
HoldPkg     = pacman glibc
Architecture = auto
Color
ILoveCandy
VerbosePkgLists
DisableDownloadTimeout
ParallelDownloads = 16
DownloadUser = alpm
SigLevel = Required DatabaseOptional
LocalFileSigLevel = PackageRequired

[cachyos-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos-core-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos-extra-v4]
Include = /etc/pacman.d/cachyos-v4-mirrorlist

[cachyos]
Include = /etc/pacman.d/cachyos-mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist
EOF
mkdir -p /root/aria2-download/
tee /root/aria2-download/updateURLs.txt > /dev/null <<EOF
https://cdn.cachyos.org/repo/x86_64/cachyos/cachyos.db
https://cdn.cachyos.org/repo/x86_64/cachyos/cachyos.db.sig
https://mirrors.aliyun.com/archlinux/core/os/x86_64/core.db
https://mirrors.aliyun.com/archlinux/extra/os/x86_64/extra.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-$ARCH/cachyos-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-$ARCH/cachyos-$ARCH.db.sig
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-core-$ARCH/cachyos-core-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-core-$ARCH/cachyos-core-$ARCH.db.sig
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-extra-$ARCH/cachyos-extra-$ARCH.db
https://cdn.cachyos.org/repo/x86_64_$ARCH/cachyos-extra-$ARCH/cachyos-extra-$ARCH.db.sig
EOF
pacman -S aria2 openssh --noconfirm
echo 'root:dragon' | chpasswd
sed -i \
    -e 's/^#PasswordAuthentication yes/PasswordAuthentication yes/' \
    -e 's/^#PermitRootLogin prohibit-password/PermitRootLogin yes/' \
    /etc/ssh/sshd_config
systemctl enable NetworkManager sshd
```

#### 示例

由于部分内容与前面的步骤高度重复，此处不再重复列举示例。

![11-01](../statics/graduation-internship/11-01.png)

#### 说明

这一部分主要重复了之前的步骤，因此示例和说明将予以简化。

- `echo 'root:dragon' | chpasswd`：将 root 密码设置为 `dragon`
- `sed -i ...`：配置 root 的远程登录权限
- `systemctl enable NetworkManager sshd`：启用网络和 SSH 服务

### 配置引导

#### 操作步骤

```bash
aria2c https://get.zfsbootmenu.org/efi -d /efi/EFI/zbm --min-split-size=1M
pacman -Sp efibootmgr \
| grep -v '^file://' \
| awk '{ print; print $0".sig" }' \
| aria2c -j16 -x16 -s16 -d /var/cache/pacman/pkg --retry-wait=5 --max-tries=5 --user-agent='Mozilla/5.0' --allow-overwrite=true --min-split-size=1M -i -

pacman -S efibootmgr --noconfirm
EFI_NAME=$(ls /efi/EFI/zbm/)
efibootmgr --create --disk "$DISK" --part "$BOOT_PART" --label 'ZFSBootMenu' --loader "\EFI\zbm\\${EFI_NAME}"
```

#### 示例

在网络状况不佳的情况下，此下载操作极有可能失败

![12-01](../statics/graduation-internship/12-01.png)
![12-02](../statics/graduation-internship/12-02.png)

#### 说明

下载 ZFSBootMenu EFI 文件，安装 `efibootmgr`，并为 ZFSBootMenu 创建一个 UEFI 启动项。

### 重启系统

#### 操作步骤

```bash
exit
umount -n -R /mnt
zpool export tank
sync
reboot
```

#### 示例

![13-01](../statics/graduation-internship/13-01.png)

#### 说明

安全地退出 `chroot` 环境，卸载所有挂载点，导出 ZFS 存储池（这是正确的退出方式），然后重启进入新安装的 ZFS 系统。

### 结束

启动界面：

![end1](../statics/graduation-internship/end1.png)

密码提示：

![end2](../statics/graduation-internship/end2.png)

至此，基于 ZFS 全盘加密和 CPU 指令集优化的 Arch Linux 系统安装已完成。系统可正常启动，并提示输入密码解锁。

## 实训成果

### ZFS 全盘加密

成功部署了一个采用 ZFS 全盘加密的 Arch Linux 系统，完成了从磁盘初始化、分区、创建 ZFS 存储池、数据集规划到系统安装的整个过程。

根文件系统位于加密的 ZFS 存储池中，启动时需要输入密码才能解锁。

### 指令集优化

实现了深度 CPU 指令集优化。
    通过设置 `ARCH='v4'` 并使用 `linux-cachyos-hardened-zfs` 内核，系统在 Intel Core i5-1135G7 上启用了完整的 AVX-512 指令集支持，从而提升了计算密集型任务的性能。

### ZFS 根分区启动支持

构建了可引导的 ZFSBootMenu 环境。
    成功配置了 EFI 系统分区，部署了 ZFSBootMenu 引导加载程序，支持多内核/快照引导、加密解锁提示、系统日志输出及其他高级功能。

### 远程登录

系统支持远程管理和维护。
    启用了支持 root 登录的 SSH 服务，并配置了 NetworkManager，确保系统在安装完成后即可通过网络访问，以便进行后续操作和调试。

## 实训体会

### ZFS 优势显著但复杂度高

ZFS 的原生加密、快照、压缩和自愈等功能极大地提升了系统的健壮性和灵活性。
    然而，初始化、挂载和启动配置必须严格遵循顺序；即使微小的失误也可能导致无法启动。

### 镜像与下载工具的优化显著提升效率

在相同的网络条件下，使用 aria2 下载软件包数据库和软件包的速度比 pacman 快约 15 倍。
    实际测试中，安装时间从 20 分钟缩短至约 1 分 20 秒，这在大规模部署中展现出明显的优势。

### 安全与性能的平衡

全盘加密虽能确保数据安全，但每次启动时都需要手动输入密码。
    禁用 `atime`、启用 `autotrim` 以及应用压缩策略可提升 SSD 性能，但必须根据工作负载进行调整。

### 复现难度高

偶尔会出现无法解释的错误。

对 CPU 有严格要求（例如支持 AVX-512）。

学生可以使用替代方案来绕过 CPU 限制，例如将 `ARCH='v4'` 改为 `ARCH='v3'`，以支持更广泛的处理器。
