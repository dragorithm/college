# Feature

## 功能标识(Feature Flags)

ZFS 的磁盘格式最初是用一个单一的版本号来标记的, 每当格式发生变化时, 这个数字就会增加。
    在 ZFS 由单一组织开发时, 这种编号方式是合适的

但在 OpenZFS 的分布式开发模式下, 版本号方式就不再适用, 因为任何一次编号的变化, 都需要所有实现之间对磁盘格式的每一次修改达成一致

OpenZFS 引入了**功能标识**作为传统版本编号的替代方案。
    它允许为每一次磁盘格式的变化定义一个唯一命名的池属性
    这种方式支持:

独立的格式变化

彼此依赖的格式变化

## 兼容性

当一个存储池所使用的所有功能都被多个 OpenZFS 实现支持时, 该磁盘格式在这些实现之间是可移植的

启用后具有排他性的功能应定期移植至所有发行版(这句话是对开发全新feature的说的)

## 参考资料

[ZFS 功能标识](http://web.archive.org/web/20160419064650/http://blog.delphix.com/csiden/files/2012/01/ZFS_Feature_Flags.pdf)
(Christopher Siden, 2012-01, 收录于 Internet Archive Wayback Machine), 其中特别提到"...传统的版本号仍然存在于池版本 1-28..."

[zpool-features(7) man 手册页](https://openzfs.github.io/openzfs-docs/man/7/zpool-features.7.html) - OpenZFS

[zpool-features (5)](http://illumos.org/man/5/zpool-features) - illumos

## 按操作系统实现的功能标识

由于信息内容过多且容易阅读, 推荐直接看官方文档

[Feature flags implementation per OS](https://openzfs.github.io/openzfs-docs/Basic%20Concepts/Feature%20Flags.html#feature-flags-implementation-per-os)

## 备注

1. [在FreeBSD上使用OpenZFS <= 2.1版本中都不能选择Edonr作为校验算法](https://github.com/openzfs/zfs/pull/12735)
2. [虽然 Illumos 的加密逻辑几乎相同, 但它的格式与 OpenZFS 的加密格式并非 100% 兼容,
    因此在跨平台加密时可能会遇到问题](https://illumos.topicbox.com/groups/discuss/Ta2162fbb2358fa0e)

## 源

[2026-03-12T03:53:25.426195Z](https://openzfs.github.io/openzfs-docs/Basic%20Concepts/Feature%20Flags.html#)
