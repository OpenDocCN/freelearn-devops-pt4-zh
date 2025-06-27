# 第十五章：评估

# 第一章，Ceph 规划

1.  可靠的自主管理分布式对象存储

1.  可扩展哈希下的受控复制

1.  电源丧失保护

1.  一致性

1.  BlueStore

1.  Sage Weil

1.  RADOS 块设备（RBD）

1.  对象存储守护进程（OSD）

# 第二章，使用容器部署 Ceph

1.  Vagrant

1.  否

1.  Rook

1.  容器化技术

1.  Playbook

# 第三章，BlueStore

1.  BlueStore

1.  RocksDB

1.  压缩

1.  延迟写入

1.  使用`ceph-objectstore-tool`

1.  Snappy

1.  x10

# 第四章，Ceph 与非原生协议

1.  iSCSI、NFS 和 SMB

1.  iSCSI

1.  SMB

1.  Ganesha

1.  Pacemaker corosync

1.  安全性

# 第五章，RADOS 池和客户端访问

1.  Reed Solomon，Cauchy

1.  读取、修改、写入

1.  两个奇偶校验碎片的擦除编码可以加速

1.  扁平化

1.  MDS

1.  跨多个 MDS 的分片性能

1.  RadosGW

1.  S3 和 Swift

# 第六章，使用 Librados 进行开发

1.  移除第三方协议的性能开销或访问 Ceph 的完整功能集

1.  在对象被修改时通知监视器

1.  C、C++、Python、PHP 和 Java

# 第七章，使用 Ceph RADOS 类进行分布式计算

1.  OSD

1.  Lua 和 C++

1.  性能，利用所有 Ceph 节点 CPU 的联合处理能力

1.  不稳定和不需要的数据损坏

# 第八章，监控 Ceph

1.  8443

1.  ceph-mgr — Ceph 管理器

1.  在读取或比较对象内容时，scrub 检测到错误

1.  包含一个或多个对象的 PG 正在恢复到新的 OSD

1.  尽可能多

# 第九章，调优 Ceph

1.  错误—PG 分布在新集群中不会均匀，并且可能根据工作负载随时间变化

1.  完整条带写入只需要一次写入操作，而部分写入则需要更高的写入惩罚

1.  高 GHz

1.  CPU 速度、磁盘延迟和网络速度

1.  Ceph 平衡器

# 第十章，Ceph 分层

1.  提高性能或允许使用较低成本的容量层

1.  bcache, dmcache

1.  Bloom 过滤器中存储的命中集

1.  `min_read_recency_for_promote`

1.  对少量热点对象有强烈偏向的工作负载

# 第十一章，故障排除

1.  `ceph pg repair <pg id>`

1.  `ceph tell osd.0 injectargs --debug-osd 0/5`

1.  Ceph 集群长时间不处于`HEALTH_OK`状态

1.  `ceph pg query`

1.  Scrubbing 是读取密集型操作，可能会使磁盘的 I/O 超负荷

# 第十二章，灾难恢复

1.  `rbd-mirror`

1.  正确

1.  `ceph-objectstore-tool`

1.  错误，它可能存在于当前未参与集群的 OSD 中

1.  缺失的文件名
