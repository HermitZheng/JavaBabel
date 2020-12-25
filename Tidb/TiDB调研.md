

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

### JDBC

#### 使用 Batch 批量插入更新

对于批量插入更新，如果插入记录较多，可以选择使用 [addBatch/executeBatch API](https://www.tutorialspoint.com/jdbc/jdbc-batch-processing)。通过 `addBatch` 的方式将多条 SQL 的插入更新记录先缓存在客户端，然后在 `executeBatch` 时一起发送到数据库服务器。

> 注意：
>
> 对于 MySQL Connector/J 实现，默认 Batch 只是将多次 `addBatch` 的 SQL 发送时机延迟到调用 `executeBatch` 的时候，但**实际网络发送还是会一条条的发送，通常不会降低与数据库服务器的网络交互次数**。
>
> 如果希望 Batch 网络发送**批量插入**，需要在 JDBC 连接参数中配置 **`rewriteBatchedStatements=true`**。



#### 使用 StreamingResult 流式获取执行结果

一般情况下，为提升执行效率，JDBC 会默认**提前获取查询结果并将其保存在客户端内存**中。但在查询返回超大结果集的场景中，客户端会希望数据库服务器减少向客户端一次返回的记录数，等客户端在有限内存处理完一部分后再去向服务器要下一批。

在 JDBC 中通常有以下两种处理方式：

- 设置 `FetchSize` 为 `Integer.MIN_VALUE`让**客户端不缓存**，客户端通过 `StreamingResult` 的方式从网络连接上**流式读取**执行结果。
- 使用 Cursor Fetch 首先需设置 `FetchSize`为正整数且在 JDBC URL 中配置 `useCursorFetch=true`。

TiDB 中同时支持两种方式，但**更推荐使用第一种将 `FetchSize` 设置为 `Integer.MIN_VALUE` 的方式**，比第二种功能实现更简单且执行效率更高。



### Mybatis

MyBatis 是目前比较流行的 Java 数据访问框架，主要用于管理 SQL 并完成结果集和 Java 对象的来回映射工作。MyBatis 和 TiDB 兼容性很好，从历史 issue 可以看出 MyBatis 很少出现问题。这里主要关注如下几个配置。

#### Mapper 参数

MyBatis 的 Mapper 中支持两种参数：

- `select 1 from t where id = #{param1}` 会作为 prepare 语句转换为 `select 1 from t where id = ?` 进行 prepare， 并使用实际参数来复用执行。
- `select 1 from t where id = ${param2}` 会做文本替换为 `select 1 from t where id = 1` 执行，如果这条语句被 prepare 成了不同参数，可能会**导致 TiDB 缓存大量的 prepare 语句**，并且这种方式执行 SQL 有**注入**安全风险。



#### 动态 SQL Batch

要支持将多条 insert 语句自动重写为 `insert ... values(...), (...), ...` 的形式，除了前面所说的在 JDBC 配置 `rewriteBatchedStatements=true` 外，MyBatis 还可以使用动态 SQL 的 **foreach** 来半自动生成 batch insert。比如下面的 mapper:

```xml
<insert id="insertTestBatch" parameterType="java.util.List" fetchSize="1">
  insert into test
   (id, v1, v2)
  values
  <foreach item="item" index="index" collection="list" separator=",">
  (
   #{item.id}, #{item.v1}, #{item.v2}
  )
  </foreach>
  on duplicate key update v2 = v1 + values(v1)
</insert>
```

会生成一个 `insert on duplicate key update` 语句，values 后面的 `(?, ?, ?)` 数目是根据传入的 list 个数决定，最终效果和使用 `rewriteBatchStatements=true` 类似，**可以有效减少客户端和 TiDB 的网络交互次数**，同样需要注意 prepare 后超过 `prepStmtCacheSqlLimit` 限制导致不缓存 prepare 语句的问题。



#### Streaming 结果

前面介绍了在 JDBC 中如何使用流式读取结果，除了 JDBC 相应的配置外，在 MyBatis 中如果希望读取超大结果集合也需要注意：

- 可以通过在 mapper 配置中对单独一条 SQL **设置 `fetchSize`**，效果等同于调用 JDBC `setFetchSize`。
- 可以使用带 `ResultHandler` 的查询接口来避免一次获取整个结果集。
- 可以使用 `Cursor` 类来进行流式读取。

对于使用 xml 配置映射，可以通过在映射 `<select>` 部分配置 `fetchSize="-2147483648"`(`Integer.MIN_VALUE`) 来流式读取结果。

```xml
<select id="getAll" resultMap="postResultMap" fetchSize="-2147483648">
  select * from post;
</select>
```

```java
public List<Integer> selectForwardOnly() {
    final List<Integer> list = new ArrayList<>();
    testMapper.selectForwardOnly(new ResultHandler<Integer>() {
        @Override
        public void handleResult(ResultContext<? extends Integer> resultContext) {
            /**回调处理逻辑 */
            list.add(resultContext.getResultObject());
        }
    });
    return list;
}
```

而使用代码配置映射，则可以使用 `@Options(fetchSize = Integer.MIN_VALUE)` 并返回 `Cursor` 从而让 SQL 结果能被流式读取。

```java
@Select("select * from post")
@Options(fetchSize = Integer.MIN_VALUE)
Cursor<Post> queryAllPost();
```



#### ExecutorType

在 `openSession` 的时候可以选择 `ExecutorType`，MyBatis 支持三种 `executor`：

- `Simple`：每次执行都会向 JDBC 进行 prepare 语句的调用（如果 JDBC 配置有开启 `cachePrepStmts`，重复的 prepare 语句会复用）。
- `Reuse`：在 `executor` 中缓存 prepare 语句，这样不用 JDBC 的 `cachePrepStmts` 也能减少重复 prepare 语句的调用。
- `Batch`：每次更新只有在 `addBatch` 到 query 或 commit 时才会调用 `executeBatch` 执行，如果 JDBC 层开启了 `rewriteBatchStatements`，则会尝试改写，没有开启则会一条条发送。

通常默认值是 `Simple`，需要在调用 `openSession` 时改变 `ExecutorType`。如果是 Batch 执行，会遇到事务中前面的 update 或 insert 都非常快，而在**读数据或 commit 事务时比较慢**的情况，这实际上是正常的，在排查慢 SQL 时需要注意。

```java
configuration.setDefaultExecutorType(ExecutorType.Batch);
```



### Spring Transaction

在应用代码中业务可能会通过使用 [Spring Transaction](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html) 和 AOP 切面的方式来启停事务。

通过在方法定义上添加 `@Transactional` 注解标记方法，AOP 将会在方法前开启事务，方法返回结果前 commit 事务。如果遇到类似业务，可以通过查找代码 `@Transactional` 来确定事务的开启和关闭时机。需要特别注意有内嵌的情况，**如果发生内嵌，Spring 会根据 [Propagation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Propagation.html) 配置使用不同的行为，因为 TiDB 未支持 savepoint，所以不支持嵌套事务。**



## 例子





### 在小米的应用实践

目前 TiDB 在小米MIUI主要应用在：

- 小米手机桌面负一屏的快递业务
- 商业广告交易平台素材抽审平台

**这两个业务场景每天读写量均达到上亿级，上线之后，整个服务稳定运行；接下来计划逐步上线更多的业务场景，小米阅读目前正在积极的针对订单系统做迁移测试。**

小米关系型存储数据库首选 MySQL，单机 2.6T 磁盘。由于小米手机销量的快速上升和 MIUI 负一屏用户量的快速增加，导致负一屏快递业务数据的数据量增长非常快， **每天的读写量级均分别达到上亿级别，数据快速增长导致单机出现瓶颈，比如性能明显下降、可用存储空间不断降低、大表 DDL 无法执行等，不得不面临数据库扩展的问题。** 比如，我们有一个业务场景（智能终端），需要定时从几千万级的智能终端高频的向数据库写入各种监控及采集数据，**MySQL 基于 Binlog 的单线程复制模式**，很容易造成**从库延迟**，并且**堆积**越来越严重。

**对于 MySQL 来讲，最直接的方案就是采用分库分表的水平扩展方式，综合来看并不是最优的方案，比如对于业务来讲，对业务代码的侵入性较大；对于 DBA 来讲提升管理成本，后续需要不断的拆分扩容，即使有中间件也有一定的局限性。** 同样是上面的智能终端业务场景，从业务需求看，需要从多个业务维度进行查询，并且业务维度可能随时进行扩展，分表的方案基本不能满足业务的需求。



#### 兼容性对比

**TiDB 支持包括跨行事务、JOIN、子查询在内的绝大多数 MySQL 的语法，可以直接使用 MySQL 客户端连接；对于已用 MySQL 的业务来讲，基本可以无缝切换到 TiDB。**

二者简单对比如下几方面：

- 功能支持
  - TiDB 尚不支持如下几项：
    - 增加、删除主键
    - 非 UTF8 字符集
    - 视图（即将支持）、存储过程、触发器、部分内置函数
    - Event
    - 全文索引、空间索引
- 默认设置
  - 字符集、排序规则、sql_mode、lower_case_table_names 几项默认值不同。
- 事务
  - TiDB 使用乐观事务模型，提交后注意检查返回值。
  - TiDB 限制单个事务大小，保持事务尽可能的小。
- TiDB 支持绝大多数的 Online DDL。
- 另，一些 MySQL 语法在 TiDB 中可以解析通过，不会产生任何作用，例如： create table 语句中 engine、partition 选项都是在解析后忽略。
- 详细信息可以访问官网：[与 MySQL 兼容性对比](https://pingcap.com/docs-cn/v3.0/reference/mysql-compatibility/)。



#### 迁移过程

整个迁移分为 2 大块：数据迁移、流量迁移。

##### 数据迁移

数据迁移分为增量数据、存量数据两部分。

- 对于存量数据，可以使用逻辑备份、导入的方式，除了传统的逻辑导入外，官方还提供一款物理导入的工具 TiDB Lightning。
- 对于增量备份可以使用 TiDB 提供的 Syncer （新版已经更名为 DM - Data Migration）来保证数据同步。

Syncer 结构如图 6，主要依靠各种 Rule 来实现不同的过滤、合并效果，一个同步源对应一个 Syncer 进程，同步 Sharding 数据时则要多个 Syncer 进程。

![图 6 Syncer 结构图](https://download.pingcap.com/images/blog-cn/user-case-xiaomi/6.png)

使用 Syncer 需要注意：

- 做好同步前检查，包含 server-id、log_bin、binlog_format 是否为 ROW、binlog_row_image 是否为 FULL、同步相关用户权限、Binlog 信息等。
- 使用严格数据检查模式，数据不合法则会停止。数据迁移之前最好针对数据、表结构做检查。
- 做好监控，TiDB 提供现成的监控方案。
- 对于已经分片的表同步到同一个 TiDB 集群，要做好预先检查。确认同步场景是否可以用 route-rules 表达，检查分表的唯一键、主键在数据合并后是否冲突等。

##### 流量迁移

流量切换到 TiDB 分为两部分：读、写流量迁移。每次切换保证灰度过程，观察周期为 1~2 周，做好回滚措施。

- 读流量切换到 TiDB，这个过程中回滚比较简单，灰度无问题，则全量切换。
- 再将写入切换到 TiDB，需要考虑好数据回滚方案或者采用双写的方式（需要断掉 Syncer）。