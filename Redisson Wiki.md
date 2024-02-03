# 概览
> Redisson is a Redis Java client with features of In-Memory Data Grid. It provides more convenient and easiest way to work with Redis. Redisson objects provides a separation of concern, which allows you to keep focus on the data modeling and application logic.
> Compatible with follow Redis cloud providers:
> - [AWS ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Replication.html)
> - [AWS ElastiCache Cluster](https://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Clusters.html)
> - [Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/)
> - [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/)
> - [HUAWEI Distributed Cache Service for Redis](https://www.huaweicloud.com/en-us/product/dcs.html)
> - [Aiven Redis hosting](https://aiven.io/redis)
> - [RLEC Multiple Active Proxy](https://docs.redislabs.com/latest/rs/administering/designing-production/networking/multiple-active-proxy/)
> - Many others
> 
Use [Redis commands mapping](#znY80) table to find Redisson method for a particular Redis command.
> Based on [Netty](http://netty.io/) framework. [Redis](http://redis.io/) 3.0+ and JDK 1.8+ compatible.
> Try [Redisson PRO](https://redisson.pro/) with **ultra-fast performance** and **support by SLA**

Redisson 是一个具有内存数据网格功能的 Redis Java 客户端。它提供了更方便、最简单的使用 Redis 的方法。 Redisson 对象提供了关注点分离，使您能够将注意力集中在数据建模和应用程序逻辑上。 与以下 Redis 云提供商兼容：

- [AWS ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Replication.html)
- [AWS ElastiCache Cluster](https://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Clusters.html)
- [Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/)
- [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/)
- [HUAWEI Distributed Cache Service for Redis](https://www.huaweicloud.com/en-us/product/dcs.html)
- [Aiven Redis hosting](https://aiven.io/redis)
- [RLEC Multiple Active Proxy](https://docs.redislabs.com/latest/rs/administering/designing-production/networking/multiple-active-proxy/)

使用 [Redis 命令映射表](#znY80)查找特定 Redis 命令的 Redisson 方法。 基于Netty框架。 Redis 3.0+ 和 JDK 1.8+ 兼容。 尝试 [Redisson PRO](https://redisson.pro/) 的超快性能和 SLA 支持。
# 配置
## 程序化配置
由 Config 对象实例执行的编程配置。例如：
```java
Config config = new Config();
config.setTransportMode(TransportMode.EPOLL);
config.useClusterServers()
       // use "rediss://" for SSL connection
      .addNodeAddress("perredis://127.0.0.1:7181");

RedissonClient redisson = Redisson.create(config);
```
## 声明式配置
Redisson 还可以通过使用用户提供的 YAML 格式的文本文件以声明方式进行配置
### 基于 YAML 文件的配置
Redisson 配置可以以 YAML 格式存储。
使用 `config.fromYAML` 方法读取以 YAML 格式存储的配置：
```
Config config = Config.fromYAML(new File("config-file.yaml"));  
RedissonClient redisson = Redisson.create(config);
```
使用 `config.toYAML` 方法以 YAML 格式编写配置：
```
Config config = new Config();
// ... many settings are set here
String yamlFormat = config.toYAML();
```
环境变量应包含在 `${...}` 中。比如：
```
singleServerConfig:
  address: "redis://127.0.0.1:${REDIS_PORT}"
```
默认值符合 shell 格式。比如：
```
singleServerConfig:
  address: "redis://127.0.0.1:${REDIS_PORT:-6379}"
```
## 常用设置
以下设置属于 `org.redisson.Config` 对象，并且对于所有模式都是通用的：
### codec
默认值：`org.redisson.codec.Kryo5Codec`
Redis 数据编解码器。在读写Redis数据时使用。有[多种实现方式](#tX4M3)可供选择
### connectionListener
默认值：`null`
当Redisson连接/断开与Redis服务器的连接时触发的连接监听器。
### lazyInitialization
默认值：`false`
定义Redisson是否仅在第一次Redis调用时而不是在Redisson实例创建期间连接到Redis。
`true`- 仅当第一次调用 Redis 时才连接到 Redis
`false`- 在Redisson实例创建期间连接到Redis
### nettyThreads
默认值：32
Redisson 使用的所有内部 Redis 客户端之间共享的线程数。 Netty线程用于Redis响应解码和命令发送。 0 = 核心数量 * 2
### nettyHook
默认值：空对象
Netty hook 应用于 Netty Bootstrap 和 Channel 对象
### executor
使用外部 ExecutorService 处理 RTopic、RRemoteService 调用处理程序和 RExecutorService 任务的所有侦听器。
### eventLoopGroup
使用外部 EventLoopGroup。 EventLoopGroup 通过自己的线程处理与 Redis 服务器绑定的所有 Netty 连接。默认情况下，每个 Redisson 客户端都会创建自己的 EventLoopGroup。因此，如果同一个 JVM 中有多个 Redisson 实例，那么在它们之间共享一个 EventLoopGroup 会很有用。
可选的值有`io.netty.channel.epoll.EpollEventLoopGroup`、`io.netty.channel.kqueue.KQueueEventLoopGroup` 和 `io.netty.channel.nio.NioEventLoopGroup`。
### transportMode
默认值：`TransportMode.NIO`
可选的值有：
`TransportMode.NIO`，
`TransportMode.EPOLL` - classpath中需要有`netty-transport-native-epoll`库
`TransportMode.KQUEUE` - classpath中需要有`netty-transport-native-kqueue`库
### threads
默认值：16
线程用于执行 RTopic 对象的侦听器逻辑、RRemoteService、RTopic 对象和 RExecutorService 任务的调用处理程序。
### protocol
默认值：`RESP2`
定义 Redis 协议版本。可用值：`RESP2`、`RESP3`
### lockWatchdogTimeout
默认值：`30000`
RLock 对象看门狗超时（以毫秒为单位）。仅当获取 RLock 对象时未使用`leaseTimeout` 参数时才使用此参数。如果看门狗未将其延长到下一个 `lockWatchdogTimeout` 时间间隔，则锁定将在 `lockWatchdogTimeout` 后过期。这可以防止由于 Redisson 客户端崩溃或任何其他原因无法以正确方式释放锁而导致无限锁定。
### checkLockSyncedSlaves
默认值：`true`
> Defines whether to check synchronized slaves amount with actual slaves amount after lock acquisition.

定义获取锁后是否检查同步从库数量和实际从库数量。
### slavesSyncTimeout
默认值：`1000`
> Defines slaves synchronization timeout in milliseconds applied to each operation of RLock, RSemaphore, RPermitExpirableSemaphore objects.

定义应用于 RLock、RSemaphore、RPermitExpirableSemaphore 对象的每个操作的从站同步超时（以毫秒为单位）。
### reliableTopicWatchdogTimeout
默认值：`600000`
可靠主题看门狗超时（以毫秒为单位）。如果看门狗没有将其延长到下一个超时时间间隔，可靠主题订阅者将在超时后过期。这可以防止由于 Redisson 客户端崩溃或订阅者无法再使用消息的任何其他原因而导致主题中存储的消息无限增长。
### addressResolverGroupFactory
默认值：`org.redisson.connection.SequentialDnsAddressResolverFactory`
允许自定义实现 [DnsAddressResolverGroup](https://github.com/netty/netty/blob/4.1/resolver-dns/src/main/java/io/netty/resolver/dns/DnsAddressResolverGroup.java)。
可选的实现有：

- `org.redisson.connection.DnsAddressResolverGroupFactory` - 使用操作系统提供的默认 DNS 服务器列表
- `org.redisson.connection.SequentialDnsAddressResolverFactory` - 使用操作系统提供的默认 DNS 服务器列表，并允许控制对 DNS 服务器的请求的并发级别
- `org.redisson.connection.RoundRobinDnsAddressResolverGroupFactory` - 以循环模式（round robin mode）使用操作系统提供的默认 DNS 服务器列表
### useScriptCache
默认值：`false`
定义是否在Redis端使用Lua脚本缓存。大多数 Redisson 方法都是基于 Lua 脚本的，打开此设置可以提高此类方法的执行速度并节省网络流量。
### keepPubSubOrder
默认值：`true`
定义是保持 PubSub 消息按到达顺序处理还是同时处理消息。此设置仅适用于每个通道的 PubSub 消息
### minCleanUpDelay
默认值：`5`
定义过期条目（entries）清理过程的最短延迟（以秒为单位）。应用于 `JCache`、`RSetCache`、`RClusteredSetCache`、`RMapCache`、`RListMultimapCache`、`RSetMultimapCache`、`RLocalCachedMapCache`、`RClusteredLocalCachedMapCache` 对象。
### maxCleanUpDelay
默认值：`1800`
定义过期条目（entries）清理过程的最大延迟（以秒为单位）。应用于 `JCache`、`RSetCache`、`RClusteredSetCache`、`RMapCache`、`RListMultimapCache`、`RSetMultimapCache`、`RLocalCachedMapCache`、`RClusteredLocalCachedMapCache` 对象。
### cleanUpKeysAmount
默认值：`100`
定义在过期条目（entries）的清理过程中每次操作删除的过期键数量。应用于 `JCache`、`RSetCache`、`RClusteredSetCache`、`RMapCache`、`RListMultimapCache`、`RSetMultimapCache`、`RLocalCachedMapCache`、`RClusteredLocalCachedMapCache` 对象。
### useThreadClassLoader
默认值：`true`
定义是否向 Codec 提供 Thread ContextClassLoader。使用 Thread.getContextClassLoader() 可以解决 Redis 响应解码期间出现的 ClassNotFoundException 错误。如果 Tomcat 和部署的应用程序中都使用 Redisson，则可能会出现此错误。
### meterMode
默认值：`ALL`
定义 Micrometer 统计收集模式。
_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_
可用的模式有：

- `ALL` - 收集 Redis 和 Redisson 对象统计信息
- `REDIS` - 仅收集 Redis 统计信息
- `OBJECTS` - 仅收集 Redisson 对象统计信息
### meterRegistryProvider
默认值：`null`
定义用于收集 Redisson 对象的各种统计信息的 Micrometer 注册表提供程序。请参阅[统计监控部分](https://github.com/redisson/redisson/wiki/14.-Integration-with-frameworks#1410-statistics-monitoring-jmx-and-other-systems)以获取所有可用提供商的列表。
_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_
### performanceMode
默认值：`LOWER_LATENCY_MODE_2`
定义命令处理引擎性能模式。由于所有值都是特定于应用程序的（正常值除外），因此建议尝试所有这些值。
可选的值有：

- `HIGHER_THROUGHPUT` - 将命令处理器引擎切换到 **更高吞吐量 **模式
- `LOWER_LATENCY_AUTO` - 将命令处理器引擎切换到 **较低延迟模式 **并自动检测最佳设置
- `LOWER_LATENCY_MODE_3` - 使用预定义设置集 #3 将命令处理器引擎切换到 **较低延迟模式**
- `LOWER_LATENCY_MODE_2` - 使用预定义设置集 #2 将命令处理器引擎切换到 **较低延迟模式**
- `LOWER_LATENCY_MODE_1` - 使用预定义设置集 #1 将命令处理器引擎切换到 **较低延迟模式**
- `NORMAL` - 将命令处理器引擎切换到正常模式

_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_
##  集群模式 Cluster mode
集群模式可用于任何托管。与 AWS ElastiCache 集群、Amazon MemoryDB 和 Azure Redis 缓存兼容。 
编程配置示例：
```java
Config config = new Config();
config.useClusterServers()
    .setScanInterval(2000) // cluster state scan interval in milliseconds
    // use "rediss://" for SSL connection
    .addNodeAddress("redis://127.0.0.1:7000", "redis://127.0.0.1:7001")
    .addNodeAddress("redis://127.0.0.1:7002");

RedissonClient redisson = Redisson.create(config);
```
### 集群设置
有关 Redis 服务器集群配置的文档位于[此处](https://redis.io/docs/management/scaling/)。通过以下行激活集群连接模式： `ClusterServersConfig clusterConfig = config.useClusterServers();`下面列出了 ClusterServersConfig 设置。
#### checkSlotsCoverage
默认值：`true`
在 Redisson 启动期间启用集群插槽检查。
#### nodeAddresses
以`host:port`格式添加 Redis 集群节点或 Redis 端点地址。 Redisson 自动发现集群拓扑。使用 `rediss://` 协议进行 SSL 连接。
#### scanInterval
默认值：`1000`
扫描间隔（以毫秒为单位）。应用于Redis集群拓扑扫描。
#### readMode
默认值：`SLAVE`
设置用于读操作的节点类型。可用值：

- `SLAVE` - 从从节点读取，如果没有可用的 SLAVES，则使用 MASTER
- `MASTER` - 从主节点读取
- `MASTER_SLAVE` - 从主节点和从节点读取
#### subscriptionMode
默认值：`MASTER`
设置用于订阅操作的节点类型。可用值：

- `MASTER` - 订阅主节点
- `SLAVE` - 订阅从节点
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
订阅（发布/订阅）通道的最小空闲连接池大小。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### subscriptionConnectionPoolSize
默认值：`50`
订阅（发布/订阅）通道的最大连接池大小。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用
#### slaveConnectionMinimumIdleSize
默认值：`24`
Redis 从节点每个从节点的最小空闲连接量。
#### masterConnectionMinimumIdleSize
默认值：`24`
Redis 主节点每个主节点的最小空闲连接量。主节点也被添加为从节点，并保持空闲连接作为从节点。这些连接保留在从连接池中，但处于非活动状态。当您的集群丢失所有从站时，它们会被保留。在这种情况下，主设备开始用作从设备，空闲连接变为活动状态。
#### masterConnectionPoolSize
默认值：`24`
Redis 主节点最大连接池大小
#### idleConnectionTimeout
默认值：`10000`
如果连接池在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任何 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### password
默认值：`null`
Redis服务器认证密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 连接。
#### tcpKeepAliveCount
默认值：`0`
定义在断开连接之前 TCP 应发送的保活探测的最大数量。 0值表示使用系统默认设置。
#### tcpKeepAliveIdle
默认值：`0`
定义 TCP 开始发送保活探测之前连接需要保持空闲的时间（以秒为单位）。 0值表示使用系统默认设置。
#### tcpKeepAliveInterval
默认值：`0`
定义各个保活探测之间的时间（以秒为单位）。 0值表示使用系统默认设置。
#### tcpUserTimeout
默认值：`0`
定义在 TCP 强制关闭连接之前，传输的数据可能保持未确认状态或缓冲的数据可能保持未传输状态（由于窗口大小为零）的最长时间（以毫秒为单位）。 0值表示使用系统默认设置。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### subscriptionTimeout
默认值：`7500`
定义每个频道订阅应用的订阅超时（以毫秒为单位）。
#### natMapper
默认值：no mapper
定义 NAT 映射器接口，该接口映射 Redis URI 对象并应用于所有 Redis 连接。可用于将内部 Redis 服务器 IP 映射到外部 IP。可用的实现：`org.redisson.api.HostPortNatMapper` 和 `org.redisson.api.HostNatMapper`。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
#### slots
默认值：`231`
用于数据分区的分区数量。 Set、Map、BitSet、Bloom filter、Spring Cache 和 Hibernate Cache 结构支持的数据分区。
_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_
### 集群 YAML 配置格式
以下是 YAML 格式的集群配置示例。所有属性名称均与 `ClusterServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
clusterServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedSlaveReconnectionInterval: 3000
  failedSlaveCheckInterval: 60000
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 24
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 24
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  subscriptionMode: "SLAVE"
  nodeAddresses:
  - "redis://127.0.0.1:7004"
  - "redis://127.0.0.1:7001"
  - "redis://127.0.0.1:7000"
  scanInterval: 1000
  pingConnectionInterval: 30000
  keepAlive: false
  tcpNoDelay: true
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 副本模式 Replicated mode
在复制模式下，将轮询每个节点的角色，以确定是否发生故障转移，从而产生新的主节点。与 `AWS ElastiCache`（非集群）、`Azure Redis Cache`（非集群）和 `Google Cloud Memorystore for Redis High Availability`兼容。
编程配置示例：
```java
Config config = new Config();
config.useReplicatedServers()
    .setScanInterval(2000) // master node change scan interval
    // use "rediss://" for SSL connection
    .addNodeAddress("redis://127.0.0.1:7000", "redis://127.0.0.1:7001")
    .addNodeAddress("redis://127.0.0.1:7002");

RedissonClient redisson = Redisson.create(config);
```
### 副本设置
通过以下行激活副本模式：
`ReplicatedServersConfig replicatedConfig = config.useReplicatedServers();`
下面列出了`Replicated ServersConfig` 设置：
#### nodeAddresses
以`host:port`格式添加Redis节点地址。可以一次添加多个节点。应定义所有节点（主节点和从节点）。对于 Aiven Redis 来说，托管单个主机名就足够了。使用 rediss:// 协议进行 SSL 连接。
#### scanInterval
默认值：`1000`
副本节点扫描间隔（以毫秒为单位）。
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### monitorIPChanges
默认值：`false`
检查配置中定义的每个 Redis 主机名，以了解扫描过程中 IP 地址的变化
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
Redis 从节点每个从节点的最小空闲订阅（发布/订阅）连接量。
#### subscriptionConnectionPoolSize
默认值：`50`
每个从节点的 Redis 从节点最大订阅（发布/订阅）连接池大小。
#### slaveConnectionMinimumIdleSize
默认值：`24`
Redis 从节点每个从节点的最小空闲连接量。
#### slaveConnectionPoolSize
默认值：`64`
Redis 从节点每个从节点的最大连接池大小。
#### masterConnectionMinimumIdleSize
默认值：`24`
Redis 主节点每个从节点的最小空闲连接量。 Redisson 的空闲连接会保留给当选为 master 的 Slave。因为该节点仍然在从连接池中，但处于非活动状态。当您的集群丢失所有从站时，会保留这些连接。在这种情况下，主设备用作从设备，并且空闲连接开始使用。
#### masterConnectionPoolSize
默认值：`64`
Redis 主节点最大连接池大小。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### readMode
默认值：`SLAVE`
设置用于读操作的节点类型。可用值：
● `SLAVE` - 从从节点读取，如果没有可用的 SLAVES，则使用 MASTER
● `MASTER` - 从主节点读取
● `MASTER_SLAVE` - 从主节点和从节点读取
#### subscriptionMode
默认值：`MASTER`
设置用于订阅操作的节点类型。可用值：

- `MASTER` - 订阅主节点
- `SLAVE` - 订阅从节点
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### failedSlaveReconnectionInterval
默认值：`3000`
Redis Slave 被排除在可用服务器的内部列表之外时尝试重新连接的时间间隔。每次发生超时事件时，Redisson 都会尝试连接到已断开连接的 Redis 服务器。值以毫秒为单位。
#### failedSlaveCheckInterval
默认值：`60000`
当该服务器上第一次执行Redis命令失败的时间间隔达到定义值时，执行命令失败的Redis Slave节点将从内部可用节点列表中排除。值以毫秒为单位。
#### database
默认值：`0`
用于Redis连接的数据库索引。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 功能。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 副本模式的 YAML 配置格式
下面是 YAML 格式的副本模式配置示例。所有属性名称均与 `ReplicatedServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
replicatedServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedSlaveReconnectionInterval: 3000
  failedSlaveCheckInterval: 60000
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 24
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 24
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  subscriptionMode: "SLAVE"
  nodeAddresses:
  - "redis://redishost1:2812"
  - "redis://redishost2:2815"
  - "redis://redishost3:2813"
  scanInterval: 1000
  monitorIPChanges: false
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 单实例模式 Single instance mode
单实例模式可用于任何托管。支持 Azure Redis 缓存和 Google Cloud Memorystore for Redis。 编程配置示例：
```java
// connects to 127.0.0.1:6379 by default
RedissonClient redisson = Redisson.create();

Config config = new Config();
config.useSingleServer().setAddress("redis://myredisserver:6379");
RedissonClient redisson = Redisson.create(config);
```
### 单实例设置
#### address
Redis 服务器地址，采用`host:port`格式。使用 `rediss:// `协议进行 SSL 连接。
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
最小空闲 Redis 订阅连接数。
#### subscriptionConnectionPoolSize
默认值：`50`
Redis 订阅连接最大池大小。
#### connectionMinimumIdleSize
默认值：`24`
最小空闲 Redis 连接数。
#### connectionPoolSize
默认值：`64`
Redis 连接最大池大小。
#### dnsMonitoringInterval
默认值：`5000`
DNS更改监控间隔。应用程序必须确保 JVM DNS 缓存 TTL 足够低以支持此操作。设置-1 禁用。[代理模式](#VCW2j)支持单个主机名的多个 IP 绑定。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### database
默认值：`0`
用于Redis连接的数据库索引。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 功能。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 单实例模式的 YAML 配置格式
以下是 YAML 格式的单实例配置示例。所有属性名称都与 `SingleServerConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address: "redis://127.0.0.1:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 24
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 哨兵模式 Sentinel mode
编程配置示例：
```java
Config config = new Config();
config.useSentinelServers()
    .setMasterName("mymaster")
    // use "rediss://" for SSL connection
    .addSentinelAddress("redis://127.0.0.1:26389", "redis://127.0.0.1:26379")
    .addSentinelAddress("redis://127.0.0.1:26319");

RedissonClient redisson = Redisson.create(config);
```
### 哨兵设置
[文档](https://redis.io/topics/sentinel)涵盖了Redis服务器哨兵配置。通过以下行激活哨兵连接模式： `SentinelServersConfig SentinelConfig = config.useSentinelServers(); `
`SentinerServersConfig`设置以下列出：
#### checkSentinelsList
默认值：`true`
在 Redisson 启动期间启用哨兵列表检查。
#### dnsMonitoringInterval
默认值：`5000`
DNS更改监控间隔。应用程序必须确保 JVM DNS 缓存 TTL 足够低以支持此操作。设置-1 禁用。
#### checkSlaveStatusWithSyncing
默认值：`true`
检查从节点 master-link-status 字段的状态是否为 ok。
#### masterName
Redis Sentinel 服务器和主变更监控任务使用的主服务器名称。
#### addSentinelAddress
在主机中添加Redis Sentinel节点`host:port`格式。可以一次添加多个节点。
#### readMode
默认值：`SLAVE`
设置用于读操作的节点类型。可用值：

- `SLAVE` - 从从节点读取，如果没有可用的 SLAVES，则使用 MASTER
- `MASTER` - 从主节点读取
- `MASTER_SLAVE` - 从主节点和从节点读取
#### subscriptionMode
默认值：`SLAVE`
设置用于订阅操作的节点类型。可用值：

- `MASTER` - 订阅主节点
- `SLAVE` - 订阅从节点
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
Redis 从节点每个从节点的最小空闲订阅（发布/订阅）连接量。
#### subscriptionConnectionPoolSize
默认值：`50`
每个从节点的 Redis 从节点最大订阅（发布/订阅）连接池大小。
#### slaveConnectionMinimumIdleSize
默认值：`24`
Redis 从节点每个从节点的最小空闲连接量。
#### slaveConnectionPoolSize
默认值：`64`
Redis 从节点每个从节点的最大连接池大小。
#### masterConnectionMinimumIdleSize
默认值：`24`
Redis 主节点每个从节点的最小空闲连接量。 Redisson 的空闲连接会保留给当选为 master 的 Slave。因为该节点仍然在从连接池中，但处于非活动状态。当您的集群丢失所有从站时，会保留这些连接。在这种情况下，主设备用作从设备，并且空闲连接开始使用。
#### masterConnectionPoolSize
默认值：`64`
Redis 主节点最大连接池大小。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### failedSlaveReconnectionInterval
默认值：`3000`
Redis Slave 被排除在可用服务器的内部列表之外时尝试重新连接的时间间隔。每次发生超时事件时，Redisson 都会尝试连接到已断开连接的 Redis 服务器。值以毫秒为单位。
#### failedSlaveCheckInterval
默认值：`60000`
当该服务器上第一次执行Redis命令失败的时间间隔达到定义值时，执行命令失败的Redis Slave节点将从内部可用节点列表中排除。值以毫秒为单位。
#### database
默认值：`0`
用于Redis连接的数据库索引。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### sentinelPassword
默认值：`null`
Redis Sentinel 服务器身份验证的密码。仅当 Sentinel 密码与主站和从站密码不同时才使用。
#### sentinelUsername
默认值：`null`
用于身份验证的 Redis Sentinel 服务器的用户名。仅当 Sentinel 用户名与主站和从站的用户名不同时才使用。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### sentinelsDiscovery
默认值：`true`
启用哨兵发现。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 功能。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### natMapper
默认值：no mapper
定义 NAT 映射器接口，该接口映射 Redis URI 对象并应用于所有 Redis 连接。可用于将内部 Redis 服务器 IP 映射到外部 IP。可用的实现：`org.redisson.api.HostPortNatMapper` 和 `org.redisson.api.HostNatMapper`。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 哨兵模式的 YAML 配置格式
以下是 YAML 格式的哨兵配置示例。所有属性名称均与 `SentinelServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
sentinelServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedSlaveReconnectionInterval: 3000
  failedSlaveCheckInterval: 60000
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 24
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 24
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  subscriptionMode: "SLAVE"
  sentinelAddresses:
  - "redis://127.0.0.1:26379"
  - "redis://127.0.0.1:26389"
  masterName: "mymaster"
  database: 0
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 主从模式 Master slave mode
编程配置示例：
```java
Config config = new Config();
config.useMasterSlaveServers()
    // use "rediss://" for SSL connection
    .setMasterAddress("redis://127.0.0.1:6379")
    .addSlaveAddress("redis://127.0.0.1:6389", "redis://127.0.0.1:6332", "redis://127.0.0.1:6419")
    .addSlaveAddress("redis://127.0.0.1:6399");

RedissonClient redisson = Redisson.create(config);
```
### 主从模式设置
涵盖 Redis 服务器主/从配置的文档位于[此处](https://redis.io/docs/management/replication/)。通过以下行激活主从连接模式： 
`MasterSlaveServersConfig masterSlaveConfig = config.useMasterSlaveServers();`
下面列出了 `MasterSlaveServersConfig` 设置：
#### dnsMonitoringInterval
默认值：`5000`
DNS更改监控间隔。应用程序必须确保 JVM DNS 缓存 TTL 足够低以支持此操作。设置-1 禁用。
#### masterAddress
Redis主节点地址，格式为`host:port`。使用 `rediss://` 协议进行 SSL 连接。
#### addSlaveAddress
添加Redis从节点地址，格式为`host:port`。可以一次添加多个节点。使用 `rediss://`协议进行 SSL 连接。
#### readMode
默认值：`SLAVE`
设置用于读操作的节点类型。可用值：

- `SLAVE` - 从从节点读取，如果没有可用的 SLAVES，则使用 MASTER
- `MASTER` - 从主节点读取
- `MASTER_SLAVE` - 从主节点和从节点读取
#### subscriptionMode
默认值：`SLAVE`
设置用于订阅操作的节点类型。可用值：

- `MASTER` - 订阅主节点
- `SLAVE` - 订阅从节点
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
Redis 从节点每个从节点的最小空闲订阅（发布/订阅）连接量。
#### subscriptionConnectionPoolSize
默认值：`50`
每个从节点的 Redis 从节点最大订阅（发布/订阅）连接池大小。
#### slaveConnectionMinimumIdleSize
默认值：`24`
Redis 从节点每个从节点的最小空闲连接量。
#### slaveConnectionPoolSize
默认值：`64`
Redis 从节点每个从节点的最大连接池大小。
#### masterConnectionMinimumIdleSize
默认值：`24`
Redis 主节点每个从节点的最小空闲连接量。 Redisson 的空闲连接会保留给当选为 master 的 Slave。因为该节点仍然在从连接池中，但处于非活动状态。当您的集群丢失所有从站时，会保留这些连接。在这种情况下，主设备用作从设备，并且空闲连接开始使用。
#### masterConnectionPoolSize
默认值：`64`
Redis 主节点最大连接池大小。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### failedSlaveReconnectionInterval
默认值：`3000`
Redis Slave 被排除在可用服务器的内部列表之外时尝试重新连接的时间间隔。每次发生超时事件时，Redisson 都会尝试连接到已断开连接的 Redis 服务器。值以毫秒为单位。
#### failedSlaveCheckInterval
默认值：`60000`
当该服务器上第一次执行Redis命令失败的时间间隔达到定义值时，执行命令失败的Redis Slave节点将从内部可用节点列表中排除。值以毫秒为单位。
#### database
默认值：`0`
用于Redis连接的数据库索引。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 连接。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 主从模式的 YAML 配置格式
下面是 YAML 格式的主从配置示例。所有属性名称均与 `MasterSlaveServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
masterSlaveServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedSlaveReconnectionInterval: 3000
  failedSlaveCheckInterval: 60000
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 24
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 24
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  subscriptionMode: "SLAVE"
  slaveAddresses:
  - "redis://127.0.0.1:6381"
  - "redis://127.0.0.1:6380"
  masterAddress: "redis://127.0.0.1:6379"
  database: 0
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 代理模式 Proxy mode
代理模式支持单个或多个Redis数据库（包括与双活复制同步）用于读/写操作。每个 Redis 主机名可能会解析为多个 IP 地址。
根据 [proxyMode](#xB5CE) 设置的值，有两种模式：

1. 所有 Redis 节点都是主节点，用于通过负载均衡器进行读/写操作
2. 单个主节点用于读/写操作，其余为空闲辅助节点

故障节点检测由 [scanMode](#WqR0A) 设置管理。
与 `Redis Enterprise Multiple Active Proxy`，`Redis Enterprise Active-Active databases` 和 `Azure Redis Cache active-active replication` 适配。

_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_

编程配置示例：
```java
Config config = new Config();
// use "rediss://" for SSL connection
config.useProxyServers().addAddress("redis://myredisserver1:6379", "redis://myredisserver2:6379");

RedissonClient redisson = Redisson.create(config);
```
### 代理模式设置
通过以下行激活代理服务器连接模式： 
`ProxyServersConfig proxyConfig = config.useProxyServers();` 
下面列出了 `ProxyServersConfig` 设置：
#### addresses
Redis 代理服务器地址采用`host:port`格式。如果定义了单个主机名并启用了 DNS 监控，则所有解析的 ip 都将被视为代理节点并由负载均衡器使用。使用 `rediss://` 协议进行 SSL 连接。
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
Redis 从节点每个从节点的最小空闲订阅（发布/订阅）连接量。
#### subscriptionConnectionPoolSize
默认值：`50`
每个从节点的 Redis 从节点最大订阅（发布/订阅）连接池大小。
#### connectionMinimumIdleSize
默认值：`24`
最小空闲 Redis 连接数。
#### connectionPoolSize
默认值：`64`
Redis 连接最大池大小。
#### scanMode
默认值：`PING`
定义扫描模式以检测失败的 Redis 节点。可用值： 
`PING` - 使用 PING 命令检查每个 Redis 节点。如果Redis节点无法响应则认为是故障节点。 
`PUBSUB` - 消息通过每个 Redis 节点的 pubsub 通道发送，并且应该由所有其他 Redis 节点接收。如果Redis节点无法订阅或接收消息，那么它被认为是失败的节点。
#### proxyMode
默认值：`ALL_ACTIVE`
定义代理模式。
可用值：
`FIRST_ACTIVE` - 主（活动）数据库是地址列表中的第一个地址，其余是故障转移后使用的空闲辅助节点。 
`ALL_ACTIVE` - 所有数据库都是主数据库（活动数据库）并用于读/写操作
#### scanInterval
默认值：`1000`
副本节点扫描间隔（以毫秒为单位）。
#### scanTimeout
默认值：`3000`
定义每个 Redis 节点应用的代理节点扫描超时（以毫秒为单位）。
#### dnsMonitoringInterval
默认值：`5000`
DNS更改监控间隔。应用程序必须确保 JVM DNS 缓存 TTL 足够低以支持此操作。设置-1 禁用。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### database
默认值：`0`
用于Redis连接的数据库索引。
#### failedNodeReconnectionInterval
默认值：`3000`
当重试间隔达到时，Redisson 会尝试连接到已断开连接的 Redis 节点。成功重新连接后，Redis 节点即可执行读/写操作。
#### failedNodeCheckInterval
默认值：`180000`
如果 Redis 节点在指定的时间间隔内继续无法执行命令，则该节点将无法执行读/写操作，并被标记为已断开连接。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 连接。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 代理模式的 YAML 配置格式
以下是 YAML 格式的代理模式配置示例。所有属性名称均与 `ProxyServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
proxyServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  addresses: "redis://127.0.0.1:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 24
  connectionPoolSize: 64
  database: 0
  dnsMonitoringInterval: 5000
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```
## 多集群模式 Multi cluster mode
支持具有主动-被动数据复制关系的多个集群设置。
与 [AWS Redis Global Datastore](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Redis-Global-Datastore.html) 适配。
具有所有可用主节点的集群成为主节点。主节点可用性扫描间隔由 scanInterval 设置定义。

_此设置仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_

编程配置示例：
```java
Config config = new Config();
config.useMultiClusterServers()
    .setScanInterval(2000) // cluster state scan interval in milliseconds
    // use "rediss://" for SSL connection
    .addAddress("redis://cluster1:7000", "redis://cluster2:70002");

RedissonClient redisson = Redisson.create(config);
```
### 多集群模式设置
通过以下行激活多集群连接模式： 
`ClusterServersConfig clusterConfig = config.useMultiClusterServers(); `
下面列出了 `ClusterServersConfig` 设置：
#### checkSlotsCoverage
默认值：`true`
在 Redisson 启动期间启用集群插槽检查。
#### addresses
Redis 代理服务器地址采用`host:port`格式。如果定义了单个主机名并启用了 DNS 监控，则所有解析的 ip 都将被视为代理节点并由负载均衡器使用。使用 `rediss://` 协议进行 SSL 连接。
#### scanInterval
默认值：`5000`
扫描间隔（以毫秒为单位）。应用于Redis集群拓扑扫描以及主备集群扫描过程。处理主集群和辅助集群之间的故障转移。具有所有可用主节点的集群成为主节点。
#### slots
默认值：`231`
用于数据分区的分区数量。 Set、Map、BitSet、Bloom filter、Spring Cache 和 Hibernate Cache 结构支持的数据分区。
#### readMode
默认值：`SLAVE`
设置用于读操作的节点类型。可用值：
● `SLAVE` - 从从节点读取，如果没有可用的 SLAVES，则使用 MASTER
● `MASTER` - 从主节点读取
● `MASTER_SLAVE` - 从主节点和从节点读取
#### datastoreMode
默认值：`ACTIVE_PASSIVE`
定义数据存储模式。
可用值： 
`ACTIVE` - 仅使用主（活动）集群 
`ACTIVE_PASSIVE` - 主（主动）集群用于读/写操作，辅助（被动）集群仅用于读操作
`WRITE_ACTIVE_READ_PASSIVE` - 主（主动）集群用于写入操作，辅助（被动）集群仅用于读取操作
#### subscriptionMode
默认值：`SLAVE`
设置用于订阅操作的节点类型。可用值：

- `MASTER` - 订阅主节点
- `SLAVE` - 订阅从节点
#### loadBalancer
默认值：`org.redisson.connection.balancer.RoundRobinLoadBalancer`
多个 Redis 服务器的连接负载均衡器。可用的实现：

- `org.redisson.connection.balancer.CommandsLoadBalancer`
- `org.redisson.connection.balancer.WeightedRoundRobinBalancer`
- `org.redisson.connection.balancer.RoundRobinLoadBalancer`
- `org.redisson.connection.balancer.RandomLoadBalancer`
#### subscriptionConnectionMinimumIdleSize
默认值：`1`
Redis 从节点每个从节点的最小空闲订阅（发布/订阅）连接量。
#### subscriptionConnectionPoolSize
默认值：`50`
每个从节点的 Redis 从节点最大订阅（发布/订阅）连接池大小。
#### slaveConnectionMinimumIdleSize
默认值：`24`
Redis 从节点每个从节点的最小空闲连接量。
#### slaveConnectionPoolSize
默认值：`64`
Redis 从节点每个从节点的最大连接池大小。
#### masterConnectionMinimumIdleSize
默认值：`24`
Redis 主节点每个从节点的最小空闲连接量。 Redisson 的空闲连接会保留给当选为 master 的 Slave。因为该节点仍然在从连接池中，但处于非活动状态。当您的集群丢失所有从站时，会保留这些连接。在这种情况下，主设备用作从设备，并且空闲连接开始使用。
#### masterConnectionPoolSize
默认值：`64`
Redis 主节点最大连接池大小。
#### idleConnectionTimeout
默认值：`10000`
如果池连接在超时时间内未使用并且当前连接数大于最小空闲连接池大小，则它将关闭并从池中删除。值以毫秒为单位。
#### connectTimeout
默认值：`10000`
连接到任意 Redis 服务器期间的超时（以毫秒为单位）。
#### timeout
默认值：`3000`
Redis 服务器响应超时（以毫秒为单位）。 Redis 命令发送成功后开始倒计时。
#### retryAttempts
默认值：`3`
失败重试次数。如果 Redis 命令在重试后仍无法发送到 Redis 服务器，则会抛出错误。但如果发送成功，则会开始计算超时。
#### retryInterval
默认值：`1500`
时间间隔（以毫秒为单位），之后将执行另一次尝试发送 Redis 命令。
#### password
默认值：`null`
Redis服务器认证的密码。
#### username
默认值：`null`
Redis 服务器身份验证的用户名。需要 Redis 6.0+。
#### credentialsResolver
默认值：空
定义在连接期间调用的凭据解析器以进行 Redis 服务器身份验证。返回每个 Redis 节点地址的 Credentials 对象，它包含用户名和密码字段。允许指定动态更改的 Redis 凭据。
#### subscriptionsPerConnection
默认值：`5`
每个订阅连接的订阅数限制。由 `RTopic`、`RPatternTopic`、`RLock`、`RSemaphore`、`RCountDownLatch`、`RClusteredLocalCachedMap`、`RClusteredLocalCachedMapCache`、`RLocalCachedMap`、`RLocalCachedMapCache` 对象和 Hibernate READ_WRITE 缓存策略使用。
#### clientName
默认值：`null`
客户端连接的名称。
#### sslProtocols
默认值：`null`
定义允许的 SSL 协议的数组。 示例值：`TLSv1.3`、`TLSv1.2`、`TLSv1.1`、`TLSv1`
#### sslEnableEndpointIdentification
默认值：`true`
在握手期间启用 SSL 端点识别，从而防止中间人攻击。
#### sslProvider
默认值：`JDK`
定义用于处理 SSL 连接的 SSL 提供程序（JDK 或 OPENSSL）。 OPENSSL 被认为是更快的实现，需要在类路径中添加 `netty-tcnative-boringssl-static` 。
#### sslTruststore
默认值：`null`
定义 SSL 信任库的路径。它存储用于识别 SSL 连接的服务器端的证书。 SSL 信任库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslTruststorePassword
默认值：`null`
定义 SSL 信任库的密码。
#### sslKeystore
默认值：`null`
定义 SSL 密钥库的路径。它存储私钥和与其公钥相对应的证书。如果 SSL 连接的服务器端需要客户端身份验证，则使用。 SSL 密钥库会在每次创建新连接时读取，并且可以动态重新加载。
#### sslKeystorePassword
默认值：`null`
定义 SSL 密钥库的密码
#### pingConnectionInterval
默认值：`30000`
此设置允许使用 PING 命令检测并重新连接断开的连接。 PING 命令发送间隔以毫秒为单位定义。当 netty lib 不调用已关闭连接的 `ChannelInactive` 方法时很有用。设置 0 禁用。
#### keepAlive
默认值：`false`
启用 TCP keepAlive 功能。
#### tcpNoDelay
默认值：`true`
启用 TCP noDelay 功能。
#### natMapper
默认值：no mapper
定义 NAT 映射器接口，该接口映射 Redis URI 对象并应用于所有 Redis 连接。可用于将内部 Redis 服务器 IP 映射到外部 IP。可用的实现：`org.redisson.api.HostPortNatMapper` 和 `org.redisson.api.HostNatMapper`。
#### nameMapper
默认值：mo mapper
定义名称映射器，将 Redisson 对象名称映射到自定义名称。应用于所有 Redisson 对象
#### commandMapper
默认值：no mapper
定义命令映射器，将 Redis 命令名称映射到自定义名称。适用于所有 Redis 命令。
### 多集群模式的 YAML 配置格式
以下是 YAML 格式的集群配置示例。所有属性名称均与 `ClusterServersConfig` 和 `Config` 对象属性名称匹配。
```yaml
---
multiClusterServersConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  failedSlaveReconnectionInterval: 3000
  failedSlaveCheckInterval: 60000
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  slaveConnectionMinimumIdleSize: 24
  slaveConnectionPoolSize: 64
  masterConnectionMinimumIdleSize: 24
  masterConnectionPoolSize: 64
  readMode: "SLAVE"
  datastoreMode: "ACTIVE_PASSIVE"
  subscriptionMode: "SLAVE"
  addresses:
  - "redis://cluster1:7004"
  - "redis://cluster2:7001"
  - "redis://cluster3:7000"
  scanInterval: 5000
  pingConnectionInterval: 30000
  keepAlive: false
  tcpNoDelay: true
threads: 16
nettyThreads: 32
codec: !<org.redisson.codec.Kryo5Codec> {}
transportMode: "NIO"
```


# Operations execution 调用方式
Redisson 实例是完全线程安全的。可以通过 `RedissonClient` 接口访问具有同步/异步方法的对象。
`Reactive` 和 `RxJava3` 方法分别通过 `RedissonReactiveClient` 和 `RedissonRxClient` 接口。
Redisson 对每个操作实施自动重试策略。重试策略由 `retryAttempts` 和 `retryInterval` 设置控制。这些设置应用于每个 `Redisson` 对象。成功发送 Redis 命令后，开始计算超时设置。
每个 Redisson 对象实例都可以覆盖上面的设置。这些设置适用于给定 Redisson 对象实例的每个方法。
这是 RBucket 对象的示例：
```java
RedissonClient client = Redisson.create(config);
RBucket<MyObject> bucket = client.getBucket('myObject');

// sync way
bucket.get();
// async way
RFuture<MyObject> result = bucket.getAsync();

// instance with overridden retryInterval and timeout parameters
RBucket<MyObject> bucket = client.getBucket(PlainOptions.name('myObject')
                                                        .timeout(Duration.ofSeconds(3))
                                                        .retryInterval(Duration.ofSeconds(5)));

```
## 异步调用
大多数 Redisson 对象都使用镜像到同步方法的异步方法来扩展异步接口。像这样：
```java
// RAtomicLong extends RAtomicLongAsync
RAtomicLongAsync longObject = client.getAtomicLong("myLong");
RFuture<Boolean> future = longObject.compareAndSetAsync(1, 401);
```
异步方法返回实现了 `CompletionStage` 接口的 `RFuture` 对象。
```java
future.whenComplete((res, exception) -> {

    // handle both result and exception

});


// or
future.thenAccept(res -> {

    // handle result

}).exceptionally(exception -> {

    // handle exception

});

```
避免在未来的侦听器中使用阻塞方法。由 netty 线程执行的侦听器和侦听器中的延迟可能会导致 Redis 请求/响应处理中出现错误。使用以下方法（自定执行线程池）在监听器中执行阻塞方法：
```java
future.whenCompleteAsync((res, exception) -> {

    // handle both result and exception

}, executor);


// or
future.thenAcceptAsync(res -> {

    // handle result

}, executor).exceptionallyAsync(exception -> {

    // handle exception

}, executor);
```
## 响应式调用
Redisson 为大多数对象公开了 [Reactive Streams](http://reactive-streams.org/) API，并基于两种实现：

1. 基于 [Project Reactor](https://projectreactor.io/) 的实现。代码示例：
```java
RedissonReactiveClient client = redissonClient.reactive();

RAtomicLongReactive atomicLong = client.getAtomicLong("myLong");
Mono<Boolean> cs = longObject.compareAndSet(10, 91);
Mono<Long> get = longObject.get();

get.doOnSuccess(res -> {
   // ...
}).subscribe();
```

2. 基于[ RxJava3](https://github.com/ReactiveX/RxJava) 的实现。代码示例：
```java
RedissonRxClient client = redissonClient.rxJava();

RAtomicLongRx atomicLong = client.getAtomicLong("myLong");
Single<Boolean> cs = longObject.compareAndSet(10, 91);
Single<Long> get = longObject.get();

get.doOnSuccess(res -> {
   // ...
}).subscribe();
```
# Data serialization 数据序列化
Redisson 使用数据序列化来编码和解码通过 Redis 服务器的网络链接接收或发送的字节。许多流行的编解码器可供使用：

| Codec class name | Description |
| --- | --- |
| org.redisson.codec.Kryo5Codec | [Kryo 5](https://github.com/EsotericSoftware/kryo)  binary codec (**Android** compatible) **Default codec** |
| org.redisson.codec.KryoCodec | [Kryo 4](https://github.com/EsotericSoftware/kryo)  binary codec |
| org.redisson.codec.JsonJacksonCodec | [Jackson JSON](https://github.com/FasterXML/jackson)  codec. Stores type information in @class field (**Android** compatible) |
| org.redisson.codec.TypedJsonJacksonCodec | Jackson JSON codec which doesn't store type id (@class field) during encoding and dosn't require it for decoding |
| org.redisson.codec.AvroJacksonCodec | [Avro](http://avro.apache.org/)  binary json codec |
| org.redisson.codec.ProtobufCodec | [Protobuf](https://github.com/protocolbuffers/protobuf)  codec |
| org.redisson.codec.SmileJacksonCodec | [Smile](http://wiki.fasterxml.com/SmileFormatSpec)  binary json codec |
| org.redisson.codec.CborJacksonCodec | [CBOR](http://cbor.io/)  binary json codec |
| org.redisson.codec.MsgPackJacksonCodec | [MsgPack](http://msgpack.org/)  binary json codec |
| org.redisson.codec.IonJacksonCodec | [Amazon Ion](https://amzn.github.io/ion-docs/)  codec |
| org.redisson.codec.SerializationCodec | JDK Serialization binary codec (**Android** compatible) |
| org.redisson.codec.LZ4Codec | [LZ4](https://github.com/jpountz/lz4-java)  compression codec. Uses Kryo5Codec for serialization by default |
| org.redisson.codec.LZ4CodecV2 | [LZ4 Apache Commons](https://github.com/apache/commons-compress)  compression codec. Uses Kryo5Codec for serialization by default |
| org.redisson.codec.SnappyCodecV2 | Snappy compression codec based on [snappy-java](https://github.com/xerial/snappy-java)  project. Uses Kryo5Codec for serialization by default |
| org.redisson.codec.MarshallingCodec | [JBoss Marshalling](https://github.com/jboss-remoting/jboss-marshalling)  binary codec **Deprecated!** |
| org.redisson.client.codec.StringCodec | String codec |
| org.redisson.client.codec.LongCodec | Long codec |
| org.redisson.client.codec.ByteArrayCodec | Byte array codec |
| org.redisson.codec.CompositeCodec | Allows to mix different codecs as one |

# 数据分区（分片）
_此功能仅在 _[_Redisson PRO_](https://redisson.pro/)_ 版本中可用。_

1. 基于Redis的Java对象的分区（分片）

所有 Redisson Java 对象都与 Redis 集群兼容，但它们的状态不会缩放/分区到集群中的多个 Redis 主节点。 Redisson PRO 为其中一些提供数据分区。此功能具有以下几个优点：

- 单个Redisson对象的状态均匀分布在Redis主节点上，而不是单个主节点上。这可以避免 Redis OutOfMemory 问题。 将读/写操作扩展到所有 Redis 主节点。
- 将读/写操作扩展到所有 Redis 主节点。

**Redisson默认将数据拆分为231个分区**。分区的最小数量为 3。分区均匀分布在所有 Redis 集群节点上。这意味着每个节点包含几乎相等数量的分区。对于Redis集群默认分区数量（231）和4个主节点来说，每个节点包含近57个数据分区。 5 个主节点集群的每个节点 46 个数据分区，依此类推。该功能的实现得益于 Redisson 中使用的特殊槽分配算法。
支持 Set、Map、BitSet、Bloom filter、Spring Cache、Hibernate Cache、JCache 和 Micronaut Cache 结构的数据分区。

2. Redis 设置的分区（分片）

具有有限内存量的多个 Redis 设置可以连接起来并用作单个分区（分片）Redis 环境。
```java
RedissonClient redisson1 = ...;
RedissonClient redisson2 = ...;
RedissonClient redisson3 = ...;

RedissonClient redisson = ShardedRedisson.create(redisson1, redisson2, redisson3);
```
# 常见操作
每个 Redisson 对象都实现 `RObject` 和 `RExpirable` 接口。
使用示例：
```java
RObject object = redisson.get...()

object.sizeInMemory();

object.delete();

object.rename("newname");

object.isExists();

// catch expired event
object.addListener(new ExpiredObjectListener() {
   ...
});

// catch delete event
object.addListener(new DeletedObjectListener() {
   ...
});
```
Redisson对象的名称是Redis中的键。
```java
RMap map = redisson.getMap("mymap");
map.getName(); // = mymap
```
所有对 Redis 键的操作都由 `RKeys` 接口公开。使用示例：
```java
RKeys keys = redisson.getKeys();

Iterable<String> allKeys = keys.getKeys();

Iterable<String> foundedKeys = keys.getKeysByPattern('key*');

long numOfDeletedKeys = keys.delete("obj1", "obj2", "obj3");

long deletedKeysAmount = keys.deleteByPattern("test?");

String randomKey = keys.randomKey();

long keysAmount = keys.count();

keys.flushall();

keys.flushdb();
```
# Distributed objects 分布式对象
正常的 Java 对象保存在机器的 JVM 上，Redission 实现了一些基本的 Java 对象（如AtomicLong、LongAdder）的 API，将对象保存在 Redis 上，对其他机器可见，称为分布式对象。
## Object holder
基于 Redis 的 RBucket 对象的Java实现是任何类型对象的持有者。大小限制为 512Mb。 代码示例：
```java
RBucket<AnyObject> bucket = redisson.getBucket("anyObject");

bucket.set(new AnyObject(1));
AnyObject obj = bucket.get();

bucket.trySet(new AnyObject(3));
bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
bucket.getAndSet(new AnyObject(6));
```
Async接口使用的代码示例：
```java
RBucket<AnyObject> bucket = redisson.getBucket("anyObject");

RFuture<Void> future = bucket.setAsync(new AnyObject(1));
RFuture<AnyObject> objfuture = bucket.getAsync();

RFuture<Boolean> tsFuture = bucket.trySetAsync(new AnyObject(3));
RFuture<Boolean> csFuture = bucket.compareAndSetAsync(new AnyObject(4), new AnyObject(5));
RFuture<AnyObject> gsFuture = bucket.getAndSetAsync(new AnyObject(6));
```
Reactive接口使用的代码示例：
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RBucketReactive<AnyObject> bucket = redisson.getBucket("anyObject");

Mono<Void> mono = bucket.set(new AnyObject(1));
Mono<AnyObject> objMono = bucket.get();

Mono<Boolean> tsMono = bucket.trySet(new AnyObject(3));
Mono<Boolean> csMono = bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
Mono<AnyObject> gsMono = bucket.getAndSet(new AnyObject(6));
```
RxJava3接口使用的代码示例：
```java
RedissonRxClient redisson = redissonClient.rxJava();
RBucketRx<AnyObject> bucket = redisson.getBucket("anyObject");

Completable rx = bucket.set(new AnyObject(1));
Maybe<AnyObject> objRx = bucket.get();

Single<Boolean> tsRx = bucket.trySet(new AnyObject(3));
Single<Boolean> csRx = bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
Maybe<AnyObject> gsRx = bucket.getAndSet(new AnyObject(6));
```

使用 RBuckets 接口对多个 RBucket 对象执行操作： 代码示例：
```java
RBuckets buckets = redisson.getBuckets();

// get all bucket values
Map<String, V> loadedBuckets = buckets.get("myBucket1", "myBucket2", "myBucket3");

Map<String, Object> map = new HashMap<>();
map.put("myBucket1", new MyObject());
map.put("myBucket2", new MyObject());

// sets all or nothing if some bucket is already exists
buckets.trySet(map);
// store all at once
buckets.set(map);
```
Async接口使用的代码示例：
```java
RBuckets buckets = redisson.getBuckets();

// get all bucket values
RFuture<Map<String, V>> bucketsFuture = buckets.getAsync("myBucket1", "myBucket2", "myBucket3");

Map<String, Object> map = new HashMap<>();
map.put("myBucket1", new MyObject());
map.put("myBucket2", new MyObject());

// sets all or nothing if some bucket is already exists
RFuture<Boolean> tsFuture = buckets.trySetAsync(map);
// store all at once
RFuture<Void> sFuture = buckets.setAsync(map);
```
Reactive接口使用的代码示例：
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RBucketsReactive buckets = redisson.getBuckets();

// get all bucket values
Mono<Map<String, V>> bucketsMono = buckets.getAsync("myBucket1", "myBucket2", "myBucket3");

Map<String, Object> map = new HashMap<>();
map.put("myBucket1", new MyObject());
map.put("myBucket2", new MyObject());

// sets all or nothing if some bucket is already exists
Mono<Boolean> tsMono = buckets.trySet(map);
// store all at once
Mono<Void> sMono = buckets.set(map);
```
RxJava3接口使用的代码示例：
```java
RedissonRxClient redisson = redissonClient.rxJava();
RBucketsRx buckets = redisson.getBuckets();

// get all bucket values
Single<Map<String, V>> bucketsRx = buckets.get("myBucket1", "myBucket2", "myBucket3");

Map<String, Object> map = new HashMap<>();
map.put("myBucket1", new MyObject());
map.put("myBucket2", new MyObject());

// sets all or nothing if some bucket is already exists
Single<Boolean> tsRx = buckets.trySet(map);
// store all at once
```
## Binary stream holder
基于 Redis 的 `RBinaryStream` 对象的 Java 实现保存字节序列。它扩展了`RBucket`接口，大小限制为512Mb。

Code example:
```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");

byte[] content = ...
stream.set(content);
stream.getAndSet(content);
stream.trySet(content);
stream.compareAndSet(oldContent, content);
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBucketAsync.html)** interface** usage:
```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");

byte[] content = ...
RFuture<Void> future = stream.set(content);
RFuture<byte[]> future = stream.getAndSet(content);
RFuture<Boolean> future = stream.trySet(content);
RFuture<Boolean> future = stream.compareAndSet(oldContent, content);
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBinaryStreamReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RBinaryStreamReactive stream = redisson.getBinaryStream("anyStream");

ByteBuffer content = ...
Mono<Void> mono = stream.set(content);
Mono<byte[]> mono = stream.getAndSet(content);
Mono<Boolean> mono = stream.trySet(content);
Mono<Boolean> mono = stream.compareAndSet(oldContent, content);

Mono<Integer> mono = stream.write(content);
stream.position(0);
Mono<Integer> mono = stream.read(b);
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBinaryStreamRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RBinaryStreamRx stream = redisson.getBinaryStream("anyStream");

ByteBuffer content = ...
Completable rx = stream.set(content);
Maybe<byte[]> rx = stream.getAndSet(content);
Single<Boolean> rx = stream.trySet(content);
Single<Boolean> rx = stream.compareAndSet(oldContent, content);

Single<Integer> rx = stream.write(content);
stream.position(0);
Single<Integer> rx = stream.read(b);
```
Code example of [java.io.InputStream](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html) and [java.io.OutputStream](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html) interfaces usage:
```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");

InputStream is = stream.getInputStream();
byte[] readBuffer = ...
is.read(readBuffer);

OutputStream os = stream.getOuputStream();
byte[] contentToWrite = ...
os.write(contentToWrite);
```
Code example of [java.nio.channels.SeekableByteChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/SeekableByteChannel.html) interface usage:
```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");

SeekableByteChannel sbc = stream.getChannel();
ByteBuffer readBuffer = ...
sbc.read(readBuffer);

sbc.position(0);

ByteBuffer contentToWrite = ...
sbc.write(contentToWrite);

sbc.truncate(234);
```
Code example of [java.nio.channels.AsynchronousByteChannel](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/AsynchronousByteChannel.html) interface usage:
```java
RBinaryStream stream = redisson.getBinaryStream("anyStream");

AsynchronousByteChannel sbc = stream.getAsynchronousChannel();
ByteBuffer readBuffer = ...
sbc.read(readBuffer);

ByteBuffer contentToWrite = ...
sbc.write(contentToWrite);
```
## Geospatial holder
基于 Redis 的 RGeo 对象的 Java 实现是地理空间项目的持有者。

Code example:
```java
RGeo<String> geo = redisson.getGeo("test");

geo.add(new GeoEntry(13.361389, 38.115556, "Palermo"), 
        new GeoEntry(15.087269, 37.502669, "Catania"));

Double distance = geo.dist("Palermo", "Catania", GeoUnit.METERS);
Map<String, GeoPosition> positions = geo.pos("test2", "Palermo", "test3", "Catania", "test1");

List<String> cities = geo.search(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
Map<String, GeoPosition> citiesWithPositions = geo.searchWithPosition(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RGeoAsync.html)** interface** usage:
```java
RGeo<String> geo = redisson.getGeo("test");

RFuture<Long> addFuture = geo.addAsync(new GeoEntry(13.361389, 38.115556, "Palermo"), 
                                       new GeoEntry(15.087269, 37.502669, "Catania"));

RFuture<Double> distanceFuture = geo.distAsync("Palermo", "Catania", GeoUnit.METERS);
RFuture<Map<String, GeoPosition>> positionsFuture = geo.posAsync("test2", "Palermo", "test3", "Catania", "test1");

RFuture<List<String>> citiesFuture = geo.searchAsync(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
RFuture<Map<String, GeoPosition>> citiesWithPositions = geo.searchWithPositionAsync(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RGeoReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RGeoReactive<String> bucket = redisson.getGeo("test");

Mono<Long> addFuture = geo.add(new GeoEntry(13.361389, 38.115556, "Palermo"), 
                               new GeoEntry(15.087269, 37.502669, "Catania"));

Mono<Double> distanceFuture = geo.dist("Palermo", "Catania", GeoUnit.METERS);
Mono<Map<String, GeoPosition>> positionsFuture = geo.pos("test2", "Palermo", "test3", "Catania", "test1");

Mono<List<String>> citiesFuture = geo.search(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
Mono<Map<String, GeoPosition>> citiesWithPositions = geo.searchWithPosition(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RGeoRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RGeoRx<String> bucket = redisson.getGeo("test");

Single<Long> addFuture = geo.add(new GeoEntry(13.361389, 38.115556, "Palermo"), 
                                 new GeoEntry(15.087269, 37.502669, "Catania"));

Single<Double> distanceFuture = geo.dist("Palermo", "Catania", GeoUnit.METERS);
Single<Map<String, GeoPosition>> positionsFuture = geo.pos("test2", "Palermo", "test3", "Catania", "test1");

Single<List<String>> citiesFuture = geo.search(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
Single<Map<String, GeoPosition>> citiesWithPositions = geo.searchWithPosition(GeoSearchArgs.from(15, 37).radius(200, GeoUnit.KILOMETERS));
```
## BitSet
基于Redis的RBitSet对象的Java实现提供了类似于`java.util.BitSet`的API。它表示根据需要增长的位向量。 Redis 的大小限制为 `4 294 967 295` 位。
Code example:
```java
RBitSet set = redisson.getBitSet("simpleBitset");

set.set(0, true);
set.set(1812, false);

set.clear(0);

set.and("anotherBitset");
set.xor("anotherBitset");
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBitSetAsync.html)** interface** usage:
```java
RBitSetAsync set = redisson.getBitSet("simpleBitset");

RFuture<Boolean> setFuture = set.setAsync(0, true);
RFuture<Boolean> setFuture = set.setAsync(1812, false);

RFuture<Void> clearFuture = set.clearAsync(0);

RFuture<Void> andFuture = set.andAsync("anotherBitset);
RFuture<Void> xorFuture = set.xorAsync("anotherBitset");
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBitSetReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RBitSetReactive stream = redisson.getBitSet("simpleBitset");

Mono<Boolean> setMono = set.set(0, true);
Mono<Boolean> setMono = set.set(1812, false);

Mono<Void> clearMono = set.clear(0);

Mono<Void> andMono = set.and("anotherBitset);
Mono<Void> xorMono = set.xor("anotherBitset");
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RBitSetRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RBitSetRx stream = redisson.getBitSet("simpleBitset");

Single<Boolean> setRx = set.set(0, true);
Single<Boolean> setRx = set.set(1812, false);

Completable clearRx = set.clear(0);

Completable andRx = set.and("anotherBitset);
Completable xorRx = set.xor("anotherBitset");
```
### BitSet 数据分区（分布式 roaring 位图）
尽管“RBitSet”对象与集群兼容，但其内容无法跨多个 Redis 主节点进行扩展。 BitSet 数据分区仅在集群模式下可用，并由单独的 RClusteredBitSet 对象实现。它使用roaring 位图结构的分布式实现。大小受整个 Redis 集群内存的限制。有关分区的更多详细信息请参见[此处](#b2Cfd)。
以下是所有可用 BitSet 实现的列表：

| RedissonClient method name | Data partitioning support | Ultra-fast read/write |
| --- | --- | --- |
| getBitSet() _open-source version_ | ❌ | ❌ |
| getBitSet() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ✔️ |
| getClusteredBitSet() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ |

Code example:
```java
RClusteredBitSet set = redisson.getClusteredBitSet("simpleBitset");
set.set(0, true);
set.set(1812, false);
set.clear(0);
set.addAsync("e");
set.xor("anotherBitset");
```
## AtomicLong
基于 Redis 的 `AtomicLong` 对象的 Java 实现提供了类似于 `java.util.concurrent.atomic.AtomicLong` 对象的 API。
Code example:
```java
RAtomicLong atomicLong = redisson.getAtomicLong("myAtomicLong");
atomicLong.set(3);
atomicLong.incrementAndGet();
atomicLong.get();
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicLongAsync.html)** interface** usage:
```java
RAtomicLongAsync atomicLong = redisson.getAtomicLong("myAtomicLong");

RFuture<Void> setFuture = atomicLong.setAsync(3);
RFuture<Long> igFuture = atomicLong.incrementAndGetAsync();
RFuture<Long> getFuture = atomicLong.getAsync();
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicLongReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RAtomicLongReactive atomicLong = redisson.getAtomicLong("myAtomicLong");

Mono<Void> setMono = atomicLong.set(3);
Mono<Long> igMono = atomicLong.incrementAndGet();
RFuture<Long> getMono = atomicLong.getAsync();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicLongRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RAtomicLongRx atomicLong = redisson.getAtomicLong("myAtomicLong");

Completable setMono = atomicLong.set(3);
Single<Long> igMono = atomicLong.incrementAndGet();
Single<Long> getMono = atomicLong.getAsync();
```
## AtomicDouble
基于 Redis 的 AtomicDouble 对象的 Java 实现。
Code example:
```java
RAtomicDouble atomicDouble = redisson.getAtomicDouble("myAtomicDouble");
atomicDouble.set(2.81);
atomicDouble.addAndGet(4.11);
atomicDouble.get();
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicDoubleAsync.html)** interface** usage:
```java
RAtomicDoubleAsync atomicDouble = redisson.getAtomicDouble("myAtomicDouble");

RFuture<Void> setFuture = atomicDouble.setAsync(2.81);
RFuture<Double> agFuture = atomicDouble.addAndGetAsync(4.11);
RFuture<Double> getFuture = atomicDouble.getAsync();
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicDoubleReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RAtomicDoubleReactive atomicDouble = redisson.getAtomicDouble("myAtomicDouble");

Mono<Void> setMono = atomicDouble.set(2.81);
Mono<Double> agMono = atomicDouble.addAndGet(4.11);
Mono<Double> getMono = atomicDouble.get();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RAtomicDoubleRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RAtomicDoubleRx atomicDouble = redisson.getAtomicDouble("myAtomicDouble");

Completable setMono = atomicDouble.set(2.81);
Single<Double> igMono = atomicDouble.addAndGet(4.11);
Single<Double> getMono = atomicDouble.get();
```
## Topic
基于Redis的RTopic对象的Java实现实现了发布/订阅机制。它允许订阅使用同名的 RTopic 对象的多个实例发布的事件。 
重新连接到 Redis 或 Redis 故障转移后，侦听器会自动重新订阅。在没有连接到 Redis 期间发送的所有消息都会丢失。使用可靠主题进行可靠交付。
Code example:
```java
RTopic topic = redisson.getTopic("myTopic");
int listenerId = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RTopic topic = redisson.getTopic("myTopic");
long clientsReceivedMessage = topic.publish(new SomeObject());
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTopicAsync.html)** interface** usage:
```java
RTopicAsync topic = redisson.getTopic("myTopic");
RFuture<Integer> listenerFuture = topic.addListenerAsync(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RTopicAsync topic = redisson.getTopic("myTopic");
RFuture<Long> publishFuture = topic.publishAsync(new SomeObject());
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTopicReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RTopicReactive topic = redisson.getTopic("myTopic");
Mono<Integer> listenerMono = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RTopicReactive topic = redisson.getTopic("myTopic");
Mono<Long> publishMono = topic.publish(new SomeObject());
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTopicRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RTopicRx topic = redisson.getTopic("myTopic");
Single<Integer> listenerMono = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RTopicRx topic = redisson.getTopic("myTopic");
Single<Long> publishMono = topic.publish(new SomeObject());
```
### Topic pattern
基于Redis的`RPatternTopic`对象的Java实现。**它允许通过指定的全局样式模式订阅多个主题**。 重新连接到 Redis 或 Redis 故障转移后，侦听器会自动重新订阅。 模式示例：

- topic? subscribes to topic1, topicA ...
- topic?_my subscribes to topic_my, topic123_my, topicTEST_my ...
- topic[ae] subscribes to topica and topice only

Code example:
```java
// subscribe to all topics by `topic*` pattern
RPatternTopic patternTopic = redisson.getPatternTopic("topic*");
int listenerId = patternTopic.addListener(Message.class, new PatternMessageListener<Message>() {
    @Override
    public void onMessage(String pattern, String channel, Message msg) {
        //...
    }
});
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPatternTopicAsync.html)** interface** usage:
```java
RPatternTopicAsync patternTopic = redisson.getPatternTopic("topic*");
RFuture<Integer> listenerFuture = patternTopic.addListenerAsync(Message.class, new PatternMessageListener<Message>() {
    @Override
    public void onMessage(String pattern, String channel, Message msg) {
        //...
    }
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPatternTopicReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RTopicReactive patternTopic = redisson.getPatternTopic("topic*");
Mono<Integer> listenerMono = patternTopic.addListener(Message.class, new PatternMessageListener<Message>() {
    @Override
    public void onMessage(String pattern, String channel, Message msg) {
        //...
    }
});
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPatternTopicRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RTopicRx patternTopic = redisson.getPatternTopic("topic*");
Single<Integer> listenerSingle = patternTopic.addListener(Message.class, new PatternMessageListener<Message>() {
    @Override
    public void onMessage(String pattern, String channel, Message msg) {
        //...
    }
});
```
### Sharded topic
基于 Redis 的`RShardedTopic`对象的Java实现实现了分片发布/订阅机制。它允许订阅使用具有相同名称的 `RShardedTopic` 对象的多个实例发布的事件。**订阅/发布操作仅在 Cluster 中与特定主题名称绑定的 Redis 节点上执行（发布和订阅都在固定的Redis节点）**。对于 `RTopic` 对象，通过 `RShardedTopic` 发布的消息不会在所有 Redis 节点上广播。这减少了网络带宽和 Redis 负载。 重新连接到 Redis 或 Redis 故障转移后，侦听器会自动重新订阅。在没有连接到 Redis 期间发送的所有消息都会丢失。使用可靠主题进行可靠交付。
Code example:
```java
RShardedTopic topic = redisson.getShardedTopic("myTopic");
int listenerId = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RShardedTopic topic = redisson.getShardedTopic("myTopic");
long clientsReceivedMessage = topic.publish(new SomeObject());
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RShardedTopicAsync.html)** interface** usage:
```java
RShardedTopicAsync topic = redisson.getShardedTopic("myTopic");
RFuture<Integer> listenerFuture = topic.addListenerAsync(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RShardedTopicAsync topic = redisson.getShardedTopic("myTopic");
RFuture<Long> publishFuture = topic.publishAsync(new SomeObject());
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RShardedTopicReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RShardedTopicReactive topic = redisson.getShardedTopic("myTopic");
Mono<Integer> listenerMono = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RShardedTopicReactive topic = redisson.getShardedTopic("myTopic");
Mono<Long> publishMono = topic.publish(new SomeObject());
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RShardedTopicRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RShardedTopicRx topic = redisson.getShardedTopic("myTopic");
Single<Integer> listenerMono = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(String channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RShardedTopicRx topic = redisson.getShardedTopic("myTopic");
Single<Long> publishMono = topic.publish(new SomeObject());
```
## Bloom filter
用于 Java 的基于 Redis 的分布式 RBloomFilter 布隆过滤器。包含的位数限制为 2^32。 使用前必须通过 `tryInit(expectedInsertions, falseProbability)` 方法初始化容量大小。
```java
RBloomFilter<SomeObject> bloomFilter = redisson.getBloomFilter("sample");
// initialize bloom filter with 
// expectedInsertions = 55000000
// falseProbability = 0.03
bloomFilter.tryInit(55000000L, 0.03);

bloomFilter.add(new SomeObject("field1Value", "field2Value"));
bloomFilter.add(new SomeObject("field5Value", "field8Value"));

bloomFilter.contains(new SomeObject("field1Value", "field8Value"));
bloomFilter.count();
```
### Bloom filter 数据分区
_This feature available only in _[Redisson PRO](https://redisson.pro/)_ edition._
尽管“RBloomFilter”对象与集群兼容，但其内容无法跨多个 Redis 主节点进行扩展。 Bloom Filter 数据分区支持仅在集群模式下可用，并由单独的 RClusteredBloomFilter 对象实现。该实现使用更高效的分布式Redis内存分配算法。它允许“缩小”所有 Redis 节点上未使用位消耗的内存空间。每个实例的状态分布在 Redis 集群中的所有节点上。包含的位数限制为 2^64。有关分区的更多详细信息请参见[此处](#b2Cfd)。
下面是所有可用的 BloomFilter 实现的列表：

| RedissonClient method name | Data partitioning support | Ultra-fast read/write |
| --- | --- | --- |
| getBloomFilter() _open-source version_ | ❌ | ❌ |
| getBloomFilter() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ✔️ |
| getClusteredBloomFilter() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ |

```java
RClusteredBloomFilter<SomeObject> bloomFilter = redisson.getClusteredBloomFilter("sample");
// initialize bloom filter with 
// expectedInsertions = 255000000
// falseProbability = 0.03
bloomFilter.tryInit(255000000L, 0.03);
bloomFilter.add(new SomeObject("field1Value", "field2Value"));
bloomFilter.add(new SomeObject("field5Value", "field8Value"));
bloomFilter.contains(new SomeObject("field1Value", "field8Value"));
```
## HyperLogLog
用于 Java 的基于 Redis 的分布式 RhyperLogLog 对象。概率数据结构可让您以极高的空间效率维护数百万个项目的计数。 它具有 Async、Reactive 和 RxJava3 接口:
```java
RHyperLogLog<Integer> log = redisson.getHyperLogLog("log");
log.add(1);
log.add(2);
log.add(3);

log.count();
```
## LongAdder
基于Redis的LongAdder对象的Java实现提供了类似于`java.util.concurrent.atomic.LongAdder`对象的API。 它在客户端维护内部 LongAdder 对象，并为递增和递减操作提供卓越的性能。比类似的 AtomicLong 对象快 12000 倍。适用于分布式度量对象。 代码示例：
```java
RLongAdder atomicLong = redisson.getLongAdder("myLongAdder");
atomicLong.add(12);
atomicLong.increment();
atomicLong.decrement();
atomicLong.sum();
```
如果不再使用对象，则应将其销毁，但如果 Redisson 关闭，则无需调用 destroy 方法。
```java
RLongAdder atomicLong = ...
atomicLong.destroy();
```
## DoubleAdder
基于Redis的DoubleAdder对象的Java实现提供了类似于`java.util.concurrent.atomic.DoubleAdder`对象的API。 它在客户端维护内部 DoubleAdder 对象，并为递增和递减操作提供卓越的性能。比类似的 AtomicDouble 对象快 12000 倍。适用于分布式度量对象。 代码示例：
```java
RLongDouble atomicDouble = redisson.getLongDouble("myLongDouble");
atomicDouble.add(12);
atomicDouble.increment();
atomicDouble.decrement();
atomicDouble.sum();
```
如果不再使用对象，则应将其销毁，但如果 Redisson 关闭，则无需调用 destroy 方法。
```java
RLongDouble atomicDouble = ...
atomicDouble.destroy();
```
## RateLimiter
用于 Java 的基于 Redis 的分布式 RateLimiter 对象限制来自所有线程（无论 Redisson 实例如何）或来自使用同一 Redisson 实例的所有线程的总调用速率。不保证公平。
Code example:
```java
RRateLimiter limiter = redisson.getRateLimiter("myLimiter");
// Initialization required only once.
// 5 permits per 2 seconds
limiter.trySetRate(RateType.OVERALL, 5, 2, RateIntervalUnit.SECONDS);

// acquire 3 permits or block until they became available       
limiter.acquire(3);
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRateLimiterAsync.html)** interface** usage:
```java
RRateLimiter limiter = redisson.getRateLimiter("myLimiter");
// Initialization required only once.
// 5 permits per 2 seconds
RFuture<Boolean> setRateFuture = limiter.trySetRate(RateType.OVERALL, 5, 2, RateIntervalUnit.SECONDS);

// acquire 3 permits or block until they became available       
RFuture<Void> aquireFuture = limiter.acquire(3);
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRateLimiterReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RRateLimiterReactive limiter = redisson.getRateLimiter("myLimiter");
// Initialization required only once.
// 5 permits per 2 seconds
Mono<Boolean> setRateMono = limiter.trySetRate(RateType.OVERALL, 5, 2, RateIntervalUnit.SECONDS);

// acquire 3 permits or block until they became available       
Mono<Void> aquireMono = limiter.acquire(3);
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRateLimiterRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RRateLimiterRx limiter = redisson.getRateLimiter("anyObject");

// Initialization required only once.
// 5 permits per 2 seconds
Single<Boolean> setRateRx = limiter.trySetRate(RateType.OVERALL, 5, 2, RateIntervalUnit.SECONDS);

// acquire 3 permits or block until they became available       
Completable aquireRx = limiter.acquire(3);
```
## Reliable Topic
基于 Redis 的 `RReliableTopic` 对象的 Java 实现实现了可靠的消息传递的发布/订阅机制。如果 Redis 连接中断，所有丢失的消息都会在重新连接到 Redis 后传递。当 Redisson 收到消息并提交给主题侦听器进行处理时，消息被视为已送达。 
每个 `RReliableTopic` 对象实例（订阅者）都有自己的看门狗，该看门狗在注册第一个侦听器时启动。如果看门狗没有将其延长到下一个超时时间间隔，订阅者将在 `org.redisson.config.Config#reliableTopicWatchdogTimeout` 超时后过期。这可以防止由于 Redisson 客户端崩溃或订阅者无法使用消息时的任何其他原因导致主题中存储的消息无限增长。 
重新连接到 Redis 或 Redis 故障转移后，主题侦听器会自动重新订阅。
Code example:
```java
RReliableTopic topic = redisson.getReliableTopic("anyTopic");
topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(CharSequence channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RReliableTopic topic = redisson.getReliableTopic("anyTopic");
long subscribersReceivedMessage = topic.publish(new SomeObject());
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RReliableTopicAsync.html)** interface** usage:
```java
RReliableTopicAsync topic = redisson.getReliableTopic("anyTopic");
RFuture<String> listenerFuture = topic.addListenerAsync(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(CharSequence channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RReliableTopicAsync topic = redisson.getReliableTopic("anyTopic");
RFuture<Long> future = topic.publishAsync(new SomeObject());
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RReliableTopicReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();

RReliableTopicReactive topic = redisson.getReliableTopic("anyTopic");
Mono<String> listenerMono = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(CharSequence channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RReliableTopicReactive topic = redisson.getReliableTopic("anyTopic");
Mono<Long> publishMono = topic.publish(new SomeObject());
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RReliableTopicRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();

RReliableTopicRx topic = redisson.getReliableTopic("anyTopic");
Single<String> listenerRx = topic.addListener(SomeObject.class, new MessageListener<SomeObject>() {
    @Override
    public void onMessage(CharSequence channel, SomeObject message) {
        //...
    }
});

// in other thread or JVM
RReliableTopicRx topic = redisson.getReliableTopic("anyTopic");
Single<Long> publisRx = topic.publish(new SomeObject());
```
## Id generator
基于 Redis 的 Java Id 生成器 `RIdGenerator` 生成唯一的数字，但不是单调递增的。在第一次请求时，一批 id 号被分配并缓存在 Java 端，直到耗尽。这种方法可以比 `RAtomicLong` 更快地生成 id。 默认分配大小为 5000。默认起始值为 0。
Code example:
```java
RIdGenerator generator = redisson.getIdGenerator("generator");

// Initialize with start value = 12 and allocation size = 20000
generator.tryInit(12, 20000);

long id = generator.nextId();
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RIdGeneratorAsync.html)** interface** usage:
```java
RIdGenerator generator = redisson.getIdGenerator("generator");

// Initialize with start value = 12 and allocation size = 20000
RFuture<Boolean> initFuture = generator.tryInitAsync(12, 20000);

RFuture<Long> idFuture = generator.nextIdAsync();
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RIdGeneratorReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RIdGenerator generator = redisson.getIdGenerator("generator");

// Initialize with start value = 12 and allocation size = 20000
Mono<Boolean> initMono = generator.tryInit(12, 20000);

Mono<Long> idMono = generator.nextId();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RIdGeneratorRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RIdGenerator generator = redisson.getIdGenerator("generator");

// Initialize with start value = 12 and allocation size = 20000
Single<Boolean> initRx = generator.tryInit(12, 20000);

Single<Long> idRx = generator.nextId();
```
## JSON object holder
`RJsonBucket` 对象的 Java 实现使用 `JSON.*` Redis 命令以 JSON 格式存储数据。 JSON 数据编码/解码由 `JsonCodec` 处理，这是必需参数。可用的实现是 `org.redisson.codec.JacksonCodec`。
### JSON object holder local cache
Redisson 提供带有本地缓存的 JSON 对象持有者实现。 
本地缓存 - 所谓的近缓存，用于加速读取操作并避免网络往返。它在 Redisson 端缓存整个 JSON 对象，并且执行读取操作的速度比常见实现快 45 倍。具有相同名称的本地缓存实例连接到相同的发布/订阅通道。该通道用于在实例之间交换无效事件。

| RedissonClient method name | Local cache | Ultra-fast read/write |
| --- | --- | --- |
| getJsonBucket() _open-source version_ | ❌ | ❌ |
| getJsonBucket() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ✔️ |
| getLocalCachedJsonBucket() [Redisson PRO](https://redisson.pro/) _ version_ | ✔️ | ✔️ |

Code example:
```java
RJsonBucket<AnyObject> bucket = redisson.getJsonBucket("anyObject", new JacksonCodec<>(AnyObject.class));

bucket.set(new AnyObject(1));
AnyObject obj = bucket.get();

bucket.trySet(new AnyObject(3));
bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
bucket.getAndSet(new AnyObject(6));

List<String> values = bucket.get(new JacksonCodec<>(new TypeReference<List<String>>() {}), "values");
long aa = bucket.arrayAppend("$.obj.values", "t3", "t4");
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RJsonBucketAsync.html)** interface** usage:
```java
RJsonBucket<AnyObject> bucket = redisson.getJsonBucket("anyObject", new JacksonCodec<>(AnyObject.class));

RFuture<Void> future = bucket.setAsync(new AnyObject(1));
RFuture<AnyObject> objfuture = bucket.getAsync();

RFuture<Boolean> tsFuture = bucket.trySetAsync(new AnyObject(3));
RFuture<Boolean> csFuture = bucket.compareAndSetAsync(new AnyObject(4), new AnyObject(5));
RFuture<AnyObject> gsFuture = bucket.getAndSetAsync(new AnyObject(6));

RFutue<List<String>> gFuture = bucket.getAsync(new JacksonCodec<>(new TypeReference<List<String>>() {}), "obj.values");
RFutue<Long> aaFuture = bucket.arrayAppendAsync("$.obj.values", "t3", "t4");
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RJsonBucketReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RJsonBucketReactive<AnyObject> bucket = redisson.getJsonBucket("anyObject", new JacksonCodec<>(AnyObject.class));

Mono<Void> mono = bucket.set(new AnyObject(1));
Mono<AnyObject> objMono = bucket.get();

Mono<Boolean> tsMono = bucket.trySet(new AnyObject(3));
Mono<Boolean> csMono = bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
Mono<AnyObject> gsMono = bucket.getAndSet(new AnyObject(6));

Mono<List<String>> vsMono = bucket.get(new JacksonCodec<>(new TypeReference<List<String>>() {}), "values");
Mono<Long> aaMono = bucket.arrayAppend("$.obj.values", "t3", "t4");
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RJsonBucketRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RJsonBucketRx<AnyObject> bucket = redisson.getJsonBucket("anyObject", new JacksonCodec<>(AnyObject.class));

Completable rx = bucket.set(new AnyObject(1));
Maybe<AnyObject> objRx = bucket.get();

Single<Boolean> tsRx = bucket.trySet(new AnyObject(3));
Single<Boolean> csRx = bucket.compareAndSet(new AnyObject(4), new AnyObject(5));
Maybe<AnyObject> gsRx = bucket.getAndSet(new AnyObject(6));

Single<List<String>> valuesRx = bucket.get(new JacksonCodec<>(new TypeReference<List<String>>() {}), "values");
Single<Long> aaRx = bucket.arrayAppend("$.obj.values", "t3", "t4");
```
# Distributed collections 分布式集合
## Map
Java 中基于 Redis 的分布式 Map 对象实现了 `ConcurrentMap` 接口。该对象是完全线程安全的。考虑使用 `Live Object 服务`将 POJO 对象存储为 Redis Map。 Redis 使用序列化状态来检查键的唯一性，而不是键的 hashCode()/equals() 方法。 它具有 Async、Reactive 和 RxJava3 接口。
如果 Map 主要用于读取操作和/或网络往返是不可取的，请使用具有`本地缓存`支持的 Map。
```java
RMap<String, SomeObject> map = redisson.getMap("anyMap");
SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

// use fast* methods when previous value is not required
map.fastPut("a", new SomeObject());
map.fastPutIfAbsent("d", new SomeObject());
map.fastRemove("b");

RFuture<SomeObject> putAsyncFuture = map.putAsync("321");
RFuture<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```
RMap 对象允许为每个键绑定一个 `Lock/ReadWriteLock/Semaphore/CountDownLatch` 对象：
```java
RMap<MyKey, MyValue> map = redisson.getMap("anyMap");
MyKey k = new MyKey();
RLock keyLock = map.getLock(k);
keyLock.lock();
try {
   MyValue v = map.get(k);
   // process value ...
} finally {
   keyLock.unlock();
}

RReadWriteLock rwLock = map.getReadWriteLock(k);
rwLock.readLock().lock();
try {
   MyValue v = map.get(k);
   // process value ...
} finally {
   keyLock.readLock().unlock();
}
```
### Map eviction, local cache and data partitioning
Redisson提供了各种Map结构实现，具有三个重要特性：
`本地缓存` - 所谓的近缓存，用于加速读取操作并避免网络往返。它在 Redisson 端缓存 Map 条目，并且执行读取操作的速度比常见实现快 45 倍。具有相同名称的本地缓存实例连接到相同的发布/订阅通道。该通道用于在所有实例之间交换更新/无效事件。本地缓存存储不使用关键对象的 `hashCode()`/`equals()` 方法，而是使用序列化状态的哈希值。
`数据分区` - 尽管 Map 对象与集群兼容，但其内容不会跨集群中的多个 Redis 主节点进行缩放/分区。数据分区允许扩展 Redis 集群中单个 Map 实例的可用内存、读/写操作和entry淘汰过程。
`entry淘汰` - 允许定义每个 Map entry 的生存时间或最大空闲时间。 Redis 哈希结构不支持淘汰，因此它是在 Redisson 端通过自定义计划任务来完成的，该任务会删除过期的entry。淘汰任务通过每个唯一对象名称执行 getMapCache() 方法启动一次。因此，即使实例未被使用并且entry已过期，也应该通过 getMapCache() 方法来启动淘汰过程。这会导致每个唯一的映射对象名称需要额外的 Redis 调用和淘汰任务。
`高级entry淘汰` - entry淘汰过程的改进版本。不使用entry淘汰任务。
 
下面是所有可用的 Map 实现的列表：

| RedissonClient method name | Local cache | Data partitioning | Entry eviction | Advanced entry eviction | Ultra-fast read/write |
| --- | --- | --- | --- | --- | --- |
| getMap() _open-source version_ | ❌ | ❌ | ❌ | ❌ | ❌ |
| getMapCache() _open-source version_ | ❌ | ❌ | ✔️ | ❌ | ❌ |
| getLocalCachedMap() _open-source version_ | ✔️ | ❌ | ❌ | ❌ | ❌ |
| getMap() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ❌ | ❌ | ❌ | ✔️ |
| getMapCache() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ❌ | ✔️ | ❌ | ✔️ |
| getMapCacheV2() _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ❌ | ✔️ | ✔️ |
| getLocalCachedMap() [Redisson PRO](https://redisson.pro/) _ version_ | ✔️ | ❌ | ❌ | ❌ | ✔️ |
| getLocalCachedMapCache() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ❌ | ✔️ | ❌ | ✔️ |
| getClusteredMap() _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ❌ | ❌ | ✔️ |
| getClusteredMapCache() _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ✔️ | ❌ | ✔️ |
| getClusteredLocalCachedMap() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ | ❌ | ❌ | ✔️ |
| getClusteredLocalCachedMapCache() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ | ✔️ | ❌ | ✔️ |

Redisson还提供`Spring Cache`和`JCache`实现。
#### 淘汰任务
具有淘汰支持的 Map 对象实现 `org.redisson.api.RMapCache` 接口并扩展 `java.util.concurrent.ConcurrentMap` 接口。它还具有 `Async`、`Reactive` 和 `RxJava3`接口。 
当前的 Redis 实现没有Map entry淘汰功能。因此，过期的entry会被 `org.redisson.eviction.EvictionScheduler` 不时清理。它一次删除 100 个过期entry。任务启动时间自动调整，取决于上次删除的过期entry数量，变化范围为 5 秒到半小时。因此，如果 clean 任务每次删除 100 个entry，它将每 5 秒执行一次（最小执行延迟）。但如果当前过期entry数量低于前一个，则执行延迟将增加 1.5 倍，反之则减少。 建议对每个 Redisson 客户端实例使用同名的 RMapCache 对象的单个实例。
 代码示例：
```java
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap", MapCacheOptions.defaults());
// or
RMapCacheV2<String, SomeObject> map = redisson.getMapCacheV2("anyMap");
// or
RMapCacheV2<String, SomeObject> map = redisson.getMapCacheV2("anyMap", MapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getLocalCachedMapCache("anyMap", LocalCachedMapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredLocalCachedMapCache("anyMap", LocalCachedMapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap", MapCacheOptions.defaults());


// ttl = 10 minutes, 
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES);
// ttl = 10 minutes, maxIdleTime = 10 seconds
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES, 10, TimeUnit.SECONDS);

// ttl = 3 seconds
map.putIfAbsent("key2", new SomeObject(), 3, TimeUnit.SECONDS);
// ttl = 40 seconds, maxIdleTime = 10 seconds
map.putIfAbsent("key2", new SomeObject(), 40, TimeUnit.SECONDS, 10, TimeUnit.SECONDS);

// if object is not used anymore
map.destroy();
```
#### 本地缓存
具有本地缓存支持的 Map 对象实现了 `org.redisson.api.RLocalCachedMap` ，它扩展了 `java.util.concurrent.ConcurrentMap` 接口。该对象是完全线程安全的。
建议为每个 Redisson 客户端实例使用每个名称的 `LocalCachedMap` 实例的单个实例。应在具有相同名称的所有实例中使用相同的 `LocalCachedMapOptions` 对象。
在对象创建期间可以提供以下选项：
```java
      LocalCachedMapOptions options = LocalCachedMapOptions.defaults()

      // Defines whether to store a cache miss into the local cache.
      // Default value is false.
      .storeCacheMiss(false);

      // Defines store mode of cache data.
      // Follow options are available:
      // LOCALCACHE - store data in local cache only and use Redis only for data update/invalidation.
      // LOCALCACHE_REDIS - store data in both Redis and local cache.
      .storeMode(StoreMode.LOCALCACHE_REDIS)

      // Defines Cache provider used as local cache store.
      // Follow options are available:
      // REDISSON - uses Redisson own implementation
      // CAFFEINE - uses Caffeine implementation
      .cacheProvider(CacheProvider.REDISSON)

      // Defines local cache eviction policy.
      // Follow options are available:
      // LFU - Counts how often an item was requested. Those that are used least often are discarded first.
      // LRU - Discards the least recently used items first
      // SOFT - Uses soft references, entries are removed by GC
      // WEAK - Uses weak references, entries are removed by GC
      // NONE - No eviction
     .evictionPolicy(EvictionPolicy.NONE)

      // If cache size is 0 then local cache is unbounded.
     .cacheSize(1000)

      // Defines strategy for load missed local cache updates after Redis connection failure.
      //
      // Follow reconnection strategies are available:
      // CLEAR - Clear local cache if map instance has been disconnected for a while.
      // LOAD - Store invalidated entry hash in invalidation log for 10 minutes
      //        Cache keys for stored invalidated entry hashes will be removed 
      //        if LocalCachedMap instance has been disconnected less than 10 minutes
      //        or whole cache will be cleaned otherwise.
      // NONE - Default. No reconnection handling
     .reconnectionStrategy(ReconnectionStrategy.NONE)

      // Defines local cache synchronization strategy.
      //
      // Follow sync strategies are available:
      // INVALIDATE - Default. Invalidate cache entry across all LocalCachedMap instances on map entry change
      // UPDATE - Insert/update cache entry across all LocalCachedMap instances on map entry change
      // NONE - No synchronizations on map changes
     .syncStrategy(SyncStrategy.INVALIDATE)

      // time to live for each map entry in local cache
     .timeToLive(10000)
      // or
     .timeToLive(10, TimeUnit.SECONDS)

      // max idle time for each map entry in local cache
     .maxIdle(10000)
      // or
     .maxIdle(10, TimeUnit.SECONDS);
```
代码示例：
```java
RLocalCachedMap<String, Integer> map = redisson.getLocalCachedMap("test", LocalCachedMapOptions.defaults());
// or
RLocalCachedMap<String, SomeObject> map = redisson.getLocalCachedMapCache("anyMap", LocalCachedMapCacheOptions.defaults());
// or
RLocalCachedMap<String, SomeObject> map = redisson.getClusteredLocalCachedMapCache("anyMap", LocalCachedMapCacheOptions.defaults());
// or
RLocalCachedMap<String, SomeObject> map = redisson.getClusteredLocalCachedMap("anyMap", LocalCachedMapOptions.defaults());

        
String prevObject = map.put("123", 1);
String currentObject = map.putIfAbsent("323", 2);
String obj = map.remove("123");

// use fast* methods when previous value is not required
map.fastPut("a", 1);
map.fastPutIfAbsent("d", 32);
map.fastRemove("b");

RFuture<String> putAsyncFuture = map.putAsync("321");
RFuture<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```
如果对象不再使用，则应将其销毁，但如果 Redisson 关闭，则无需调用 destroy 方法。
```java
RLocalCachedMap<String, Integer> map = ...
map.destroy();
```
#### 如何加载数据以避免无效消息流量
代码示例：
```java
    public void loadData(String cacheName, Map<String, String> data) {
        RLocalCachedMap<String, String> clearMap = redisson.getLocalCachedMap(cacheName, 
                LocalCachedMapOptions.defaults().cacheSize(1).syncStrategy(SyncStrategy.INVALIDATE));
        RLocalCachedMap<String, String> loadMap = redisson.getLocalCachedMap(cacheName, 
                LocalCachedMapOptions.defaults().cacheSize(1).syncStrategy(SyncStrategy.NONE));
        
        loadMap.putAll(data);
        clearMap.clearLocalCache();
    }
```
#### 数据分区
具有数据分区支持的Map对象实现了`org.redisson.api.RClusteredMap`，它扩展了`java.util.concurrent.ConcurrentMap`接口。在[此处](#b2Cfd)阅读有关数据分区的更多详细信息。 代码示例：
```java
RClusteredMap<String, SomeObject> map = redisson.getClusteredMap("anyMap");
// or
RClusteredMap<String, SomeObject> map = redisson.getClusteredLocalCachedMapCache("anyMap", LocalCachedMapCacheOptions.defaults());
// or
RClusteredMap<String, SomeObject> map = redisson.getClusteredLocalCachedMap("anyMap", LocalCachedMapOptions.defaults());
// or
RClusteredMap<String, SomeObject> map = redisson.getClusteredMapCache("anyMap");

SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");
```
### Map 持久化
Redisson 允许将 Map 数据与 Redis 存储一起存储在外部存储中。 
用例： 

1. Redisson Map 对象作为应用程序和外部存储之间的缓存
2. 提高 Redisson Map 数据的持久性和被淘汰entry的寿命
3. 缓存数据库、Web 服务或任何其他数据源
#### Read-through 策略
如果 Redisson Map 对象中不存在请求的entry，则将使用提供的 MapLoader 对象加载该entry。代码示例：
```java
        MapLoader<String, String> mapLoader = new MapLoader<String, String>() {
            
            @Override
            public Iterable<String> loadAllKeys() {
                List<String> list = new ArrayList<String>();
                Statement statement = conn.createStatement();
                try {
                    ResultSet result = statement.executeQuery("SELECT id FROM student");
                    while (result.next()) {
                        list.add(result.getString(1));
                    }
                } finally {
                    statement.close();
                }

                return list;
            }
            
            @Override
            public String load(String key) {
                PreparedStatement preparedStatement = conn.prepareStatement("SELECT name FROM student where id = ?");
                try {
                    preparedStatement.setString(1, key);
                    ResultSet result = preparedStatement.executeQuery();
                    if (result.next()) {
                        return result.getString(1);
                    }
                    return null;
                } finally {
                    preparedStatement.close();
                }
            }
        };
```
配置示例：
```java
MapOptions<K, V> options = MapOptions.<K, V>defaults()
                              .loader(mapLoader);

MapCacheOptions<K, V> mcoptions = MapCacheOptions.<K, V>defaults()
                              .loader(mapLoader);


RMap<K, V> map = redisson.getMap("test", options);
// or
RMapCache<K, V> map = redisson.getMapCache("test", mcoptions);
// or with performance boost up to 45x times 
RLocalCachedMap<K, V> map = redisson.getLocalCachedMap("test", options);
// or with performance boost up to 45x times 
RLocalCachedMapCache<K, V> map = redisson.getLocalCachedMapCache("test", mcoptions);
```
#### Write-through (同步) 策略
当 Map entry被更新时，方法不会返回，直到 Redisson 使用 `MapWriter` 对象在外部存储中更新它。代码示例：
```java
        MapWriter<String, String> mapWriter = new MapWriter<String, String>() {
            
            @Override
            public void write(Map<String, String> map) {
                PreparedStatement preparedStatement = conn.prepareStatement("INSERT INTO student (id, name) values (?, ?)");
                try {
                    for (Entry<String, String> entry : map.entrySet()) {
                        preparedStatement.setString(1, entry.getKey());
                        preparedStatement.setString(2, entry.getValue());
                        preparedStatement.addBatch();
                    }
                    preparedStatement.executeBatch();
                } finally {
                    preparedStatement.close();
                }
            }
            
            @Override
            public void delete(Collection<String> keys) {
                PreparedStatement preparedStatement = conn.prepareStatement("DELETE FROM student where id = ?");
                try {
                    for (String key : keys) {
                        preparedStatement.setString(1, key);
                        preparedStatement.addBatch();
                    }
                    preparedStatement.executeBatch();
                } finally {
                    preparedStatement.close();
                }
            }
        };
```
配置示例：
```java
MapOptions<K, V> options = MapOptions.<K, V>defaults()
                              .writer(mapWriter)
                              .writeMode(WriteMode.WRITE_THROUGH);

MapCacheOptions<K, V> mcoptions = MapCacheOptions.<K, V>defaults()
                              .writer(mapWriter)
                              .writeMode(WriteMode.WRITE_THROUGH);


RMap<K, V> map = redisson.getMap("test", options);
// or
RMapCache<K, V> map = redisson.getMapCache("test", mcoptions);
// or with performance boost up to 45x times 
RLocalCachedMap<K, V> map = redisson.getLocalCachedMap("test", options);
// or with performance boost up to 45x times 
RLocalCachedMapCache<K, V> map = redisson.getLocalCachedMapCache("test", mcoptions);
```
#### Write-behind (同步) 策略
Map对象的更新会批量累积，并通过MapWriter对象以定义的延迟异步写入外部存储。 `writeBehindDelay` - 批量写入或删除操作的延迟。默认值为 1000 毫秒。 `writeBehindBatchSize` - 批次大小。每个批次包含地图条目写入或删除命令。默认值为 50。
配置示例：
```java
MapOptions<K, V> options = MapOptions.<K, V>defaults()
                              .writer(mapWriter)
                              .writeMode(WriteMode.WRITE_BEHIND)
                              .writeBehindDelay(5000)
                              .writeBehindBatchSize(100);

MapCacheOptions<K, V> mcoptions = MapCacheOptions.<K, V>defaults()
                              .writer(mapWriter)
                              .writeMode(WriteMode.WRITE_BEHIND)
                              .writeBehindDelay(5000)
                              .writeBehindBatchSize(100);


RMap<K, V> map = redisson.getMap("test", options);
// or
RMapCache<K, V> map = redisson.getMapCache("test", mcoptions);
// or with performance boost up to 45x times 
RLocalCachedMap<K, V> map = redisson.getLocalCachedMap("test", options);
// or with performance boost up to 45x times 
RLocalCachedMapCache<K, V> map = redisson.getLocalCachedMapCache("test", mcoptions);
```
此功能可用于 `RMap`、`RMapCache`、`RLocalCachedMap` 和 `RLocalCachedMapCache` 对象。 使用 `RLocalCachedMap` 和 `RLocalCachedMapCache` 对象可将 Redis 读取操作提高高达 **45 倍**，并为数据库、Web 服务或任何其他数据源提供几乎即时的速度。
### Map 侦听器
Redisson 允许为实现 `RMapCache` 接口的 Map 对象绑定侦听器并跟踪 Map 数据的后续事件：
Entry 创建 - `org.redisson.api.map.event.EntryCreatedListener`
Entry 过期 - `org.redisson.api.map.event.EntryExpiredListener`
Entry 删除 - `org.redisson.api.map.event.EntryRemovedListener`
Entry 更新 - `org.redisson.api.map.event.EntryUpdatedListener`

使用示例：
```java
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap", MapCacheOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getLocalCachedMapCache("anyMap", LocalCachedMapCacheOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredLocalCachedMapCache("anyMap", LocalCachedMapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap", MapCacheOptions.defaults());


int updateListener = map.addListener(new EntryUpdatedListener<Integer, Integer>() {
     @Override
     public void onUpdated(EntryEvent<Integer, Integer> event) {
          event.getKey(); // key
          event.getValue() // new value
          event.getOldValue() // old value
          // ...
     }
});

int createListener = map.addListener(new EntryCreatedListener<Integer, Integer>() {
     @Override
     public void onCreated(EntryEvent<Integer, Integer> event) {
          event.getKey(); // key
          event.getValue() // value
          // ...
     }
});

int expireListener = map.addListener(new EntryExpiredListener<Integer, Integer>() {
     @Override
     public void onExpired(EntryEvent<Integer, Integer> event) {
          event.getKey(); // key
          event.getValue() // value
          // ...
     }
});

int removeListener = map.addListener(new EntryRemovedListener<Integer, Integer>() {
     @Override
     public void onRemoved(EntryEvent<Integer, Integer> event) {
          event.getKey(); // key
          event.getValue() // value
          // ...
     }
});

map.removeListener(updateListener);
map.removeListener(createListener);
map.removeListener(expireListener);
map.removeListener(removeListener);
```
### LRU/LFU 有界 Map
实现 `RMapCache` 接口的 Map 对象可以使用[最近最少使用 (LRU) ](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU)或 [最少经常使用 (LFU) ](https://en.wikipedia.org/wiki/Least_frequently_used)顺序进行限制。有界映射允许在定义的限制内存储映射 entry 并按定义的顺序删除 entry。 
用例：Redis 内存有限。
```java
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap", MapCacheOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getLocalCachedMapCache("anyMap", LocalCachedMapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredLocalCachedMapCache("anyMap", LocalCachedMapOptions.defaults());
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap");
// or
RMapCache<String, SomeObject> map = redisson.getClusteredMapCache("anyMap", MapCacheOptions.defaults());


// tries to set limit map to 10 entries using LRU eviction algorithm
map.trySetMaxSize(10);
// ... using LFU eviction algorithm
map.trySetMaxSize(10, EvictionMode.LFU);

// set or change limit map to 10 entries using LRU eviction algorithm
map.setMaxSize(10);
// ... using LFU eviction algorithm
map.setMaxSize(10, EvictionMode.LFU);

map.put("1", "2");
map.put("3", "3", 1, TimeUnit.SECONDS);
```
## Multimap 多值Map
基于 Redis 的 Java `Multimap` 允许为每个键绑定多个值。该对象是完全线程安全的。 Redis 将键数量限制为 4 294 967 295 个元素。 Redis 使用序列化状态来检查键的唯一性，而不是键的 hashCode()/equals() 方法。 它具有 Async、Reactive 和 RxJava3 接口。
### Set based 多值Map
基于集合的 Multimap 不允许每个键的值重复。
```java
RSetMultimap<SimpleKey, SimpleValue> map = redisson.getSetMultimap("myMultimap");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

Set<SimpleValue> allValues = map.get(new SimpleKey("0"));

List<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
Set<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

Set<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```
### List based 多值Map
Java 的基于列表的 Multimap 对象按插入顺序存储条目，并允许映射到键的值重复。
```java
RListMultimap<SimpleKey, SimpleValue> map = redisson.getListMultimap("test1");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

List<SimpleValue> allValues = map.get(new SimpleKey("0"));

Collection<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
List<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

List<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```
### 多值Map的数据淘汰
Java 的 Multimap 对象，具有通过分离的 MultimapCache 对象实现的逐出支持。分别有用于设置和列表多重映射的 `RSetMultimapCache` 和 `RListMultimapCache` 对象。 
过期的条目由 `org.redisson.EvictionScheduler` 清理。它一次删除 100 个过期条目。任务启动时间自动调整，取决于上次删除的过期条目数量，变化范围为 1 秒到 2 小时。因此，如果 clean 任务每次删除 100 个条目，那么它将每秒执行一次（最小执行延迟）。但如果当前过期条目数量低于之前的条目数量，则执行延迟将增加 1.5 倍。 
RSetMultimapCache 示例：
```java
RSetMultimapCache<String, String> multimap = redisson.getSetMultimapCache("myMultimap");
multimap.put("1", "a");
multimap.put("1", "b");
multimap.put("1", "c");

multimap.put("2", "e");
multimap.put("2", "f");

multimap.expireKey("2", 10, TimeUnit.MINUTES);
```
## Set
Java 中基于 Redis 的 Set 对象实现了 `java.util.Set` 接口。该对象是完全线程安全的。通过元素状态比较保持元素的唯一性。 Redis 将大小限制为 `4 294 967 295` 个元素。 Redis 使用序列化状态来检查值的唯一性，而不是值的 `hashCode()`/`equals()` 方法。 它具有 `Async`、`Reactive` 和 `RxJava3` 接口。
```java
RSet<SomeObject> set = redisson.getSet("anySet");
set.add(new SomeObject());
set.remove(new SomeObject());
```
RSet 对象允许为每个值绑定一个 `Lock/ReadWriteLock/Semaphore/CountDownLatch` 对象：
```java
RSet<MyObject> set = redisson.getSet("anySet");
MyObject value = new MyObject();
RLock lock = map.getLock(value);
lock.lock();
try {
   // process value ...
} finally {
   lock.unlock();
}
```
### Set 数据淘汰和数据分区
Redisson 提供了各种 Set 结构实现，具有两个重要特性：
`数据分区` - 尽管 Set 对象与集群兼容，但其内容不会跨集群中的多个 Redis 主节点进行缩放/分区。数据分区允许扩展 Redis 集群中单个 Set 实例的可用内存、读/写操作和条目驱逐过程。 
`条目驱逐` - 允许定义每个设定值的生存时间。 Redis 设置结构不支持驱逐，因此它是在 Redisson 端通过自定义计划任务完成的，该任务会删除过期的条目。逐出任务由每个唯一对象名称的 getSetCache() 方法执行启动一次。如果对象实例未使用且具有过期元素，则应通过 getSetCache() 方法获取该对象实例以启动驱逐过程。这会导致每个唯一设置的对象名称需要额外的 Redis 调用和驱逐任务。 
`高级条目驱逐` - 条目驱逐过程的改进版本。不使用条目驱逐任务。 下面是所有可用的 Map 实现的列表：

| RedissonClient method name | Data partitioning | Entry eviction | Advanced entry eviction | Ultra-fast read/write |
| --- | --- | --- | --- | --- |
| getSet() _open-source version_ | ❌ | ❌ | ❌ | ❌ |
| getSetCache() _open-source version_ | ❌ | ✔️ | ❌ | ❌ |
| getSet() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ❌ | ❌ | ✔️ |
| getSetCache() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ✔️ | ❌ | ✔️ |
| getSetCacheV2() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ❌ | ✔️ | ✔️ |
| getClusteredSet() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ❌ | ❌ | ✔️ |
| getClusteredSetCache() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ | ❌ | ✔️ |

#### Eviction 数据淘汰 (驱逐)
具有驱逐支持的 Map 对象实现了 `org.redisson.api.RSetCache` ，它扩展了 `java.util.Set` 接口。它具有 Async、Reactive 和 RxJava3 接口。 
当前的 Redis 实现没有设置值驱逐功能。因此过期的条目会被 `org.redisson.eviction.EvictionScheduler` 清理。它一次删除 300 个过期条目。任务启动时间自动调整，取决于上次删除的过期条目数量，变化范围为 1 秒到 1 小时。因此，如果 clean 任务每次删除 300 个条目，那么它将每秒执行一次（最小执行延迟）。但如果当前过期值数量低于前一个，则执行延迟将增加 1.5 倍。 代码示例：
```java
RSetCache<SomeObject> set = redisson.getSetCache("mySet");
// or
RMapCache<SomeObject> set = redisson.getClusteredSetCache("mySet");

// ttl = 10 minutes, 
set.add(new SomeObject(), 10, TimeUnit.MINUTES);

// if object is not used anymore
map.destroy();
```
#### 数据分区
具有数据分区支持的 Map 对象实现了`org.redisson.api.RClusteredSet`，它扩展了`java.util.Set`接口。在[此处](#b2Cfd)阅读有关数据分区的更多详细信息。 
代码示例：
```java
RClusteredSet<SomeObject> set = redisson.getClusteredSet("mySet");
// or
RClusteredSet<SomeObject> set = redisson.getClusteredSetCache("mySet");

// ttl = 10 minutes, 
map.add(new SomeObject(), 10, TimeUnit.MINUTES);
```
## SortedSet
基于 Redis 的 Java 分布式 `SortedSet` 实现了 `java.util.SortedSet` 接口。该对象是完全线程安全的。它使用比较器对元素进行排序并保持唯一性。对于 String 数据类型，由于性能增益，建议使用 `LexSortedSet` 对象。
```java
RSortedSet<Integer> set = redisson.getSortedSet("anySet");
set.trySetComparator(new MyComparator()); // set object comparator
set.add(3);
set.add(1);
set.add(2);

set.removeAsync(0);
set.addAsync(5);
```
## ScoredSortedSet
基于 Redis 的分布式 `ScoredSortedSet` 对象。按元素插入期间定义的分数对元素进行排序。通过元素状态比较保持元素的唯一性。 它具有 Async、Reactive 和 RxJava3 接口。 Redis 将集合大小限制为 `4 294 967 295` 个元素。
```java
RScoredSortedSet<SomeObject> set = redisson.getScoredSortedSet("simple");

set.add(0.13, new SomeObject(a, b));
set.addAsync(0.251, new SomeObject(c, d));
set.add(0.302, new SomeObject(g, d));

set.pollFirst();
set.pollLast();

int index = set.rank(new SomeObject(g, d)); // get element index
Double score = set.getScore(new SomeObject(g, d)); // get element score
```
### ScoredSortedSet 数据分区
尽管“RScoredSortedSet”对象与集群兼容，但其内容无法跨多个 Redis 主节点进行扩展。 
`RScoredSortedSet` 数据分区仅在集群模式下可用，并由单独的 `RClusteredScoredSortedSet` 对象实现。大小受整个 Redis 集群内存的限制。有关分区的更多信息请参见[此处](#b2Cfd)。 下面是所有可用的 `RScoredSortedSet` 实现的列表：

| RedissonClient method name | Data partitioning support | Ultra-fast read/write |
| --- | --- | --- |
| getScoredSortedSet() _open-source version_ | ❌ | ❌ |
| getScoredSortedSet() [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ✔️ |
| getClusteredScoredSortedSet() _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ |

代码示例：
```java
RClusteredScoredSortedSet set = redisson.getClusteredScoredSortedSet("simpleBitset");
set.add(1.1, "v1");
set.add(1.2, "v2");
set.add(1.3, "v3");

ScoredEntry<String> s = set.firstEntry();
ScoredEntry<String> e = set.pollFirstEntry();
```
## LexSortedSet
Java 中基于 Redis 的分布式 Set 对象仅允许 String 对象并实现` java.util.Set<String>` 接口。它保持元素按字典顺序排列，并通过元素状态比较保持元素的唯一性。 它具有 Async、Reactive 和 RxJava3 接口。
```java
RLexSortedSet set = redisson.getLexSortedSet("simple");
set.add("d");
set.addAsync("e");
set.add("f");

set.lexRangeTail("d", false);
set.lexCountHead("e");
set.lexRange("d", true, "z", false);
```
## List
Java 中基于 Redis 的分布式 `List` 对象实现了 `java.util.List` 接口。它保持元素的插入顺序。 
它具有 Async、Reactive 和 RxJava3 接口。 Redis 将列表大小限制为 `4 294 967 295` 个元素。
```java
RList<SomeObject> list = redisson.getList("anyList");
list.add(new SomeObject());
list.get(0);
list.remove(new SomeObject());
```
## Queue
Java 中基于 Redis 的分布式无界队列对象实现了` java.util.Queue` 接口。该对象是完全线程安全的。 
它具有 Async、Reactive 和 RxJava3 接口。
```java
RQueue<SomeObject> queue = redisson.getQueue("anyQueue");
queue.add(new SomeObject());
SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
```
## Deque
Java 中基于 Redis 的分布式无界 Deque 对象实现了 `java.util.Deque` 接口。该对象是完全线程安全的。 
它具有 Async、Reactive 和 RxJava3 接口。
```java
RDeque<SomeObject> queue = redisson.getDeque("anyDeque");
queue.addFirst(new SomeObject());
queue.addLast(new SomeObject());
SomeObject obj = queue.removeFirst();
SomeObject someObj = queue.removeLast();
```
## Blocking Queue
Java 的基于 Redis 的分布式无界 `BlockingQueue `对象实现了 `java.util.concurrent.BlockingQueue` 接口。该对象是完全线程安全的。 
它具有 Async、Reactive 和 RxJava3 接口。
```java
RBlockingQueue<SomeObject> queue = redisson.getBlockingQueue("anyQueue");

queue.offer(new SomeObject());

SomeObject obj = queue.peek();
SomeObject obj = queue.poll();
SomeObject obj = queue.poll(10, TimeUnit.MINUTES);
```
`poll`、`pollFromAny`、`pollLastAndOfferFirstTo` 和 `take` 方法在重新连接到 Redis 服务器或故障转移期间会自动重新订阅。
## Bounded Blocking Queue 有界阻塞队列
基于 Redis 的 Java 分布式 `BoundedBlockingQueue` 实现了 `java.util.concurrent.BlockingQueue` 接口。 Redis 将 `BoundedBlockingQueue` 大小限制为 `4 294 967 295` 个元素。该对象是完全线程安全的。该对象是完全线程安全的。 
使用前应通过 `trySetCapacity()` 方法定义一次队列容量：
```java
RBoundedBlockingQueue<SomeObject> queue = redisson.getBoundedBlockingQueue("anyQueue");
// returns `true` if capacity set successfully and `false` if it already set.
queue.trySetCapacity(2);

queue.offer(new SomeObject(1));
queue.offer(new SomeObject(2));
// will be blocked until free space available in queue
queue.put(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```
`poll`、`pollFromAny`、`pollLastAndOfferFirstTo` 和 `take` 方法将在重新连接到 Redis 服务器或 Redis 服务器故障转移期间自动重新订阅。
## Blocking Deque
基于Redis的`BlockingDeque`的Java实现实现了`java.util.concurrent.BlockingDeque`接口。该对象是完全线程安全的。该对象是完全线程安全的。 它具有 Async、Reactive 和 RxJava3 接口。 
```java
RBlockingDeque<Integer> deque = redisson.getBlockingDeque("anyDeque");
deque.putFirst(1);
deque.putLast(2);
Integer firstValue = queue.takeFirst();
Integer lastValue = queue.takeLast();
Integer firstValue = queue.pollFirst(10, TimeUnit.MINUTES);
Integer lastValue = queue.pollLast(3, TimeUnit.MINUTES);
```
`poll`、`pollFromAny`、`pollLastAndOfferFirstTo` 和 `take` 方法在重新连接到 Redis 服务器或故障转移期间会自动重新订阅。
## Blocking Fair Queue 阻塞公平队列
基于Redis的分布式BlockingFairQueue for Java实现了`java.util.concurrent.BlockingQueue`接口。该对象是完全线程安全的。 
当消费者在网络的不同部分排队时：有些离redis更近，有些离redis更远。由于网络延迟，“更远”消费者将从队列中获得较少数量的消息。反过来，“更接近”的消费者将获得更高的金额，这可能会导致客户端超载。 
具有公平轮询的阻塞队列保证了 `poll` 和 `take` 方法的访问顺序，并允许获得均匀分布的消耗。 
```java
RBlockingFairQueue queue = redisson.getBlockingFairQueue("myQueue");
queue.offer(new SomeObject());

SomeObject element = queue.peek();
SomeObject element = queue.poll();
SomeObject element = queue.poll(10, TimeUnit.MINUTES);
SomeObject element = queue.take();
```
_此功能仅在 Redisson PRO 版本中可用。_
## Blocking Fair Deque
基于 Redis 的 Java 分布式 `BlockingFairDeque` 实现了 `java.util.concurrent.BlockingDeque` 接口。该对象是完全线程安全的。 
当 deque 消费者位于网络的不同部分时：有些距离 Redis 较近，有些距离较远。由于网络延迟，“更远”消费者将从队列中获得较少数量的消息。反过来，“更接近”的消费者将获得更高的金额，这可能会导致客户端超载。 具有公平轮询的阻塞双端队列保证了 `poll`、`take`、`pollFirst`、`takeFirst`、`pollLast` 和 `takeLast` 方法的访问顺序，并允许获得均匀分布的消耗。 
```java
RBlockingFairDeque deque = redisson.getBlockingFairDeque("myDeque");
deque.offer(new SomeObject());

SomeObject firstElement = queue.peekFirst();
SomeObject firstElement = queue.pollFirst();
SomeObject firstElement = queue.pollFirst(10, TimeUnit.MINUTES);
SomeObject firstElement = queue.takeFirst();

SomeObject lastElement = queue.peekLast();
SomeObject lastElement = queue.pollLast();
SomeObject lastElement = queue.pollLast(10, TimeUnit.MINUTES);
SomeObject lastElement = queue.takeLast();
```
此功能仅在 Redisson PRO 版本中可用。
## Delayed Queue
Java 中基于 Redis 的 `DelayedQueue` 对象允许以指定的延迟将每个元素传输到目标队列。目标队列可以是任何实现 RQueue 接口的队列。该对象是完全线程安全的。 
对于用于向消费者传递消息的指数退避策略可能很有用。如果应用程序重新启动，则应创建延迟队列的实例，以便将挂起的项目添加到目标队列。 
```java
RBlockingQueue<String> distinationQueue = ...
RDelayedQueue<String> delayedQueue = getDelayedQueue(distinationQueue);
// move object to distinationQueue in 10 seconds
delayedQueue.offer("msg1", 10, TimeUnit.SECONDS);
// move object to distinationQueue in 1 minutes
delayedQueue.offer("msg2", 1, TimeUnit.MINUTES);


// msg1 will appear in 10 seconds
distinationQueue.poll(15, TimeUnit.SECONDS);

// msg2 will appear in 2 seconds
distinationQueue.poll(2, TimeUnit.SECONDS);
```
如果对象不再使用，则应将其销毁，但如果 Redisson 关闭，则无需调用 destroy 方法。 
```java
RDelayedQueue<String> delayedQueue = ...
delayedQueue.destroy();
```
## Priority Queue
基于Redis的`PriorityQueue`的Java实现实现了`java.util.Queue`接口。元素按照 `java.lang.Comparable` 接口或定义的 `java.util.Comparator `的自然顺序排序。该对象是完全线程安全的。 使用 `trySetComparator()` 方法定义自己的 `java.util.Comparator`。 
代码示例：
```java
public class Entry implements Comparable<Entry>, Serializable {

    private String key;
    private Integer value;

    public Entry(String key, Integer value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public int compareTo(Entry o) {
        return key.compareTo(o.key);
    }

}

RPriorityQueue<Entry> queue = redisson.getPriorityQueue("anyQueue");
queue.add(new Entry("b", 1));
queue.add(new Entry("c", 1));
queue.add(new Entry("a", 1));

// Entry [a:1]
Entry e = queue.poll();
// Entry [b:1]
Entry e = queue.poll();
// Entry [c:1]
Entry e = queue.poll();
```
## Priority Deque
基于Redis的`PriorityDeque`的Java实现实现了`java.util.Deque`接口。元素按照 `java.lang.Comparable` 接口或定义的 `java.util.Comparator` 的自然顺序排序。该对象是完全线程安全的。 使用 `trySetComparator()` 方法定义自己的 java.util.Comparator。 
代码示例：
```java
public class Entry implements Comparable<Entry>, Serializable {

    private String key;
    private Integer value;

    public Entry(String key, Integer value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public int compareTo(Entry o) {
        return key.compareTo(o.key);
    }

}

RPriorityDeque<Entry> queue = redisson.getPriorityDeque("anyQueue");
queue.add(new Entry("b", 1));
queue.add(new Entry("c", 1));
queue.add(new Entry("a", 1));

// Entry [a:1]
Entry e = queue.pollFirst();
// Entry [c:1]
Entry e = queue.pollLast();
```
## Priority Blocking Queue
基于Redis的`PriorityBlockingQueue`的Java实现类似于JDK `java.util.concurrent.PriorityBlockingQueue`对象。元素按照 `java.lang.Comparable` 接口或定义的 `java.util.Comparator` 的自然顺序排序。该对象是完全线程安全的。 
使用 `trySetComparator()` 方法定义自己的 java.util.Comparator。
 `poll`、`pollLastAndOfferFirstTo` 和 `take` 方法在重新连接到 Redis 服务器或故障转移期间会自动重新订阅。 代码示例：
```java
public class Entry implements Comparable<Entry>, Serializable {

    private String key;
    private Integer value;

    public Entry(String key, Integer value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public int compareTo(Entry o) {
        return key.compareTo(o.key);
    }

}

RPriorityBlockingQueue<Entry> queue = redisson.getPriorityBlockingQueue("anyQueue");
queue.add(new Entry("b", 1));
queue.add(new Entry("c", 1));
queue.add(new Entry("a", 1));

// Entry [a:1]
Entry e = queue.take();
```
## Priority Blocking Deque
基于Redis的`PriorityBlockingDeque`的Java实现实现了`java.util.concurrent.BlockingDeque`接口。元素按照 `java.lang.Comparable` 接口或定义的 `java.util.Comparator` 的自然顺序排序。该对象是完全线程安全的。 
使用 `trySetComparator()` 方法定义自己的 `java.util.Comparator`。 
`poll`、`pollLastAndOfferFirstTo`、`take` 方法在重新连接到 Redis 服务器或故障转移期间会自动重新订阅。 
代码示例：
```java
public class Entry implements Comparable<Entry>, Serializable {

    private String key;
    private Integer value;

    public Entry(String key, Integer value) {
        this.key = key;
        this.value = value;
    }

    @Override
    public int compareTo(Entry o) {
        return key.compareTo(o.key);
    }

}

RPriorityBlockingDeque<Entry> queue = redisson.getPriorityBlockingDeque("anyQueue");
queue.add(new Entry("b", 1));
queue.add(new Entry("c", 1));
queue.add(new Entry("a", 1));

// Entry [a:1]
Entry e = queue.takeFirst();
// Entry [c:1]
Entry e = queue.takeLast();
```
## Stream
基于Redis的Stream对象的Java实现包装了Redis Stream功能。基本上它允许创建消费者组来消费生产者添加的数据。该对象是完全线程安全的。
```java
RStream<String, String> stream = redisson.getStream("test");

StreamMessageId sm = stream.add(StreamAddArgs.entry("0", "0"));

stream.createGroup("testGroup");

StreamId id1 = stream.add(StreamAddArgs.entry("1", "1"));
StreamId id2 = stream.add(StreamAddArgs.entry("2", "2"));

Map<StreamId, Map<String, String>> group = stream.readGroup("testGroup", "consumer1", StreamReadGroupArgs.neverDelivered());

// return entries in pending state after read group method execution
Map<StreamMessageId, Map<String, String>> pendingData = stream.pendingRange("testGroup", "consumer1", StreamMessageId.MIN, StreamMessageId.MAX, 100);

// transfer ownership of pending messages to a new consumer
List<StreamMessageId> transferedIds = stream.fastClaim("testGroup", "consumer2", 1, TimeUnit.MILLISECONDS, id1, id2);

// mark pending entries as correctly processed
long amount = stream.ack("testGroup", id1, id2);
```
Code example of [Async interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RStreamAsync.html) usage:
```java
RStream<String, String> stream = redisson.getStream("test");

RFuture<StreamMessageId> smFuture = stream.addAsync(StreamAddArgs.entry("0", "0"));

RFuture<Void> groupFuture = stream.createGroupAsync("testGroup");

RFuture<StreamId> id1Future = stream.addAsync(StreamAddArgs.entry("1", "1"));
RFuture<StreamId> id2Future = stream.addAsync(StreamAddArgs.entry("2", "2"));

RFuture<Map<StreamId, Map<String, String>>> groupResultFuture = stream.readGroupAsync("testGroup", "consumer1", StreamReadGroupArgs.neverDelivered());

// return entries in pending state after read group method execution
RFuture<Map<StreamMessageId, Map<String, String>>> pendingDataFuture = stream.pendingRangeAsync("testGroup", "consumer1", StreamMessageId.MIN, StreamMessageId.MAX, 100);

// transfer ownership of pending messages to a new consumer
RFuture<List<StreamMessageId>> transferedIdsFuture = stream.fastClaim("testGroup", "consumer2", 1, TimeUnit.MILLISECONDS, id1, id2);

// mark pending entries as correctly processed
RFuture<Long> amountFuture = stream.ackAsync("testGroup", id1, id2);

amountFuture.whenComplete((res, exception) -> {
    // ...
});
```
Code example of [Reactive interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RStreamReactive.html) usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RStreamReactive<String, String> stream = redisson.getStream("test");

Mono<StreamMessageId> smMono = stream.add(StreamAddArgs.entry("0", "0"));

Mono<Void> groupMono = stream.createGroup("testGroup");

Mono<StreamId> id1Mono = stream.add(StreamAddArgs.entry("1", "1"));
Mono<StreamId> id2Mono = stream.add(StreamAddArgs.entry("2", "2"));

Mono<Map<StreamId, Map<String, String>>> groupMono = stream.readGroup("testGroup", "consumer1", StreamReadGroupArgs.neverDelivered());

// return entries in pending state after read group method execution
Mono<Map<StreamMessageId, Map<String, String>>> pendingDataMono = stream.pendingRange("testGroup", "consumer1", StreamMessageId.MIN, StreamMessageId.MAX, 100);

// transfer ownership of pending messages to a new consumer
Mono<List<StreamMessageId>> transferedIdsMono = stream.fastClaim("testGroup", "consumer2", 1, TimeUnit.MILLISECONDS, id1, id2);

// mark pending entries as correctly processed
Mono<Long> amountMono = stream.ack("testGroup", id1, id2);

amountMono.doOnNext(res -> {
    // ...
}).subscribe();
```
Code example of [RxJava3 interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RStreamRx.html) usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RStreamRx<String, String> stream = redisson.getStream("test");

Single<StreamMessageId> smRx = stream.add(StreamAddArgs.entry("0", "0"));

Completable groupRx = stream.createGroup("testGroup");

Single<StreamId> id1Rx = stream.add(StreamAddArgs.entry("1", "1"));
Single<StreamId> id2Rx = stream.add(StreamAddArgs.entry("2", "2"));

Single<Map<StreamId, Map<String, String>>> groupRx = stream.readGroup("testGroup", "consumer1", StreamReadGroupArgs.neverDelivered());

// return entries in pending state after read group method execution
Single<Map<StreamMessageId, Map<String, String>>> pendingDataRx = stream.pendingRange("testGroup", "consumer1", StreamMessageId.MIN, StreamMessageId.MAX, 100);

// transfer ownership of pending messages to a new consumer
Single<List<StreamMessageId>> transferedIdsRx = stream.fastClaim("testGroup", "consumer2", 1, TimeUnit.MILLISECONDS, id1, id2);

// mark pending entries as correctly processed
Single<Long> amountRx = stream.ack("testGroup", id1, id2);

amountRx.doOnSuccess(res -> {
    // ...
}).subscribe();
```
## Ring Buffer
基于Redis的`RingBuffer`的Java实现实现了`java.util.Queue`接口。如果队列容量已满，此结构会从头部逐出元素。该对象是完全线程安全的。 
使用前应通过 `trySetCapacity()` 方法初始化容量大小。
代码示例：
```java
RRingBuffer<Integer> buffer = redisson.getRingBuffer("test");

// buffer capacity is 4 elements
buffer.trySetCapacity(4);

buffer.add(1);
buffer.add(2);
buffer.add(3);
buffer.add(4);

// buffer state is 1, 2, 3, 4

buffer.add(5);
buffer.add(6);

// buffer state is 3, 4, 5, 6
```
Code example of [Async interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRingBufferAsync.html) usage:
```java
RRingBuffer<Integer> buffer = redisson.getRingBuffer("test");

// buffer capacity is 4 elements
RFuture<Boolean> capacityFuture = buffer.trySetCapacityAsync(4);

RFuture<Boolean> addFuture = buffer.addAsync(1);
RFuture<Boolean> addFuture = buffer.addAsync(2);
RFuture<Boolean> addFuture = buffer.addAsync(3);
RFuture<Boolean> addFuture = buffer.addAsync(4);

// buffer state is 1, 2, 3, 4

RFuture<Boolean> addFuture = buffer.addAsync(5);
RFuture<Boolean> addFuture = buffer.addAsync(6);

// buffer state is 3, 4, 5, 6

addFuture.whenComplete((res, exception) -> {
    // ...
});
```
Code example of [Reactive interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRingBufferReactive.html) usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RRingBufferReactive<Integer> buffer = redisson.getRingBuffer("test");

// buffer capacity is 4 elements
Mono<Boolean> capacityMono = buffer.trySetCapacity(4);

Mono<Boolean> addMono = buffer.add(1);
Mono<Boolean> addMono = buffer.add(2);
Mono<Boolean> addMono = buffer.add(3);
Mono<Boolean> addMono = buffer.add(4);

// buffer state is 1, 2, 3, 4

Mono<Boolean> addMono = buffer.add(5);
Mono<Boolean> addMono = buffer.add(6);

// buffer state is 3, 4, 5, 6

addMono.doOnNext(res -> {
    // ...
}).subscribe();
```
Code example of [RxJava3 interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RRingBufferRx.html) usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RRingBufferRx<Integer> buffer = redisson.getRingBuffer("test");

// buffer capacity is 4 elements
Single<Boolean> capacityRx = buffer.trySetCapacity(4);

Single<Boolean> addRx = buffer.add(1);
Single<Boolean> addRx = buffer.add(2);
Single<Boolean> addRx = buffer.add(3);
Single<Boolean> addRx = buffer.add(4);

// buffer state is 1, 2, 3, 4

Single<Boolean> addRx = buffer.add(5);
Single<Boolean> addRx = buffer.add(6);

// buffer state is 3, 4, 5, 6

addRx.doOnSuccess(res -> {
    // ...
}).subscribe();
```
## Transfer Queue
基于Redis的`TransferQueue`的Java实现实现了`java.util.concurrent.TransferQueue`接口。提供一组传输方法，仅当消息成功传递给消费者时才返回。该对象是完全线程安全的。 
`poll` 和 `take` 方法在重新连接到 Redis 服务器或故障转移期间自动重新订阅。 
代码示例：
```java
RTransferQueue<String> queue = redisson.getTransferQueue("myCountDownLatch");

queue.transfer("data");
// or try transfer immediately
queue.tryTransfer("data");
// or try transfer up to 10 seconds
queue.tryTransfer("data", 10, TimeUnit.SECONDS);

// in other thread or JVM

queue.take();
// or
queue.poll();
```
Code example of [Async interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTransferQueueAsync.html) usage:
```java
RTransferQueue<String> queue = redisson.getTransferQueue("myCountDownLatch");

RFuture<Void> future = queue.transferAsync("data");
// or try transfer immediately
RFuture<Boolean> future = queue.tryTransferAsync("data");
// or try transfer up to 10 seconds
RFuture<Boolean> future = queue.tryTransferAsync("data", 10, TimeUnit.SECONDS);

// in other thread or JVM

RFuture<String> future = queue.takeAsync();
// or
RFuture<String> future = queue.pollAsync();

future.whenComplete((res, exception) -> {
    // ...
});
```
Code example of [Reactive interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTransferQueueReactive.html) usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RTransferQueueReactive<String> queue = redisson.getTransferQueue("myCountDownLatch");

Mono<Void> mono = queue.transfer("data");
// or try transfer immediately
Mono<Boolean> mono = queue.tryTransfer("data");
// or try transfer up to 10 seconds
Mono<Boolean> mono = queue.tryTransfer("data", 10, TimeUnit.SECONDS);

// in other thread or JVM

Mono<String> mono = queue.take();
// or
Mono<String> mono = queue.poll();

mono.doOnNext(res -> {
    // ...
}).subscribe();
```
Code example of [RxJava3 interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RTransferQueueRx.html) usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RTransferQueueRx<String> queue = redisson.getTransferQueue("myCountDownLatch");

Completable res = queue.transfer("data");
// or try transfer immediately
Single<Boolean> resRx = queue.tryTransfer("data");
// or try transfer up to 10 seconds
Single<Boolean> resRx = queue.tryTransfer("data", 10, TimeUnit.SECONDS);

// in other thread or JVM

Single<String> resRx = queue.take();
// or
Maybe<String> resRx = queue.poll();

resRx.doOnSuccess(res -> {
    // ...
}).subscribe();
```
## Time Series
基于 Redis 的 `TimeSeries` 对象的 Java 实现允许按时间戳存储值并定义每个条目的 TTL（生存时间）。值按时间戳排序。该对象是完全线程安全的。 
代码示例：
```java
RTimeSeries<String> ts = redisson.getTimeSeries("myTimeSeries");

ts.add(201908110501, "10%");
ts.add(201908110502, "30%");
ts.add(201908110504, "10%");
ts.add(201908110508, "75%");

// entry time-to-live is 10 hours
ts.add(201908110510, "85%", 10, TimeUnit.HOURS);
ts.add(201908110510, "95%", 10, TimeUnit.HOURS);

String value = ts.get(201908110508);
ts.remove(201908110508);

Collection<String> values = ts.pollFirst(2);
Collection<String> range = ts.range(201908110501, 201908110508);
```
Code example of [Async interface](https://www.javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RTimeSeriesAsync.html) usage:
```java
RTimeSeries<String> ts = redisson.getTimeSeries("myTimeSeries");

RFuture<Void> future = ts.addAsync(201908110501, "10%");
RFuture<Void> future = ts.addAsync(201908110502, "30%");
RFuture<Void> future = ts.addAsync(201908110504, "10%");
RFuture<Void> future = ts.addAsync(201908110508, "75%");

// entry time-to-live is 10 hours
RFuture<Void> future = ts.addAsync(201908110510, "85%", 10, TimeUnit.HOURS);
RFuture<Void> future = ts.addAsync(201908110510, "95%", 10, TimeUnit.HOURS);

RFuture<String> future = ts.getAsync(201908110508);
RFuture<Boolean> future = ts.removeAsync(201908110508);

RFuture<Collection<String>> future = t.pollFirstAsync(2);
RFuture<Collection<String>> future = t.rangeAsync(201908110501, 201908110508);

future.whenComplete((res, exception) -> {
    // ...
});
```
Code example of [Reactive interface](https://www.javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RTimeSeriesReactive.html) usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RTimeSeriesReactive<String> ts = redisson.getTimeSeries("myTimeSeries");

Mono<Void> mono = ts.add(201908110501, "10%");
Mono<Void> mono = ts.add(201908110502, "30%");
Mono<Void> mono = ts.add(201908110504, "10%");
Mono<Void> mono = ts.add(201908110508, "75%");

// entry time-to-live is 10 hours
Mono<Void> mono = ts.add(201908110510, "85%", 10, TimeUnit.HOURS);
Mono<Void> mono = ts.add(201908110510, "95%", 10, TimeUnit.HOURS);

Mono<String> mono = ts.get(201908110508);
Mono<Boolean> mono = ts.remove(201908110508);

Mono<Collection<String>> mono = ts.pollFirst(2);
Mono<Collection<String>> mono = ts.range(201908110501, 201908110508);

mono.doOnNext(res -> {
    // ...
}).subscribe();
```
Code example of [RxJava3 interface](https://www.javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RTimeSeriesRx.html) usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RTimeSeriesRx<String> ts = redisson.getTimeSeries("myTimeSeries");

Completable rx = ts.add(201908110501, "10%");
Completable rx = ts.add(201908110502, "30%");
Completable rx = ts.add(201908110504, "10%");
Completable rx = ts.add(201908110508, "75%");

// entry time-to-live is 10 hours
Completable rx = ts.add(201908110510, "85%", 10, TimeUnit.HOURS);
Completable rx = ts.add(201908110510, "95%", 10, TimeUnit.HOURS);

Maybe<String> rx = ts.get(201908110508);
Single<Boolean> rx = ts.remove(201908110508);

Single<Collection<String>> rx = ts.pollFirst(2);
Single<Collection<String>> rx = ts.range(201908110501, 201908110508);

rx.doOnSuccess(res -> {
    // ...
}).subscribe();
```
# Distributed locks and synchronizers 分布式锁和同步器
## Lock
基于 Redis 的 Java 分布式可重入 Lock 对象并实现 `Lock` 接口。 
如果获取锁的 Redisson 实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
可以定义获取锁时的`leaseTime`参数。在指定的时间间隔后锁定将自动释放。 
RLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore` 对象。 
代码示例：
```java
RLock lock = redisson.getLock("myLock");

// traditional lock method
lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RLock lock = redisson.getLock("myLock");

RFuture<Void> lockFuture = lock.lockAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Void> lockFuture = lock.lockAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Boolean> lockFuture = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);

lockFuture.whenComplete((res, exception) -> {

    // ...

    lock.unlockAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RLockReactive lock = redisson.getLock("myLock");

Mono<Void> lockMono = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Void> lockMono = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Boolean> lockMono = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockMono.doOnNext(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RLockRx lock = redisson.getLock("myLock");

Completable lockRes = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Completable lockRes = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Boolean> lockRes = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockRes.doOnSuccess(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
## Fair Lock
Java 中基于 Redis 的分布式可重入公平 Lock 对象实现了 `Lock` 接口。 
公平锁保证线程将按照它们请求的顺序获取它。所有等待线程都会排队，如果某个线程已死亡，则 Redisson 会等待其返回 5 秒。例如，如果 5 个线程由于某种原因死亡，则延迟将为 25 秒。 
如果获取锁的 Redisson 实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
可以定义获取锁时的`leaseTime`参数。在指定的时间间隔后锁定将自动释放。 
RLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore `对象。 
代码示例：
```java
RLock lock = redisson.getFairLock("myLock");

// traditional lock method
lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RLock lock = redisson.getFairLock("myLock");

RFuture<Void> lockFuture = lock.lockAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Void> lockFuture = lock.lockAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Boolean> lockFuture = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);

lockFuture.whenComplete((res, exception) -> {
    // ...
    lock.unlockAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RLockReactive lock = redisson.getFairLock("myLock");

Mono<Void> lockMono = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Void> lockMono = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Boolean> lockMono = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockMono.doOnNext(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RLockRx lock = redisson.getFairLock("myLock");

Completable lockRes = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Completable lockRes = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Boolean> lockRes = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockRes.doOnSuccess(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
## MultiLock
基于 Redis 的分布式 `MultiLock` 对象允许对 Lock 对象进行分组并将它们作为单个锁进行处理。每个`RLock`对象可能属于不同的Redisson实例。 
如果获取 MultiLock 的 Redisson 实例崩溃，则此类 MultiLock 可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
可以定义获取锁时的`leaseTime`参数。在指定的时间间隔后锁定将自动释放。 
MultiLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore` 对象。 
代码示例：
```java
RLock lock1 = redisson1.getLock("lock1");
RLock lock2 = redisson2.getLock("lock2");
RLock lock3 = redisson3.getLock("lock3");

RLock multiLock = anyRedisson.getMultiLock(lock1, lock2, lock3);

// traditional lock method
multiLock.lock();

// or acquire lock and automatically unlock it after 10 seconds
multiLock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = multiLock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        multiLock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RLock lock1 = redisson1.getLock("lock1");
RLock lock2 = redisson2.getLock("lock2");
RLock lock3 = redisson3.getLock("lock3");

RLock multiLock = anyRedisson.getMultiLock(lock1, lock2, lock3);

RFuture<Void> lockFuture = multiLock.lockAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Void> lockFuture = multiLock.lockAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Boolean> lockFuture = multiLock.tryLockAsync(100, 10, TimeUnit.SECONDS);

lockFuture.whenComplete((res, exception) -> {
    // ...
    multiLock.unlockAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient anyRedisson = redissonClient.reactive();

RLockReactive lock1 = redisson1.getLock("lock1");
RLockReactive lock2 = redisson2.getLock("lock2");
RLockReactive lock3 = redisson3.getLock("lock3");

RLockReactive multiLock = anyRedisson.getMultiLock(lock1, lock2, lock3);

Mono<Void> lockMono = multiLock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Void> lockMono = multiLock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Boolean> lockMono = multiLock.tryLock(100, 10, TimeUnit.SECONDS);

lockMono.doOnNext(res -> {
    // ...
})
.doFinally(multiLock.unlock())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient anyRedisson = redissonClient.rxJava();

RLockRx lock1 = redisson1.getLock("lock1");
RLockRx lock2 = redisson2.getLock("lock2");
RLockRx lock3 = redisson3.getLock("lock3");

RLockRx multiLock = anyRedisson.getMultiLock(lock1, lock2, lock3);

Completable lockRes = multiLock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Completable lockRes = multiLock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Boolean> lockRes = multiLock.tryLock(100, 10, TimeUnit.SECONDS);

lockRes.doOnSuccess(res -> {
    // ...
})
.doFinally(multiLock.unlock())
.subscribe();
```
## RedLock
该对象已被弃用。请改用 `RLock` 或 `RFencedLock`。
## ReadWriteLock
Java 中基于 Redis 的分布式可重入 `ReadWriteLock` 对象实现了 `ReadWriteLock` 接口。读锁和写锁都实现了 RLock 接口。
允许多个 ReadLock 所有者，并且只允许一个 WriteLock 所有者。 
如果获取锁的 Redisson 实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
Redisson还允许在锁获取期间指定`leaseTime`参数。在指定的时间间隔后锁定将自动释放。 
RLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore` 对象。 
代码示例：
```java
RReadWriteLock rwlock = redisson.getReadWriteLock("myLock");

RLock lock = rwlock.readLock();
// or
RLock lock = rwlock.writeLock();

// traditional lock method
lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RReadWriteLock rwlock = redisson.getReadWriteLock("myLock");

RLock lock = rwlock.readLock();
// or
RLock lock = rwlock.writeLock();

RFuture<Void> lockFuture = lock.lockAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Void> lockFuture = lock.lockAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Boolean> lockFuture = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);

lockFuture.whenComplete((res, exception) -> {

    // ...

    lock.unlockAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();

RReadWriteLockReactive rwlock = redisson.getReadWriteLock("myLock");

RLockReactive lock = rwlock.readLock();
// or
RLockReactive lock = rwlock.writeLock();

Mono<Void> lockMono = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Void> lockMono = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Boolean> lockMono = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockMono.doOnNext(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();

RReadWriteLockRx rwlock = redisson.getReadWriteLock("myLock");

RLockRx lock = rwlock.readLock();
// or
RLockRx lock = rwlock.writeLock();

Completable lockRes = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Completable lockRes = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Boolean> lockRes = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockRes.doOnSuccess(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
## Semaphore
Java 中基于 Redis 的分布式 `Semaphore` 对象与 `Semaphore` 对象类似。 
可以在使用前初始化，但这不是必需的，通过 `trySetPermits(permits)` 方法获得可用的许可数量。 
代码示例：
```java
RSemaphore semaphore = redisson.getSemaphore("mySemaphore");

// acquire single permit
semaphore.acquire();

// or acquire 10 permits
semaphore.acquire(10);

// or try to acquire permit
boolean res = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
boolean res = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire 10 permit
boolean res = semaphore.tryAcquire(10);

// or try to acquire 10 permits or wait up to 15 seconds
boolean res = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        semaphore.release();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RSemaphoreAsync.html)** interface** usage:
```java
RSemaphore semaphore = redisson.getSemaphore("mySemaphore");

// acquire single permit
RFuture<Void> acquireFuture = semaphore.acquireAsync();

// or acquire 10 permits
RFuture<Void> acquireFuture = semaphore.acquireAsync(10);

// or try to acquire permit
RFuture<Boolean> acquireFuture = semaphore.tryAcquireAsync();

// or try to acquire permit or wait up to 15 seconds
RFuture<Boolean> acquireFuture = semaphore.tryAcquireAsync(15, TimeUnit.SECONDS);

// or try to acquire 10 permit
RFuture<Boolean> acquireFuture = semaphore.tryAcquireAsync(10);

// or try to acquire 10 permits or wait up to 15 seconds
RFuture<Boolean> acquireFuture = semaphore.tryAcquireAsync(10, 15, TimeUnit.SECONDS);

acquireFuture.whenComplete((res, exception) -> {
    // ...
    semaphore.releaseAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RSemaphoreReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();

RSemaphoreReactive semaphore = redisson.getSemaphore("mySemaphore");

// acquire single permit
Mono<Void> acquireMono = semaphore.acquire();

// or acquire 10 permits
Mono<Void> acquireMono = semaphore.acquire(10);

// or try to acquire permit
Mono<Boolean> acquireMono = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
Mono<Boolean> acquireMono = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire 10 permit
Mono<Boolean> acquireMono = semaphore.tryAcquire(10);

// or try to acquire 10 permits or wait up to 15 seconds
Mono<Boolean> acquireMono = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);

acquireMono.doOnNext(res -> {
    // ...
})
.doFinally(semaphore.release())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RSemaphoreRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();

RSemaphoreRx semaphore = redisson.getSemaphore("mySemaphore");

// acquire single permit
Completable acquireRx = semaphore.acquire();

// or acquire 10 permits
Completable acquireRx = semaphore.acquire(10);

// or try to acquire permit
Single<Boolean> acquireRx = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
Single<Boolean> acquireRx = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire 10 permit
Single<Boolean> acquireRx = semaphore.tryAcquire(10);

// or try to acquire 10 permits or wait up to 15 seconds
Single<Boolean> acquireRx = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);

acquireRx.doOnSuccess(res -> {
    // ...
})
.doFinally(semaphore.release())
.subscribe();
```
## PermitExpirableSemaphore
基于 Redis 的 Java 分布式信号量对象，为每个获取的许可提供租用时间参数支持。每个许可证都由自己的 ID 标识，并且只能使用其 ID 来释放。 
应在使用前通过 `trySetPermits(permits)` 方法初始化可用的许可量。允许通过 `addPermits(permits)` 方法增加/减少可用许可证的数量。 
代码示例：
```java
RPermitExpirableSemaphore semaphore = redisson.getPermitExpirableSemaphore("mySemaphore");

semaphore.trySetPermits(23);

// acquire permit
String id = semaphore.acquire();

// or acquire permit with lease time in 10 seconds
String id = semaphore.acquire(10, TimeUnit.SECONDS);

// or try to acquire permit
String id = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
String id = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire permit with least time 15 seconds or wait up to 10 seconds
String id = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);
if (id != null) {
    try {
        ...
    } finally {
        semaphore.release(id);
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPermitExpirableSemaphoreAsync.html)** interface** usage:
```java
RPermitExpirableSemaphore semaphore = redisson.getPermitExpirableSemaphore("mySemaphore");

RFuture<Boolean> setFuture = semaphore.trySetPermitsAsync(23);

// acquire permit
RFuture<String> acquireFuture = semaphore.acquireAsync();

// or acquire permit with lease time in 10 seconds
RFuture<String> acquireFuture = semaphore.acquireAsync(10, TimeUnit.SECONDS);

// or try to acquire permit
RFuture<String> acquireFuture = semaphore.tryAcquireAsync();

// or try to acquire permit or wait up to 15 seconds
RFuture<String> acquireFuture = semaphore.tryAcquireAsync(15, TimeUnit.SECONDS);

// or try to acquire permit with least time 15 seconds or wait up to 10 seconds
RFuture<String> acquireFuture = semaphore.tryAcquireAsync(10, 15, TimeUnit.SECONDS);
acquireFuture.whenComplete((id, exception) -> {
    // ...
    semaphore.releaseAsync(id);
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPermitExpirableSemaphoreReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();

RPermitExpirableSemaphoreReactive semaphore = redisson.getPermitExpirableSemaphore("mySemaphore");

Mono<Boolean> setMono = semaphore.trySetPermits(23);

// acquire permit
Mono<String> acquireMono = semaphore.acquire();

// or acquire permit with lease time in 10 seconds
Mono<String> acquireMono = semaphore.acquire(10, TimeUnit.SECONDS);

// or try to acquire permit
Mono<String> acquireMono = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
Mono<String> acquireMono = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire permit with least time 15 seconds or wait up to 10 seconds
Mono<String> acquireMono = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);

acquireMono.flatMap(id -> {
    // ...
    return semaphore.release(id);
}).subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RPermitExpirableSemaphoreRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();

RPermitExpirableSemaphoreRx semaphore = redisson.getPermitExpirableSemaphore("mySemaphore");

Single<Boolean> setRx = semaphore.trySetPermits(23);

// acquire permit
Single<String> acquireRx = semaphore.acquire();

// or acquire permit with lease time in 10 seconds
Single<String> acquireRx = semaphore.acquire(10, TimeUnit.SECONDS);

// or try to acquire permit
Maybe<String> acquireRx = semaphore.tryAcquire();

// or try to acquire permit or wait up to 15 seconds
Maybe<String> acquireRx = semaphore.tryAcquire(15, TimeUnit.SECONDS);

// or try to acquire permit with least time 15 seconds or wait up to 10 seconds
Maybe<String> acquireRx = semaphore.tryAcquire(10, 15, TimeUnit.SECONDS);

acquireRx.flatMap(id -> {
    // ...
    return semaphore.release(id);
}).subscribe();
```
## CountDownLatch
Java 中基于 Redis 的分布式 CountDownLatch 对象具有与 `CountDownLatch` 对象类似的结构。 
使用前应通过 `trySetCount(count)` 方法初始化计数。 
代码示例：
```java
RCountDownLatch latch = redisson.getCountDownLatch("myCountDownLatch");

latch.trySetCount(1);
// await for count down
latch.await();

// in other thread or JVM
RCountDownLatch latch = redisson.getCountDownLatch("myCountDownLatch");
latch.countDown();
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RCountDownLatchAsync.html)** interface** usage:
```java
RCountDownLatch latch = redisson.getCountDownLatch("myCountDownLatch");

RFuture<Boolean> setFuture = lock.trySetCountAsync(1);
// await for count down
RFuture<Void> awaitFuture = latch.awaitAsync();

// in other thread or JVM
RCountDownLatch latch = redisson.getCountDownLatch("myCountDownLatch");
RFuture<Void> countFuture = latch.countDownAsync();
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RCountDownLatchReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RCountDownLatchReactive latch = redisson.getCountDownLatch("myCountDownLatch");

Mono<Boolean> setMono = latch.trySetCount(1);
// await for count down
Mono<Void> awaitMono = latch.await();

// in other thread or JVM
RCountDownLatchReactive latch = redisson.getCountDownLatch("myCountDownLatch");
Mono<Void> countMono = latch.countDown();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RCountDownLatchRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RCountDownLatchRx latch = redisson.getCountDownLatch("myCountDownLatch");

Single<Boolean> setRx = latch.trySetCount(1);
// await for count down
Completable awaitRx = latch.await();

// in other thread or JVM
RCountDownLatchRx latch = redisson.getCountDownLatch("myCountDownLatch");
Completable countRx = latch.countDown();
```
## Spin Lock
基于 Redis 的 Java 分布式可重入 SpinLock 对象并实现 `Lock` 接口。 
由于 Lock 对象中的 pubsub 使用，每短时间间隔获取/释放数千个或更多的锁可能会导致达到网络吞吐量限制和 Redis CPU 过载。发生这种情况是由于 Redis pubsub 的性质 - 消息被分发到 Redis 集群中的所有节点。 Spin Lock 默认使用 `Exponential Backoff` 策略来获取锁，而不是 pubsub 通道。 
如果获取锁的 Redisson 实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
可以定义获取锁时的`leaseTime`参数。在指定的时间间隔后锁定将自动释放。 
RLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore` 对象。 
代码示例：
```java
RLock lock = redisson.getSpinLock("myLock");

// traditional lock method
lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RLock lock = redisson.getSpinLock("myLock");

RFuture<Void> lockFuture = lock.lockAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Void> lockFuture = lock.lockAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Boolean> lockFuture = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);

lockFuture.whenComplete((res, exception) -> {

    // ...

    lock.unlockAsync();
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RLockReactive lock = redisson.getSpinLock("myLock");

Mono<Void> lockMono = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Void> lockMono = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Boolean> lockMono = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockMono.doOnNext(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RLockRx lock = redisson.getSpinLock("myLock");

Completable lockRes = lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
Completable lockRes = lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Boolean> lockRes = lock.tryLock(100, 10, TimeUnit.SECONDS);

lockRes.doOnSuccess(res -> {
    // ...
})
.doFinally(lock.unlock())
.subscribe();
```
## Fenced Lock 
Java 中基于 Redis 的分布式可重入 FencedLock 对象并实现了 Lock 接口。 
这种类型的锁会维护 fencing token，以避免 Client 获取锁由于长时间 GC 暂停或其他原因而延迟并且无法检测到它不再拥有锁的情况。要解决此问题，可以通过锁定方法或 getToken() 方法返回令牌。应由此锁保护的服务检查令牌是否大于或等于前一个令牌，如果条件为假，则拒绝操作。 
如果获取锁的 Redisson 实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson 维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为 30 秒，可以通过 `Config.lockWatchdogTimeout` 设置进行更改。 
可以定义获取锁时的leaseTime参数。在指定的时间间隔后锁定将自动释放。 
RLock 对象的行为符合 Java Lock 规范。这意味着只有锁所有者线程才能解锁它，否则将抛出 `IllegalMonitorStateException`。否则考虑使用 `RSemaphore` 对象。 
代码示例：
```java
RFencedLock lock = redisson.getFencedLock("myLock");

// traditional lock method
Long token = lock.lockAndGetToken();

// or acquire lock and automatically unlock it after 10 seconds
token = lock.lockAndGetToken(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Long token = lock.tryLockAndGetToken(100, 10, TimeUnit.SECONDS);
if (token != null) {
    try {
        // check if token >= old token
        ...
    } finally {
        lock.unlock();
    }
}
```
Code example of [Async](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockAsync.html)** interface** usage:
```java
RFencedLock lock = redisson.getFencedLock("myLock");

RFuture<Long> lockFuture = lock.lockAndGetTokenAsync();

// or acquire lock and automatically unlock it after 10 seconds
RFuture<Long> lockFuture = lock.lockAndGetTokenAsync(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
RFuture<Long> lockFuture = lock.tryLockAndGetTokenAsync(100, 10, TimeUnit.SECONDS);

long threadId = Thread.currentThread().getId();
lockFuture.whenComplete((token, exception) -> {
    if (token != null) {
        try {
            // check if token >= old token
            ...
        } finally {
            lock.unlockAsync(threadId);
        }
    }
});
```
Code example of [Reactive](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockReactive.html)** interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RFencedLockReactive lock = redisson.getFencedLock("myLock");

Mono<Long> lockMono = lock.lockAndGetToken();

// or acquire lock and automatically unlock it after 10 seconds
Mono<Long> lockMono = lock.lockAndGetToken(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Mono<Long> lockMono = lock.tryLockAndGetToken(100, 10, TimeUnit.SECONDS);

long threadId = Thread.currentThread().getId();
lockMono.doOnNext(token -> {
    if (token != null) {
        try {
            // check if token >= old token
            ...
        } finally {
            lock.unlock(threadId);
        }
    }
})
.subscribe();
```
Code example of [RxJava3](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/RLockRx.html)** interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RFencedLockRx lock = redisson.getFencedLock("myLock");

Single<Long> lockRes = lock.lockAndGetToken();

// or acquire lock and automatically unlock it after 10 seconds
Single<Long> lockRes = lock.lockAndGetToken(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
Single<Long> lockRes = lock.tryLock(100, 10, TimeUnit.SECONDS);

long threadId = Thread.currentThread().getId();
lockRes.doOnSuccess(token -> {
    if (token != null) {
        try {
            // check if token >= old token
            ...
        } finally {
            lock.unlock(threadId);
        }
    }
})
.subscribe();
```
# Distributed services 分布式服务
## Remote service 远程服务
Redisson 提供 Java 远程服务来使用 Redis 执行远程过程调用。远程接口可以有任何类型的方法参数和结果对象。 Redis用于存储方法请求和相应的执行结果。 
RemoteService 提供两种类型的 `RRemoteService` 实例：

- **Server side instance** (服务器端实例) - 执行远程方法（工作实例）。例子：
```java
RRemoteService remoteService = redisson.getRemoteService();
SomeServiceImpl someServiceImpl = new SomeServiceImpl();

// register remote service before any remote invocation
// can handle only 1 invocation concurrently
remoteService.register(SomeServiceInterface.class, someServiceImpl);

// register remote service able to handle up to 12 invocations concurrently
remoteService.register(SomeServiceInterface.class, someServiceImpl, 12);
```

- **Client side instance** (客户端示例) - 调用远程方法。例子：
```java
RRemoteService remoteService = redisson.getRemoteService();
SomeServiceInterface service = remoteService.get(SomeServiceInterface.class);

String result = service.doSomeStuff(1L, "secondParam", new AnyParam());
```
客户端和服务器端实例应使用相同的远程接口，并由使用相同服务器连接配置创建的 redisson 实例支持。客户端和服务器端实例可以在同一个 JVM 中运行。客户端和/或服务器实例的数量没有限制。 （注意：虽然 redisson 不强制执行任何限制，但 Redis 的限制仍然适用。） 
如果 1 个以上工作线程可用，则远程调用将以并行模式执行。
![](https://camo.githubusercontent.com/5584cc9d3a5b155f0ea84a810774d95135d0319f54836db202ccfb1351d8a412/68747470733a2f2f7265646973736f6e2e6f72672f72656d6f7465536572766963652e706e67#from=url&id=S4uLN&originHeight=686&originWidth=690&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
并行执行器总数计算如下：`T` = `R` * `N` 
`T` - 可用并行执行器总数 `R` - Redisson 服务器端实例数量 `N` - 服务注册期间定义的执行器数量。
超过此数量的命令将排队等候下一个可用的执行器。 
如果只有 1 个工作线程可用，则远程调用将以顺序模式执行。在这种情况下，只能同时处理一个命令，其余命令将排队。
![](https://camo.githubusercontent.com/b4754fe6b03ea16bfcf7fb61dbeaf577a4a89606c566c9a82eaba83c4d896111/68747470733a2f2f7265646973736f6e2e6f72672f72656d6f746553657276696365322e706e67#from=url&id=BP8F4&originHeight=686&originWidth=256&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
### Remote service. Message flow
RemoteService 每次调用都会创建两个队列。一个队列用于请求（由服务器端实例侦听），另一个队列用于确认响应和结果响应（由客户端实例侦听）。 Ack-response 用于确定方法执行器是否收到请求。如果在 ack 超时期间没有出现，则将抛出 `RemoteServiceAckTimeoutException`。 
下面描述了每个远程调用的消息流。
![](https://camo.githubusercontent.com/998f1a31504145430354459d00de09e264cc0f8378327f9207940b5551810243/68747470733a2f2f7265646973736f6e2e6f72672f72656d6f746553657276696365332e706e67#from=url&id=BaYsW&originHeight=561&originWidth=745&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
### Remote service. Fire-and-forget and ack-response modes
每个远程调用的 RemoteService 选项可以通过 `RemoteInitationOptions` 对象定义。这些选项允许更改超时并跳过确认响应和/或结果响应。例子：
```java
// 1 second ack timeout and 30 seconds execution timeout
RemoteInvocationOptions options = RemoteInvocationOptions.defaults();

// no ack but 30 seconds execution timeout
RemoteInvocationOptions options = RemoteInvocationOptions.defaults().noAck();

// 1 second ack timeout then forget the result
RemoteInvocationOptions options = RemoteInvocationOptions.defaults().noResult();

// 1 minute ack timeout then forget about the result
RemoteInvocationOptions options = RemoteInvocationOptions.defaults().expectAckWithin(1, TimeUnit.MINUTES).noResult();

// no ack and forget about the result (fire and forget)
RemoteInvocationOptions options = RemoteInvocationOptions.defaults().noAck().noResult();

RRemoteService remoteService = redisson.getRemoteService();
YourService service = remoteService.get(YourService.class, options);
```
### Remote service. Asynchronous, Reactive and RxJava3 calls
可以使用 Async、Reactive 和 RxJava3 Api 执行远程方法。 
**异步远程接口**。接口应使用 `@RRemoteAsync` 进行注释。方法签名与远程接口中的方法匹配并返回`RFuture`对象。 
**反应式远程接口**。接口应使用`@RRemoteReactive` 进行注释。方法签名与远程接口中的方法匹配并返回`reactor.core.publisher.Mono`对象。 
**RxJava3 远程接口**。接口应使用`@RRemoteRx` 进行注释。方法签名与远程接口中的方法匹配，并返回以下对象之一：`io.reactivex.Completable`、`io.reactivex.Single`、`io.reactivex.Maybe`。 
不必列出所有方法，只需列出那些需要的方法。下面是远程服务接口的示例：
```java
public interface RemoteInterface {

    Long someMethod1(Long param1, String param2);

    void someMethod2(MyObject param);

    MyObject someMethod3();

}
```
**Asynchronous Remote interface** and method call example:
```java
@RRemoteAsync(RemoteInterface.class)
public interface RemoteInterfaceAsync {

    RFuture<Long> someMethod1(Long param1, String param2);

    RFuture<Void> someMethod2(MyObject param);

}

RedissonClient redisson = Redisson.create(config);

RRemoteService remoteService = redisson.getRemoteService();
RemoteInterfaceAsync asyncService = remoteService.get(RemoteInterfaceAsync.class);

asyncService.someMethod1(1L, "myparam");
```
**Reactive Remote interface** and method call example:
```java
@RRemoteReactive(RemoteInterface.class)
public interface RemoteInterfaceReactive {

    Mono<Long> someMethod1(Long param1, String param2);

    Mono<Void> someMethod2(MyObject param);

}

RedissonReactiveClient redisson = Redisson.createReactive(config);

RRemoteService remoteService = redisson.getRemoteService();
RemoteInterfaceReactive reactiveService = remoteService.get(RemoteInterfaceReactive.class);

reactiveService.someMethod1(1L, "myparam");
```
**RxJava3 Remote interface** and method call example:
```java
@RRemoteRx(RemoteInterface.class)
public interface RemoteInterfaceRx {

    Single<Long> someMethod1(Long param1, String param2);

    Completable someMethod2(MyObject param);

}

RedissonRxClient redisson = Redisson.createRx(config);

RRemoteService remoteService = redisson.getRemoteService();
RemoteInterfaceReactive rxService = remoteService.get(RemoteInterfaceRx.class);

rxService.someMethod1(1L, "myparam");
```
### Remote service. Asynchronous, Reactive and RxJava3 call cancellation
远程服务提供在其执行的任何阶段取消调用的能力。分为三个阶段： 

1. 队列中的远程调用请求 
2. 远程服务收到远程调用请求，但尚未处理，且 Ack 响应尚未发送 
3. 远程调用执行正在进行中 

要处理第三阶段，您需要检查远程服务代码中的 `Thread.currentThread().isInterrupted()` 状态。这是一个例子：
```java
public interface MyRemoteInterface {

    Long myBusyMethod(Long param1, String param2);

}

@RRemoteAsync(MyRemoteInterface.class)
public interface MyRemoteInterfaceAsync {

    RFuture<Long> myBusyMethod(Long param1, String param2);

}

@RRemoteReactive(MyRemoteInterface.class)
public interface MyRemoteInterfaceReactive {

    Mono<Long> myBusyMethod(Long param1, String param2);

}

@RRemoteRx(MyRemoteInterface.class)
public interface MyRemoteInterfaceRx {

    Single<Long> myBusyMethod(Long param1, String param2);

}


// remote service implementation
public class MyRemoteServiceImpl implements MyRemoteInterface {

   public Long myBusyMethod(Long param1, String param2) {
       for (long i = 0; i < Long.MAX_VALUE; i++) {
           iterations.incrementAndGet();
           if (Thread.currentThread().isInterrupted()) {
                System.out.println("interrupted! " + i);
                return;
           }
       }
   }

}

RRemoteService remoteService = redisson.getRemoteService();
ExecutorService executor = Executors.newFixedThreadPool(5);
// register remote service using separate
// ExecutorService used to execute remote invocation
MyRemoteInterface serviceImpl = new MyRemoteServiceImpl();
remoteService.register(MyRemoteInterface.class, serviceImpl, 5, executor);

// call Asynchronous method
MyRemoteInterfaceAsync asyncService = remoteService.get(MyRemoteInterfaceAsync.class);
RFuture<Long> future = asyncService.myBusyMethod(1L, "someparam");
// cancel invocation
future.cancel(true);

// call Reactive method
MyRemoteInterfaceReactive reactiveService = remoteService.get(MyRemoteInterfaceReactive.class);
Mono<Long> mono = reactiveService.myBusyMethod(1L, "someparam");
Disposable disp = mono.doOnSubscribe(s -> s.request(1)).subscribe();
// cancel invocation
disp.dispose();

// call RxJava3 method
MyRemoteInterfaceRx asyncService = remoteService.get(MyRemoteInterfaceRx.class);
Single<Long> single = asyncService.myBusyMethod(1L, "someparam");
Disposable disp = single.subscribe();
// cancel invocation
disp.dispose();
```
## Live Object service 分布式对象
### 介绍
Live Object可以理解为标准Java对象的增强版本，它的实例引用不仅可以在单个 JVM 中的线程之间共享，还可以在不同机器上的不同 JVM 之间共享。维基百科将其描述为： 
> Live分布式对象（也简称Live Object）是指分布式多方（或点对点）协议的运行实例，从面向对象的角度来看，作为一个具有明确身份的实体，可以封装内部状态和执行线程，并且表现出明确定义的外部可见行为。 

Redisson Live Object (RLO) 通过运行时构建的代理类将 Java 类内的所有字段映射到 Redis 哈希来实现这一想法。每个字段的所有 getters/setters 方法都被转换为在 redis 哈希上操作的 hget/hset 命令，使得连接到同一 redis 服务器的任何客户端都可以访问它。 getters/setters/constructors 不能由 Lombok 等字节码工具生成。其他方法应该使用 getter 而不是字段。众所周知，对象的字段值代表了它的状态；将它们存储在远程存储库 redis 中，使其成为分布式对象。该对象是 Redisson Live 对象。 
通过使用 RLO，在应用程序和/或服务器之间共享对象与在独立应用程序中共享对象相同。这消除了序列化和反序列化的需要，同时降低了编程模型的复杂性：对一个字段所做的更改（几乎^）可以立即被其他进程、应用程序和服务器访问。 
由于 Redis 服务器是单线程应用程序，因此对活动对象的所有字段访问都会以原子方式自动执行：读取值时不会更改值。 通过 RLO，您可以将 Redis 服务器视为所有连接的 JVM 的共享堆空间。
### 用法
Redisson为Live Object提供了不同的注释。创建和使用活动对象需要`@RId`和`@REntity`注释。
```java
@REntity
public class MyObject {

    @RId
    private String id;
    @RIndex
    private String value;
    private transient SimpleObject simpleObject;
    private MyObject parent;

    public MyObject(String id) {
        this.id = id;
    }

    public MyObject() {
    }

    // getters and setters

}
```
请注意：标有`transient`关键字的字段不会存储在 Redis 中。
要开始使用它，您应该使用以下方法之一：

- `RLiveObjectService.attach()` - 将对象附加到 Redis。丢弃分离实例中已有的所有字段值 
- `RLiveObjectService.merge()` - 使用给定的对象状态覆盖 Redis 中的当前对象状态
- `RLiveObjectService.persist()` - 仅存储新对象

代码示例：
```java
RLiveObjectService service = redisson.getLiveObjectService();
MyLiveObject myObject = new MyLiveObject();
myObject1.setId("1");
// current state of myObject is now persisted and attached to Redis
myObject = service.persist(myObject);

MyLiveObject myObject = new MyLiveObject("1");
// current state of myObject is now cleared and attached to Redis
myObject = service.attach(myObject);

MyLiveObject myObject = new MyLiveObject();
myObject.setId("1");
// current state of myObject is now merged to already existed object and attached to Redis
myObject = service.merge(myObject);
myObject.setValue("somevalue");

// get Live Object by Id
MyLiveObject myObject = service.get(MyLiveObject.class, "1");

// find Live Objects by value field
Collection<MyLiveObject> myObjects = service.find(MyLiveObject.class, Conditions.in("value", "somevalue", "somevalue2"));

Collection<MyLiveObject> myObjects = service.find(MyLiveObject.class, Conditions.and(Conditions.in("value", "somevalue", "somevalue2"), Conditions.eq("secondfield", "test")));
```
RLO 中的字段类型几乎可以是任何类型，从 Java util 类到集合/映射类型，当然还有您自己的自定义对象，只要它可以由提供的编解码器进行编码和解码即可。有关编解码器的更多详细信息可以在“高级用法”部分中找到。 
为了使 RLO 的行为尽可能接近标准 Java 对象，Redisson 自动将以下标准 Java 字段类型转换为 Redisson RObject 支持的计数器类型。

| Standard Java Class | Converted Redisson Class |
| --- | --- |
| SortedSet.class | RedissonSortedSet.class |
| Set.class | RedissonSet.class |
| ConcurrentMap.class | RedissonMap.class |
| Map.class | RedissonMap.class |
| BlockingDeque.class | RedissonBlockingDeque.class |
| Deque.class | RedissonDeque.class |
| BlockingQueue.class | RedissonBlockingQueue.class |
| Queue.class | RedissonQueue.class |
| List.class | RedissonList.class |

如果某一字段类型与多个条目匹配，则转换会优先选择靠近表顶部的字段。即`LinkedList`实现了`Deque`、`List`、`Queue`，因此它将被转换为`RedissonDeque`。 
这些Redisson类的实例也在Redis中保留它们的状态/值/条目，对它们的更改会直接反映到Redis中，而不在本地VM中保留值。
### Search by Object properties 按对象属性搜索
Redisson 为 Live Object 提供全面的搜索引擎。要使属性参与搜索，应使用 @RIndex 注释对其进行注释。
使用示例：
```java
@REntity
public class MyObject {

    @RId
    private String id;

    @RIndex
    private String field1;

    @RIndex
    private Integer field2;
    
    @RIndex
    private Long field3;
}
```
可以使用不同类型的搜索条件：
`Conditions.and `- **AND **condition for collection of nested conditions
`Conditions.eq` - **EQUALS** condition which restricts property to defined value
`Conditions.or` - **OR** condition for collection of nested conditions
`Conditions.in` - **IN** condition which restricts property to set of defined values
`Conditions.gt` - **GREATER THAN** condition which restricts property to defined value
`Conditions.ge` - **GREATER THAN ON EQUAL** condition which restricts property to defined value
`Conditions.lt` - **LESS THAN** condition which restricts property to defined value
`Conditions.le` - **LESS THAN ON EQUAL** condition which restricts property to defined value

一旦对象存储在 Redis 中，我们就可以运行搜索：

|  Search index partitioning in cluster | Ultra-fast search engine | Low JVM memory consumption |  |
| --- | --- | --- | --- |
|  Open-source version   | ❌ | ❌ | ❌ |
|  [Redisson PRO](https://redisson.pro/)  version   | ✔️ | ✔️ | ✔️ |

```java
RLiveObjectService liveObjectService = redisson.getLiveObjectService();

liveObjectService.persist(new MyObject());

// find all objects where field1 = value and field2 < 12 or field1 = value2 and field2 > 23 or field3 in [1, 2]

Collection<MyObject> objects = liveObjectService.find(MyObject.class, 
            Conditions.or(Conditions.and(Conditions.eq("field1", "value"), Conditions.lt("field2", 12)), 
                          Conditions.and(Conditions.eq("field1", "value2"), Conditions.gt("field2", 23)), 
                          Conditions.in("field3", 1L, 2L));
```
仅当 Redis `notify-keyspace-events` 设置包含字母 Kx 时，搜索索引才会在 `expireXXX()` 方法调用后过期。
### 高级用法
如前所述，RLO 类是代理类，可以在需要时构建，然后根据其原始类缓存在 `RedissonClient` 实例中。此过程可能有点慢，建议通过 `RedissonLiveObjectService` 为任何类型的延迟敏感应用程序预先注册所有 Redisson Live Object 类。如果不再需要某个类，该服务还可用于注销该类。当然，它可以用来检查该类是否已经注册。
```java
RLiveObjectService service = redisson.getLiveObjectService();
service.registerClass(MyClass.class);
service.unregisterClass(MyClass.class);
Boolean registered = service.isClassRegistered(MyClass.class);
```
### Annotations 注解
`@REntity` 
应用于一个类。每种类型的 RLO 的行为都可以通过 @REntity 注释的属性进行自定义。您可以指定每个属性以对其行为进行精细控制。 

- **namingScheme** - 您可以指定一个命名方案，告诉 Redisson 如何为此类的每个实例分配键名称。它用于创建对现有 Redisson 活动对象的引用并在 redis 中具体化一个新对象。它默认使用Redisson提供的`DefaultNamingScheme`。 
- **codec** - 您可以告诉 Redisson 您要为 RLO 使用哪个编解码器类。 Redisson将使用实例池根据类类型来定位实例。默认为Redisson提供的`JsonJacksonCodec`。 
- **fieldTransformation** - 您还可以为 RLO 指定字段转换模式。如前所述，为了使所有内容尽可能接近标准 Java，Redisson 会自动将常用 Java util 类的字段转换为 Redisson 兼容类。这使用 `ANNOTATION_BASED` 作为默认值。您可以将其设置为 `IMPLMENTATION_BASED` ，这将跳过转换。

`@RId`
应用于某个字段。定义该类的主键字段。该字段的值用于创建对现有 RLO 的引用。带有此注释的字段是唯一一个其值也保存在本地虚拟机中的字段。每个类只能有一个 `RId` 注释。 
如果您希望以编程方式生成该字段的值，您可以为 @RId 注释提供生成器策略。默认生成器为空。
`@RIndex` 
应用于某个字段。指定该字段用于搜索索引。允许通过 `RLiveObjectService.find` 方法执行基于该字段的搜索查询。
`@RObjectField` 
应用于某个字段。允许指定与 `@REntity` 中指定的不同的`namingScheme`和/或`codec`。
`@RCascade` 
应用于某个字段。指定将定义的级联类型应用于活动对象字段中包含的一个或多个对象。有不同的级联类型可供选择：

- `RCascadeType.ALL` - 包括所有级联类型 
- `RCascadeType.PERSIST` - `RLiveObjectService.persist()` 方法调用期间级联持久操作 
- `RCascadeType.DETACH` - `RLiveObjectService.detach()` 方法调用期间级联分离操作 
- `RCascadeType.MERGE` - `RLiveObjectService.merge()` 方法调用期间级联合并操作 
- `RCascadeType.DELETE` - `RLiveObjectService.delete()` 方法调用期间级联删除操作。
### Restrictions 限制
如上所述，`RId`字段的类型不能是Array类型。这是由于 `DefaultNamingScheme` 目前还无法序列化和反序列化 Array 类型。一旦改进了 `DefaultNamingScheme`，就可以解除此限制。由于 `RId` 字段被编码为底层 RMap 使用的键名的一部分，**因此创建只有一个字段的 RLO 是没有意义的**。对于这种类型的使用，最好使用 `RBucket`。
## Distributed executor service 分布式执行器服务
### 介绍
基于 Redis 的 `RExecutorService` 对象实现 `ExecutorService` 接口来运行 Callable、Runnable 和 Lambda 任务。任务和结果对象使用定义的编解码器进行序列化，并存储在两个请求/响应 Redis 队列中。 Redisson实例和任务id可以注入到任务对象中。任务 ID 大小为 128 位且全局唯一。这允许快速有效地处理 Redis 数据并执行分布式计算。
### Workers 
Worker运行通过`RExecutorService`接口提交的任务。 Worker 应使用 `WorkerOptions` 设置对象进行注册。每个工作线程按照请求的顺序从 Redis 队列的头部轮询一个任务。轮询任务由 `ExecutorService` 执行。如果 `WorkerOptions` 中的 `ExecutorService` 未定义，则使用 Redisson 配置中的全局默认 `ExecutorService`。 
如果您需要注册和执行工作程序的独立 jar，请考虑使用 Redisson 节点。 `WorkerOptions` 公开以下设置：
```java
    WorkerOptions options = WorkerOptions.defaults()
    
    // Defines workers amount used to execute tasks.
    // Default is 1.
    .workers(2)

    // Defines Spring BeanFactory instance to execute tasks 
    // with Spring's '@Autowired', '@Value' or JSR-330's '@Inject' annotation.
    .beanFactory(beanFactory)

    // Defines custom ExecutorService to execute tasks
    // Config.executor is used by default.
    .executorService()

    // Defines task timeout since task execution start moment
    .taskTimeout(60, TimeUnit.SECONDS)

    // add listener which is invoked on task successed event
    .addListener(new TaskSuccessListener() {
         public void onSucceeded(String taskId, T result) {
             // ...
         }
    })

    // add listener which is invoked on task failed event
    .addListener(new TaskFailureListener() {
         public void onFailed(String taskId, Throwable exception) {
             // ...
         }
    })

    // add listener which is invoked on task started event
    .addListener(new TaskStartedListener() {
         public void onStarted(String taskId) {
             // ...
         }
    })

    // add listener which is invoked on task finished event
    .addListener(new TaskFinishedListener() {
         public void onFinished(String taskId) {
             // ...
         }
    });
```
使用上面定义的选项进行 worker 注册的代码示例：
```java
RExecutorService executor = redisson.getExecutorService("myExecutor");
executor.registerWorkers(options);
```
### Tasks
Redisson 节点不需要类路径中存在任务类。任务类由Redisson节点ClassLoader自动加载。因此，每个新任务类不需要重新启动 Redisson 节点。
Example with Callable task:
```java
public class CallableTask implements Callable<Long> {

    @RInject
    private RedissonClient redissonClient;

    @RInject
    private String taskId;

    @Override
    public Long call() throws Exception {
        RMap<String, Integer> map = redissonClient.getMap("myMap");
        Long result = 0;
        for (Integer value : map.values()) {
            result += value;
        }
        return result;
    }

}
```
Example with Runnable task:
```java
public class RunnableTask implements Runnable {

    @RInject
    private RedissonClient redissonClient;

    @RInject
    private String taskId;

    private long param;

    public RunnableTask() {
    }

    public RunnableTask(long param) {
        this.param = param;
    }

    @Override
    public void run() {
        RAtomicLong atomic = redissonClient.getAtomicLong("myAtomic");
        atomic.addAndGet(param);
    }

}
```
在 ExecutorService 获取期间可以提供以下选项：
```java
ExecutorOptions options = ExecutorOptions.defaults()

// Defines task retry interval at the end of which task is executed again.
// ExecutorService worker re-schedule task execution retry every 5 seconds.
//
// Set 0 to disable.
//
// Default is 5 minutes
options.taskRetryInterval(10, TimeUnit.MINUTES);
```
```java
RExecutorService executorService = redisson.getExecutorService("myExecutor", options);
executorService.submit(new RunnableTask(123));

RExecutorService executorService = redisson.getExecutorService("myExecutor", options);
Future<Long> future = executorService.submit(new CallableTask());
Long result = future.get();
```
Example with Lambda task:
```java
RExecutorService executorService = redisson.getExecutorService("myExecutor", options);
Future<Long> future = executorService.submit((Callable & Serializable)() -> {
    System.out.println("task has been executed!");
});
Long result = future.get();
```
每个 Redisson 节点都已准备好使用 RedissonClient，可以使用 `@RInject` 注释进行注入。
### Tasks with Spring beans
Redisson 不仅允许使用 `@RInject` 注释注入 RedissonClient 和任务 id，还支持 Spring 的 `@Autowire`、`@Value` 和 JSR-330 的 `@Inject` 注释。注入的Spring Bean服务接口应该实现`Serializable`。 
Redisson使用Spring的BeanFactory对象进行注入。它应该在 Redisson 节点配置中定义。 可调用任务的示例：
Example with Callable task:
```java
public class CallableTask implements Callable<Integer> {

    @Autowired
    private MySpringService service;

    @Value("myProperty")
    private String prop;

    @RInject
    private RedissonClient redissonClient;

    @Override
    public Integer call() throws Exception {
        RMap<String, Integer> map = redissonClient.getMap(prop);

        Integer result = service.someMethod();
        map.put("test", result);
        return result;
    }

}
```
```java
RExecutorService executorService = redisson.getExecutorService("myExecutor", options);
Future<Integer> future = executorService.submit(new CallableTask());
Integer result = future.get();
```
### Task execution cancellation
通过 `RFuture.cancel()` 或 `RExecutorService.cancelTask()` 方法可以轻松取消任何提交的任务。要处理任务执行已经在进行中的情况，您需要使用 `Thread.currentThread().isInterrupted()` 调用检查线程状态是否中断：
```java
public class CallableTask implements Callable<Long> {

    @RInject
    private RedissonClient redissonClient;

    @Override
    public Long call() throws Exception {
        RMap<String, Integer> map = redissonClient.getMap("myMap");
        Long result = 0;
        // map contains many entries
        for (Integer value : map.values()) {
           if (Thread.currentThread().isInterrupted()) {
                // task has been canceled
                return null;
           }
           result += value;
        }
        return result;
    }

}

RExecutorService executorService = redisson.getExecutorService("myExecutor");

// submit tasks synchronously for asynchronous execution.
RExecutorFuture<Long> future = executorService.submit(new CallableTask());

// submit tasks asynchronously for asynchronous execution.
RExecutorFuture<Long> future = executorService.submitAsync(new CallableTask());

// cancel future
future.cancel(true);

// cancel by taskId
executorService.cancelTask(future.getTaskId());
```
## Distributed scheduled executor service
### 介绍
基于Redis的`RScheduledExecutorService`实现了`ScheduledExecutorService`接口来调度Callable、Runnable和Lambda任务。计划任务是一项需要在未来某个特定时间执行一次或多次的作业。任务和结果对象被序列化并存储在请求/响应 Redis 队列中。 Redisson实例和任务id可以注入到任务对象中。任务 ID 大小为 128 位且全局唯一。这允许快速有效地处理 Redis 数据并执行分布式计算。
### Workers
Worker运行通过`RScheduledExecutorService`接口提交的任务。 Worker 应使用 WorkerOptions 设置对象进行注册。每个工作线程按照请求的顺序从 Redis 队列的头部轮询一个任务。轮询任务由 ExecutorService 执行。如果 WorkerOptions 中的 ExecutorService 未定义，则使用 Redisson 配置中的全局默认 ExecutorService。 
如果您需要注册和执行工作程序的独立 jar，请考虑使用 Redisson 节点。 WorkerOptions 公开以下设置：
```java
    WorkerOptions options = WorkerOptions.defaults()
    
    // Defines workers amount used to execute tasks.
    // Default is 1.
    .workers(2)

    // Defines Spring BeanFactory instance to execute tasks 
    // with Spring's '@Autowired', '@Value' or JSR-330's '@Inject' annotation.
    .beanFactory(beanFactory)

    // Defines custom ExecutorService to execute tasks
    // Config.executor is used by default.
    .executorService()

    // Defines task timeout since task execution start moment
    .taskTimeout(60, TimeUnit.SECONDS)

    // add listener which is invoked on task successed event
    .addListener(new TaskSuccessListener() {
         public void onSucceeded(String taskId, T result) {
             // ...
         }
    })

    // add listener which is invoked on task failed event
    .addListener(new TaskFailureListener() {
         public void onFailed(String taskId, Throwable exception) {
             // ...
         }
    })

    // add listener which is invoked on task started event
    .addListener(new TaskStartedListener() {
         public void onStarted(String taskId) {
             // ...
         }
    })

    // add listener which is invoked on task finished event
    .addListener(new TaskFinishedListener() {
         public void onFinished(String taskId) {
             // ...
         }
    });
```
使用上面定义的选项进行 worker 注册的代码示例：
```java
RScheduledExecutorService executor = redisson.getExecutorService("myExecutor");
executor.registerWorkers(options);
```
### Scheduling a task
Redisson 节点不需要 classpath 中存在任务类。任务类由Redisson节点ClassLoader自动加载。因此，每个新任务类不需要重新启动 Redisson 节点。
Example with Callable task:
```java
public class CallableTask implements Callable<Long> {

    @RInject
    private RedissonClient redissonClient;

    @Override
    public Long call() throws Exception {
        RMap<String, Integer> map = redissonClient.getMap("myMap");
        Long result = 0;
        for (Integer value : map.values()) {
            result += value;
        }
        return result;
    }

}
```
在 ExecutorService 获取期间可以提供以下选项：
```java
ExecutorOptions options = ExecutorOptions.defaults()

// Defines task retry interval at the end of which task is executed again.
// ExecutorService worker re-schedule task execution retry every 5 seconds.
//
// Set 0 to disable.
//
// Default is 5 minutes
options.taskRetryInterval(10, TimeUnit.MINUTES);
```
```java
RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor");
ScheduledFuture<Long> future = executorService.schedule(new CallableTask(), 10, TimeUnit.MINUTES);
Long result = future.get();
```
Example with Lambda task:
```java
RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor", options);
ScheduledFuture<Long> future = executorService.schedule((Callable & Serializable)() -> {
    System.out.println("task has been executed!");
}, 10, TimeUnit.MINUTES);
Long result = future.get();
```
Example with Runnable task:
```java
public class RunnableTask implements Runnable {

    @RInject
    private RedissonClient redissonClient;

    private long param;

    public RunnableTask() {
    }

    public RunnableTask(long param) {
        this.param= param;
    }

    @Override
    public void run() {
        RAtomicLong atomic = redissonClient.getAtomicLong("myAtomic");
        atomic.addAndGet(param);
    }

}

RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor");
ScheduledFuture<?> future1 = executorService.schedule(new RunnableTask(123), 10, TimeUnit.HOURS);
// ...
ScheduledFuture<?> future2 = executorService.scheduleAtFixedRate(new RunnableTask(123), 10, 25, TimeUnit.HOURS);
// ...
ScheduledFuture<?> future3 = executorService.scheduleWithFixedDelay(new RunnableTask(123), 5, 10, TimeUnit.HOURS);
```
### Scheduling a task with Spring beans
Redisson不仅允许使用`@RInject`注解注入RedissonClient，还支持Spring的`@Autowire`、`@Value`和JSR-330的@Inject注解。注入的Spring Bean服务接口应该实现`Serializable`。 
Redisson使用Spring的BeanFactory对象进行注入。它应该在 Redisson 节点配置中定义。
Example with Callable task:
```java
public class RunnableTask implements Runnable {

    @Autowired
    private MySpringService service;

    @Value("myProperty")
    private String prop;

    @RInject
    private RedissonClient redissonClient;

    private long param;

    public RunnableTask() {
    }

    public RunnableTask(long param) {
        this.param = param;
    }

    @Override
    public void run() {
        RMap<String, Integer> map = redissonClient.getMap(prop);

        Integer result = service.someMethod();
        map.put("test", result);
        return result;
    }

}
```
```java
RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor");
ScheduledFuture<?> future1 = executorService.schedule(new RunnableTask(123), 10, TimeUnit.HOURS);
// ...
ScheduledFuture<?> future2 = executorService.scheduleAtFixedRate(new RunnableTask(123), 10, 25, TimeUnit.HOURS);
// ...
ScheduledFuture<?> future3 = executorService.scheduleWithFixedDelay(new RunnableTask(123), 5, 10, TimeUnit.HOURS);
```
### Scheduling a task with cron expression
任务调度程序服务允许使用 cron 表达式定义更复杂的计划。其中完全兼容 Quartz cron 格式。
示例：
```java
RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor");
executorService.schedule(new RunnableTask(), CronSchedule.of("10 0/5 * * * ?"));
// ...
executorService.schedule(new RunnableTask(), CronSchedule.dailyAtHourAndMinute(10, 5));
// ...
executorService.schedule(new RunnableTask(), CronSchedule.weeklyOnDayAndHourAndMinute(12, 4, Calendar.MONDAY, Calendar.FRIDAY));
```
### Task scheduling cancellation
定时执行器服务提供了两种取消定时任务的方法：`RScheduledFuture.cancel()` 或 `RScheduledExecutorService.cancelTask()`方法。要处理任务执行已经在进行中的情况，您需要使用 `Thread.currentThread().isInterrupted()` 调用检查线程状态是否中断：
```java
public class RunnableTask implements Callable<Long> {

    @RInject
    private RedissonClient redissonClient;

    @Override
    public Long call() throws Exception {
        RMap<String, Integer> map = redissonClient.getMap("myMap");
        Long result = 0;
        // map contains many entries
        for (Integer value : map.values()) {
           if (Thread.currentThread().isInterrupted()) {
                // task has been canceled
                return null;
           }
           result += value;
        }
        return result;
    }

}

RScheduledExecutorService executorService = redisson.getExecutorService("myExecutor");

// submit tasks synchronously for asynchronous execution.
RScheduledFuture<Long> future = executorService.schedule(new RunnableTask(), CronSchedule.dailyAtHourAndMinute(10, 5));

// submit tasks asynchronously for asynchronous execution.
RScheduledFuture<Long> future = executorService.scheduleAsync(new RunnableTask(), CronSchedule.dailyAtHourAndMinute(10, 5));

// cancel future
future.cancel(true);

// cancel by taskId
executorService.cancelTask(future.getTaskId());
```
## Distributed MapReduce service
### 介绍
Redisson提供MapReduce编程模型来处理Redis中存储的大量数据。基于不同实现和 Google 的 MapReduce 的想法。支持不同的对象：`RMap`、`RMapCache`、`RLocalCachedMap`、`RClusteredMap`、`RClusteredMapCache`、`RClusteredLocalCachedMap`、`RClusteredSet`、`RClusteredSetCache`、`RSet`、`RSetCache`、`RList`、`RSortedSet`、`RScoredSortedSet`、`RQueue`、`RBlockingQueue`、`RDeque`、`RBlockingDeque`、`RPriorityQueue` 和 `RPriorityDeque`。 
基于几个对象的 MapReduce 模型：`RMapper/RCollectionMapper`、`RReducer` 和 `RCollator`。 
Map 和Reduce 阶段的所有任务都跨Redisson 节点执行。 
对于 `RLocalCachedMap`、`RClusteredMap`、`RClusteredMapCache`、`RClusteredLocalCachedMap`、`RClusteredSet`、`RClusteredSetCache` 对象，Map 阶段被拆分为多个子任务并行执行。对于其他对象，Map 阶段在单个子任务中执行。 Reduce阶段总是被分割成多个子任务并并行执行。子任务数量等于注册的MapReduce worker总数。
#### RMapper 仅适用于 Map 对象，并将每个 Map 的条目转换为中间键/值对。
```java
public interface RMapper<KIn, VIn, KOut, VOut> extends Serializable {

    void map(KIn key, VIn value, RCollector<KOut, VOut> collector);
    
}
```
#### RCollectionMapper 仅适用于 Collection 对象，并将每个 Collection 的条目转换为中间键/值对
```java
public interface RCollectionMapper<VIn, KOut, VOut> extends Serializable {

    void map(VIn value, RCollector<KOut, VOut> collector);
    
}
```
#### RReducer 减少中间键/值对的列表
```java
public interface RReducer<K, V> extends Serializable {

    V reduce(K reducedKey, Iterator<V> values);
    
}
```
#### RCollator 将缩减结果转换为单个对象
```java
public interface RCollator<K, V, R> extends Serializable {

    R collate(Map<K, V> resultMap);
    
}
```
每种类型的任务都能够使用 `@RInject` 注释注入 RedissonClient，如下所示：
```java
    public class WordMapper implements RMapper<String, String, String, Integer> {

        @RInject
        private RedissonClient redissonClient;

        @Override
        public void map(String key, String value, RCollector<String, Integer> collector) {

            // ...

            redissonClient.getAtomicLong("mapInvocations").incrementAndGet();
        }
        
    }

```
默认情况下，MapReduce 的执行时间不受限制，但如果需要，可以定义超时：
```java
             = list.<String, Integer>mapReduce()
                   .mapper(new WordMapper())
                   .reducer(new WordReducer())
                   .timeout(60, TimeUnit.MINUTES);
```
### Map 示例
MapReduce 可用于映射类型对象：`RMap`、`RMapCache` 和 `RLocalCachedMap`。 下面是使用 MapReduce 的字数统计示例：

1. Mapper 对象应用于每个 Map 条目，并按空格分割值以分隔单词。
```java
    public class WordMapper implements RMapper<String, String, String, Integer> {

        @Override
        public void map(String key, String value, RCollector<String, Integer> collector) {
            String[] words = value.split("[^a-zA-Z]");
            for (String word : words) {
                collector.emit(word, 1);
            }
        }
        
    }
```

2. Reducer 对象计算每个单词的总和。
```java
    public class WordReducer implements RReducer<String, Integer> {

        @Override
        public Integer reduce(String reducedKey, Iterator<Integer> iter) {
            int sum = 0;
            while (iter.hasNext()) {
               Integer i = (Integer) iter.next();
               sum += i;
            }
            return sum;
        }
        
    }
```

3. Collator 对象计算单词总数。
```java
    public class WordCollator implements RCollator<String, Integer, Integer> {

        @Override
        public Integer collate(Map<String, Integer> resultMap) {
            int result = 0;
            for (Integer count : resultMap.values()) {
                result += count;
            }
            return result;
        }
        
    }
```

4. 以下是如何一起运行：
```java
    RMap<String, String> map = redisson.getMap("wordsMap");
    map.put("line1", "Alice was beginning to get very tired"); 
    map.put("line2", "of sitting by her sister on the bank and");
    map.put("line3", "of having nothing to do once or twice she");
    map.put("line4", "had peeped into the book her sister was reading");
    map.put("line5", "but it had no pictures or conversations in it");
    map.put("line6", "and what is the use of a book");
    map.put("line7", "thought Alice without pictures or conversation");

    RMapReduce<String, String, String, Integer> mapReduce
             = map.<String, Integer>mapReduce()
                  .mapper(new WordMapper())
                  .reducer(new WordReducer());

    // count occurrences of words
    Map<String, Integer> mapToNumber = mapReduce.execute();

    // count total words amount
    Integer totalWordsAmount = mapReduce.execute(new WordCollator());
```
### Collection 示例
MapReduce 可用于集合类型对象：`RSet`、`RSetCache`、`RList`、`RSortedSet`、`RScoredSortedSet`、`RQueue`、`RBlockingQueue`、`RDeque`、`RBlockingDeque`、`RPriorityQueue` 和 `RPriorityDeque`。
下面是使用 MapReduce 的字数统计示例：
```java
public class WordMapper implements RCollectionMapper<String, String, Integer> {

    @Override
    public void map(String value, RCollector<String, Integer> collector) {
        String[] words = value.split("[^a-zA-Z]");
        for (String word : words) {
            collector.emit(word, 1);
        }
    }

}
```
```java
public class WordReducer implements RReducer<String, Integer> {

    @Override
    public Integer reduce(String reducedKey, Iterator<Integer> iter) {
        int sum = 0;
        while (iter.hasNext()) {
            Integer i = (Integer) iter.next();
            sum += i;
        }
        return sum;
    }

}
```
```java
public class WordCollator implements RCollator<String, Integer, Integer> {

    @Override
    public Integer collate(Map<String, Integer> resultMap) {
        int result = 0;
        for (Integer count : resultMap.values()) {
            result += count;
        }
        return result;
    }

}
```
```java
RList<String> list = redisson.getList("myList");
list.add("Alice was beginning to get very tired"); 
list.add("of sitting by her sister on the bank and");
list.add("of having nothing to do once or twice she");
list.add("had peeped into the book her sister was reading");
list.add("but it had no pictures or conversations in it");
list.add("and what is the use of a book");
list.add("thought Alice without pictures or conversation");

RCollectionMapReduce<String, String, Integer> mapReduce
= list.<String, Integer>mapReduce()
.mapper(new WordMapper())
.reducer(new WordReducer());

// count occurrences of words
Map<String, Integer> mapToNumber = mapReduce.execute();

// count total words amount
Integer totalWordsAmount = mapReduce.execute(new WordCollator());
```
## RediSearch service
Redisson 提供 RediSearch 集成。支持`RMap`和`RJSONBucket`对象的字段索引、查询执行和聚合。 
它具有 Async、Reactive 和 RxJava3 接口。
### 运行查询
Code example for RMap object:
```java
RMap<String, SimpleObject> m = redisson.getMap("doc:1", new CompositeCodec(StringCodec.INSTANCE, redisson.getConfig().getCodec()));
m.put("v1", new SimpleObject("name1"));
m.put("v2", new SimpleObject("name2"));
RMap<String, SimpleObject> m2 = redisson.getMap("doc:2", new CompositeCodec(StringCodec.INSTANCE, redisson.getConfig().getCodec()));
m2.put("v1", new SimpleObject("name3"));
m2.put("v2", new SimpleObject("name4"));

RSearch s = redisson.getSearch();
s.createIndex("idx", IndexOptions.defaults()
              .on(IndexType.HASH)
              .prefix(Arrays.asList("doc:")),
              FieldIndex.text("v1"),
              FieldIndex.text("v2"));

SearchResult r = s.search("idx", "*", QueryOptions.defaults()
                          .returnAttributes(new ReturnAttribute("v1"), new ReturnAttribute("v2")));
```
Code example for JSON object:
```java
public class TestClass {

    private List<Integer> arr;
    private String value;

    public TestClass() {
    }

    public TestClass(List<Integer> arr, String value) {
        this.arr = arr;
        this.value = value;
    }

    public List<Integer> getArr() {
        return arr;
    }

    public TestClass setArr(List<Integer> arr) {
        this.arr = arr;
        return this;
    }

    public String getValue() {
        return value;
    }

    public TestClass setValue(String value) {
        this.value = value;
        return this;
    }
}


RJsonBucket<TestClass> b = redisson.getJsonBucket("doc:1", new JacksonCodec<>(TestClass.class));
b.set(new TestClass(Arrays.asList(1, 2, 3), "hello"));

RSearch s = redisson.getSearch(StringCodec.INSTANCE);
s.createIndex("idx", IndexOptions.defaults()
              .on(IndexType.JSON)
              .prefix(Arrays.asList("doc:")),
              FieldIndex.numeric("$..arr").as("arr"),
              FieldIndex.text("$..value").as("val"));

SearchResult r = s.search("idx", "*", QueryOptions.defaults()
                          .returnAttributes(new ReturnAttribute("arr"), new ReturnAttribute("val")));
```
### Aggregation 聚合
Code example for Map object:
```java
RMap<String, SimpleObject> m = redisson.getMap("doc:1", new CompositeCodec(StringCodec.INSTANCE, redisson.getConfig().getCodec()));
m.put("v1", new SimpleObject("name1"));
m.put("v2", new SimpleObject("name2"));
RMap<String, SimpleObject> m2 = redisson.getMap("doc:2", new CompositeCodec(StringCodec.INSTANCE, redisson.getConfig().getCodec()));
m2.put("v1", new SimpleObject("name3"));
m2.put("v2", new SimpleObject("name4"));

RSearch s = redisson.getSearch();
s.createIndex("idx", IndexOptions.defaults()
              .on(IndexType.HASH)
              .prefix(Arrays.asList("doc:")),
              FieldIndex.text("v1"),
              FieldIndex.text("v2"));

AggregationResult r = s.aggregate("idx", "*", AggregationOptions.defaults()
                                  .load("v1", "v2"));
```
Code example for JSON object:
```java
public class TestClass {

    private List<Integer> arr;
    private String value;

    public TestClass() {
    }

    public TestClass(List<Integer> arr, String value) {
        this.arr = arr;
        this.value = value;
    }

    public List<Integer> getArr() {
        return arr;
    }

    public TestClass setArr(List<Integer> arr) {
        this.arr = arr;
        return this;
    }

    public String getValue() {
        return value;
    }

    public TestClass setValue(String value) {
        this.value = value;
        return this;
    }
}

RJsonBucket<TestClass> b = redisson.getJsonBucket("doc:1", new JacksonCodec<>(TestClass.class));
b.set(new TestClass(Arrays.asList(1, 2, 3), "hello"));

RSearch s = redisson.getSearch(StringCodec.INSTANCE);
s.createIndex("idx", IndexOptions.defaults()
              .on(IndexType.JSON)
              .prefix(Arrays.asList("doc:")),
              FieldIndex.numeric("$..arr").as("arr"),
              FieldIndex.text("$..value").as("val"));

AggregationResult r = s.aggregate("idx", "*", AggregationOptions.defaults()
                                  .load("arr", "val"));
```
### Spellcheck 拼写检查
代码示例：
```java
RSearch s = redisson.getSearch();

s.createIndex("idx", IndexOptions.defaults()
              .on(IndexType.HASH)
              .prefix(Arrays.asList("doc:")),
              FieldIndex.text("t1"),
              FieldIndex.text("t2"));

s.addDict("name", "hockey", "stik");

Map<String, Map<String, Double>> res = s.spellcheck("idx", "Hocke sti", SpellcheckOptions.defaults()
                                                    .includedTerms("name"));
```
# Additional features 其他功能
## 管理 Redis 节点
Redisson提供API来管理Redis节点。
Code example of operations with **Redis Cluster** setup:
```java
RedisCluster cluster = redisson.getRedisNodes(RedisNodes.CLUSTER);
cluster.pingAll();
Collection<RedisClusterMaster> masters = cluster.getMasters();
Collection<RedisClusterMaster> slaves = cluster.getSlaves();
```
Code example of operations with **Redis Master Slave** setup:
```java
RedisMasterSlave masterSlave = redisson.getRedisNodes(RedisNodes.MASTER_SLAVE);
masterSlave.pingAll();
RedisMaster master = masterSlave.getMaster();
Collection<RedisSlave> slaves = masterSlave.getSlaves();
```
Code example of operations with **Redis Sentinel** setup:
```java
RedisSentinelMasterSlave sentinelMasterSlave = redisson.getRedisNodes(RedisNodes.SENTINEL_MASTER_SLAVE);
sentinelMasterSlave.pingAll();
RedisMaster master = sentinelMasterSlave.getMaster();
Collection<RedisSlave> slaves = sentinelMasterSlave.getSlaves();
Collection<RedisSentinel> sentinels = sentinelMasterSlave.getSentinels();
```
Code example of operations with **Redis Single** setup:
```java
RedisSingle single = redisson.getRedisNodes(RedisNodes.SINGLE);
single.pingAll();
RedisMaster instance = single.getInstance();
```
## References to Redisson objects 对象嵌套
可以以任意组合在另一个 Redisson 对象中使用一个 Redisson 对象。在这种情况下，Redisson 将使用和处理一个特殊的引用对象。使用示例：
```java
RMap<RSet<RList>, RList<RMap>> map = redisson.getMap("myMap");
RSet<RList> set = redisson.getSet("mySet");
RList<RMap> list = redisson.getList("myList");

map.put(set, list);
// With the help of the special reference object, we can even create a circular
// reference which is impossible to achieve if we were to serialize its content
set.add(list);
list.add(map);
```
您可能已经注意到，在地图对象的元素发生更改后，无需重新“保存/保留”地图对象。因为它不包含任何值而只是一个引用，这使得 Redisson 对象的行为更像标准 Java 对象。实际上，使 Redis 成为 JVM 内存的一部分而不仅仅是一个简单的存储库。 
本例中将创建1个Redis HASH、1个Redis SET和1个Redis LIST。
## 批量执行命令
可以在单个网络调用中使用 RBatch 对象批量发送多个命令。命令批处理可以减少一组命令的总体执行时间。在 Redis 中，这种方法称为管道化。在对象创建期间可以提供以下选项：
```java
BatchOptions options = BatchOptions.defaults()
// Sets execution mode
//
// ExecutionMode.REDIS_READ_ATOMIC - Store batched invocations in Redis and execute them atomically as a single command
//
// ExecutionMode.REDIS_WRITE_ATOMIC - Store batched invocations in Redis and execute them atomically as a single command
//
// ExecutionMode.IN_MEMORY - Store batched invocations in memory on Redisson side and execute them on Redis. Default mode
//
// ExecutionMode.IN_MEMORY_ATOMIC - Store batched invocations on Redisson side and executes them atomically on Redis as a single command
.executionMode(ExecutionMode.IN_MEMORY)

// Inform Redis not to send reply back to client. This allows to save network traffic for commands with batch with big
.skipResult()

// Synchronize write operations execution across defined amount of Redis slave nodes
//
// sync with 2 slaves with 1 second for timeout
.syncSlaves(2, 1, TimeUnit.SECONDS)

// Response timeout
.responseTimeout(2, TimeUnit.SECONDS)

// Retry interval for each attempt to send Redis commands batch
.retryInterval(2, TimeUnit.SECONDS);

// Attempts amount to re-send Redis commands batch if it wasn't sent due to network delays or other issues
.retryAttempts(4);
```
结果批次对象包含以下数据：
```java
// list of result objects per command in batch
List<?> responses = res.getResponses();
// amount of successfully synchronized slaves during batch execution
int slaves = res.getSyncedSlaves();
```
Code example for **Sync / Async** mode:
```java
RBatch batch = redisson.createBatch(BatchOptions.defaults());
batch.getMap("test1").fastPutAsync("1", "2");
batch.getMap("test2").fastPutAsync("2", "3");
batch.getMap("test3").putAsync("2", "5");
RFuture<Long> future = batch.getAtomicLong("counter").incrementAndGetAsync();
batch.getAtomicLong("counter").incrementAndGetAsync();

// result could be acquired through RFuture object returned by batched method
// or 
// through result list by corresponding index
future.whenComplete((res, exception) -> {
    // ...
});

BatchResult res = batch.execute();
// or
Future<BatchResult> resFuture = batch.executeAsync();

List<?> list = res.getResponses();
Long result = list.get(4);
```
Code example of [Reactive](https://www.javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RTransactionReactive.html)** interface** usage:
```java
RBatchReactive batch = redisson.createBatch(BatchOptions.defaults());
batch.getMap("test1").fastPut("1", "2");
batch.getMap("test2").fastPut("2", "3");
batch.getMap("test3").put("2", "5");
Mono<Long> commandMono = batch.getAtomicLongAsync("counter").incrementAndGet();
batch.getAtomicLongAsync("counter").incrementAndGet();

// result could be acquired through Reactive object returned by batched method
// or 
// through result list by corresponding index
commandMono.doOnNext(res -> {
    // ...
}).subscribe();

Mono<BatchResult> resMono = batch.execute();
resMono.doOnNext(res -> {
    List<?> list = res.getResponses();
    Long result = list.get(4);

    // ...
}).subscribe();
```
Code example of [RxJava3](https://www.javadoc.io/doc/org.redisson/redisson/latest/org/redisson/api/RTransactionRx.html)** interface** usage:
```java
RBatchRx batch = redisson.createBatch(BatchOptions.defaults());
batch.getMap("test1").fastPut("1", "2");
batch.getMap("test2").fastPut("2", "3");
batch.getMap("test3").put("2", "5");
Single<Long> commandSingle = batch.getAtomicLongAsync("counter").incrementAndGet();
batch.getAtomicLongAsync("counter").incrementAndGet();

// result could be acquired through RxJava3 object returned by batched method
// or 
// through result list by corresponding index
commandSingle.doOnSuccess(res -> {
    // ...
}).subscribe();

Mono<BatchResult> resSingle = batch.execute();
resSingle.doOnSuccess(res -> {
    List<?> list = res.getResponses();
    Long result = list.get(4);

    // ...
}).subscribe();
```
在集群环境中以map\reduce方式批量执行。它聚合每个节点的命令并同时发送它们，然后将每个节点得到的结果添加到公共结果列表中。
## 事务
`RMap`、`RMapCache`、`RLocalCachedMap`、`RSet`、`RSetCache` 和 `RBucket` Redisson 对象可以参与具有 ACID 属性的 Transaction。它对写操作使用锁，并维护数据修改操作列表，直到执行 `commit()` 方法。获取的锁在 `rollback()` 方法执行后被释放。如果在提交/回滚执行期间出现任何错误，实现会抛出 `org.redisson.transaction.TransactionException `。
事务的隔离级别是：`READ_COMMITTED`。
另请参阅 Spring 事务管理器部分。 另请参阅 XA 事务部分。
可用以下事务选项：
```java
TransactionOptions options = TransactionOptions.defaults()
// Synchronization data timeout between Redis master participating in transaction and its slaves.
// Default is 5000 milliseconds.
.syncSlavesTimeout(5, TimeUnit.SECONDS)

// Response timeout
// Default is 3000 milliseconds.
.responseTimeout(3, TimeUnit.SECONDS)

// Defines time interval for each attempt to send transaction if it hasn't been sent already.
// Default is 1500 milliseconds.
.retryInterval(2, TimeUnit.SECONDS)

// Defines attempts amount to send transaction if it hasn't been sent already.
// Default is 3 attempts.
.retryAttempts(3)

// If transaction hasn't committed within <code>timeout</code> it will rollback automatically.
// Default is 5000 milliseconds.
.timeout(5, TimeUnit.SECONDS);
```
对于在 Redis 集群中执行，请使用 {} 括号将数据放在同一槽上，否则 Redis 会抛出 CROSSSLOT 错误。
Redis集群的代码示例：
```java
RMap<String, String> map = transaction.getMap("myMap{user:1}");
map.put("1", "2");
String value = map.get("3");
RSet<String> set = transaction.getSet("mySet{user:1}")
set.add(value);
```
Code example of **Sync / Async** exection:
```java
RedissonClient redisson = Redisson.create(config);
RTransaction transaction = redisson.createTransaction(TransactionOptions.defaults());

RMap<String, String> map = transaction.getMap("myMap");
map.put("1", "2");
String value = map.get("3");
RSet<String> set = transaction.getSet("mySet")
set.add(value);

try {
    transaction.commit();
} catch(TransactionException e) {
    transaction.rollback();
}

// or

RFuture<Void> future = transaction.commitAsync();
future.exceptionally(exception -> {
    transaction.rollbackAsync();
});
```
Code example of **Reactive** exection:
```java
RedissonReactiveClient redisson = Redisson.create(config).reactive();
RTransactionReactive transaction = redisson.createTransaction(TransactionOptions.defaults());

RMapReactive<String, String> map = transaction.getMap("myMap");
map.put("1", "2");
Mono<String> mapGet = map.get("3");
RSetReactive<String> set = transaction.getSet("mySet")
set.add(value);

Mono<Void> mono = transaction.commit();
mono.onErrorResume(exception -> {
    return transaction.rollback();
});
```
Code example of **RxJava3** exection:
```java
RedissonRxClient redisson = Redisson.create(config).rxJava();
RTransactionRx transaction = redisson.createTransaction(TransactionOptions.defaults());

RMapRx<String, String> map = transaction.getMap("myMap");
map.put("1", "2");
Maybe<String> mapGet = map.get("3");
RSetRx<String> set = transaction.getSet("mySet")
set.add(value);

Completable res = transaction.commit();
res.onErrorResumeNext(exception -> {
    return transaction.rollback();
});
```
## XA 事务
Redisson提供了`XAResource`实现来参与JTA事务。 `RMap`、`RMapCache`、`RLocalCachedMap`、`RSet`、`RSetCache` 和 `RBucket` Redisson 对象可以参与 Transaction。 
事务隔离级别为：`READ_COMMITTED`。 
另请参阅事务章节。 另请参阅 Spring 事务章节。 
_此功能仅在 Redisson PRO 版本中可用。 _
对于在 Redis 集群中执行，请使用 {} 括号将数据放在同一槽上，否则 Redis 会抛出 CROSSSLOT 错误。
Redis 集群的代码示例：
```java
RMap<String, String> map = transaction.getMap("myMap{user:1}");
map.put("1", "2");
String value = map.get("3");
RSet<String> set = transaction.getSet("mySet{user:1}")
set.add(value);
```
代码示例：
```java
// transaction obtained from JTA compatible transaction manager
Transaction globalTransaction = transactionManager.getTransaction();

RXAResource xaResource = redisson.getXAResource();
globalTransaction.enlistResource(xaResource);

RTransaction transaction = xaResource.getTransaction();
RBucket<String> bucket = transaction.getBucket("myBucket");
bucket.set("simple");
RMap<String, String> map = transaction.getMap("myMap");
map.put("myKey", "myValue");
        
transactionManager.commit();
```
## Scripting
Redisson提供了`RScript`对象来执行Lua脚本。它具有原子性属性，用于在Redis端处理数据。脚本可以以两种模式执行： 

- Mode.READ_ONLY 脚本作为读操作执行 
- Mode.READ_WRITE 脚本作为写操作执行 

作为脚本结果对象返回以下类型之一： 

- ReturnType.BOOLEAN - 布尔类型。 
- ReturnType.INTEGER - 整数类型。 
- ReturnType.MULTI - 用户定义类型的列表类型。 
- ReturnType.STATUS - Lua 字符串类型。 
- ReturnType.VALUE - 用户定义的类型。 
- ReturnType.MAPVALUE - 地图值类型。 
- ReturnType.MAPVALUELIST - Map 值类型列表。 

代码示例：
```java
RBucket<String> bucket = redisson.getBucket("foo");
bucket.set("bar");

RScript script = redisson.getScript(StringCodec.INSTANCE);

String r = script.eval(Mode.READ_ONLY,
                       "return redis.call('get', 'foo')", 
                       RScript.ReturnType.VALUE);

// execute the same script stored in Redis lua script cache

// load lua script into Redis cache to all redis master instances
String res = script.scriptLoad("return redis.call('get', 'foo')");
// res == 282297a0228f48cd3fc6a55de6316f31422f5d17

// call lua script by sha digest
Future<Object> r1 = redisson.getScript().evalShaAsync(Mode.READ_ONLY,
   "282297a0228f48cd3fc6a55de6316f31422f5d17",
   RScript.ReturnType.VALUE, Collections.emptyList());
```
## Functions
Redisson 提供 RFunction 对象来执行 Redis 函数。它具有原子性属性，用于在Redis端处理数据。函数可以以两种模式执行： 

- Mode.READ 将函数作为读操作执行 
- Mode.WRITE 将函数作为写操作执行 

作为脚本结果对象返回以下类型之一： 

- ReturnType.BOOLEAN - 布尔类型 
- ReturnType.LONG - 长类型 
- ReturnType.LIST - 用户定义类型的列表类型 
- ReturnType.STRING - 纯字符串类型 
- ReturnType.VALUE - 用户定义的类型 
- ReturnType.MAPVALUE - 映射值类型。 Codec.getMapValueDecoder() 和 Codec.getMapValueEncoder() 方法用于数据反序列化或序列化 
- ReturnType.MAPVALUELIST - 列表类型，由Map Value 类型的对象组成。 Codec.getMapValueDecoder() 和 Codec.getMapValueEncoder() 方法用于数据反序列化或序列化 

代码示例：
```java
RFunction f = redisson.getFunction();

// load function
f.load("lib", "redis.register_function('myfun', function(keys, args) return args[1] end)");

// execute function
String value = f.call(RFunction.Mode.READ, "myfun", RFunction.ReturnType.STRING, Collections.emptyList(), "test");
```
Code example of **Async interface** usage:
```java
RFunction f = redisson.getFunction();

// load function
RFuture<Void> loadFuture = f.loadAsync("lib", "redis.register_function('myfun', function(keys, args) return args[1] end)");

// execute function
RFuture<String> valueFuture = f.callAsync(RFunction.Mode.READ, "myfun", RFunction.ReturnType.STRING, Collections.emptyList(), "test");
```
Code example of **Reactive interface** usage:
```java
RedissonReactiveClient redisson = redissonClient.reactive();
RFunctionReactive f = redisson.getFunction();

// load function
Mono<Void> loadMono = f.load("lib", "redis.register_function('myfun', function(keys, args) return args[1] end)");

// execute function
Mono<String> valueMono = f.callAsync(RFunction.Mode.READ, "myfun", RFunction.ReturnType.STRING, Collections.emptyList(), "test");
```
Code example of **RxJava3 interface** usage:
```java
RedissonRxClient redisson = redissonClient.rxJava();
RFunctionRx f = redisson.getFunction();

// load function
Completable loadMono = f.load("lib", "redis.register_function('myfun', function(keys, args) return args[1] end)");

// execute function
Maybe<String> valueMono = f.callAsync(RFunction.Mode.READ, "myfun", RFunction.ReturnType.STRING, Collections.emptyList(), "test");
```
## Low level Redis client
Redisson 使用适用于 Java 的高性能异步和无锁 Redis 客户端。它支持异步和同步模式。**最流行的用例是执行 Redisson 尚不支持的命令**。请确保 Redis 命令映射列表尚不支持所需的命令。 `org.redisson.client.protocol.RedisCommands` - 包含所有可用命令。
代码示例：
```java
// Use shared EventLoopGroup only if multiple clients are used
EventLoopGroup group = new NioEventLoopGroup();

RedisClientConfig config = new RedisClientConfig();
config.setAddress("redis://localhost:6379") // or rediss:// for ssl connection
      .setPassword("myPassword")
      .setDatabase(0)
      .setClientName("myClient")
      .setGroup(group);

RedisClient client = RedisClient.create(config);
RedisConnection conn = client.connect();
//or
CompletionStage<RedisConnection> connFuture = client.connectAsync();

// execute SET command in sync way
conn.sync(StringCodec.INSTANCE, RedisCommands.SET, "test", 0);
// execute GET command in async way
conn.async(StringCodec.INSTANCE, RedisCommands.GET, "test");

conn.close()
// or
conn.closeAsync()

client.shutdown();
// or
client.shutdownAsync();
```
# Redis commands mapping Redis命令映射
| Redis 命令 | Sync / Async API Redisson.create(config) | Reactive API redisson.reactive() | RxJava3 API redisson.rxJava() |
| --- | --- | --- | --- |
| AUTH | Config.setPassword() | - | - |
| APPEND | RBinaryStream. getOutputStream().write() | - | - |
| BZPOPMAX | RScoredSortedSet. pollLast() pollLastAsync() | RScoredSortedSetReactive. pollLast() | RScoredSortedSetRx. pollLast() |
| BZPOPMIN | RScoredSortedSet. pollFirst() pollFirstAsync() | RScoredSortedSetReactive. pollFirst() | RScoredSortedSetRx. pollFirst() |
| BZMPOP | RScoredSortedSet. pollLast() pollLastAsync() pollLastFromAny() pollLastEntriesFromAny() | RScoredSortedSetReactive. pollLast() pollLastFromAny() pollLastEntriesFromAny() | RScoredSortedSetRx. pollLast() pollLastFromAny() pollLastEntriesFromAny() |
| BITCOUNT | RBitSet. cardinality() cardinalityAsync() | RBitSetReactive. cardinality() | RBitSetRx. cardinality() |
| BITOP | RBitSet. or() and() xor() orAsync() andAsync() xorAsync() | RBitSetReactive. or() and() xor() | RBitSetRx. or() and() xor() |
| BITPOS | RBitSet. length() lengthAsync() | RBitSetReactive. length() | RBitSetRx. length() |
| BITFIELD | RBitSet. getByte() setByte() incrementAndGetByte()  getShort() setShort() incrementAndGetShort()  getInteger() setInteger() incrementAndGetInteger()  getLong() setLong() incrementAndGetLong() | RBitSetReactive. getByte() setByte() incrementAndGetByte()  getShort() setShort() incrementAndGetShort()  getInteger() setInteger() incrementAndGetInteger()  getLong() setLong() incrementAndGetLong() | RBitSetRx. getByte() setByte() incrementAndGetByte()  getShort() setShort() incrementAndGetShort()  getInteger() setInteger() incrementAndGetInteger()  getLong() setLong() incrementAndGetLong() |
| BLMPOP | RBlockingQueue. pollLastFromAny() pollFirstFromAny() pollLastFromAnyAsync() pollFirstFromAnyAsync() | RBlockingQueueReactive. pollLastFromAny() pollFirstFromAny() | RBlockingQueueRx. pollLastFromAny() pollFirstFromAny() |
| BLPOP | RBlockingQueue. take() poll() pollFromAny() takeAsync() pollAsync() pollFromAnyAsync() | RBlockingQueueReactive. take() poll() pollFromAny() | RBlockingQueueRx. take() poll() pollFromAny() |
| BLMOVE | RBlockingDeque. move() moveAsync() | RBlockingDequeReactive. move() | RBlockingDequeRx. move() |
| BRPOP | RBlockingDeque. takeLast() takeLastAsync() | RBlockingDequeReactive. takeLast() | RBlockingDequeRx. takeLast() |
| BRPOPLPUSH | RBlockingQueue. pollLastAndOfferFirstTo() pollLastAndOfferFirstToAsync() | RBlockingQueueReactive. pollLastAndOfferFirstTo() | RBlockingQueueRx. pollLastAndOfferFirstTo() |
| CONFIG GET | RedisNode. setConfig() setConfigAsync() | - | - |
| CONFIG SET | RedisNode. getConfig() getConfigAsync() | - | - |
| COPY | RObject. copy() copyAsync() | RObjectReactive. copy() | RObjectRx. copy() |
| CLIENT SETNAME | Config.setClientName() | - | - |
| CLIENT REPLY | BatchOptions.skipResult() | - | - |
| CLUSTER INFO | ClusterNode.info() | - | - |
| CLUSTER KEYSLOT | RKeys. getSlot() getSlotAsync() | RKeysReactive. getSlot() | RKeysRx. getSlot() |
| CLUSTER NODES | Used in ClusterConnectionManager |  |  |
| DECRBY | RAtomicLong. addAndGet() addAndGetAsync() | RAtomicLongReactive. addAndGet() | RAtomicLongRx. addAndGet() |
| DUMP | RObject. dump() dumpAsync() | RObjectReactive. dump() | RObjectRx. dump() |
| DBSIZE | RKeys. count() countAsync() | RKeysReactive. count() | RKeysRx.count() |
| DECR | RAtomicLong. decrementAndGet() decrementAndGetAsync() | RAtomicLongReactive. decrementAndGet() | RAtomicLongRx. decrementAndGet() |
| DEL | RObject. delete() deleteAsync()  RKeys. delete() deleteAsync() | RObjectReactive. delete()  RKeysReactive. delete() | RObjectRx. delete()  RKeysRx. delete() |
| STRLEN | RBucket. size() sizeAsync() | RBucketReactive. size() | RBucketRx. size() |
| EVAL | RScript. eval() evalAsync() | RScriptReactive. eval() | RScriptRx. eval() |
| EVALSHA | RScript. evalSha() evalShaAsync() | RScriptReactive. evalSha() | RScriptRx. evalSha() |
| EXEC | RBatch. execute() executeAsync() | RBatchReactive. execute() | RBatchRx. execute() |
| EXISTS | RObject. isExists() isExistsAsync() | RObjectReactive. isExists() | RObjectRx. isExists() |
| FLUSHALL | RKeys. flushall() flushallAsync() | RKeysReactive. flushall() | RKeysRx. flushall() |
| FLUSHDB | RKeys. flushdb() flushdbAsync() | RKeysReactive. flushdb() | RKeysRx. flushdb() |
| GETRANGE | RBinaryStream. getChannel().read() | RBinaryStreamReactive. read() | RBinaryStreamRx. read() |
| GEOADD | RGeo. add() addAsync() | RGeoReactive. add() | RGeoRx. add() |
| GEODIST | RGeo. dist() distAsync() | RGeoReactive. dist() | RGeoRx. dist() |
| GEOHASH | RGeo. hash() hashAsync() | RGeoReactive. hash() | RGeoRx. hash() |
| GEOPOS | RGeo. pos() posAsync() | RGeoReactive. pos() | RGeoRx. pos() |
| GEORADIUS | RGeo. radius() radiusAsync() radiusWithDistance() radiusWithDistanceAsync() radiusWithPosition() radiusWithPositionAsync() | RGeoReactive. radius() radiusWithDistance() radiusWithPosition() | RGeoRx. radius() radiusWithDistance() radiusWithPosition() |
| GEOSEARCH | RGeo. search() searchAsync() | RGeoReactive. search() | RGeoRx. search() |
| GEOSEARCHSTORE | RGeo. storeSearchTo() storeSearchToAsync() | RGeoReactive. storeSearchTo() | RGeoRx. storeSearchTo() |
| GET | RBucket. get() getAsync()  RBinaryStream. get() getAsync() | RBucketReactive. get()  RBinaryStreamReactive. get() | RBucketRx. get()  RBinaryStreamRx. get() |
| GETEX | RBucket. getAndExpire() getAndExpireAsync() getAndClearExpire() getAndClearExpireAsync() | RBucketReactive. getAndExpire() getAndClearExpire() | RBucketRx. getAndExpire() getAndClearExpire() |
| GETBIT | RBitSet. get() getAsync() | RBitSetReactive. get() | RBitSetRx. get() |
| GETSET | RBucket. getAndSet() getAndSetAsync()  RAtomicLong. getAndSet() getAndSetAsync()  RAtomicDouble. getAndSet() getAndSetAsync() | RBucketReactive. getAndSet()  RAtomicLongReactive. getAndSet()  RAtomicDoubleReactive. getAndSet() | RBucketRx. getAndSet()  RAtomicLongRx. getAndSet()  RAtomicDoubleRx. getAndSet() |
| HDEL | RMap. fastRemove() fastRemoveAsync() | RMapReactive. fastRemove() | RMapRx. fastRemove() |
| HEXISTS | RMap. containsKey() containsKeyAsync() | RMapReactive. containsKey() | RMapRx. containsKey() |
| HGET | RMap. get() getAsync() | RMapReactive. get() | RMapRx. get() |
| HSTRLEN | RMap. valueSize() valueSizeAsync() | RMapReactive. valueSize() | RMapRx. valueSize() |
| HGETALL | RMap. readAllEntrySet() readAllEntrySetAsync() | RMapReactive. readAllEntrySet() | RMapRx. readAllEntrySet() |
| HINCRBY | RMap. addAndGet() addAndGetAsync() | RMapReactive. addAndGet() | RMapRx. addAndGet() |
| HINCRBYFLOAT | RMap. addAndGet() addAndGetAsync() | RMapReactive. addAndGet() | RMapRx. addAndGet() |
| HKEYS | RMap. readAllKeySet() readAllKeySetAsync() | RMapReactive. readAllKeySet() | RMapRx. readAllKeySet() |
| HLEN | RMap. size() sizeAsync() | RMapReactive. size() | RMapRx. size() |
| HMGET | RMap. getAll() getAllAsync() | RMapReactive. getAll() | RMapRx. getAll() |
| HMSET | RMap. putAll() putAllAsync() | RMapReactive. putAll() | RMapRx. putAll() |
| HRANDFIELD | RMap. putAll() putAllAsync() | RMapReactive. putAll() | RMapRx. putAll() |
| HSCAN | RMap. keySet().iterator() values().iterator() entrySet().iterator() | RMapReactive. keyIterator() valueIterator() entryIterator() | RMapRx. keyIterator() valueIterator() entryIterator() |
| HSET | RMap. randomEntries() randomKeys() randomEntriesAsync() randomKeysAsync() | RMapReactive. randomEntries() randomKeys() | RMapRx. randomEntries() randomKeys() |
| HSETNX | RMap. fastPutIfAbsent() fastPutIfAbsentAsync() | RMapReactive. fastPutIfAbsent() | RMapRx. fastPutIfAbsent() |
| HVALS | RMap. readAllValues() readAllValuesAsync() | RMapReactive. readAllValues() | RMapRx. readAllValues() |
| INCR | RAtomicLong. incrementAndGet() incrementAndGetAsync() | RAtomicLongReactive. incrementAndGet() | RAtomicLongRx. incrementAndGet() |
| INCRBY | RAtomicLong. addAndGet() addAndGetAsync() | RAtomicLongReactive. addAndGet() | RAtomicLongRx. addAndGet() |
| INCRBYFLOAT | RAtomicDouble. addAndGet() addAndGetAsync() | RAtomicDoubleReactive. addAndGet() | RAtomicDoubleRx. addAndGet() |
| JSON.ARRAPPEND | RJsonBucket. arrayAppend() arrayAppendAsync() | RJsonBucketReactive. arrayAppend() | RJsonBucketRx. arrayAppend() |
| JSON.ARRINDEX | RJsonBucket. arrayIndex() arrayIndexAsync() | RJsonBucketReactive. arrayIndex() | RJsonBucketRx. arrayIndex() |
| JSON.ARRINSERT | RJsonBucket. arrayInsert() arrayInsertAsync() | RJsonBucketReactive. arrayInsert() | RJsonBucketRx. arrayInsert() |
| JSON.ARRLEN | RJsonBucket. arraySize() arraySizeAsync() | RJsonBucketReactive. arraySize() | RJsonBucketRx. arraySize() |
| JSON.ARRPOP | RJsonBucket. arrayPollLast() arrayPollFirst() arrayPop() arrayPollLastAsync() arrayPollFirstAsync() arrayPopAsync() | RJsonBucketReactive. arrayPollLast() arrayPollFirst() arrayPop() | RJsonBucketRx. arrayPollLast() arrayPollFirst() arrayPop() |
| JSON.ARRTRIM | RJsonBucket. arrayTrim() arrayTrimAsync() | RJsonBucketReactive. arrayTrim() | RJsonBucketRx. arrayTrim() |
| JSON.CLEAR | RJsonBucket. clear() clearAsync() | RJsonBucketReactive. clear() | RJsonBucketRx. clear() |
| JSON.GET | RJsonBucket. get() getAsync() | RJsonBucketReactive. get() | RJsonBucketRx. get() |
| JSON.NUMINCRBY | RJsonBucket. incrementAndGet() incrementAndGetAsync() | RJsonBucketReactive. incrementAndGet() | RJsonBucketRx. incrementAndGet() |
| JSON.OBJLEN | RJsonBucket. countKeys() countKeysAsync() | RJsonBucketReactive. countKeys() | RJsonBucketRx. countKeys() |
| JSON.OBJKEYS | RJsonBucket. getKeys() getKeysAsync() | RJsonBucketReactive. getKeys() | RJsonBucketRx. getKeys() |
| JSON.SET | RJsonBucket. set() setAsync() | RJsonBucketReactive. set() | RJsonBucketRx. set() |
| JSON.STRAPPEND | RJsonBucket. stringAppend() stringAppendAsync() | RJsonBucketReactive. stringAppend() | RJsonBucketRx. stringAppend() |
| JSON.STRLEN | RJsonBucket. stringSize() stringSizeAsync() | RJsonBucketReactive. stringSize() | RJsonBucketRx. stringSize() |
| JSON.TOGGLE | RJsonBucket. toggle() toggleAsync() | RJsonBucketReactive. toggle() | RJsonBucketRx. toggle() |
| JSON.TYPE | RJsonBucket. type() typeAsync() | RJsonBucketReactive. type() | RJsonBucketRx. type() |
| KEYS | RKeys. getKeysByPattern() getKeysByPatternAsync() | RKeysReactive. getKeysByPattern() | RKeysRx. getKeysByPattern() |
| LINDEX | RList. get() getAsync() | RListReactive. get() | RListRx. get() |
| LLEN | RList. size() sizeAsync() | RListReactive. size() | RListRx. size() |
| LMOVE | RDeque. move() moveAsync() | RDequeReactive. move() | RDequeRx. move() |
| LPOP | RQueue. poll() pollAsync() | RQueueReactive. poll() | RQueueRx. poll() |
| LPUSH | RDeque. addFirst() addFirstAsync() | RDequeReactive. addFirst() |  |
| LRANGE | RList. readAll() readAllAsync() | RListReactive.readAll() | RListRx.readAll() |
| LPUSHX | RDeque. addFirstIfExists() addFirstIfExistsAsync() | RDequeReactive. addFirstIfExists() | RDequeRx. addFirstIfExists() |
| LREM | RList. fastRemove() fastRemoveAsync() | RListReactive. fastRemove() | RListRx. fastRemove() |
| LSET | RList. fastSet() fastSetAsync() | RListReactive. fastSet() | RListRx. fastSet() |
| LTRIM | RList. trim() trimAsync() | RListReactive. trim() | RListRx. trim() |
| LINSERT | RList. addBefore() addAfter() addBeforeAsync() addAfterAsync() | RListReactive. addBefore() addAfter() | RListRx. addBefore() addAfter() |
| MULTI | RBatch. execute() executeAsync() | RBatchReactive. execute() | RBatchRx. execute() |
| MGET | RBuckets. get() getAsync() | RBucketsReactive. get() | RBucketsRx. get() |
| MSETNX | RBuckets. trySet() trySetAsync() | RBucketsReactive. trySet() | RBucketsRx. trySet() |
| MIGRATE | RObject. migrate() migrateAsync() | RObjectReactive. migrate() | RObjectRx. migrate() |
| MOVE | RObject. move() moveAsync() | RObjectReactive. move() | RObjectRx. move() |
| MSET | RBuckets. set() setAsync() | RBucketsReactive. set() | RBucketsRx. set() |
| PERSIST | RExpirable. clearExpire() clearExpireAsync() | RExpirableReactive. clearExpire() | RExpirableRx. clearExpire() |
| PEXPIRE | RExpirable. expire() expireAsync() | RExpirableReactive. expire() | RExpirableRx. expire() |
| PEXPIREAT | RExpirable. expireAt() expireAtAsync() | RExpirableReactive. expireAt() | RExpirableRx. expireAt() |
| PEXPIRETIME | RExpirable. expireTime() expireTimeAsync() | RExpirableReactive. expireTime() | RExpirableRx. expireTime() |
| PFADD | RHyperLogLog. add() addAsync() addAll() addAllAsync() | RHyperLogLogReactive. add()  addAll() | RHyperLogLogRx. add()  addAll() |
| PFCOUNT | RHyperLogLog. count() countAsync() countWith() countWithAsync() | RHyperLogLogReactive. count() countWith() | RHyperLogLogRx. count() countWith() |
| PFMERGE | RHyperLogLog. mergeWith() mergeWithAsync() | RHyperLogLogReactive. mergeWith() | RHyperLogLogRx. mergeWith() |
| PING | Node.ping() NodesGroup.pingAll() | - | - |
| PSUBSCRIBE | RPatternTopic. addListener() | RPatternTopicReactive. addListener() | RPatternTopicRx. addListener() |
| PSETEX | RBucket. set() setAsync() | RBucketReactive. set() | RBucketRx. set() |
| PTTL | RExpirable. remainTimeToLive() remainTimeToLiveAsync() | RExpirableReactive. remainTimeToLive() | RExpirableRx. remainTimeToLive() |
| PUBLISH | RTopic. publish() | RTopicReactive. publish() | RTopicRx. publish() |
| PUBSUB NUMSUB | RTopic. countSubscribers() countSubscribersAsync() | RTopicReactive. countSubscribers() | RTopicRx. countSubscribers() |
| PUNSUBSCRIBE | RPatternTopic. removeListener() | RPatternTopicReactive. removeListener() | RPatternTopicRx. removeListener() |
| RANDOMKEY | RKeys. randomKey() randomKeyAsync() | RKeysReactive. randomKey() | RKeysRx. randomKey() |
| RESTORE | RObject. restore() restoreAsync() | RObjectReactive. restore() | RObjectRx. restore() |
| RENAME | RObject. rename() renameAsync() | RObjectReactive. rename() | RObjectRx. rename() |
| RPOP | RDeque. pollLast() removeLast() pollLastAsync() removeLastAsync() | RDequeReactive. pollLast() removeLast() | RDequeRx. pollLast() removeLast() |
| RPOPLPUSH | RDeque. pollLastAndOfferFirstTo() pollLastAndOfferFirstToAsync() | RDequeReactive. pollLastAndOfferFirstTo() | RDequeRx. pollLastAndOfferFirstTo() |
| RPUSH | RList. add() addAsync() | RListReactive. add() | RListRx. add() |
| RPUSHX | RDeque. addLastIfExists() addLastIfExistsAsync() | RListReactive. addLastIfExists() | RListRx. addLastIfExists() |
| SADD | RSet. add() addAsync() | RSetReactive. add() | RSetRx. add() |
| SETRANGE | RBinaryStream. getChannel().write() | RBinaryStreamReactive. write() | RBinaryStreamRx. write() |
| SCAN | RKeys. getKeys() | RKeysReactive. getKeys() | RKeysRx. getKeys() |
| SCARD | RSet. size() sizeAsync() | RSetReactive. size() | RSetRx. size() |
| SCRIPT EXISTS | RScript. scriptExists() scriptExistsAsync() | RScriptReactive. scriptExists() | RScriptRx. scriptExists() |
| SCRIPT FLUSH | RScript. scriptFlush() scriptFlushAsync() | RScriptReactive. scriptFlush() | RScriptRx. scriptFlush() |
| SCRIPT KILL | RScript. scriptKill() scriptKillAsync() | RScriptReactive. scriptKill() | RScriptRx. scriptKill() |
| SCRIPT LOAD | RScript. scriptLoad() scriptLoadAsync() | RScriptReactive. scriptLoad() | RScriptRx. scriptLoad() |
| SDIFFSTORE | RSet. diff() diffAsync() | RSetReactive. diff() | RSetRx. diff() |
| SDIFF | RSet. readDiff() readDiffAsync() | RSetReactive. readDiff() | RSetRx. readDiff() |
| SRANDMEMBER | RSet. random() randomAsync() | RSetReactive. random() | RSetRx. random() |
| SELECT | Config.setDatabase() | - | - |
| SET | RBucket. set() setAsync() | RBucketReactive. set() | RBucketRx. set() |
| SETBIT | RBitSet. set() clear() setAsync() clearAsync() | RBitSetReactive. set() clear() | RBitSetRx. set() clear() |
| SETEX | RBucket. set() setAsync() | RBucketReactive. set() | RBucketRx. set() |
| SETNX | RBucket. setIfAbsent() setIfAbsentAsync() | RBucketReactive. setIfAbsent() | RBucketRx. setIfAbsent() |
| SISMEMBER | RSet. contains() containsAsync() | RSetReactive. contains() | RSetRx. contains() |
| SINTERSTORE | RSet. intersection() intersectionAsync() | RSetReactive. intersection() | RSetRx. intersection() |
| SINTER | RSet. readIntersection() readIntersectionAsync() | RSetReactive. readIntersection() | RSetRx. readIntersection() |
| SMEMBERS | RSet. readAll() readAllAsync() | RSetReactive. readAll() | RSetRx. readAll() |
| SMOVE | RSet. move() moveAsync() | RSetReactive. move() | RSetRx. move() |
| SORT | RList. readSort() sortTo() readSortAsync() sortToAsync() | RListReactive. readSort() sortTo() | RListRx. readSort() sortTo() |
| SPOP | RSet. removeRandom() removeRandomAsync() | RSetReactive. removeRandom() | RSetRx. removeRandom() |
| SREM | RSet. remove() removeAsync() | RSetReactive. remove() | RSetRx. remove() |
| SSCAN | RSet. iterator() | RSetReactive. iterator() | RSetRx. iterator() |
| SUBSCRIBE | RTopic. addListener() | RTopicReactive. addListener() | RTopicRx. addListener() |
| SUNION | RSet. readUnion() readUnionAsync() | RSetReactive. readUnion() | RSetRx. readUnion() |
| SUNIONSTORE | RSet. union() unionAsync() | RSetReactive. union() | RSetRx. union() |
| SWAPDB | RKeys. swapdb() swapdbAsync() | RKeysReactive. swapdb() | RKeysRx. swapdb() |
| TTL | RExpirable. remainTimeToLive() remainTimeToLiveAsync() | RExpirableReactive. remainTimeToLive() | RExpirableRx. remainTimeToLive() |
| TYPE | RKeys. getType() getTypeAsync() | RKeysReactive. getType() | RKeysRx. getType() |
| TOUCH | RObject. touch() touchAsync() | RObjectReactive. touch() | RObjectRx. touch() |
| UNSUBSCRIBE | RTopic. removeListener() | RTopicReactive. removeListener() | RTopicRx. removeListener() |
| UNLINK | RObject. unlink() unlinkAsync() | RObjectReactive. unlink() | RObjectRx. unlink() |
| WAIT | BatchOptions. sync() | BatchOptions. sync() | BatchOptions. sync() |
| WAITAOF | BatchOptions. syncAOF() | BatchOptions. syncAOF() | BatchOptions. syncAOF() |
| ZADD | RScoredSortedSet. add() addAsync() addAll() addAllAsync() | RScoredSortedSetReactive. add() addAll() | RScoredSortedSetRx. add() addAll() |
| ZCARD | RScoredSortedSet. size() sizeAsync() | RScoredSortedSetReactive. size() | RScoredSortedSetRx. size() |
| ZCOUNT | RScoredSortedSet. count() countAsync() | RScoredSortedSetReactive. count() | RScoredSortedSetRx. count() |
| ZDIFF | RScoredSortedSet. readDiff() readDiffAsync() | RScoredSortedSetReactive. readDiff() | RScoredSortedSetRx. readDiff() |
| ZDIFFSTORE | RScoredSortedSet. diff() diffAsync() | RScoredSortedSetReactive. diff() | RScoredSortedSetRx. diff() |
| ZINCRBY | RScoredSortedSet. addScore() addScoreAsync() | RScoredSortedSetReactive. addScore() | RScoredSortedSetRx. addScore() |
| ZINTER | RScoredSortedSet. readIntersection() readIntersectionAsync() | RScoredSortedSetReactive. readIntersection() | RScoredSortedSetRx. readIntersection() |
| ZREMRANGEBYRANK | RScoredSortedSet. removeRangeByRank() removeRangeByRankAsync() | RScoredSortedSetReactive. removeRangeByRank() | RScoredSortedSetRx. removeRangeByRank() |
| ZREVRANGEBYLEX | RLexSortedSet. rangeReversed() rangeReversedAsync() | RLexSortedSetReactive. rangeReversed() | RLexSortedSetSetRx. rangeReversed() |
| ZLEXCOUNT | RLexSortedSet. lexCount() lexCountHead() lexCountTail() lexCountAsync() lexCountHeadAsync() lexCountTailAsync() | RLexSortedSetReactive. lexCount() lexCountHead() lexCountTail() | RLexSortedSetRx. lexCount() lexCountHead() lexCountTail() |
| ZRANGE | RScoredSortedSet. valueRange() valueRangeAsync() | RScoredSortedSetReactive. valueRange() | RScoredSortedSetRx. valueRange() |
| ZRANDMEMBER | RScoredSortedSet. random() randomAsync() | RScoredSortedSetReactive. random() | RScoredSortedSetRx. random() |
| ZREVRANGE | RScoredSortedSet. valueRangeReversed() valueRangeReversedAsync() | RScoredSortedSetReactive. valueRangeReversed() | RScoredSortedSetRx. valueRangeReversed() |
| ZUNION | RScoredSortedSet. readUnion() readUnionAsync() | RScoredSortedSetReactive. readUnion() | RScoredSortedSetRx. readUnion() |
| ZUNIONSTORE | RScoredSortedSet. union() unionAsync() | RScoredSortedSetReactive. union() | RScoredSortedSetRx. union() |
| ZINTERSTORE | RScoredSortedSet. intersection() intersectionAsync() | RScoredSortedSetReactive. intersection() | RScoredSortedSetRx. intersection() |
| ZPOPMAX | RScoredSortedSet. pollLast() pollLastAsync() | RScoredSortedSetReactive. pollLast() | RScoredSortedSetRx. pollLast() |
| ZPOPMIN | RScoredSortedSet. pollFirst() pollFirstAsync() | RScoredSortedSetReactive. pollFirst() | RScoredSortedSetRx. pollFirst() |
| ZMPOP | RScoredSortedSet. pollFirstEntriesFromAny() pollFirstEntriesFromAnyAsync() pollLastFromAny() pollLastFromAnyAsync() pollFirstFromAny() pollFirstFromAnyAsync() | RScoredSortedSetReactive. pollFirstEntriesFromAny() pollLastFromAny() pollFirstFromAny() | RScoredSortedSetRx. pollFirstEntriesFromAny() pollLastFromAny() pollFirstFromAny() |
| ZRANGEBYLEX | RLexSortedSet. range() rangeHead() rangeTail() rangeAsync() rangeHeadAsync() rangeTailAsync() | RLexSortedSetReactive. range() rangeHead() rangeTail() | RLexSortedSetRx. range() rangeHead() rangeTail() |
| ZRANGEBYSCORE | RScoredSortedSet. valueRange() entryRange() valueRangeAsync() entryRangeAsync() | RScoredSortedSetReactive. valueRange() entryRange() | RScoredSortedSetRx. valueRange() entryRange() |
| TIME | RedissonClient. getNodesGroup(). getNode().time() getClusterNodesGroup(). getNode().time() | - | - |
| ZRANK | RScoredSortedSet. rank() rankAsync() | RScoredSortedSetReactive. rank() | RScoredSortedSetRx. rank() |
| ZREM | RScoredSortedSet. remove() removeAll() removeAsync() removeAllAsync() | RScoredSortedSetReactive. remove() removeAll() | RScoredSortedSetRx. remove() removeAll() |
| ZREMRANGEBYLEX | RLexSortedSet. removeRange() removeRangeHead() removeRangeTail() removeRangeAsync() removeRangeHeadAsync() removeRangeTailAsync() | RLexSortedSetReactive. removeRange() removeRangeHead() removeRangeTail() | RLexSortedSetRx. removeRange() removeRangeHead() removeRangeTail() |
| ZREMRANGEBYSCORE | RScoredSortedSet. removeRangeByScore() removeRangeByScoreAsync() | RScoredSortedSetReactive. removeRangeByScore() | RScoredSortedSetRx. removeRangeByScore() |
| ZREVRANGEBYSCORE | RScoredSortedSet. valueRangeReversed() entryRangeReversed() valueRangeReversedAsync() entryRangeReversedAsync() | RScoredSortedSetReactive. entryRangeReversed() valueRangeReversed() | RScoredSortedSetRx. entryRangeReversed() valueRangeReversed() |
| ZREVRANK | RScoredSortedSet. revRank() revRankAsync() | RScoredSortedSetReactive. revRank() | RScoredSortedSetRx. revRank() |
| ZSCORE | RScoredSortedSet. getScore() getScoreAsync() | RScoredSortedSetReactive. getScore() | RScoredSortedSetRx. getScore() |
| XACK | RStream. ack() ackAsync() | RStreamReactive. ack() | RStreamRx. ack() |
| XADD | RStream. add() addAsync() | RStreamReactive. add() | RStreamRx. add() |
| XAUTOCLAIM | RStream. autoClaim() autoClaimAsync() | RStreamReactive. autoClaim() | RStreamRx. autoClaim() |
| XCLAIM | RStream. claim() claimAsync() | RStreamReactive. claim() | RStreamRx. claim() |
| XDEL | RStream. remove() removeAsync() | RStreamReactive. remove() | RStreamRx. remove() |
| XGROUP | RStream. createGroup() removeGroup() updateGroup() createGroupAsync() removeGroupAsync() updateGroupAsync() | RStreamReactive. createGroup() removeGroup() updateGroup() | RStreamRx. createGroup() removeGroup() updateGroup() |
| XINFO | RStream. getInfo() listGroups() listConsumers() getInfoAsync() listGroupsAsync() listConsumersAsync() | RStreamReactive. getInfo() listGroups() listConsumers() | RStreamRx. getInfo() listGroups() listConsumers() |
| XLEN | RStream. size() sizeAsync() | RStreamReactive. size() | RStreamRx. size() |
| XPENDING | RStream. listPending() listPendingAsync() | RStreamReactive. listPending() | RStreamRx. listPending() |
| XRANGE | RStream. range() rangeAsync() | RStreamReactive. range() | RStreamRx. range() |
| XREAD | RStream. read() readAsync() | RStreamReactive. read() | RStreamRx. read() |
| XREADGROUP | RStream. readGroup() readGroupAsync() | RStreamReactive. readGroup() | RStreamRx. readGroup() |
| XREVRANGE | RStream. rangeReversed() rangeReversedAsync() | RStreamReactive. rangeReversed() | RStreamRx. rangeReversed() |
| XTRIM | RStream. trim() trimAsync() | RStreamReactive. trim() | RStreamRx. trim() |

# Standalone node
## 介绍
Redisson 提供作为独立节点运行并参与分布式计算的能力。此类节点用于运行 `MapReduce`、`ExecutorService`、`ScheduledExecutorService` 任务或 `RemoteService` 服务。所有任务都保存在 Redis 中，直到执行为止。 
打包为单个 jar 并可以[下载](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=org.redisson&a=redisson-all&v=LATEST&e=jar)。
![](https://camo.githubusercontent.com/2a2006bcb09a209f6d30ab71c6cb908ccdc5b664536696eaf8b5010a5ed501cf/68747470733a2f2f7265646973736f6e2e6f72672f6172636869746563747572652e706e67#from=url&id=bVAq8&originHeight=1236&originWidth=1698&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
## Configuration
### 设置
Redisson node 使用与 Redisson 框架相同的配置以及附加设置。通过线程设置设置 ExecutorService 可用的线程数量。
#### mapReduceWorkers
默认值：`0`
MapReduce worker的数量。`0 = current_processors_amount`
#### executorServiceWorkers
默认值：`null`
映射时，键作为服务名称，值作为 worker 数量。
#### redissonNodeInitializer
默认值：`null`
在 Redisson 节点启动期间运行的侦听器。
#### beanFactory
默认值：`null`
定义 Spring Bean Factory 实例以使用 Spring 的“@Autowired”、“@Value”或 JSR-330 的“@Inject”注释执行任务。
请参阅 `ExecutorService` 和 `ScheduledExecutorService` 的文档
### YAML 配置格式
以下是集群模式的配置示例，并附加了 YAML 格式的 Redisson 节点设置。
```java
---
clusterServersConfig:
  nodeAddresses:
  - "//127.0.0.1:7004"
  - "//127.0.0.1:7001"
  - "//127.0.0.1:7000"
  scanInterval: 1000
threads: 0

executorServiceWorkers:
  myService1: 123
  myService2: 421
redissonNodeInitializer: !<org.mycompany.MyRedissonNodeInitializer> {}
```
## 初始化阶段的侦听器
Redisson 节点允许在启动期间通过 `RedissonNodeInitializer` 侦听器执行初始化逻辑。例如，它允许注册您的远程服务实现（该实现应在 Redisson 节点的类路径中可用），或执行其他有用的逻辑。例如，通知所有订阅者新的 Redisson 节点可用。
```java
public class MyRedissonNodeInitializer implements RedissonNodeInitializer {

    @Override
    public void onStartup(RedissonNode redissonNode) {
        RMap<String, Integer> map = redissonNode.getRedisson().getMap("myMap");
        // ...
        // or
        redisson.getRemoteService("myRemoteService").register(MyRemoteService.class, new MyRemoteServiceImpl(...));
        // or
        reidsson.getTopic("myNotificationTopic").publish("New node has joined. id:" + redissonNode.getId() + " remote-server:" + redissonNode.getRemoteAddress());
    }

}
```
## 如何作为嵌入式节点运行
Redisson 节点可以嵌入到您的应用程序中：
```java
// Redisson config
Config config = ...
// Redisson Node config
RedissonNodeConfig nodeConfig = new RedissonNodeConfig(config);
Map<String, Integer> workers = new HashMap<String, Integer>();
workers.put("test", 1);
nodeConfig.setExecutorServiceWorkers(workers);

// create Redisson node
RedissonNode node = RedissonNode.create(nodeConfig);
// or create Redisson node with existing Redisson instance
RedissonNode node = RedissonNode.create(nodeConfig, redisson);

node.start();

//...

node.shutdown();
```
## 如何从命令行运行

1. 下载 [Redisson node 的jar包](https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=org.redisson&a=redisson-all&v=LATEST&e=jar)
2. 创建 yaml 配置文件
3. 使用下面的命令行运行

`java -jar redisson-all.jar config.yaml`
不要忘了加上`-Xmx`和`-Xms`参数。
## 如何在docker中运行
### 有 Redis 实例

1. 运行 Redis

`docker run -d --name redis-node redis`

2. 运行 Redisson node

`docker run -d --network container:redis-node -e JAVA_OPTS="<java-opts>" -v <path-to-config>:/opt/redisson-node/redisson.conf redisson/redisson-node`
`<path-to-config>` - Redisson node 的 YAML 配置文件的路径
`<java-opts>` - JVM 参数
### 没有 Redis 实例

1. 运行 Redisson node

`docker run -d -e JAVA_OPTS="<java-opts>" -v <path-to-config>:/opt/redisson-node/redisson.conf redisson/redisson-node`
`<path-to-config>` - Redisson node 的 YAML 配置文件的路径
`<java-opts>` - JVM 参数
# Tools 工具
## 集群管理工具
集群管理工具允许像 `redis-trib.rb` 脚本一样轻松地管理 Redis 集群节点。
### 创建 Redis 集群
下面是如何创建具有 3 个主服务器和 3 个从服务器的集群的示例。
```java
ClusterNodes clusterNodes = ClusterNodes.create()
.master("127.0.0.1:7000").withSlaves("127.0.0.1:7001", "127.0.0.1:7002")
.master("127.0.0.1:7003").withSlaves("127.0.0.1:7004")
.master("127.0.0.1:7005");
ClusterManagementTool.createCluster(clusterNodes);
```
`127.0.0.1:7000` 有两个从节点 `127.0.0.1:7001`, `127.0.0.1:7002`
`127.0.0.1:7003` 有一个从节点 `127.0.0.1:7004`
`127.0.0.1:7005` 没有从节点
### 删除 Redis 节点
下面是如何从集群中删除节点的示例。 
```java
ClusterManagementTool.removeNode("127.0.0.1:7000", "127.0.0.1:7002");
// or
redisson.getClusterNodesGroup().removeNode("127.0.0.1:7002");
```
从 `127.0.0.1:7000` 参与的集群中删除 `127.0.0.1:7002`。
### 在 Redis 节点之间移动槽
以下是如何在集群主节点之间移动插槽的示例。 
```java
ClusterManagementTool.moveSlots("127.0.0.1:7000", "127.0.0.1:7002", 23, 419, 4712, 8490);
// or
redisson.getClusterNodesGroup().moveSlots("127.0.0.1:7000", "127.0.0.1:7002", 23, 419, 4712, 8490);
```
将插槽 23、419、4712、8490 从 127.0.0.1:7002 移动到 127.0.0.1:7000 节点 
下面是如何在集群主节点之间移动插槽范围的示例。 
```java
ClusterManagementTool.moveSlotsRange("127.0.0.1:7000", "127.0.0.1:7002", 51, 9811);
// or
redisson.getClusterNodesGroup().moveSlotsRange("127.0.0.1:7000", "127.0.0.1:7002", 51, 9811);
```
将插槽范围 [51, 9811] 从 127.0.0.1:7002 移动到 127.0.0.1:7000 节点。
### 添加从节点
以下是如何将从节点添加到集群的示例。 
```java
ClusterManagementTool.addSlaveNode("127.0.0.1:7000", "127.0.0.1:7003");
// or
redisson.getClusterNodesGroup().addSlaveNode("127.0.0.1:7003");
```
将从节点 127.0.0.1:7003 添加到 127.0.0.1:7000 参与的集群。
### 添加主节点
以下是如何将主节点添加到集群的示例。 
```java
ClusterManagementTool.addMasterNode("127.0.0.1:7000", "127.0.0.1:7004");
// or
redisson.getClusterNodesGroup().addMasterNode("127.0.0.1:7004");
```
将主节点 127.0.0.1:7004 添加到 127.0.0.1:7000 参与的集群。
_此功能仅在 Redisson PRO 版本中可用。_
# Integration with frameworks 与其他框架集成
## Spring Framework
## Spring Cache
Redisson 提供了根据 Spring Cache 规范制作的基于 Redis 的 Spring Cache 实现。每个Cache实例都有两个重要的参数：`ttl`和`maxIdleTime`。如果这些设置未定义或等于 0，则数据将无限存储。
配置示例：
```java
    @Configuration
    @ComponentScan
    @EnableCaching
    public static class Application {

        @Bean(destroyMethod="shutdown")
        RedissonClient redisson() throws IOException {
            Config config = new Config();
            config.useClusterServers()
                  .addNodeAddress("redis://127.0.0.1:7004", "redis://127.0.0.1:7001");
            return Redisson.create(config);
        }

        @Bean
        CacheManager cacheManager(RedissonClient redissonClient) {
            Map<String, CacheConfig> config = new HashMap<String, CacheConfig>();

            // create "testMap" cache with ttl = 24 minutes and maxIdleTime = 12 minutes
            config.put("testMap", new CacheConfig(24*60*1000, 12*60*1000));
            return new RedissonSpringCacheManager(redissonClient, config);
        }

    }
```
缓存配置可以从 YAML 配置文件中读取：
```java
    @Configuration
    @ComponentScan
    @EnableCaching
    public static class Application {

        @Bean(destroyMethod="shutdown")
        RedissonClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
            Config config = Config.fromYAML(configFile.getInputStream());
            return Redisson.create(config);
        }

        @Bean
        CacheManager cacheManager(RedissonClient redissonClient) throws IOException {
            return new RedissonSpringCacheManager(redissonClient, "classpath:/cache-config.yaml");
        }

    }
```
### Local Cache and data partitioning
Redisson 为各种 Spring Cache 管理器提供了两个重要功能：
**本地缓存** - 所谓的近缓存，用于加速读取操作并避免网络往返。它在 Redisson 端缓存 Spring Cache 条目 (entry) ，并且执行读取操作的速度比常见实现快 45 倍。具有相同名称的本地缓存实例连接到相同的发布/订阅通道。该通道用于在所有实例之间交换更新/无效事件。本地缓存存储不使用关键对象的 hashCode()/equals() 方法，而是使用序列化状态的哈希值。 
**数据分区** - 尽管 Spring Cache 实例与集群兼容，但其内容不会跨集群中的多个 Redis 主节点进行扩展/分区。数据分区允许扩展 Redis 集群中单个 Spring Cache 实例的可用内存、读/写操作和条目驱逐过程。 
**数据淘汰** - 允许定义每个Map条目的生存时间或最大空闲时间。 Redis 哈希结构不支持驱逐，因此它是在 Redisson 端通过自定义计划任务来完成的，该任务会删除过期的条目。驱逐任务在创建缓存实例时启动。如果未使用缓存实例并且条目已过期，则应创建该实例以启动逐出过程。这会导致每个唯一的缓存实例名称需要额外的 Redis 调用和驱逐任务。 
**高级条目驱逐** - 条目驱逐过程的改进版本。不使用条目驱逐任务。 以下是可用管理器的完整列表：

| Class name | Local cache | Data partitioning | Entry eviction | Advanced entry eviction | Ultra-fast read/write |
| --- | --- | --- | --- | --- | --- |
| RedissonSpringCacheManager _open-source version_ | ❌ | ❌ | ✔️ | ❌ | ❌ |
| RedissonSpringCacheManager [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ❌ | ✔️ | ❌ | ✔️ |
| RedissonSpringCacheV2Manager _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ❌ | ✔️ | ✔️ |
| RedissonSpringLocalCachedCacheManager _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ❌ | ✔️ | ❌ | ✔️ |
| RedissonClusteredSpringCacheManager _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ✔️ | ❌ | ✔️ |
| RedissonClusteredSpringLocalCachedCacheManager _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ | ✔️ | ❌ | ✔️ |

应在 `org.redisson.spring.cache.RedissonSpringLocalCachedCacheManager` 或 `org.redisson.spring.cache.RedissonClusteredSpringLocalCachedCacheManager` 初始化期间提供以下选项对象：
```java
      LocalCachedMapOptions options = LocalCachedMapOptions.defaults()

      // Defines whether to store a cache miss into the local cache.
      // Default value is false.
      .storeCacheMiss(false);

      // Defines store mode of cache data.
      // Follow options are available:
      // LOCALCACHE - store data in local cache only.
      // LOCALCACHE_REDIS - store data in both Redis and local cache.
      .storeMode(StoreMode.LOCALCACHE_REDIS)

      // Defines Cache provider used as local cache store.
      // Follow options are available:
      // REDISSON - uses Redisson own implementation
      // CAFFEINE - uses Caffeine implementation
      .cacheProvider(CacheProvider.REDISSON)

      // Defines local cache eviction policy.
      // Follow options are available:
      // LFU - Counts how often an item was requested. Those that are used least often are discarded first.
      // LRU - Discards the least recently used items first
      // SOFT - Uses weak references, entries are removed by GC
      // WEAK - Uses soft references, entries are removed by GC
      // NONE - No eviction
     .evictionPolicy(EvictionPolicy.NONE)

      // If cache size is 0 then local cache is unbounded.
     .cacheSize(1000)

      // Used to load missed updates during any connection failures to Redis. 
      // Since, local cache updates can't be get in absence of connection to Redis. 
      // Follow reconnection strategies are available:
      // CLEAR - Clear local cache if map instance has been disconnected for a while.
      // LOAD - Store invalidated entry hash in invalidation log for 10 minutes
      //        Cache keys for stored invalidated entry hashes will be removed 
      //        if LocalCachedMap instance has been disconnected less than 10 minutes
      //        or whole cache will be cleaned otherwise.
      // NONE - Default. No reconnection handling
     .reconnectionStrategy(ReconnectionStrategy.NONE)

      // Used to synchronize local cache changes.
      // Follow sync strategies are available:
      // INVALIDATE - Default. Invalidate cache entry across all LocalCachedMap instances on map entry change
      // UPDATE - Insert/update cache entry across all LocalCachedMap instances on map entry change
      // NONE - No synchronizations on map changes
     .syncStrategy(SyncStrategy.INVALIDATE)

      // time to live for each map entry in local cache
     .timeToLive(10000)
      // or
     .timeToLive(10, TimeUnit.SECONDS)

      // max idle time for each map entry in local cache
     .maxIdle(10000)
      // or
     .maxIdle(10, TimeUnit.SECONDS);
```
每个Spring Cache实例都有两个重要的参数：`ttl`和`maxIdleTime`，如果它们没有定义或等于0，则无限存储数据。 
完整的配置示例：
```java
    @Configuration
    @ComponentScan
    @EnableCaching
    public static class Application {

        @Bean(destroyMethod="shutdown")
        RedissonClient redisson() throws IOException {
            Config config = new Config();
            config.useClusterServers()
                  .addNodeAddress("redis://127.0.0.1:7004", "redis://127.0.0.1:7001");
            return Redisson.create(config);
        }

        @Bean
        CacheManager cacheManager(RedissonClient redissonClient) {
            Map<String, CacheConfig> config = new HashMap<String, CacheConfig>();

            // define local cache settings for "testMap" cache.
            // ttl = 48 minutes and maxIdleTime = 24 minutes for local cache entries
            LocalCachedMapOptions options = LocalCachedMapOptions.defaults()
                .evictionPolicy(EvictionPolicy.LFU)
                .timeToLive(48, TimeUnit.MINUTES)
                .maxIdle(24, TimeUnit.MINUTES);
                .cacheSize(1000);
     
            // create "testMap" Redis cache with ttl = 24 minutes and maxIdleTime = 12 minutes
            LocalCachedCacheConfig cfg = new LocalCachedCacheConfig(24*60*1000, 12*60*1000, options);
            // Max size of map stored in Redis
            cfg.setMaxSize(2000);
            config.put("testMap", cfg);
            return new RedissonSpringLocalCachedCacheManager(redissonClient, config);
        }

    }
```
缓存配置可以从 YAML 配置文件中读取：
```java
    @Configuration
    @ComponentScan
    @EnableCaching
    public static class Application {

        @Bean(destroyMethod="shutdown")
        RedissonClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
            Config config = Config.fromYAML(configFile.getInputStream());
            return Redisson.create(config);
        }

        @Bean
        CacheManager cacheManager(RedissonClient redissonClient) throws IOException {
            return new RedissonSpringLocalCachedCacheManager(redissonClient, "classpath:/cache-config.yaml");
        }

    }
```
### YAML 配置格式
下面是 YAML 格式的名为 testMap 的 Spring Cache 配置：
```yaml
---
testMap:
  ttl: 1440000
  maxIdleTime: 720000
  localCacheOptions:
    invalidationPolicy: "ON_CHANGE"
    evictionPolicy: "NONE"
    cacheSize: 0
    timeToLiveInMillis: 0
    maxIdleInMillis: 0
```
_请注意：_`_localCacheOptions_`_设置仅适用于 _`_org.redisson.spring.cache.RedissonSpringLocalCachedCacheManager_`_ 和 _`_org.redisson.spring.cache.RedissonSpringClusteredLocalCachedCacheManager _`_类。_
## Hibernate Cache
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-hibernate)。
### Local cache and data partitioning
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-hibernate)。
## JCache API (JSR-107) implementation
Redisson 为 Redis 提供了 JCache API (JSR-107) 的实现。 以下是 JCache API 用法的示例。

1. 使用默认的配置文件`/redisson-jcache.yaml`
```java
MutableConfiguration<String, String> config = new MutableConfiguration<>();
        
CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", config);
```

2. 使用自定义的配置文件
```java
MutableConfiguration<String, String> config = new MutableConfiguration<>();

// yaml config
URI redissonConfigUri = getClass().getResource("redisson-jcache.yaml").toURI();
CacheManager manager = Caching.getCachingProvider().getCacheManager(redissonConfigUri, null);
Cache<String, String> cache = manager.createCache("namedCache", config);
```

3. 使用 Redisson 的配置对象
```java
MutableConfiguration<String, String> jcacheConfig = new MutableConfiguration<>();

Config redissonCfg = ...
Configuration<String, String> config = RedissonConfiguration.fromConfig(redissonCfg, jcacheConfig);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", config);
```

4. 使用 Redisson 实例对象
```java
MutableConfiguration<String, String> jcacheConfig = new MutableConfiguration<>();

RedissonClient redisson = ...
Configuration<String, String> config = RedissonConfiguration.fromInstance(redisson, jcacheConfig);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", config);
```
更多配置请阅读[配置](#mJVg8)。
提供的实现完全通过了 TCK 测试。这是[测试模块](https://github.com/cruftex/jsr107-test-zoo/tree/master/redisson-V2-test)。
### Asynchronous, Reactive and RxJava3 interfaces
除了常用的 JCache API 之外，Redisson 还提供异步、响应式和 RxJava3 API。
[Asynchronous interface](https://static.javadoc.io/org.redisson/redisson/3.11.6/org/redisson/api/CacheAsync.html). Each method returns `org.redisson.api.RFuture` object.
Example:
```java
MutableConfiguration<String, String> config = new MutableConfiguration<>();

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

CacheAsync<String, String> asyncCache = cache.unwrap(CacheAsync.class);
RFuture<Void> putFuture = asyncCache.putAsync("1", "2");
RFuture<String> getFuture = asyncCache.getAsync("1");
```
[Reactive interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/CacheReactive.html). Each method returns `reactor.core.publisher.Mono` object.
Example:
```java
MutableConfiguration<String, String> config = new MutableConfiguration<>();

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

CacheReactive<String, String> reactiveCache = cache.unwrap(CacheReactive.class);
Mono<Void> putFuture = reactiveCache.put("1", "2");
Mono<String> getFuture = reactiveCache.get("1");
```
[RxJava3 interface](https://static.javadoc.io/org.redisson/redisson/latest/org/redisson/api/CacheRx.html). Each method returns one of the following object: `io.reactivex.Completable`, `io.reactivex.Single`, `io.reactivex.Maybe`.
Example:
```java
MutableConfiguration<String, String> config = new MutableConfiguration<>();

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

CacheRx<String, String> rxCache = cache.unwrap(CacheRx.class);
Completable putFuture = rxCache.put("1", "2");
Maybe<String> getFuture = rxCache.get("1");
```
### Local cache and data partitioning
Redisson 为 JCache 实现提供了两个重要功能：
**本地缓存** - 所谓的近缓存，用于加速读取操作并避免网络往返。它在 Redisson 端缓存 JCache 条目，并且执行读取操作的速度比常见实现快 45 倍。具有相同名称的本地缓存实例连接到相同的发布/订阅通道。该通道用于在所有实例之间交换更新/无效事件。本地缓存存储不使用关键对象的 hashCode()/equals() 方法，而是使用序列化状态的哈希值。 
**数据分区** - 尽管 JCache 实例与集群兼容，但其内容无法跨集群中的多个 Redis 主节点进行扩展/分区。数据分区允许扩展 Redis 集群中单个 JCache 实例的可用内存、读/写操作和条目逐出过程。 
以下是可用管理器的完整列表：

|  Local cache | Data partitioning | Ultra-fast read/write |  |
| --- | --- | --- | --- |
| JCache _open-source version_ | ❌ | ❌ | ❌ |
| JCache [Redisson PRO](https://redisson.pro/) _ version_ | ❌ | ❌ | ✔️ |
| JCache with local cache _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ❌ | ✔️ |
| JCache with data partitioning _available only in _[Redisson PRO](https://redisson.pro/) | ❌ | ✔️ | ✔️ |
| JCache with local cache and data partitioning _available only in _[Redisson PRO](https://redisson.pro/) | ✔️ | ✔️ | ✔️ |

#### Local cache configuration:
```java
LocalCacheConfiguration<String, String> configuration = new LocalCacheConfiguration<>()

// Defines whether to store a cache miss into the local cache.
// Default value is false.
.storeCacheMiss(false);

// Defines store mode of cache data.
// Follow options are available:
// LOCALCACHE - store data in local cache only and use Redis only for data update/invalidation.
// LOCALCACHE_REDIS - store data in both Redis and local cache.
.storeMode(StoreMode.LOCALCACHE_REDIS)

// Defines Cache provider used as local cache store.
// Follow options are available:
// REDISSON - uses Redisson own implementation
// CAFFEINE - uses Caffeine implementation
.cacheProvider(CacheProvider.REDISSON)

// Defines local cache eviction policy.
// Follow options are available:
// LFU - Counts how often an item was requested. Those that are used least often are discarded first.
// LRU - Discards the least recently used items first
// SOFT - Uses weak references, entries are removed by GC
// WEAK - Uses soft references, entries are removed by GC
// NONE - No eviction
.evictionPolicy(EvictionPolicy.NONE)

// If cache size is 0 then local cache is unbounded.
.cacheSize(1000)

// Used to load missed updates during any connection failures to Redis. 
// Since, local cache updates can't be get in absence of connection to Redis. 
// Follow reconnection strategies are available:
// CLEAR - Clear local cache if map instance has been disconnected for a while.
// LOAD - Store invalidated entry hash in invalidation log for 10 minutes
//        Cache keys for stored invalidated entry hashes will be removed 
//        if LocalCachedMap instance has been disconnected less than 10 minutes
//        or whole cache will be cleaned otherwise.
// NONE - Default. No reconnection handling
.reconnectionStrategy(ReconnectionStrategy.NONE)

// Used to synchronize local cache changes.
// Follow sync strategies are available:
// INVALIDATE - Default. Invalidate cache entry across all LocalCachedMap instances on map entry change
// UPDATE - Insert/update cache entry across all LocalCachedMap instances on map entry change
// NONE - No synchronizations on map changes
.syncStrategy(SyncStrategy.INVALIDATE)

// time to live for each map entry in local cache
.timeToLive(10000)
// or
.timeToLive(10, TimeUnit.SECONDS)

// max idle time for each map entry in local cache
.maxIdle(10000)
// or
.maxIdle(10, TimeUnit.SECONDS);
```
#### Local cache usage examples:
```java
LocalCacheConfiguration<String, String> config = new LocalCacheConfiguration<>();
.setEvictionPolicy(EvictionPolicy.LFU)
.setTimeToLive(48, TimeUnit.MINUTES)
.setMaxIdle(24, TimeUnit.MINUTES);
.setCacheSize(1000);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

// or

URI redissonConfigUri = getClass().getResource("redisson-jcache.yaml").toURI();
CacheManager manager = Caching.getCachingProvider().getCacheManager(redissonConfigUri, null);
Cache<String, String> cache = manager.createCache("myCache", config);

// or 

Config redissonCfg = ...
Configuration<String, String> rConfig = RedissonConfiguration.fromConfig(redissonCfg, config);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", rConfig);
```
#### Data partitioning usage examples:
```java
ClusteredConfiguration<String, String> config = new ClusteredConfiguration<>();

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

// or

URI redissonConfigUri = getClass().getResource("redisson-jcache.yaml").toURI();
CacheManager manager = Caching.getCachingProvider().getCacheManager(redissonConfigUri, null);
Cache<String, String> cache = manager.createCache("myCache", config);

// or 

Config redissonCfg = ...
Configuration<String, String> rConfig = RedissonConfiguration.fromConfig(redissonCfg, config);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", rConfig);
```
#### Local cache with data partitioning configuration:
```java
ClusteredLocalCacheConfiguration<String, String> configuration = new ClusteredLocalCacheConfiguration<>()

// Defines whether to store a cache miss into the local cache.
// Default value is false.
.storeCacheMiss(false);

// Defines store mode of cache data.
// Follow options are available:
// LOCALCACHE - store data in local cache only and use Redis only for data update/invalidation.
// LOCALCACHE_REDIS - store data in both Redis and local cache.
.storeMode(StoreMode.LOCALCACHE_REDIS)

// Defines Cache provider used as local cache store.
// Follow options are available:
// REDISSON - uses Redisson own implementation
// CAFFEINE - uses Caffeine implementation
.cacheProvider(CacheProvider.REDISSON)

// Defines local cache eviction policy.
// Follow options are available:
// LFU - Counts how often an item was requested. Those that are used least often are discarded first.
// LRU - Discards the least recently used items first
// SOFT - Uses weak references, entries are removed by GC
// WEAK - Uses soft references, entries are removed by GC
// NONE - No eviction
.evictionPolicy(EvictionPolicy.NONE)

// If cache size is 0 then local cache is unbounded.
.cacheSize(1000)

// Used to load missed updates during any connection failures to Redis. 
// Since, local cache updates can't be get in absence of connection to Redis. 
// Follow reconnection strategies are available:
// CLEAR - Clear local cache if map instance has been disconnected for a while.
// LOAD - Store invalidated entry hash in invalidation log for 10 minutes
//        Cache keys for stored invalidated entry hashes will be removed 
//        if LocalCachedMap instance has been disconnected less than 10 minutes
//        or whole cache will be cleaned otherwise.
// NONE - Default. No reconnection handling
.reconnectionStrategy(ReconnectionStrategy.NONE)

// Used to synchronize local cache changes.
// Follow sync strategies are available:
// INVALIDATE - Default. Invalidate cache entry across all LocalCachedMap instances on map entry change
// UPDATE - Insert/update cache entry across all LocalCachedMap instances on map entry change
// NONE - No synchronizations on map changes
.syncStrategy(SyncStrategy.INVALIDATE)

// time to live for each map entry in local cache
.timeToLive(10000)
// or
.timeToLive(10, TimeUnit.SECONDS)

// max idle time for each map entry in local cache
.maxIdle(10000)
// or
.maxIdle(10, TimeUnit.SECONDS);
```
#### Local cache with data partitioning usage examples:
```java
ClusteredLocalCacheConfiguration<String, String> config = new ClusteredLocalCacheConfiguration<>();

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("myCache", config);

// or

URI redissonConfigUri = getClass().getResource("redisson-jcache.yaml").toURI();
CacheManager manager = Caching.getCachingProvider().getCacheManager(redissonConfigUri, null);
Cache<String, String> cache = manager.createCache("myCache", config);

// or 

Config redissonCfg = ...
Configuration<String, String> rConfig = RedissonConfiguration.fromConfig(redissonCfg, config);

CacheManager manager = Caching.getCachingProvider().getCacheManager();
Cache<String, String> cache = manager.createCache("namedCache", rConfig);
```
## MyBatis Cache
请参阅这个[章节](https://github.com/redisson/redisson/tree/master/redisson-mybatis)。
### Local cache and data partitioning
请参阅这个[章节](https://github.com/redisson/redisson/tree/master/redisson-mybatis)。
## Tomcat Redis Session Manager
请参阅这个[章节](https://github.com/redisson/redisson/tree/master/redisson-tomcat)。
## Spring Session
请注意，Redis `notification-keyspace-events` 设置应包含 `Exg` 字母以使 Spring Session 集成正常工作。 确保您的类路径中有 Spring Session 库，如有必要，请添加它：
**Maven**
```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-core</artifactId>
    <version>3.2.1</version>
</dependency>

<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson-spring-data-32</artifactId>
   <version>3.25.2</version>
</dependency>
```
**Gradle**
```groovy
compile 'org.springframework.session:spring-session-core:3.2.1'

compile 'org.redisson:redisson-spring-data-32:3.25.2'
```
### Usage example of Spring Http Session configuration:
Add configuration class which extends AbstractHttpSessionApplicationInitializer class:
```java
@Configuration
@EnableRedisHttpSession
public class SessionConfig extends AbstractHttpSessionApplicationInitializer { 

    @Bean
    public RedissonConnectionFactory redissonConnectionFactory(RedissonClient redisson) {
        return new RedissonConnectionFactory(redisson);
    }

    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
        Config config = Config.fromYAML(configFile.getInputStream());
        return Redisson.create(config);
    }

}
```
### Usage example of Spring WebFlux’s Session configuration:
Add configuration class which extends AbstractReactiveWebInitializer class:
```java
@Configuration
@EnableRedisWebSession
public class SessionConfig extends AbstractReactiveWebInitializer { 

    @Bean
    public RedissonConnectionFactory redissonConnectionFactory(RedissonClient redisson) {
        return new RedissonConnectionFactory(redisson);
    }

    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
        Config config = Config.fromYAML(configFile.getInputStream());
        return Redisson.create(config);
    }
}
```
### Usage example with Spring Boot configuration:

1. Add Spring Session Data Redis library in classpath:

**Maven**
```
<dependency>
   <groupId>org.springframework.session</groupId>
   <artifactId>spring-session-data-redis</artifactId>
   <version>3.2.1</version>
</dependency>
```
**Gradle**
```groovy
compile 'org.springframework.session:spring-session-data-redis:3.2.1'  
```

2. Add Redisson Spring Data Redis library in classpath:

**Maven**
```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson-spring-data-32</artifactId>
  <version>3.25.2</version>
</dependency>
```
**Gradle**
```groovy
compile 'org.redisson:redisson-spring-data-32:3.25.2'  
```

3. Define follow properties in spring-boot settings
```properties
spring.session.store-type=redis
spring.redis.redisson.file=classpath:redisson.yaml
spring.session.timeout.seconds=900
```
Try [Redisson PRO](https://redisson.pro/) with **ultra-fast performance** and **support by SLA**.
## Spring Transaction Manager
Redisson 提供了 `org.springframework.transaction.PlatformTransactionManager` 和 `org.springframework.transaction.ReactiveTransactionManager` 接口的实现来参与 Spring 事务。另请参阅[事务](#GN6dh)部分。
### Usage example of Spring Transaction Management:
```java
@Configuration
@EnableTransactionManagement
public class RedissonTransactionContextConfig {

    @Bean
    public TransactionalBean transactionBean() {
        return new TransactionalBean();
    }

    @Bean
    public RedissonTransactionManager transactionManager(RedissonClient redisson) {
        return new RedissonTransactionManager(redisson);
    }

    @Bean(destroyMethod="shutdown")
    public RedissonClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
        Config config = Config.fromYAML(configFile.getInputStream());
        return Redisson.create(config);
    }

}


public class TransactionalBean {

    @Autowired
    private RedissonTransactionManager transactionManager;

    @Transactional
    public void commitData() {
        RTransaction transaction = transactionManager.getCurrentTransaction();
        RMap<String, String> map = transaction.getMap("test1");
        map.put("1", "2");
    }

}
```
### Usage example of Reactive Spring Transaction Management:
```java
@Configuration
@EnableTransactionManagement
public class RedissonReactiveTransactionContextConfig {

    @Bean
    public TransactionalBean transactionBean() {
        return new TransactionalBean();
    }

    @Bean
    public ReactiveRedissonTransactionManager transactionManager(RedissonReactiveClient redisson) {
        return new ReactiveRedissonTransactionManager(redisson);
    }

    @Bean(destroyMethod="shutdown")
    public RedissonReactiveClient redisson(@Value("classpath:/redisson.yaml") Resource configFile) throws IOException {
        Config config = Config.fromYAML(configFile.getInputStream());
        return Redisson.createReactive(config);
    }

}

public class TransactionalBean {

    @Autowired
    private ReactiveRedissonTransactionManager transactionManager;

    @Transactional
    public Mono<Void> commitData() {
        Mono<RTransactionReactive> transaction = transactionManager.getCurrentTransaction();
        return transaction.flatMap(t -> {
            RMapReactive<String, String> map = t.getMap("test1");
            return map.put("1", "2");
        }).then();
    }

}
```
## Spring Data Redis
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-spring-data#spring-data-redis-integration)。
## Spring Boot Starter
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter#spring-boot-starter)。
## Micronaut
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-micronaut)。
## Quarkus
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-quarkus)。
## Helidon
请参阅[这章](https://github.com/redisson/redisson/tree/master/redisson-helidon)。
# Dependency list 依赖列表
以下是Redisson使用的库列表：

| Group id | Artifact Id | Version | Dependency |
| --- | --- | --- | --- |
| com.esotericsoftware | kryo | 5.4+ | **required** (if Kryo is used as codec) |
| com.esotericsoftware | reflectasm | 1.11+ | **required** (if Kryo is used as codec) |
| com.esotericsoftware | minlog | 1.3+ | **required** (if Kryo is used as codec) |
| org.objenesis | objenesis | 3.3+ | **required** (if Kryo is used as codec) |
| io.netty | netty-common | 4.1+ | **required** |
| io.netty | netty-codec | 4.1+ | **required** |
| io.netty | netty-buffer | 4.1+ | **required** |
| io.netty | netty-transport | 4.1+ | **required** |
| io.netty | netty-handler | 4.1+ | **required** |
| io.netty | netty-resolver | 4.1+ | **required** |
| io.netty | netty-resolver-dns | 4.1+ | **required** |
| com.fasterxml.jackson.dataformat | jackson-core | 2.7+ | **required** |
| com.fasterxml.jackson.dataformat | jackson-databind | 2.7+ | **required** |
| com.fasterxml.jackson.dataformat | jackson-annotations | 2.7+ | **required** |
| com.fasterxml.jackson.dataformat | jackson-dataformat-yaml | 2.7+ | **required** |
| org.yaml | snakeyaml | 1.0+ | **required** |
| net.bytebuddy | byte-buddy | 1.6+ | _optional (used by LiveObject service)_ |
| org.jodd | jodd-bean | 3.7+ | _optional (used by LiveObject service)_ |
| javax.cache | cache-api | 1.1.1 | _optional (used by JCache implementation)_ |
| io.projectreactor | reactor-core | 3.1+ | _optional (used by RedissonReactiveClient)_ |
| io.reactivex.rxjava3 | rxjava | 3.0+ | _optional (used by RedissonRxClient)_ |

# Observability
## 指标
_此功能仅在 Redisson PRO 版本中可用。_
Redisson通过`Micrometer`框架提供了与最流行的监控系统的集成。
### AppOptics
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.AppOpticsMeterRegistryProvider |
| Params | uri - AppOptics host uri hostTag - tag mapped to host apiToken - AppOptics api token numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-appoptics |

### Atlas
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.AtlasMeterRegistryProvider |
| Params | uri - Atlas host uri configUri - Atlas LWC endpoint uri to retrieve current subscriptions evalUri - Atlas LWC endpoint uri to evaluate the data for a subscription numThreads - number of threads used by scheduler (default is 4) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 10000) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-atlas |

### Azure
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.AzureMonitorMeterRegistryProvider |
| Params | instrumentationKey - instrumentation key numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-azure-monitor |

### Amazon CloudWatch
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.CloudWatchMeterRegistryProvider |
| Params | accessKey - AWS access key secretKey - AWS secret access key namespace - namespace value numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-cloudwatch |

### Datadog
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.DatadogMeterRegistryProvider |
| Params | uri - Datadog host uri hostTag - tag mapped to host apiKey - api key numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-datadog |

### Dropwizard
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.DropwizardMeterRegistryProvider |
| Params | sharedRegistryName - name used to store instance in SharedMetricRegistries nameMapper - custom implementation of io.micrometer.core.instrument.util.HierarchicalNameMapper |

### Dynatrace
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.DynatraceMeterRegistryProvider |
| Params | uri - Dynatrace host uri apiToken - api token deviceId - device id numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-dynatrace |

### Elastic
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.ElasticMeterRegistryProvider |
| Params | host - Elasticsearch host uri userName - user name password - password numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-elastic |

### Ganglia
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.GangliaMeterRegistryProvider |
| Params | host - Ganglia host address port - Ganglia port numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-ganglia |

### Graphite
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.GraphiteMeterRegistryProvider |
| Params | host - Graphite host address port - Graphite port |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-graphite |

### Humio
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.HumioMeterRegistryProvider |
| Params | uri - Humio host uri repository - repository name apiToken - api token numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-humio |

### Influx
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.InfluxMeterRegistryProvider |
| Params | uri - Influx host uri db - db name userName - user name password - password numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-influx |

### JMX
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.JmxMeterRegistryProvider |
| Params | domain - domain name sharedRegistryName - name used to store instance in SharedMetricRegistries |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-jmx |

### Kairos
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.KairosMeterRegistryProvider |
| Params | uri - Kairos host uri userName - user name password - password numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-kairos |

### NewRelic
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.NewRelicMeterRegistryProvider |
| Params | uri - NewRelic host uri apiKey - api key accountId - account id numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-new-relic |

### Prometheus
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.MeterRegistryWrapper |
| Params | registry - instance of PrometheusMeterRegistry object |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-prometheus |

### SingnalFx
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.SingnalFxMeterRegistryProvider |
| Params | apiHost - SingnalFx host uri accessToken - access token source - application instance id numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 10 secs) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-signalfx |

### Stackdriver
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.StackdriverMeterRegistryProvider |
| Params | projectId - project id numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-stackdriver |

### Statsd
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.StatsdMeterRegistryProvider |
| Params | host - Statsd host address port - Statsd port flavor - metrics format ETSY/DATADOG/TELEGRAF/SYSDIG |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-statsd |

### Wavefront
|  |  |
| --- | --- |
| Class | org.redisson.config.metrics.WavefrontMeterRegistryProvider |
| Params | uri - Wavefront host uri source - application instance id apiToken - api token numThreads - number of threads used by scheduler (default is 2) step - update interval in ISO-8601 format (default is 1 min) batchSize - number of measurements sent per request (default is 500) |
| Dependency | groupId: io.micrometer artifactId: micrometer-registry-wavefront |

### 详细指标
#### Config
```java
Config config = ... // Redisson PRO config object

// JMX
JmxMeterRegistryProvider provider = new JmxMeterRegistryProvider();
provider.setDomain("appStats");
config.setMeterRegistryProvider(provider);

// Prometheus
PrometheusMeterRegistry registry = ...
config.setMeterRegistryProvider(new MeterRegistryWrapper(registry));

// Dynatrace
DynatraceMeterRegistryProvider p = new DynatraceMeterRegistryProvider();
p.setApiToken("Hg3M0iadsQC2Pcjk6QIW0g");
p.setUri("https://qtd9012301.live.dynatrace.com/");
p.setDeviceId("myHost");
config.setMeterRegistryProvider(p);

// Influx
InfluxMeterRegistryProvider provider = new InfluxMeterRegistryProvider();
provider.setUri("http://localhost:8086/");
provider.setDb("myinfluxdb");
provider.setUserName("admin");
provider.setPassword("admin");
config.setMeterRegistryProvider(provider);
```
Redisson Config 会读取 YAML 的配置：
JMX:
```yaml
meterRegistryProvider: !<org.redisson.config.metrics.JmxMeterRegistryProvider> 
               domain: "appStats"
```
Dynatrace:
```yaml
meterRegistryProvider: !<org.redisson.config.metrics.DynatraceMeterRegistryProvider>
             apiToken: "Hg3M0iadsQC2Pcjk6QIW0g"
                  uri: "https://qtd9012301.live.dynatrace.com"
             deviceId: "myHost"
```
Influx:
```yaml
meterRegistryProvider: !<org.redisson.config.metrics.InfluxMeterRegistryProvider>
                  uri: "http://localhost:8086/"
                   db: "myinfluxdb"
             userName: "admin"
             password: "admin"
```
可以使用以下指标：
#### Configuration metrics

- `redisson.license.expiration-year` - A Gauge of the number of the license expiration year
- `redisson.license.expiration-month` - A Gauge of the number of the license expiration month
- `redisson.license.expiration-day` - A Gauge of the number of the license expiration day
- `redisson.license.active-instances` - A Gauge of the number of active Redisson PRO clients
- `redisson.executor-pool-size` - A Gauge of the number of executor threads pool size
- `redisson.netty-pool-size` - A Gauge of the number of netty threads pool size
- `netty.eventexecutor.tasks.pending` - Number of pending tasks in Netty event executor
#### Metrics per Redis node.
Base name: redisson.redis.<host>:<port>

- status - A Gauge of the number value of Redis node status [1 = connected, -1 = disconnected]
- type - A Gauge of the number value of Redis node type [1 = MASTER, 2 = SLAVE, 3 = SENTINEL]

- total-response-bytes - A Meter of the total amount of bytes received from Redis
- response-bytes - A Histogram of the number of bytes received from Redis
- total-request-bytes - A Meter of the total amount of bytes sent to Redis
- request-bytes - A Histogram of the number of bytes sent to Redis

- connections.active - A Counter of the number of busy connections
- connections.free - A Counter of the number of free connections
- connections.max-pool-size - A Counter of the number of maximum connection pool size
- connections.reconnected - A Counter of the number of reconnected connections
- connections.total - A Counter of the number of total connections in pool

- operations.total - A Meter of the number of total executed operations
- operations.total-failed - A Meter of the number of total failed operations
- operations.total-successful - A Meter of the number of total successful operations
- operations.latency - A Histogram of the number of operations latency in milliseconds
- operations.retry-attempt - A Histogram of the number of operations attempt

- publish-subscribe-connections.active - A Counter of the number of active publish subscribe connections
- publish-subscribe-connections.free - A Counter of the number of free publish subscribe connections
- publish-subscribe-connections.max-pool-size - A Counter of the number of maximum publish subscribe connection pool size
- publish-subscribe-connections.total - A Counter of the number of total publish subscribe connections in pool
#### Metrics per RRemoteService object
Base name: redisson.remote-service.<name>

- invocations.total - A Meter of the number of total executed invocations
- invocations.total-failed - A Meter of the number of total failed to execute invocations
- invocations.total-successful - A Meter of the number of total successful to execute invocations
#### Metrics per RExecutorService object
Base name: redisson.executor-service.<name>

- tasks.submitted - A Meter of the number of submitted tasks
- tasks.executed - A Meter of the number of executed tasks

- workers.active - A Gauge of the number of busy task workers
- workers.free - A Gauge of the number of free task workers
- workers.total - A Gauge of the number of total task workers
- workers.tasks-executed.total - A Meter of the number of total executed tasks by workers
- workers.tasks-executed.total-failed - A Meter of the number of total failed to execute tasks by workers
- workers.tasks-executed.total-successful - A Meter of the number of total successful to execute tasks by workers
#### Metrics per RMap object
Base name: redisson.map.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache
#### Metrics per RMapCache object
Base name: redisson.map-cache.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache
#### Metrics per RClusteredMapCache object
Base name: redisson.clustered-map-cache.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache
#### Metrics per RLocalCachedMap object
Base name: redisson.local-cached-map.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache

- local-cache.hits - A Meter of the number of get requests for data contained in local cache
- local-cache.misses - A Meter of the number of get requests for data contained in local cache
- local-cache.evictions - A Meter of the number of evictions for data contained in local cache
- local-cache.size - A Gauge of the number of local cache size
#### Metrics per RClusteredLocalCachedMap object
Base name: redisson.clustered-local-cached-map.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache

- local-cache.hits - A Meter of the number of get requests for data contained in local cache
- local-cache.misses - A Meter of the number of get requests for data contained in local cache
- local-cache.evictions - A Meter of the number of evictions for data contained in local cache
- local-cache.size - A Gauge of the number of local cache size
#### Metrics per RLocalCachedMapCache object
Base name: redisson.local-cached-map-cache.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache

- local-cache.hits - A Meter of the number of get requests for data contained in local cache
- local-cache.misses - A Meter of the number of get requests for data contained in local cache
- local-cache.evictions - A Meter of the number of evictions for data contained in local cache
- local-cache.size - A Gauge of the number of local cache size
#### Metrics per RClusteredLocalCachedMapCache object
Base name: redisson.clustered-local-cached-map-cache.<name>

- hits - A Meter of the number of get requests for data contained in cache
- misses - A Meter of the number of get requests for data not contained in cache
- puts - A Meter of the number of puts to the cache
- removals - A Meter of the number of removals from the cache

- local-cache.hits - A Meter of the number of get requests for data contained in local cache
- local-cache.misses - A Meter of the number of get requests for data contained in local cache
- local-cache.evictions - A Meter of the number of evictions for data contained in local cache
- local-cache.size - A Gauge of the number of local cache size
#### Metrics per RTopic object
Base name: redisson.topic.<name>

- messages-sent - A Meter of the number of messages sent for topic
- messages-received - A Meter of the number of messages received for topic
#### Metrics per RBucket object
Base name: redisson.bucket.<name>

- gets - A Meter of the number of get operations executed for bucket object
- sets - A Meter of the number of set operations executed for bucket object
#### Metrics per JCache object
Base name: redisson.JCache.<name>

- cache.evictions - A Meter of the number of evictions for data contained in JCache
- cache.puts - A Meter of the number of puts to the JCache
- hit.cache.gets - A Meter of the number of get requests for data contained in JCache
- miss.cache.gets - A Meter of the number of get requests for data contained in JCache
## Tracing
_此功能仅在 Redisson PRO 版本中可用。_
Redisson 通过 `Micrometer Observation API` 和 `Micrometer Tracing` 框架提供与最流行的跟踪器库的集成。此功能允许收集有关调用的 Redisson 方法的额外统计信息：名称、参数、调用路径、持续时间和频率。 
每个跟踪事件的配置指标。

- `invacation` - 带参数的 Redisson 方法。仅当 `includeArgs` 设置为 true 时才包含参数
- `object_name` - Redisson对象名称
- `address` - Redis服务器地址
- `username` - 用于连接Redis服务器的用户名
### OpenZipkin Brave
配置示例：
```java
OkHttpSender sender = OkHttpSender.create("http://localhost:9411/api/v2/spans");
AsyncZipkinSpanHandler zipkinSpanHandler = AsyncZipkinSpanHandler.create(sender);
StrictCurrentTraceContext braveCurrentTraceContext = StrictCurrentTraceContext.create();

Tracing tracing = Tracing.newBuilder()
                         .localServiceName("myservice")
                         .currentTraceContext(braveCurrentTraceContext)
                         .sampler(Sampler.ALWAYS_SAMPLE)
                         .addSpanHandler(zipkinSpanHandler)
                         .build();

config.setTracingProvider(new BraveTracingProvider(tracing));
```
需要的依赖：
groupId: `io.zipkin.reporter2`, artifactId: `zipkin-sender-okhttp3`
groupId: `io.zipkin.reporter2`, artifactId: `zipkin-reporter-brave`
### OpenTelemetry
配置示例：
```java
SpanExporter spanExporter = new ZipkinSpanExporterBuilder()
                                    .setSender(URLConnectionSender.create("http://localhost:9411/api/v2/spans"))
                                    .build();

SdkTracerProvider sdkTracerProvider = SdkTracerProvider.builder()
                                           .addSpanProcessor(BatchSpanProcessor.builder(spanExporter).build())
                                           .build();

OpenTelemetrySdk openTelemetrySdk = OpenTelemetrySdk.builder()
                                         .setPropagators(ContextPropagators.create(B3Propagator.injectingSingleHeader()))
                                         .setTracerProvider(sdkTracerProvider)
                                         .build();

io.opentelemetry.api.trace.Tracer otelTracer = openTelemetrySdk.getTracerProvider().get("io.micrometer.micrometer-tracing");

config.setTracingProvider(new OtelTracingProvider(otelTracer, openTelemetrySdk.getPropagators()));
```
需要的依赖：
groupId: `io.opentelemetry`, artifactId: `opentelemetry-exporter-zipkin`
groupId: `io.zipkin.reporter2`, artifactId: `zipkin-sender-urlconnection`
# 参考

- [redisson/redisson Wiki](https://github.com/redisson/redisson/wiki)
