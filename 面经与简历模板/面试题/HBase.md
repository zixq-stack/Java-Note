## HBase

### 概念

base是分布式、面向列的开源数据库（其实准确的说是面向列族）。HDFS为Hbase提供可靠的底层数据存储服务，MapReduce为Hbase提供高性能的计算能力，Zookeeper为Hbase提供稳定服务和Failover机制，因此我们说Hbase是一个通过大量廉价的机器解决海量数据的高速存储和读取的分布式数据库解决方案。

### 14.1.2. 列式存储

列方式所带来的重要好处之一就是，由于查询中的选择规则是通过列来定义的，因此整个数据库是自动索引化的。

![image-20240205193240502](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205194953063-1018494842.png)

这里的列式存储其实说的是列族存储，Hbase是根据列族来存储数据的。列族下面可以有非常多的列，列族在创建表的时候就必须指定。为了加深对Hbase列族的理解，下面是一个简单的关系型数据库的表和Hbase数据库的表：

![image-20240205193303924](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205194952969-157856600.png)



### Hbase 核心概念

#### Column Family列族

Column Family又叫列族，Hbase通过列族划分数据的存储，列族下面可以包含任意多的列，实现灵活的数据存取。Hbase 表的创建的时候就必须指定列族。就像关系型数据库创建的时候必须指定具体的列是一样的。Hbase的列族不是越多越好，官方推荐的是列族最好小于或者等于 3 。我们使用的场景一般是 1 个列族。

#### Rowkey（Rowkey查询，Rowkey范围扫描，全表扫描）

Rowkey的概念和mysql中的主键是完全一样的，Hbase使用Rowkey来唯一的区分某一行的数据。Hbase只支持 3 中查询方式：基于Rowkey的单行查询，基于Rowkey的范围扫描，全表扫描。

#### Region分区

**Region** ：Region的概念和关系型数据库的分区或者分片差不多。Hbase会将一个大表的数据基于Rowkey的不同范围分配到不通的Region中，每个Region负责一定范围的数据访问和存储。这样即使是一张巨大的表，由于被切割到不通的region，访问起来的时延也很低。

#### TimeStamp多版本

TimeStamp是实现Hbase多版本的关键。在Hbase中使用不同的timestame来标识相同rowkey行对应的不通版本的数据。在写入数据的时候，如果用户没有指定对应的timestamp，Hbase会自动添加一个timestamp，timestamp和服务器时间保持一致。在Hbase中，相同rowkey的数据按照timestamp倒序排列。默认查询的是最新的版本，用户可同指定timestamp的值来读取旧版本的数据。

### Hbase 核心架构

Hbase是由Client、Zookeeper、Master、HRegionServer、HDFS等几个组建组成。

![image-20240205193433727](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205194953445-922989002.png)

#### Client：

Client包含了访问Hbase的接口，另外Client还维护了对应的cache来加速Hbase的访问，比如cache的.META.元数据的信息。

#### Zookeeper：......

Hbase通过Zookeeper来做master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作。具体工作如下：

1. 通过Zoopkeeper来保证集群中只有 1 个master在运行，如果master异常，会通过竞争机制产生新的master提供服务
2. 通过Zoopkeeper来监控RegionServer的状态，当RegionSevrer有异常的时候，通过回调的形式通知Master RegionServer上下限的信息
3. 通过Zoopkeeper存储元数据的统一入口地址。

#### Hmaster

master节点的主要职责如下：

1. 为RegionServer分配Region
2. 维护整个集群的负载均衡
3. 维护集群的元数据信息发现失效的Region，并将失效的Region分配到正常RegionServer上当RegionSever失效的时候，协调对应Hlog的拆分

#### HregionServer

HregionServer直接对接用户的读写请求，是真正的“干活”的节点。它的功能概括如下：

1. 管理master为其分配的Region


2. 处理来自客户端的读写请求
3. 负责和底层HDFS的交互，存储数据到HDFS
4. 负责Region变大以后的拆分
5. 负责Storefile的合并工作

#### Region寻址方式（通过zookeeper .META）

第 1 步：Client请求ZK获取.META.所在的RegionServer的地址。
第 2 步：Client请求.META.所在的RegionServer获取访问数据所在的RegionServer地址，client会将.META.的相关信息cache下来，以便下一次快速访问。
第 3 步：Client请求数据所在的RegionServer，获取所需要的数据。

![image-20240205193553162](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205194953044-293207136.png)



#### HDFS

HDFS为Hbase提供最终的底层数据存储服务，同时为Hbase提供高可用（Hlog存储在HDFS）的支持。



### Hbase 的写逻辑

#### Hbase的写入流程

![image-20240205193638550](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205194952950-79445337.png)

从上图可以看出氛围 3 步骤：

1. **获取RegionServer**

第 1 步：Client获取数据写入的Region所在的RegionServer

2. **请求写Hlog**

第 2 步：请求写Hlog, Hlog存储在HDFS，当RegionServer出现异常，需要使用Hlog来恢复数据。

3. **请求写MemStore**

第 3 步：请求写MemStore,只有当写Hlog和写MemStore都成功了才算请求写入完成。MemStore后续会逐渐刷到HDFS中。



#### MemStore刷盘

为了提高Hbase的写入性能，当写请求写入MemStore后，不会立即刷盘。而是会等到一定的时候进行刷盘的操作。具体是哪些场景会触发刷盘的操作呢？总结成如下的几个场景：

1. **全局内存控制**

这个全局的参数是控制内存整体的使用情况，当所有memstore占整个heap的最大比
例的时候，会触发刷盘的操作。这个参数是`hbase.regionserver.global.memstore.upperLimit`，默认为整个heap内存的40%。但这并不意味着全局内存触发的刷盘操作会将所有的MemStore都进行输盘，而是通过另外一个参数`hbase.regionserver.global.memstore.lowerLimit`来控制，默认是整个heap内存的35%。当flush到所有memstore占整个heap内存的比率为35%的时候，就停止刷盘。这么做主要是为了减少刷盘对业务带来的影响，实现平滑系统负载的目的。

2. **MemStore达到上限.............**

当MemStore的大小达到hbase.hregion.memstore.flush.size大小的时候会触发刷盘，默认128M大小

3. **RegionServer的Hlog数量达到上限**

前面说到Hlog为了保证Hbase数据的一致性，那么如果Hlog太多的话，会导致故障恢复的时间太长，因此Hbase会对Hlog的最大个数做限制。当达到Hlog的最大个数的时候，会强制刷盘。这个参数是hase.regionserver.max.logs，默认是 32 个。

4. **手工触发**

可以通过hbase shell或者java api手工触发flush的操作。

5. **关闭RegionServer触发**

在正常关闭RegionServer会触发刷盘的操作，全部数据刷盘后就不需要再使用Hlog恢复数据。

6. **Region使用HLOG恢复完数据后触发..............**

当RegionServer出现故障的时候，其上面的Region会迁移到其他正常的RegionServer上，在恢复完Region的数据后，会触发刷盘，当刷盘完成后才会提供给业务访问。



### HBase vs Cassandra

|                    | HBase                                                        | Cassandra                                                    |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 语言               | Java                                                         | Java                                                         |
| 出发点             | BigTable                                                     | BigTable and Dynamo                                          |
| License            | Apache                                                       | Apache                                                       |
| Protocol           | HTTP/REST (also Thrift)                                      | Custom, binary (Thrift)                                      |
| 数据分布           | 表划分为多个 region 存在不同 region<br/server 上             | 改进的一致性哈希（虚拟节点）                                 |
| 存储目标           | 大文件                                                       | 小文件                                                       |
| 一致性             | 强一致性                                                     | 最终一致性，Quorum NRW 策略                                  |
| 高可用性           | NameNode 是 HDFS 的单点故障点                                | P2P 和去中心化设计，不会出现单点故障                         |
| 伸缩性             | Region Server 扩容，通过将自身发布到Master，Master 均匀分布 Region | 扩容需在 Hash Ring 上多个节点间调整数据分布                  |
| 读写性能           | 数据读写定位可能要通过最多 6 次的网<br/络 RPC，性能较低。    | 数据读写定位非常快                                           |
| 数据冲突处理       | 乐观并发控制（optimistic concurrency control）               | 向量时钟                                                     |
| 临时故障处理       | Region Server 宕机，重做 HLog                                | 数据回传机制：某节点宕机，hash 到该节点的新数据自动路由到下一节点做 hinted handoff，源节点恢复后，推<br/送回源节点。 |
| 永久故障恢复       | Region Server 恢复，master 重新给其<br/分配 region           | Merkle 哈希树，通过 Gossip 协议同步 Merkle Tree，维护集群节点间的数据一致性 |
| 成员通信及错误检测 | Zookeeper                                                    | 基于 Gossip                                                  |
| CAP                | 1，强一致性，0 数据丢失。<br />2，可用性<br/低。<br />3，扩容方便。 | 1，弱一致性，数据可能丢失。<br />2，可用性高。<br />3，扩容方便。 |
| 架构               | master/slave                                                 | p2p                                                          |















