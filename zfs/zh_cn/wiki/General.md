# 面向用户/系统管理员的通用文档

## 系统管理员

本页面旨在记录对系统管理员有用的信息

## 概述

ZFS是对传统存储架构的重新构想。
    ZFS 中的基本存储单位是 **存储池/池(storage pool/pool)**, 并且可以从中获得 **数据集(datasets)**, 数据集可以是挂载点(可挂载的文件系统)或块设备。
    ZFS的池是一个完整的存储架构, 能够代替RAID、分区、卷管理、fstab/exports文件、仅覆盖单个硬盘的传统文件系统(如UFS和XFS)。
    这使得能够用更少的代码完成相同的任务, 同时获得更高的可靠性和更简化的管理

只需一条命令, 即可从一组磁盘创建一个具有冗余功能的可用的文件系统, 且该文件系统在重启后仍会保留。
    这是因为 ZFS 存储池始终包含一个名为 **根数据集(root database)** 的挂载文件系统, 该文件系统会在存储池创建时被挂载。
    创建时, 存储池会被导入系统, 从而在 `zpool.cache`文件中生成一条记录。
    在导入或创建过程中, 存储池会存储系统 **唯一的主机ID(hostid)**, 为了支持多路径功能, 除非强制操作, 否则将无法将该存储池导入其他系统

ZFS 本身由三个主要层组成。
    底层是**存储池分配器(Storage Pool Allocator/SPA)**, 负责将物理磁盘组织成存储空间。
    中间层是**数据管理单元(Data Management Unit/DMU)**, 它使用 SPA 提供的存储, 以事务性的、原子方式对其进行读写。
    最上层是 **数据集层(dataset layer)**, 它将对存储池所提供的文件系统和块设备(zvols)的操作, 转换为 DMU 中的操作

### 低级存储

SPA 将磁盘池中的磁盘组织成一个 **虚拟设备(virtual devices/vdevs)**。
    该树的顶层是根 vdev。
    其直接子节点可以是除自身以外的任何类型的 vdev。
    vdev 的主要类型包括:

- mirror(支持 n 路镜像)
- raidz
  - raidz1(1 磁盘校验, 类似 RAID 5)
  - raidz2(2 磁盘校验, 类似 RAID 6)
  - raidz3(3 硬盘校验, 没有对应的 RAID 等价物)
- disk
- file(不推荐用于生产环境, 因为额外的文件系统会增加不必要的层级)

这些 vdev 中的任意数量都可以是根 vdev 的子节点, 它们被称为顶级 vdev。
    此外, 其中某些 vdev 也可以拥有子节点, 例如 mirror vdev 和 raidz vdev
    命令行工具并不支持创建 raidz 的镜像 或 镜像的 raidz, 尽管在开发者测试中会使用这种配置

使用多个顶层 vdev 会以可叠加的方式影响 IOPS, 其中总 IOPS 将等于所有顶层 vdev 的 IOPS 之和。
    因此, 任何一个主要顶层 vdev 的丢失都会导致整个存储池的丢失, 因此必须在所有顶层 vdev 上使用适当的冗余

支持的最小 vdev 或磁盘 vdev 大小为 64MB(2^26字节), 而最大大小则取决于平台, 但所有平台都应支持至少 16EB(2 ^ 64字节)的vdev

还有三种特殊类型的 vdev:

- spare
- cache
- log

spare 设备会在某个磁盘发生故障时用于替换, 前提是存储池的 autoreplace 属性已启用, 并且你的平台支持该功能。
    它不会替换 cache 设备 或 log 设备

cache 设备用于扩展 ZFS 的内存数据缓存, 该缓存取代了页面缓存(page cache), 但 mmap()除外, 在绝大多数平台上, mmap()仍然使用页面缓存。
    ZFS 使用的缓存算法是 **自适应替换缓存(Adaptive Replacement Cache/ARC)**, 其命中率高于页面缓存所使用的 **最近最少使用(Last Recently Used)** 算法。
    cache 设备的设计目的是与闪存设备一起使用。
    其中存储的数据是非持久性的, 因此可以使用低成本设备

log 设备允许 ZFS 将 **意图日志(ZFS Intent Log/ZIL)** 记录写入不同的设备(例如闪存设备), 以提升同步写入操作的性能, 随后这些记录才会写入主存储。
    除非设置了 `sync=disabled` 数据集属性, 否则会使用这些记录。
    无论是否使用 log 设备, 同步操作的更改都会先保存在内存中, 直到下一次 **事务组(transaction group)** 提交时才写出。
    只要下一次事务组提交成功完成, ZFS 就能在 log 设备丢失的情况下继续运行。
    如果系统在日志设备丢失时发生崩溃, 存储池将出现故障。
    虽然可以恢复, 但当前事务组中进行的任何同步更改都将丢失, 且存储在该存储池上的数据集将无法保留这些更改

### 数据完整性

ZFS 通过多种机制来确保数据完整性:

- 已提交的数据存储在一个 **Merkle树** 中, 并且该树会在每一次**事务组**提交时以原子方式更新
- Merkle树 使用存储在 **区块指针(block pointers)** 中的256位 **校验和(checksums)** 来防止 **错误写入(misdirected writes)**, 包括那些在较弱校验和下更容易发生碰撞的情况。
    为了提供密码学级别的强校验保证, ZFS 支持使用 **sha256** 作为校验算法, 尽管缺省算法是 **fletcher4**
- 每个 磁盘/文件 vdev 都包含四个**硬盘标识(disk labels)**, 磁盘标识在磁盘两端各有两个, 这样即使磁头跌落导致任一端的数据损坏, 也不会使这些标签被清除
- 事务组提交使用两个阶段, 以确保在事务组被视为已提交之前, 所有数据都已经写入存储。
    这也是为什么 ZFS 在每个磁盘的两端都放置了两个标识的原因。
    在机械存储设备上执行事务组提交时, 需要进行一次 完整的磁头扫描, 并使用刷新操作来确保后半部分操作不会在其他操作之前执行
- ZIL记录用于存储同步I/O的待处理更改, 这些记录是 **自校验块(self checksumming blocks)**, 只有在最后一个事务组提交之前系统进行了更改, 则在池导入时这些记录为只读
- 缺省情况下, 所有元数据都会被存储两份, 而包含标识所指向的特定事务组中池状态的对象则会被写入三份。
    ZFS会尽量将这些元数据至少间隔 1/8磁盘空间 的位置, 以防止磁头坠落导致无法恢复的损坏
- 标签中包含 **超级元数据历史(uberblock history)**, 这使得在最坏情况下, 可以将整个存储池回滚到近期某个时间点。
    使用这种恢复机制需要使用特殊命令, 因为在正常情况不应该需要它
- uberblock中包含所有 vdev GUID 的求和。
    只有当该总和与预期值相符时, uberblocks 才被视为有效。
    此机制可防止来自己销毁旧存储池的 uberblocks 被误认为有效的 uberblocks
- RAIDZ 支持 **N路镜像(N-way mirroring)** 和 最多3级奇偶校验, 以确保在使用适当冗余的情况下, 即使遇到越来越常见的 双盘故障[[1]](https://queue.acm.org/detail.cfm?id=1670144)
    (这些故障会在恢复过程中导致 RAID 5 和 双镜像阵列失效), 也不会导致 ZFS 存储池崩溃

在 FreeNAS 论坛上曾流传过一些错误信息, 称在不使用 ECC 内存的情况下,
    ZFS 的数据完整性功能会比其他文件系统差[[2]](https://forums.freenas.org/index.php?threads/ecc-vs-non-ecc-ram-and-zfs.15449/)。
    这一说法已被彻底驳斥[[3]](http://jrs-s.net/2015/02/03/will-zfs-and-non-ecc-ram-kill-your-data/)。
    所有软件都需要 ECC 内存[[4]](http://open-zfs.org/wiki/Hardware#ECC_Memory), 才能实现可靠运行, 而在这方面, ZFS 与任何其他文件系统并无不同

## 启动过程

在传统的 POSIX 系统上, 启动过程如下:

1. CPU开始执行 BIOS
2. BIOS会进行基本的硬件初始化, 并加载 bootloader
3. bootloader 会加载内核, 并传递包含 rootfs 的驱动器信息
4. 内核会进行进一步初始化, 挂载 rootfs, 然后启动 /sbin/init
5. /sbin/init 会运行脚本来启动其他所有内容, 这包括从 fstab 挂载文件系统, 以及从 exports 导出 NFS 共享

在这一过程中存在一些变体, 例如, 在 Illumos 上, bootloader 会通过 boot_archive 加载内核模块; 在 FreeBSD 上, 则会逐个加载模块。
    某些 Linux 系统会加载 initramfs, 它是一个临时的 rootfs, 包含需要加载的模块, 并将部分启动逻辑移入 **用户态(userland)**。
    initramfs 会挂载真正的 rootfs, 并通过称为 pivot_root 的操作切换到它。
    此外, EFI 系统能够直接加载操作系统内核, 从而省略 bootloader 阶段

以下是 ZFS 对启动过程做出的更改:

1. 当 rootfs 位于ZFS上时, 必须在内核能够挂载它之前导入存储池。
    在 Illumos 和 FreeBSD 上, bootloader 会将存储池信息传递给内核, 以便内核导入 root pool 并挂载 rootfs。
    在 Linux 上, 必须使用 initramfs, 直到 bootloader 对动态创建 initramfs 的支持被实现
2. 无论是否存在 root pool, 已导入的存储池都必须出现。
    这是通过从`zpool.cache`文件中读取已导入池的列表来实现的, 该文件在大多数平台上位于`/etc/zfs/zpool.cache`, 在 FreeBSD 上位于`/boot/zfs/zpool.cache`。
    该文件以 **XDR 编码的 nvlist** 格式存储, 可以通过不带参数执行`zdb`命令来读取
3. 在存储池导入之后, 必须挂载文件系统, 并导出任何文件系统共享或iSCSI LUN。
    如果某个数据集的 mountpoint property 被设置为 `legacy`, 则可以使用 fstab。
    否则, boot 脚本会在存储池导入后通过运行`zfs mount -a`来挂载所有数据集。
    同样地, 通过 NFS 或 SMB 共享的文件系统数据集, 以及通过 iSCSI 共享的 zvol 数据集, 将在挂载完成后通过`zfs share -a`进行导出或共享。
    并非所有平台都支持`zfs share -a`处理所有共享类型。
    在不支持通过`zfs share -a`实现自动化的平台上, 必须始终使用传统方法

以下表格列出了不同平台上 `zfs share -a` 命令支持的协议:

| **平台**     | **iSCSI** | **NFS** | **SMB** |
| ------------ | --------- | ------- | ------- |
| **FreeBSD**  | No        | Yes     | No      |
| **Illumos**  | Yes       | Yes     | Yes     |
| **Linux**    | No        | Yes     | Yes     |
| **Mac OS X** | No        | No      | No      |
| **其它系统** | No?       | No?     | No?     |

## 创建存储池

ZFS的管理是通过`zpool`和`zfs`命令进行的。
    要创建一个存储池, 可以使用`zpool create poolname ...`。
    在 poolname 后的部分是 vdev 树的规范。
    根数据集缺省会被挂载在`/poolname`, 除非使用`zpool create -m none`。
    可以通过将`none`替换为其它挂载点路径来指定其它挂载点

在顶层vdev关键字指定之前的任何文件或磁盘 vdev 都会成为 顶级vdev。
    在顶层 vdev 关键字之后指定的任何内容都将成为该 vdev 的子 vdev。
    如果顶级 vdev 内的磁盘或文件大小不相等, 那么其可用存储空间将被限制在其中最小的那一个。
    对于生产环境的存储池, 重要的是不要创建那些不是 raidz, mirror, log, cache 或 spare 的顶层 vdev

更多信息可以在你所使用平台的 zpool 手册页(man page) 中找到。
    在 Illumos 上, 它位于 1m 章节; 在 Linux 上, 它位于 第8章节

### 重要信息

#### 扇区对其(Sector Alignment)

ZFS 会尝试通过从硬盘中提取物理扇区大小来确保正确的对其。
    对于每个顶层 vdev, 会使用其中 **最大的物理扇区大小**, 以避免出现部分扇区修改的额外开销。
    如果磁盘报告的物理扇区大小不正确, 这种机制将无法正确工作。
    更多细节可参考`Performance Tuning`页面

#### 整盘优化(Whole disk tweaks)

当提供的是整块磁盘时, ZFS 会在不同平台上尝试进行一些小的优化。
    **在 Illumos 上**, ZFS会启用磁盘缓存以提升性能。如果提供的是分区, 则不会启用, 以保护可能共享该磁盘且无法容忍磁盘缓存的其它文件系统(例如UFS)。
    **在 Linux 上**, I/O调度器(IO elevator)会被设置为 **noop**, 以减少 CPU 开销。
    因为 ZFS 拥有自己的内部 I/O调度器, 这使得 Linux 的调度器变得多余。
    更多细节可参考`Performance Tuning`页面

#### 磁盘错误恢复控制(Drive error recovery control)

ZFS 目前尚未实现的一项重要调整, 是针对磁盘的 **错误恢复控制(error recovery control)** 。
    当磁盘遇到读取错误时, 它会反复重试读取操作, 希望从略微不同的角度读取能够成功, 从而使其能够正确重写该扇区。
    它会持续进行此操作, 直到达到了超时时间, 在许多硬盘上, 这通常是 **7秒**。
    在此期间, 磁盘可能无法处理其它 I/O 请求, 这会对 IOPS 造成严重影响, 因为事务组提交会等待所有 I/O 操作完成后才会开始下一项提交。
    在 ZFS 尚未对其控制的硬盘自动设置此项正确, 系统管理员应在每次启动时使用 `smartctl` 等工具以及系统本地配置文件来完成此操作

#### 驱动器 和/或 驱动控制器挂起(Drive and/or drive controller hangs)

同样的, 如果某个驱动器或驱动控制器发生挂起, 它会导致整个存储池挂起, 并触发 **死机(deadman)** 计时器。
    在 Illumos 上, 这会导致系统重启; 在 Linux 上, 这会尝试记录故障, 但不会执行其他操作。
    死机计时器的缺省超时时间为 1000 秒

#### 硬件(Hardware)

ZFS 的设计目标是使用通用硬件也能可靠地存储数据, 但在某些硬件配置本身就存在严重可靠性问题, 以至于没有任何文件系统能够弥补这些问题。
    有关硬件配置的建议可以在 [Hardware](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/Hardware.html#) 页面中找到

#### 性能(Performance)

ZFS 会自动进行自我调优, 但仍有一些可调参数能够让特定 APP 受益。
    这些内容在 [Performance tuning](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/index.html#performance-and-tuning) 页面中有详细说明

#### 冗余(Redundancy)

两块硬盘同时发生故障的情况相当常见 [[5]](https://queue.acm.org/detail.cfm?id=1670144), 因此不应该在生存环境中将 `raidz1 vdevs` 用于数据存储。
    两硬盘镜像(2-disk mirrors)同样存在风险, 但风险承担不及 raidz1 vdevs, 除非仅使用两块硬盘。
    这是因为, 在使用超过两个硬盘的情况下, 两块硬盘同时故障在镜像中发生的概率, 低于 raidz1 等奇偶校验配置中 任意两块硬盘发生故障的概率更低。
    但如果仅使用两个硬盘, 此时两种的故障风险相同。
    相同的风险也适用于 RAID 1 和 RAID 5

单磁盘 vdev(不包括日志和缓存设备) 绝不应该在生存环境使用, 因为它们不具备冗余性。
    不过, 包含单磁盘 vdev 的存储池可以通过 `zpool attach` 添加硬盘, 将其转换为镜像, 从而实现冗余

## 常规管理

常规管理在存储池层面通过`zpool`命令执行, 在数据集层面通过`zfs`命令执行。
    相关文档可在对应平台的`zpool`和`zfs`手册页中找到。
    在 Linux 系统中位于第 8 节, 在 Illumos 系统中位于第 1m 节

参阅 [平台/发行版文档](https://openzfs.org/wiki/System_Administration#Platform.2FDistribution_documentation)。
    需要注意的是, 一个平台的手册页通常也适用于其他平台

## 其他资源

译者提醒: 部分链接可能已经失效

### 第三方工具

- [bdrewery/zfstools](https://github.com/bdrewery/zfstools): 提供用于快照管理的工具, 使用 Ruby 编写
- [fearedbliss/bliss-zfs-scripts](https://github.com/fearedbliss/bliss-zfs-scripts): 在 MPL2.0 许可下提供 "ZFS 快照 & 备份管理", 专为 Gentoo Linux 设计
- [jimsalterjrs/sanoid](https://github.com/jimsalterjrs/sanoid): 基于策略的快照管理与复制工具, 采用 GPL 许可, 在 Linux 和 FreeBSD 上经过测试
- [Rudd-O/zfs-tools](https://github.com/Rudd-O/zfs-tools): 提供快照管理与复制工具, 使用 Python 编写
- [ZFS Watcher](http://zfswatcher.damicon.fi/): 一个用于监控存储池并发送通知的守护进程

### 通用文档

- [Features](https://openzfs.org/wiki/Features)
- [出版物与会议演讲(Publications and conference talks)](https://openzfs.org/wiki/Publications)
- [History](https://openzfs.org/wiki/History)
- [性能调优(Performance tuning)](https://openzfs.github.io/openzfs-docs/Performance%20and%20Tuning/index.html#performance-and-tuning)
- 手册页: [zdb](http://illumos.org/man/1m/zdb), [zfs](http://illumos.org/man/1m/zfs), [zpool](http://illumos.org/man/1m/zpool),
    [zpool-features](http://illumos.org/man/5/zpool-features), [zstreamdump](http://illumos.org/man/1m/zstreamdump) - 来自Illumos; 如果能提供更易阅读的版本将更受欢迎
- [Oracle Solaris ZFS管理指南](http://docs.oracle.com/cd/E26505_01/html/E37384/index.html) - 很大程度上仍然适用
- [ZFS – The Last Word in File Systems](http://www.snia.org/sites/default/files2/sdc_archives/2008_presentations/monday/JeffBonwick-BillMoore_ZFS.pdf) - 来自 SNIA 2008 的概述性介绍
- [Aaron Toponce 的 ZFS 系列博客文章](https://pthree.org/2012/04/17/install-zfs-on-debian-gnulinux/)
- [Kateley Co 的视频教程](http://kateleyco.com/?page_id=783)
- [RAID‑Z 条带宽度](http://blog.delphix.com/matt/2014/06/06/zfs-stripe-width/)
- [RAID‑Z 计算工具](http://wintelguy.com/raidcalc.pl) - 包含 RAID‑Z1/Z2/Z3 的计算功能

### 平台/发行版文档

#### FreeBSD

- 手册页: [zdb](http://www.freebsd.org/cgi/man.cgi?query=zdb&manpath=FreeBSD+8.4-RELEASE), [zfs](http://www.freebsd.org/cgi/man.cgi?query=zfs&manpath=FreeBSD+8.4-RELEASE),
    [zpool](http://www.freebsd.org/cgi/man.cgi?query=zpool&manpath=FreeBSD+8.4-RELEASE),[zpool-features](http://www.freebsd.org/cgi/man.cgi?query=zpool-features&manpath=FreeBSD+8.4-RELEASE),
    [zstreamdump](http://www.freebsd.org/cgi/man.cgi?query=zstreamdump&manpath=FreeBSD+8.4-RELEASE)
- The FreeBSD Handbook 中的 [ZFS 章节](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/zfs.html)
- [FreeBSD ZFS Wiki](https://wiki.freebsd.org/ZFS)

#### Gentoo

- [Wiki page](https://wiki.gentoo.org/wiki/ZFS)
- [Richard Yao 的 Gentoo 安装笔记](https://github.com/ryao/zfs-overlay/blob/master/zfs-install)

#### illumos

- 手册页: [zdb](http://illumos.org/man/1m/zdb), [zfs](http://illumos.org/man/1m/zfs), [zpool](http://illumos.org/man/1m/zpool),
    [zpool-features](http://illumos.org/man/5/zpool-features), [zstreamdump](http://illumos.org/man/1m/zstreamdump)
- [Wiki page](http://wiki.illumos.org/display/illumos/ZFS)
- [OpenIndiana ZFS 管理指南](http://wiki.openindiana.org/oi/ZFS)

## 页脚

[原页面](https://openzfs.org/wiki/System_Administration) 最后编辑于 2026 年 1 月 27 日 00:42

除非另有说明, 内容均以 [Creative Commons Attribution-ShareAlike](https://creativecommons.org/licenses/by-sa/3.0/deed.zh-hans) 许可协议提供

[关于 OpenZFS](https://openzfs.org/wiki/About_OpenZFS)

[ZFSwiki 为移动端页面准备的页面](https://openzfs.org/w/index.php?title=System_Administration&mobileaction=toggle_view_mobile)
