可以把分布式基础按“从单机到分布式，从正常路径到异常路径”的顺
序学。不要一开始就按算法名学，比如先学 Raft、Paxos、CAP，这样
容易悬空。更适合你的顺序是下面这条线：

计算机基础
-> 网络通信
-> RPC
-> 并发与状态
-> 分片与路由
-> 元数据
-> 副本与一致性
-> 故障处理
-> 调度与负载均衡
-> 分布式存储
-> 可观测性
-> 共识算法

第一组：必须先补牢的底层基础

这一组是分布式系统的地基。

操作系统
网络
并发
存储

重点不是学得特别深，而是要知道系统为什么会慢、为什么会失败。

你需要掌握：

进程、线程、协程
锁、条件变量、原子操作
内存、页表、mmap
磁盘、SSD、顺序读写、随机读写
TCP、连接、超时、重传
带宽、延迟、吞吐、P99

对于 KV Cache 场景，尤其要理解：

GPU HBM 很快但容量小
CPU Memory 容量大但访问慢
SSD 更慢但容量更大
跨机器访问比本机访问更慢
跨 GPU / 跨节点搬运 KV Cache 代价很高

这一组的目标是：你看到一个系统设计时，能大概判断“这个东西可能
慢在哪里”。

第二组：RPC 和远程调用

分布式系统不是本地函数调用，而是远程调用。

你需要学：

RPC 是什么
请求/响应模型
序列化和反序列化
timeout
retry
幂等
限流
熔断
连接池

最关键的一句话是：

远程调用失败，不代表服务端没有执行。

比如：

evict_block_group(bg_1) 超时

可能是：

请求没发到服务端
服务端执行了但响应丢了
服务端还在执行
服务端执行失败

所以你要学会设计幂等接口：

evict(block_group_id, epoch)
migrate(block_group_id, src, dst, epoch)
pin(block_group_id, request_id)
unpin(block_group_id, request_id)

这一组的目标是：你能理解为什么分布式接口都要考虑超时、重试和
幂等。

第三组：并发状态和状态机

这组对你现在最重要。

KV Cache、Block、BlockGroup 本质上都是有生命周期的状态对象。

你应该学：

状态机
状态转移
并发读写
CAS
锁
版本号
epoch
引用计数
pin/unpin

以 BlockGroup 为例，你要能设计出这样的状态：

CREATED
LOCAL
PINNED
MIGRATING_OUT
MIGRATING_IN
REMOTE
EVICTING
EVICTED
FAILED

然后回答：

哪些状态可以读？
哪些状态可以迁移？
哪些状态可以淘汰？
哪些状态可以释放？
迁移失败怎么办？
淘汰过程中又被访问怎么办？
旧 owner 恢复后还能不能继续写？

这一组的目标是：你能把一个分布式对象的生命周期讲清楚。

第四组：分片、路由和元数据

当数据放不下一台机器，就要分片。

你需要学：

哈希分片
一致性哈希
Range 分片
路由表
元数据服务
本地元数据缓存
元数据失效

在 KV Cache 里，元数据可能是：

block_id -> block_group_id
block_group_id -> node_id
block_group_id -> device_id
block_group_id -> memory_tier
block_group_id -> state
block_group_id -> epoch
request_id -> block_group_list

这里最重要的问题是：

数据在哪里？
谁说了算？
客户端看到的是不是旧位置？
迁移过程中路由怎么更新？

这一组的目标是：你能理解一个请求如何找到它需要的 KV Cache。

第五组：副本和一致性

数据有副本之后，就会有一致性问题。

你需要学：

主从副本
多副本读写
强一致性
最终一致性
读写一致性
过期读
写冲突

但要注意，KV Cache 里要区分两类东西：

KV 数据本身：很多时候是不可变 block。
元数据状态：经常变化，必须更认真处理一致性。

比如：

block 的 K/V 内容生成后不再修改
block_group 的 location/state/ref_count/pin_count 会一直变

所以：

KV 数据可以更偏缓存语义
元数据要更偏强一致语义

这一组的目标是：你能判断哪些数据必须强一致，哪些数据可以最终
一致。

第六组：故障处理

分布式系统一定会失败，所以要专门学失败路径。

你需要学：

节点宕机
进程崩溃
网络分区
请求超时
慢节点
磁盘故障
GPU 故障
数据损坏

然后结合 KV Cache 思考：

worker 挂了，它持有的 block group 怎么办？
迁移到一半失败怎么办？
调度器以为 cache 在 A，但实际已经迁到 B 怎么办？
请求取消后 ref_count 没释放怎么办？
节点恢复后带着旧 epoch 继续写怎么办？

这组里面要重点学：

heartbeat
failure detector
lease
fencing token
epoch
后台修复任务
补偿机制

这一组的目标是：你能设计“失败后系统怎么恢复”。

第七组：迁移、淘汰和缓存策略

这一组开始贴近你的业务。

你需要学：

cache hit / miss
LRU
LFU
TTL
Cost-aware eviction
cache migration
cache prefetch
cache placement
多级缓存

KV Cache 里尤其关注：

以 block 还是 block group 为单位迁移？
什么情况下从 GPU HBM 下沉到 CPU Memory？
什么时候迁移到远端？
哪些 cache 不能淘汰？
如何根据 ref_count 和 pin_count 判断是否安全？

典型问题：

显存满了，应该删哪个 block group？
远端有 cache，本地没有，要不要拉回来？
请求马上要 decode，cache 却正在迁移，怎么办？

这一组的目标是：你能理解 KV Cache 管理策略。

第八组：调度和负载均衡

当有多台机器、多张 GPU、多份模型副本时，就要学调度。

你需要学：

Round Robin
Least Loaded
Power of Two Choices
Locality-aware Scheduling
Cost-based Scheduling
队列
优先级
资源隔离

KV Cache 场景里，调度不能只看 GPU 空不空，还要看：

cache 在哪里
拉取 cache 的成本
当前 GPU HBM 剩余多少
当前 batch 是否能合并
请求上下文长度
decode 预计持续多久
网络是否拥塞

这一组的目标是：你能理解为什么“把请求调到空闲 GPU”不一定是最
优。

第九组：可观测性

分布式系统不靠猜，要靠指标、日志和链路追踪。

你需要学：

Metrics
Logs
Traces
告警
Dashboard
P50 / P99 / P999
错误率
队列长度

KV Cache 相关指标可以重点看：

cache_hit_rate
cache_miss_rate
cache_eviction_count
cache_migration_bytes
cache_migration_latency
gpu_memory_used_bytes
remote_read_latency
metadata_update_latency
request_ttft
request_tpot
p99_decode_latency

日志里要关注：

request_id
block_group_id
node_id
device_id
epoch
state
latency
error_code

这一组的目标是：线上变慢或出错时，你知道先看什么。

第十组：共识算法和分布式协调

这组可以稍微放后一点学。

你需要了解：

Raft
Paxos
Leader Election
Log Replication
Quorum
etcd
ZooKeeper
Consul

不要一开始就追求手写 Raft。你先知道它解决什么问题：

选主
元数据强一致
配置变更
分布式锁
服务发现
集群成员管理

在你的场景里，它可能用于：

scheduler leader election
metadata server 高可用
block_group owner 记录
集群节点状态管理

这一组的目标是：你知道什么时候需要 etcd/Raft 这类组件，什么时
候不需要。

推荐学习顺序

我建议你按这个顺序一步一步学：

1. 网络基础：TCP、延迟、吞吐、P99
2. RPC：timeout、retry、幂等
3. 并发状态：锁、CAS、状态机
4. KV Cache 对象建模：block、block group、metadata
5. 分片和路由：数据如何找到
6. 一致性：强一致、最终一致、epoch
7. 故障处理：节点挂了、迁移失败、旧 owner 写入
8. 缓存策略：迁移、淘汰、pin/ref_count
9. 调度策略：locality-aware、cost-based
10. 可观测性：metrics、logs、traces
11. Raft / etcd / ZooKeeper
12. CAP / PACELC / 分布式理论总结

最适合你当前阶段的学习主线是：

一个 BlockGroup 从创建到释放的一生

围绕它学就不会散：

创建时：元数据怎么写？
使用时：如何 pin？如何 ref_count？
访问时：怎么路由？
迁移时：状态怎么变？epoch 怎么检查？
淘汰时：能不能删？删谁？
失败时：如何恢复？
观测时：看哪些指标？

如果你能把这条链路讲清楚，分布式基础就不再是零散概念，而是能
和你实习工作直接对应起来。
