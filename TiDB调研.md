

## 总体架构







## 存储模型TiKV

TiKV 的选择是 **Key-Value 模型**，并且提供**有序遍历**方法。简单来讲，可以将 TiKV 看做一个巨大的 Map，其中 Key 和 Value 都是原始的 Byte 数组，在这个 Map 中，Key 按照 Byte 数组总的原始二进制比特位比较顺序排列。

1. 这是一个巨大的 Map，也就是存储的是 Key-Value pair
2. 这个 Map 中的 Key-Value pair 按照 Key 的**二进制顺序有序**，也就是我们可以 Seek 到某一个 Key 的位置，然后不断的调用 Next 方法以递增的顺序获取比这个 Key 大的 Key-Value



### 存储引擎RocksDB

 TiKV 没有选择直接向磁盘上写数据，而是把数据保存在 **RocksDB** 中，具体的数据落地由 RocksDB 负责。RocksDB 是一个非常优秀的开源的单机存储引擎，可以简单的认为 RocksDB 是一个单机的 Key-Value Map。



### 一致性算法Raft

如何保证单机失效的情况下，数据不丢失，不出错？简单来说，我们需要想办法把数据复制到多台机器上，这样一台机器挂了，我们还有其他的机器上的副本；复杂来说，我们还需要这个复制方案是可靠、高效并且能处理副本失效的情况。

Raft 是一个一致性协议，提供几个重要的功能：

1. Leader 选举
2. 成员变更
3. 日志复制

TiKV 利用 Raft 来做数据复制，**每个数据变更都会落地为一条 Raft 日志**，通过 Raft 的日志复制功能，将数据安全可靠地同步到 Group 的多数节点中。

到这里我们总结一下，通过单机的 RocksDB，我们可以将数据快速地存储在磁盘上；通过 Raft，我们可以将数据复制到多台机器上，以防单机失效。数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。通过实现 Raft，我们拥有了一个分布式的 KV，现在再也不用担心某台机器挂掉了。



### Region

对于一个 KV 系统，将数据分散在多台机器上有两种比较典型的方案：一种是**按照 Key 做 Hash**，根据 Hash 值选择对应的存储节点；另一种是**分 Range**，**某一段连续的 Key 都保存在一个存储节点上**。TiKV 选择了**第二种**方式，将整个 Key-Value 空间分成很多段，每一段是一系列连续的 Key，我们将每一段叫做一个 **Region**，并且我们会尽量保持每个 Region 中保存的数据不超过一定的大小(这个大小可以配置，目前默认是 96mb)。每一个 Region 都可以用 StartKey 到 EndKey 这样一个左闭右开区间来描述。

将数据划分成 Region 后，我们将会做 **两件重要的事情**：

- 以 Region 为单位，将数据分散在集群中所有的节点上，并且尽量保证**每个节点上服务的 Region 数量差不多**
  - 系统会有一个组件来负责将 Region 尽可能均匀的散布在集群中所有的节点上，这样一方面实现了**存储容量的水平扩展**（增加新的结点后，会自动将其他节点上的 Region 调度过来），另一方面也实现了**负载均衡**（不会出现某个节点有很多数据，其他节点上没什么数据的情况）。
  - 也会有一个组件**记录 Region 在节点上面的分布情况**，也就是通过任意一个 Key 就能查询到这个 Key 在哪个 Region 中，以及这个 Region 目前在哪个节点上。
- 以 Region 为单位做 **Raft 的复制和成员管理**
  - TiKV 是以 Region 为单位做数据的复制，也就是**一个 Region 的数据会保存多个副本**，我们将每一个副本叫做一个 **Replica**。Replica 之间是通过 Raft 来保持数据的一致，一个 Region 的多个 Replica 会保存在不同的节点上，构成一个 Raft Group。
  - 其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower。所有的读和写都是通过 Leader 进行，再由 Leader 复制给 Follower。 



### MVCC

TiKV 的 MVCC 实现是通过在 Key 后面添加 **Version** 来实现的。

```
	Key1 -> Value
	Key2 -> Value
	……
	KeyN -> Value
	
	Key1-Version3 -> Value
	Key1-Version2 -> Value
	Key1-Version1 -> Value
	……
	Key2-Version4 -> Value
	Key2-Version3 -> Value
	Key2-Version2 -> Value
	Key2-Version1 -> Value
	……
	KeyN-Version2 -> Value
	KeyN-Version1 -> Value
	……
```

**对于同一个 Key 的多个版本，我们把版本号较大的放在前面，版本号小的放在后面。**当用户通过一个 Key + Version 来获取 Value 的时候，可以将 Key 和 Version 构造出 MVCC 的 Key，也就是 Key-Version。然后可以直接 Seek(Key-Version)，定位到**第一个大于等于**这个 Key-Version （最新版本）的位置。



## 从SQL关系模型到KV结构

对于一个 Table 来说，需要存储的数据包括三部分：

1. 表的元信息
2. Table 中的 Row
3. 索引数据

对于 Row，可以选择行存或者列存，这两种各有优缺点。TiDB 面向的首要目标是 OLTP 业务，这类业务需要支持快速地读取、保存、修改、删除一行数据，所以采用**行存储**是比较合适的。

### 索引Index

对于 Index，TiDB 不止需要支持 Primary Index，还需要支持 Secondary Index。Index 的作用的辅助查询，提升查询性能，以及保证某些 Constraint。查询的时候有两种模式，一种是点查，另一种是 Range 查询。Index 还分为 Unique Index 和 非 Unique Index，这两种都需要支持。

TiDB 对每个表分配一个 TableID，每一个索引都会分配一个 IndexID，每一行分配一个 RowID（如果表有整数型的 Primary Key，那么会用 Primary Key 的值当做 RowID），其中 TableID 在整个集群内唯一，IndexID/RowID 在表内唯一，这些 ID 都是 int64 类型。

每行数据按照如下规则进行编码成 Key-Value pair：

```
Key: tablePrefix{tableID}_recordPrefixSep{rowID}
Value: [col1, col2, col3, col4]
// index
Key: tablePrefix{tableID}_indexPrefixSep{indexID}_indexedColumnsValue
Value: rowID
```



## TiDB Server

TiDB Servers 这一层的节点都是无状态的节点，本身并不存储数据，节点之间完全对等。TiDB Server 这一层最重要的工作是处理用户请求，执行 SQL 运算逻辑。

### SQL运算

将 SQL 查询映射为对 KV 的查询，再通过 KV 接口获取对应的数据，最后执行各种计算。比如 `Select count(*) from user where name="TiDB";` 这样一个语句，我们需要读取表中所有的数据，然后检查 `Name` 字段是否是 `TiDB`，如果是的话，则返回这一行。

这样一个操作流程转换为 KV 操作流程：

- 构造出 Key Range：一个表中所有的 RowID 都在 `[0, MaxInt64)` 这个范围内，那么我们用 0 和 MaxInt64 根据 Row 的 Key 编码规则，就能构造出一个 `[StartKey, EndKey)` 的左闭右开区间
- 扫描 Key Range：根据上面构造出的 Key Range，读取 TiKV 中的数据
- 过滤数据：对于读到的每一行数据，计算 `name="TiDB"` 这个表达式，如果为真，则向上返回这一行，否则丢弃这一行数据
- 计算 Count：对符合要求的每一行，累计到 Count 值上面

这个方案肯定是可以 Work 的，但是并不能 Work 的很好，原因是显而易见的：

1. 在扫描数据的时候，每一行都要通过 KV 操作同 TiKV 中读取出来，至少有一次 RPC 开销，如果需要扫描的数据很多，那么这个开销会非常大
2. 并不是所有的行都有用，如果不满足条件，其实可以不读取出来
3. 符合要求的行的值并没有什么意义，实际上这里只需要有几行数据这个信息就行

### 分布式 SQL 运算





## 实践



### 在小米的应用实践

目前 TiDB 在小米MIUI主要应用在：

- 小米手机桌面负一屏的快递业务
- 商业广告交易平台素材抽审平台

**这两个业务场景每天读写量均达到上亿级，上线之后，整个服务稳定运行；接下来我们计划逐步上线更多的业务场景，小米阅读目前正在积极的针对订单系统做迁移测试。**







