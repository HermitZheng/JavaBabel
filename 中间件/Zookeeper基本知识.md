

## Zookeeper的配置参数

**配置参数详解(主要是%ZOOKEEPER_HOME%/conf/zoo.cfg文件)**

**clientPort**客户端连接server的端口，即**对外服务端口**，一般设置为**2181**吧。

**dataDir**存储**快照文件snapshot**的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。

**tickTime**ZK中的一个**时间单元**。ZK中所有时间都是以这个时间单元为基础，进行整数倍配置的。例如，session的最小超时时间是2*tickTime。

**dataLogDir**事务日志输出目录。尽量给**事务日志**的输出配置单独的磁盘或是挂载点，这将极大的提升ZK性能。 

