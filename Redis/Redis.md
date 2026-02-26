参与复制的Redis实例划分为**主节点(master)和从节点(slave)**。默认情况下，Redis都是主节点。**每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点，master以写为主，Slave以读为主。**

接下来的内容主要包括以下几方面：

Redis高级：

- Redis主从
- Redis哨兵
- Redis分片集群
- Redis数据结构
- Redis内存回收
- Redis缓存一致性

微服务高级：

- Eureka和Nacos对比
- Ribbon和SpringCloudLoadBalancer
- Hystix和Sentinel
- 限流算法

# 1.Redis主从

单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离。

## 1.1.主从集群结构

下图就是一个简单的Redis主从集群结构：

![img](./assets/1770364511907-58.png)

如图所示，集群中有一个master节点、两个slave节点（现在叫replica）。当我们通过Redis的Java客户端访问主从集群时，应该做好路由：

- 如果是写操作，应该访问master节点，master会自动将数据同步给两个slave节点
- 如果是读操作，建议访问各个slave节点，从而分担并发压力

## 1.2.搭建主从集群

我们会在同一个虚拟机中利用3个Docker容器来搭建主从集群，容器信息如下：

| **容器名** | **角色** | **IP**          | **映射****端口** |
| :--------- | :------- | :-------------- | :--------------- |
| r1         | master   | 192.168.150.101 | 7001             |
| r2         | slave    | 192.168.150.101 | 7002             |
| r3         | slave    | 192.168.150.101 | 7003             |

### 1.2.1.启动多个Redis实例

我们利用课前资料提供的docker-compose文件来构建主从集群：

![img](./assets/1770364511682-1.png)

文件内容如下：

```YAML
version: "3.2"

services:
  r1:
    image: redis
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001"]
  r2:
    image: redis
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002"]
  r3:
    image: redis
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003"]
```

将其上传至虚拟机的`/root/redis`目录下：

![img](./assets/1770364511682-2.png)

执行命令，运行集群：

```Bash
docker compose up -d
```

结果：

![img](./assets/1770364511683-3.png)

查看docker容器，发现都正常启动了：

![img](./assets/1770364511683-4.png)

由于采用的是host模式，我们看不到端口映射。不过能直接在宿主机通过ps命令查看到Redis进程：

![img](./assets/1770364511684-5.png)

### 1.2.2.建立集群

虽然我们启动了3个Redis实例，但是它们并没有形成主从关系。我们需要通过命令来配置主从关系：

```Bash
# Redis5.0以前
slaveof <masterip> <masterport>
# Redis5.0以后
replicaof <masterip> <masterport>
```

有临时和永久两种模式：

- 永久生效：在redis.conf文件中利用`slaveof`命令指定`master`节点
- 临时生效：直接利用redis-cli控制台输入`slaveof`命令，指定`master`节点

我们测试临时模式，首先连接`r2`，让其以`r1`为master

```Bash
# 连接r2
docker exec -it r2 redis-cli -p 7002
# 认r1主，也就是7001
slaveof 192.168.150.101 7001
```

然后连接`r3`，让其以`r1`为master

```Bash
# 连接r3
docker exec -it r3 redis-cli -p 7003
# 认r1主，也就是7001
slaveof 192.168.150.101 7001
```

然后连接`r1`，查看集群状态：

```Bash
# 连接r1
docker exec -it r1 redis-cli -p 7001
# 查看集群状态
info replication
```

结果如下：

```Bash
127.0.0.1:7001> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.150.101,port=7002,state=online,offset=140,lag=1
slave1:ip=192.168.150.101,port=7003,state=online,offset=140,lag=1
master_failover_state:no-failover
master_replid:16d90568498908b322178ca12078114e6c518b86
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:140
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:140
```

可以看到，当前节点`r1:7001`的角色是`master`，有两个slave与其连接：

- `slave0`：`port`是`7002`，也就是`r2`节点
- `slave1`：`port`是`7003`，也就是`r3`节点

### 1.2.3.测试

依次在`r1`、`r2`、`r3`节点上执行下面命令：

```Bash
set num 123

get num
```

你会发现，只有在`r1`这个节点上可以执行`set`命令（**写操作**），其它两个节点只能执行`get`命令（**读操作**）。也就是说读写操作已经分离了。

## 1.3.主从同步原理

![image-20260206162728097](./assets/image-20260206162728097.png)



![image-20260206170417396](./assets/image-20260206170417396.png)

在刚才的主从测试中，我们发现`r1`上写入Redis的数据，在`r2`和`r3`上也能看到，这说明主从之间确实完成了数据同步。

那么这个同步是如何完成的呢？

### 1.3.1.全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程：

![img](./assets/1770364511684-6.png)

这里有一个问题，`master`如何得知`salve`是否是第一次来同步呢？？

有几个概念，可以作为判断依据：

- **`Replication Id`**：简称`replid`，是数据集的标记，replid一致则是同一数据集。每个`master`都有唯一的`replid`，`slave`则会继承`master`节点的`replid`。
- **`offset`**：偏移量，随着记录在`repl_baklog`中的数据增多而逐渐增大。`slave`完成同步时也会记录当前同步的`offset`。如果`slave`的`offset`小于`master`的`offset`，说明`slave`数据落后于`master`，需要更新。

因此`slave`做数据同步，必须向`master`声明自己的`replication id `和`offset`，`master`才可以判断到底需要同步哪些数据。

由于我们在执行`slaveof`命令之前，所有redis节点都是`master`，有自己的`replid`和`offset`。

当我们第一次执行`slaveof`命令，与`master`建立主从关系时，发送的`replid`和`offset`是自己的，与`master`肯定不一致。

`master`判断发现`slave`发送来的`replid`与自己的不一致，说明这是一个全新的slave，就知道要做全量同步了。

`master`会将自己的`replid`和`offset`都发送给这个`slave`，`slave`保存这些信息到本地。自此以后`slave`的`replid`就与`master`一致了。

因此，**master****判断一个节点是否是第一次同步的依据，就是看replid是否一致**。流程如图：

![img](./assets/1770364511685-7.png)

完整流程描述：

- `slave`节点请求增量同步
- `master`节点判断`replid`，发现不一致，拒绝增量同步
- `master`将完整内存数据生成`RDB`，发送`RDB`到`slave`
- `slave`清空本地数据，加载`master`的`RDB`
- `master`将`RDB`期间的命令记录在`repl_baklog`，并持续将log中的命令发送给`slave`
- `slave`执行接收到的命令，保持与`master`之间的同步

来看下`r1`节点的运行日志：

![img](./assets/1770364511685-8.png)

再看下`r2`节点执行`replicaof`命令时的日志：

![img](./assets/1770364511686-9.png)

与我们描述的完全一致。

### 1.3.2.增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输个slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。

什么是增量同步？就是只更新slave与master存在差异的部分数据。如图：

![img](./assets/1770364511686-10.png)

那么master怎么知道slave与自己的数据差异在哪里呢?

### 1.3.3.repl_baklog原理

master怎么知道slave与自己的数据差异在哪里呢?

这就要说到全量同步时的`repl_baklog`文件了。这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。

`repl_baklog`中会记录Redis处理过的命令及`offset`，包括master当前的`offset`，和slave已经拷贝到的`offset`：

![img](./assets/1770364511686-11.png)

slave与master的offset之间的差异，就是salve需要增量拷贝的数据了。

随着不断有数据写入，master的offset逐渐变大，slave也不断的拷贝，追赶master的offset：

![img](./assets/1770364511687-12.png)

直到数组被填满：

![img](./assets/1770364511687-13.png)

此时，如果有新的数据写入，就会覆盖数组中的旧数据。不过，旧的数据只要是绿色的，说明是已经被同步到slave的数据，即便被覆盖了也没什么影响。因为未同步的仅仅是红色部分：

![img](./assets/1770364511687-14.png)

但是，如果slave出现网络阻塞，导致master的`offset`远远超过了slave的`offset`： 

![img](./assets/1770364511688-15.png)

如果master继续写入新数据，master的`offset`就会覆盖`repl_baklog`中旧的数据，直到将slave现在的`offset`也覆盖：

![img](./assets/1770364511688-16.png)

棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却发现自己的`offset`都没有了，无法完成增量同步了。只能做**全量同步**。

`repl_baklog`大小有上限，写满后会覆盖最早的数据。如果slave断开时间过久，导致尚未备份的数据被覆盖，则无法基于`repl_baklog`做增量同步，只能再次全量同步。

## 1.4.主从同步优化

主从同步可以保证主从数据的一致性，非常重要。

可以从以下几个方面来优化Redis主从就集群：

- 在master中配置`repl-diskless-sync  yes`启用无磁盘复制，避免全量同步时的磁盘IO。
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高`repl_baklog`的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用`主-从-从`链式结构，减少master压力

`主-从-从`架构图：

![img](./assets/1770364511688-17.png)

简述全量同步和增量同步区别？

- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave

什么时候执行全量同步？

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时

什么时候执行增量同步？

- slave节点断开又恢复，并且在`repl_baklog`中能找到offset时

# 2.Redis哨兵

主从结构中master节点的作用非常重要，一旦故障就会导致集群不可用。那么有什么办法能保证主从集群的高可用性呢？

## 2.1.哨兵工作原理

Redis提供了`哨兵`（`Sentinel`）机制来监控主从集群监控状态，确保集群的高可用性。

### 2.1.1.哨兵作用

哨兵集群作用原理图：

![img](./assets/1770364511688-18.png)

哨兵的作用如下：

- **状态监控**：`Sentinel` 会不断检查您的`master`和`slave`是否按预期工作
- **故障恢复（failover）**：如果`master`故障，`Sentinel`会将一个`slave`提升为`master`。当故障实例恢复后会成为`slave`
- **状态通知**：`Sentinel`充当`Redis`客户端的服务发现来源，当集群发生`failover`时，会将最新集群信息推送给`Redis`的客户端

那么问题来了，`Sentinel`怎么知道一个Redis节点是否宕机呢？

### 2.1.2.状态监控

`Sentinel`基于心跳机制监测服务状态，每隔1秒向集群的每个节点发送ping命令，并通过实例的响应结果来做出判断：

- **主观下线（sdown）**：如果某sentinel节点发现某Redis节点未在规定时间响应，则认为该节点主观下线。
- **客观下线(odown)**：若超过指定数量（通过`quorum`设置）的sentinel都认为该节点主观下线，则该节点客观下线。quorum值最好超过Sentinel节点数量的一半，Sentinel节点数量至少3台。

如图：

![img](./assets/1770364511689-19.png)

一旦发现master故障，sentinel需要在salve中选择一个作为新的master，选择依据是这样的：

- 首先会判断slave节点与master节点断开时间长短，如果超过`down-after-milliseconds * 10`则会排除该slave节点
- 然后判断slave节点的`slave-priority`值，越小优先级越高，如果是0则永不参与选举（默认都是1）。
- 如果`slave-prority`一样，则判断slave节点的`offset`值，越大说明数据越新，优先级越高
- 最后是判断slave节点的`run_id`大小，越小优先级越高（`通过info server可以查看run_id`）。

对应的官方文档如下：

https://redis.io/docs/management/sentinel/#replica-selection-and-priority

问题来了，当选出一个新的master后，该如何实现身份切换呢？

大概分为两步：

- 在多个`sentinel`中选举一个`leader`
- 由`leader`执行`failover`

### 2.1.3.选举leader

首先，Sentinel集群要选出一个执行`failover`的Sentinel节点，可以成为`leader`。要成为`leader`要满足两个条件：

- 最先获得超过半数的投票
- 获得的投票数不小于`quorum`值

而sentinel投票的原则有两条：

- 优先投票给目前得票最多的
- 如果目前没有任何节点的票，就投给自己

比如有3个sentinel节点，`s1`、`s2`、`s3`，假如`s2`先投票：

- 此时发现没有任何人在投票，那就投给自己。`s2`得1票
- 接着`s1`和`s3`开始投票，发现目前`s2`票最多，于是也投给`s2`，`s2`得3票
- `s2`称为`leader`，开始故障转移

不难看出，**谁先****投票****，谁就会称为****leader**，那什么时候会触发投票呢？

答案是**第一个确认****master****客观下线****的人****会立刻发起****投票****，一定会成为****leader**。

OK，`sentinel`找到`leader`以后，该如何完成`failover`呢？

### 2.1.4.failover

我们举个例子，有一个集群，初始状态下7001为`master`，7002和7003为`slave`：

![img](./assets/1770364511689-20.png)

假如master发生故障，slave1当选。则故障转移的流程如下：

1）`sentinel`给备选的`slave1`节点发送`slaveof no one`命令，让该节点成为`master`

![img](./assets/1770364511689-21.png)

2）`sentinel`给所有其它`slave`发送`slaveof 192.168.150.101 7002` 命令，让这些节点成为新`master`，也就是`7002`的`slave`节点，开始从新的`master`上同步数据。

![img](./assets/1770364511689-22.png)

3）最后，当故障节点恢复后会接收到哨兵信号，执行`slaveof 192.168.150.101 7002`命令，成为`slave`：

![img](./assets/1770364511689-23.png)

## 2.2.搭建哨兵集群

首先，我们停掉之前的redis集群：

```Bash
# 老版本DockerCompose
docker-compose down

# 新版本Docker
docker compose down
```

然后，我们找到课前资料提供的sentinel.conf文件：

![img](./assets/1770364511690-24.png)

其内容如下：

```Bash
sentinel announce-ip "192.168.150.101"
sentinel monitor hmaster 192.168.150.101 7001 2
sentinel down-after-milliseconds hmaster 5000
sentinel failover-timeout hmaster 60000
```

说明：

- `sentinel announce-ip "192.168.150.101"`：声明当前sentinel的ip
- `sentinel monitor hmaster 192.168.150.101 7001 2`：指定集群的主节点信息 
  - `hmaster`：主节点名称，自定义，任意写
  - `192.168.150.101 7001`：主节点的ip和端口
  - `2`：认定`master`下线时的`quorum`值
- `sentinel down-after-milliseconds hmaster 5000`：声明master节点超时多久后被标记下线
- `sentinel failover-timeout hmaster 60000`：在第一次故障转移失败后多久再次重试

我们在虚拟机的`/root/redis`目录下新建3个文件夹：`s1`、`s2`、`s3`:

![img](./assets/1770364511690-25.png)

将课前资料提供的`sentinel.conf`文件分别拷贝一份到3个文件夹中。

接着修改`docker-compose.yaml`文件，内容如下：

```YAML
version: "3.2"

services:
  r1:
    image: redis
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001"]
  r2:
    image: redis
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002", "--slaveof", "192.168.150.101", "7001"]
  r3:
    image: redis
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003", "--slaveof", "192.168.150.101", "7001"]
  s1:
    image: redis
    container_name: s1
    volumes:
      - /root/redis/s1:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27001"]
  s2:
    image: redis
    container_name: s2
    volumes:
      - /root/redis/s2:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27002"]
  s3:
    image: redis
    container_name: s3
    volumes:
      - /root/redis/s3:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27003"]
```

直接运行命令，启动集群：

```Shell
docker-compose up -d
```

运行结果：

![img](./assets/1770364511691-26.png)

我们以s1节点为例，查看其运行日志：

`docker logs -f s1`

```Bash
# Sentinel ID is 8e91bd24ea8e5eb2aee38f1cf796dcb26bb88acf
# +monitor master hmaster 192.168.150.101 7001 quorum 2
* +slave slave 192.168.150.101:7003 192.168.150.101 7003 @ hmaster 192.168.150.101 7001
* +sentinel sentinel 5bafeb97fc16a82b431c339f67b015a51dad5e4f 192.168.150.101 27002 @ hmaster 192.168.150.101 7001
* +sentinel sentinel 56546568a2f7977da36abd3d2d7324c6c3f06b8d 192.168.150.101 27003 @ hmaster 192.168.150.101 7001
* +slave slave 192.168.150.101:7002 192.168.150.101 7002 @ hmaster 192.168.150.101 7001
```

可以看到`sentinel`已经联系到了`7001`这个节点，并且与其它几个哨兵也建立了链接。哨兵信息如下：

- `27001`：`Sentinel ID`是`8e91bd24ea8e5eb2aee38f1cf796dcb26bb88acf`
- `27002`：`Sentinel ID`是`5bafeb97fc16a82b431c339f67b015a51dad5e4f`
- `27003`：`Sentinel ID`是`56546568a2f7977da36abd3d2d7324c6c3f06b8d`

## 2.3.演示failover

接下来，我们演示一下当主节点故障时，哨兵是如何完成集群故障恢复（failover）的。

我们连接`7001`这个`master`节点，然后通过命令让其休眠60秒，模拟宕机：

```Bash
# 连接7001这个master节点，通过sleep模拟服务宕机，60秒后自动恢复
docker exec -it r1 redis-cli -p 7001 DEBUG sleep 60
# docker stop r1
```

稍微等待一段时间后，会发现sentinel节点触发了`failover`：

![img](./assets/1770364511691-27.png)

如果重启7001`docker start r1`

7001变成slave

## 2.4.总结

Sentinel的三个作用是什么？

- 集群监控
- 故障恢复
- 状态通知

Sentinel如何判断一个redis实例是否健康？

- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线（`sdown`）
- 如果大多数sentinel都认为实例主观下线，则判定服务客观下线（`odown`）

故障转移步骤有哪些？

- 首先要在`sentinel`中选出一个`leader`，由leader执行`failover`
- 选定一个`slave`作为新的`master`，执行`slaveof noone`，切换到master模式
- 然后让所有节点都执行`slaveof` 新master
- 修改故障节点配置，添加`slaveof` 新master

sentinel选举leader的依据是什么？

- 票数超过sentinel节点数量1半
- 票数超过quorum数量
- 一般情况下最先发起failover的节点会当选

sentinel从slave中选取master的依据是什么？

- 首先会判断slave节点与master节点断开时间长短，如果超过`down-after-milliseconds`` * 10`则会排除该slave节点
- 然后判断slave节点的`slave-priority`值，越小优先级越高，如果是0则永不参与选举（默认都是1）。
- 如果`slave-prority`一样，则判断slave节点的`offset`值，越大说明数据越新，优先级越高
- 最后是判断slave节点的`run_id`大小，越小优先级越高（`通过info server可以查看run_id`）。



## 2.5.RedisTemplate连接哨兵集群（自学）

分为三步：

- 1）引入依赖
- 2）配置哨兵地址
- 3）配置读写分离

### 2.5.1.引入依赖

就是SpringDataRedis的依赖：

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2.5.2.配置哨兵地址

连接哨兵集群与传统单点模式不同，不再需要设置每一个redis的地址，而是直接指定哨兵地址：

```YAML
spring:
  redis:
    sentinel:
      master: hmaster # 集群名
      nodes: # 哨兵地址列表
        - 192.168.150.101:27001
        - 192.168.150.101:27002
        - 192.168.150.101:27003
```

### 2.5.3.配置读写分离

最后，还要配置读写分离，让java客户端将写请求发送到master节点，读请求发送到slave节点。定义一个bean即可：

```Java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

这个bean中配置的就是读写策略，包括四种：

- `MASTER`：从主节点读取
- `MASTER_PREFERRED`：优先从`master`节点读取，`master`不可用才读取`slave`
- `REPLICA`：从`slave`节点读取
- `REPLICA_PREFERRED`：优先从`slave`节点读取，所有的`slave`都不可用才读取`master`

# 3.Redis分片集群

主从模式可以解决高可用、高并发读的问题。但依然有两个问题没有解决：

- 海量数据存储
- 高并发写

要解决这两个问题就需要用到分片集群了。分片的意思，就是把数据拆分存储到不同节点，这样整个集群的存储数据量就更大了。

Redis分片集群的结构如图：

![img](./assets/1770364511691-28.jpeg)

分片集群特征：

-  集群中有多个master，每个master保存不同分片数据 ，解决海量数据存储问题
-  每个master都可以有多个slave节点 ，确保高可用
-  master之间通过ping监测彼此健康状态 ，类似哨兵作用
-  客户端请求可以访问集群任意节点，最终都会被转发到数据所在节点 

## 3.1.搭建分片集群

Redis分片集群最少也需要3个master节点，由于我们的机器性能有限，我们只给每个master配置1个slave，形成最小的分片集群：

![img](./assets/1770364511692-29.jpeg)

计划部署的节点信息如下：

| 容器名 | 角色   | IP              | 映射端口 |
| ------ | ------ | --------------- | -------- |
| r1     | master | 192.168.150.101 | 7001     |
| r2     | master | 192.168.150.101 | 7002     |
| r3     | master | 192.168.150.101 | 7003     |
| r4     | slave  | 192.168.150.101 | 7004     |
| r5     | slave  | 192.168.150.101 | 7005     |
| r6     | slave  | 192.168.150.101 | 7006     |

### 3.1.1.集群配置

分片集群中的Redis节点必须开启集群模式，一般在配置文件中添加下面参数：

```Bash
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

其中有3个我们没见过的参数：

- `cluster-enabled`：是否开启集群模式
- `cluster-config-file`：集群模式的配置文件名称，无需手动创建，由集群自动维护
- `cluster-node-timeout`：集群中节点之间心跳超时时间

一般搭建部署集群肯定是给每个节点都配置上述参数，不过考虑到我们计划用`docker-compose`部署，因此可以直接在启动命令中指定参数，偷个懒。

在虚拟机的`/root`目录下新建一个`redis-cluster`目录，然后在其中新建一个`docker-compose.yaml`文件，内容如下：

```YAML
version: "3.2"

services:
  r1:
    image: redis
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r2:
    image: redis
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r3:
    image: redis
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r4:
    image: redis
    container_name: r4
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7004", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r5:
    image: redis
    container_name: r5
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7005", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r6:
    image: redis
    container_name: r6
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7006", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
```

**注意**：使用Docker部署Redis集群，network模式必须采用host

### 3.1.2.启动集群

进入`/root/redis-cluster`目录，使用命令启动redis：

```Bash
docker-compose up -d
```

启动成功，可以通过命令查看启动进程：

```Bash
ps -ef | grep redis
# 结果：
root       4822   4743  0 14:29 ?        00:00:02 redis-server *:7002 [cluster]
root       4827   4745  0 14:29 ?        00:00:01 redis-server *:7005 [cluster]
root       4897   4778  0 14:29 ?        00:00:01 redis-server *:7004 [cluster]
root       4903   4759  0 14:29 ?        00:00:01 redis-server *:7006 [cluster]
root       4905   4775  0 14:29 ?        00:00:02 redis-server *:7001 [cluster]
root       4912   4732  0 14:29 ?        00:00:01 redis-server *:7003 [cluster]
```

可以发现每个redis节点都以cluster模式运行。不过节点与节点之间并未建立连接。

接下来，我们使用命令创建集群：

```Bash
# 进入任意节点容器
docker exec -it r1 bash
# 然后，执行命令
redis-cli --cluster create --cluster-replicas 1 \
192.168.150.101:7001 192.168.150.101:7002 192.168.150.101:7003 \
192.168.150.101:7004 192.168.150.101:7005 192.168.150.101:7006
```

命令说明：

- `redis-cli --cluster`：代表集群操作命令
- `create`：代表是创建集群
- `--cluster-replicas 1` ：指定集群中每个`master`的副本个数为1
  - 此时`节点总数 ÷ (replicas + 1)` 得到的就是`master`的数量`n`。因此节点列表中的前`n`个节点就是`master`，其它节点都是`slave`节点，随机分配到不同`master`

输入命令后控制台会弹出下面的信息：

![img](./assets/1770364511693-30.png)

这里展示了集群中`master`与`slave`节点分配情况，并询问你是否同意。节点信息如下：

- `7001`是`master`，节点`id`后6位是`da134f`
- `7002`是`master`，节点`id`后6位是`862fa0`
- `7003`是`master`，节点`id`后6位是`ad5083`
- `7004`是`slave`，节点`id`后6位是`391f8b`，认`ad5083`（7003）为`master`
- `7005`是`slave`，节点`id`后6位是`e152cd`，认`da134f`（7001）为`master`
- `7006`是`slave`，节点`id`后6位是`4a018a`，认`862fa0`（7002）为`master`

输入`yes`然后回车。会发现集群开始创建，并输出下列信息：

![img](./assets/1770364511694-31.png)

接着，我们可以通过命令查看集群状态：

```Bash
redis-cli -p 7001 cluster nodes
```

结果：

![img](./assets/1770364511695-32.png)

## 3.2.散列插槽

数据要分片存储到不同的Redis节点，肯定需要有分片的依据，这样下次查询的时候才能知道去哪个节点查询。很多数据分片都会采用一致性hash算法。而Redis则是利用散列插槽（**`hash slot`**）的方式实现数据分片。

详见官方文档：

https://redis.io/docs/management/scaling/#redis-cluster-101

在Redis集群中，共有16384个`hash slots`，集群中的每一个master节点都会分配一定数量的`hash slots`。具体的分配在集群创建时就已经指定了：

![img](./assets/1770364511696-33.png)

如图中所示：

- Master[0]，本例中就是7001节点，分配到的插槽是0~5460
- Master[1]，本例中就是7002节点，分配到的插槽是5461~10922
- Master[2]，本例中就是7003节点，分配到的插槽是10923~16383

当我们读写数据时，Redis基于`CRC16` 算法对`key`做`hash`运算，得到的结果与`16384`取余，就计算出了这个`key`的`slot`值。然后到`slot`所在的Redis节点执行读写操作。

不过`hash slot`的计算也分两种情况：

- 当`key`中包含`{}`时，根据`{}`之间的字符串计算`hash slot`
- 当`key`中不包含`{}`时，则根据整个`key`字符串计算`hash slot`

例如：

- key是`user`，则根据`user`来计算hash slot
- key是`user:{age}`，则根据`age`来计算hash slot

我们来测试一下，先于`7001`建立连接：

```Bash
# 进入容器
docker exec -it r1 bash
# 进入redis-cli
redis-cli -p 7001
# 测试
set user jack
```

会发现报错了：

![img](./assets/1770364511696-34.png)

提示我们`MOVED 5474`，其实就是经过计算，得出`user`这个`key`的`hash slot` 是`5474`，而`5474`是在`7002`节点，不能在`7001`上写入！！

说好的任意节点都可以读写呢？

这是因为我们连接的方式有问题，连接集群时，要加`-c`参数：

```Bash
# 通过7001连接集群
redis-cli -c -p 7001
# 存入数据
set user jack
```

结果如下：

![img](./assets/1770364511696-35.png)

可以看到，客户端自动跳转到了`5474`这个`slot`所在的`7002`节点。

现在，我们添加一个新的key，这次加上`{}`：

```Bash
# 试一下key中带{}
set user:{age} 21

# 再试一下key中不带{}
set age 20
```

结果如下：

![img](./assets/1770364511697-36.png)

可以看到`user:{age}`和`age`计算出的`slot`都是`741`。

## 3.3.故障转移

分片集群的节点之间会互相通过ping的方式做心跳检测，超时未回应的节点会被标记为下线状态。当发现master下线时，会将这个master的某个slave提升为master。

我们先打开一个控制台窗口，利用命令监测集群状态：

```Bash
watch docker exec -it r1 redis-cli -p 7001 cluster nodes
```

命令前面的watch可以每隔一段时间刷新执行结果，方便我们实时监控集群状态变化。

接着，我们故技重施，利用命令让某个master节点休眠。比如这里我们让`7002`节点休眠，打开一个新的ssh控制台，输入下面命令：

```Bash
docker exec -it r2 redis-cli -p 7002 DEBUG sleep 30
```

可以观察到，集群发现7002宕机，标记为下线：

![img](./assets/1770364511697-37.png)

过了一段时间后，7002原本的小弟7006变成了`master`：

![img](./assets/1770364511697-38.png)

而7002被标记为`slave`，而且其`master`正好是7006，主从地位互换。

## 3.4.总结

Redis分片集群如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

如何将同一类数据固定的保存在同一个Redis实例？

- Redis计算key的插槽值时会判断key中是否包含`{}`，如果有则基于`{}`内的字符计算插槽
- 数据的key中可以加入`{类型}`，例如key都以`{typeId}`为前缀，这样同类型数据计算的插槽一定相同

## 3.5.Java客户端连接分片集群（选学）

RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致，参考`2.5节`：

1）引入redis的starter依赖

2）配置分片集群地址

3）配置读写分离

与哨兵模式相比，其中只有分片集群的配置方式略有差异，如下：

```YAML
spring:
  redis:
    cluster:
      nodes:
        - 192.168.150.101:7001
        - 192.168.150.101:7002
        - 192.168.150.101:7003
        - 192.168.150.101:8001
        - 192.168.150.101:8002
        - 192.168.150.101:8003
```

# 4.Redis数据结构

我们常用的Redis数据类型有5种，分别是：

- String
- List
- Set
- SortedSet
- Hash

还有一些高级数据类型，比如Bitmap、HyperLogLog、GEO等，其底层都是基于上述5种基本数据类型。因此在Redis的源码中，其实只有5种数据类型。

## 4.1.RedisObject

不管是任何一种数据类型，最终都会封装为RedisObject格式，它是一种结构体，C语言中的一种结构，可以理解为Java中的类。

结构大概是这样的：

![img](./assets/1770364511697-39.png)

可以看到整个结构体中并不包含真实的数据，仅仅是对象头信息，内存占用的大小为4+4+24+32+64 = 128bit

也就是16字节，然后指针`ptr`指针指向的才是真实数据存储的内存地址。所以RedisObject的内存开销是很大的。

属性中的`encoding`就是当前对象底层采用的**数据结构**或**编码方式**，可选的有11种之多：

| **编号** | **编码方式**            | **说明**               |
| :------- | :---------------------- | :--------------------- |
| 0        | OBJ_ENCODING_RAW        | raw编码动态字符串      |
| 1        | OBJ_ENCODING_INT        | long类型的整数的字符串 |
| 2        | OBJ_ENCODING_HT         | hash表（也叫dict）     |
| 3        | OBJ_ENCODING_ZIPMAP     | 已废弃                 |
| 4        | OBJ_ENCODING_LINKEDLIST | 双端链表               |
| 5        | OBJ_ENCODING_ZIPLIST    | 压缩列表               |
| 6        | OBJ_ENCODING_INTSET     | 整数集合               |
| 7        | OBJ_ENCODING_SKIPLIST   | 跳表                   |
| 8        | OBJ_ENCODING_EMBSTR     | embstr编码的动态字符串 |
| 9        | OBJ_ENCODING_QUICKLIST  | 快速列表               |
| 10       | OBJ_ENCODING_STREAM     | Stream流               |
| 11       | OBJ_ENCODING_LISTPACK   | 紧凑列表               |

Redis中的5种不同的数据类型采用的底层数据结构和编码方式如下：

| **数据类型** | **编码方式**                                                 |
| :----------- | :----------------------------------------------------------- |
| STRING       | `int`、`embstr`、`raw`                                       |
| LIST         | `LinkedList和ZipList`(3.2以前)、`QuickList`（3.2以后）       |
| SET          | `intset`、`HT`                                               |
| ZSET         | `ZipList`（7.0以前）、`Listpack`（7.0以后）、`HT`、`SkipList` |
| HASH         | `ZipList`（7.0以前）、`Listpack`（7.0以后）、`HT`            |



## 4.2.SkipList

SkipList的特点： 

- 跳跃表是一个有序的双向链表 
- 每个节点都可以包含多层指针,层数是1到32之间的随机数 
- 不同层指针到下一个节点的跨度不同，层级越高，跨度越大 
- 增删改查效率与红黑树基本一致，实现却更简单。但空间复杂度更高

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：

- 元素按照升序排列存储
- 节点可能包含多个指针，指针跨度不同。

传统链表只有指向前后元素的指针，因此只能顺序依次访问。如果查找的元素在链表中间，查询的效率会比较低。而SkipList则不同，它内部包含跨度不同的多级指针，可以让我们跳跃查找链表中间的元素，效率非常高。

其结构如图：

![img](./assets/1770364511697-40.png)

我们可以看到1号元素就有指向3、5、10的多个指针，查询时就可以跳跃查找。例如我们要找大小为14的元素，查找的流程是这样的：

![img](./assets/1770364511697-41.png)

- 首先找元素1节点最高级指针，也就是4级指针，起始元素大小为1，指针跨度为9，可以判断出目标元素大小为10。由于14比10大，肯定要从10这个元素向下接着找。
- 找到10这个元素，发现10这个元素的最高级指针跨度为5，判断出目标元素大小为15，大于14，需要判断下级指针
- 10这个元素的2级指针跨度为3，判断出目标元素为13，小于14，因此要基于元素13接着找
- 13这个元素最高级级指针跨度为2，判断出目标元素为15，比14大，需要判断下级指针。
- 13的下级指针跨度为1，因此目标元素是14，刚好于目标一致，找到。

这种多级指针的查询方式就避免了传统链表的逐个遍历导致的查询效率下降问题。在对有序数据做随机查询和排序时效率非常高。

跳表的结构体如下：

```C
typedef struct zskiplist {
    // 头尾节点指针
    struct zskiplistNode *header, *tail;
    // 节点数量
    unsigned long length;
    // 最大的索引层级
    int level;
} zskiplist;
```

可以看到SkipList主要属性是header和tail，也就是头尾指针，因此它是支持双向遍历的。

跳表中节点的结构体如下：

```C
typedef struct zskiplistNode {
    sds ele; // 节点存储的字符串
    double score;// 节点分数，排序、查找用
    struct zskiplistNode *backward; // 前一个节点指针
    struct zskiplistLevel {
        struct zskiplistNode *forward; // 下一个节点指针
        unsigned long span; // 索引跨度
    } level[]; // 多级索引数组
} zskiplistNode;
```

每个节点中都包含ele和score两个属性，其中score是得分，也就是节点排序的依据。ele则是节点存储的字符串数据指针。

其内存结构如下：

![img](./assets/1770364511698-42.png)

## 4.3.SortedSet

![image-20260207160113678](./assets/image-20260207160113678.png)

**面试题**：Redis的`SortedSet`底层的数据结构是怎样的？

**答**：SortedSet是有序集合，底层的存储的每个数据都包含element和score两个值。score是得分，element则是字符串值。SortedSet会根据每个element的score值排序，形成有序集合。

- 每组数据都包含score和member
- member唯一
- 可根据score排序

它支持的操作很多，比如：

- 根据element查询score值
- 按照score值升序或降序查询element

要实现根据element查询对应的score值，就必须实现element与score之间的键值映射。SortedSet底层是基于**HashTable**来实现的。

要实现对score值排序，并且查询效率还高，就需要有一种高效的有序数据结构，SortedSet是基于**跳表**实现的。

加分项：因为SortedSet底层需要用到两种数据结构，对内存占用比较高。因此Redis底层会对SortedSet中的元素大小做判断。如果**元素大小****小于128**且**每个元素都小于64字节**，SortedSet底层会采用**ZipList**，也就是**压缩列**表来代替**HashTable**和**SkipList**

不过，`ZipList`存在连锁更新问题，因此而在Redis7.0版本以后，`ZipList`又被替换为**Listpack**（紧凑列表）。

Redis源码中`zset`，也就是`SortedSet`的结构体如下：

```C
typedef struct zset {
    dict *dict; // dict，底层就是HashTable
    zskiplist *zsl; // 跳表
} zset;
```

其内存结构如图：

![img](./assets/1770364511698-43.png)

# 5.Redis内存回收

Redis之所以性能强，最主要的原因就是基于内存存储。然而单节点的Redis其内存大小不宜过大，会影响持久化或主从同步性能。

我们可以通过修改redis.conf文件，添加下面的配置来配置Redis的最大内存：

```Properties
maxmemory 1gb
```

当内存达到上限，就无法存储更多数据了。因此，Redis内部会有两套内存回收的策略：

- 内存过期策略
- 内存淘汰策略

## 5.1.内存过期处理

存入Redis中的数据可以配置过期时间，到期后再次访问会发现这些数据都不存在了，也就是被过期清理了。

### 5.1.1.过期命令

Redis中通过`expire`命令可以给KEY设置`TTL`（过期时间），例如：

```Bash
# 写入一条数据
set num 123
# 设置20秒过期时间
expire num 20
```

不过set命令本身也可以支持过期时间的设置：

```Shell
# 写入一条数据并设置20s过期时间
set num EX 20
```

当过期时间到了以后，再去查询数据，会发现数据已经不存在。 

### 5.1.2.过期策略

![image-20260207160958513](./assets/image-20260207160958513.png)

那么问题来了：

- Redis如何判断一个KEY是否过期呢？
- Redis又是何时删除过期KEY的呢？

Redis不管有多少种数据类型，本质是一个`KEY-VALUE`的键值型数据库，而这种键值映射底层正式基于HashTable来实现的，在Redis中叫做Dict.

来看下RedisDB的底层源码：

```C
typedef struct redisDb {
    dict dict;                 / The keyspace for this DB , 也就是存放KEY和VALUE的哈希表*/
    dict *expires;              /* 同样是哈希表，但保存的是设置了TTL的KEY，及其到期时间*/
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
    dict *ready_keys;           /* Blocked keys that received a PUSH */
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS /
    int id;                     / Database ID, 0 ~ 15 /
    long long avg_ttl;          / Average TTL, just for stats /
    unsigned long expires_cursor; / Cursor of the active expire cycle. */
    list *defrag_later;         /* List of key names to attempt to defrag one by one, gradually. */
} redisDb;
```

现在回答第一个问题：

**面试题**：Redis如何判断KEY是否过期呢？

**答**：在Redis中会有两个Dict，也就是HashTable，其中一个记录KEY-VALUE键值对，另一个记录KEY和过期时间。要判断一个KEY是否过期，只需要到记录过期时间的Dict中根据KEY查询即可。

Redis是何时删除过期KEY的呢？

Redis并不会在KEY过期时立刻删除KEY，因为要实现这样的效果就必须给每一个过期的KEY设置时钟，并监控这些KEY的过期状态。无论对CPU还是内存都会带来极大的负担。

Redis的过期KEY删除策略有两种：

- 惰性删除
- 周期删除

**惰性删除**，顾明思议就是过期后不会立刻删除。那在什么时候删除呢？

Redis会在每次访问KEY的时候判断当前KEY有没有设置过期时间，如果有，过期时间是否已经到期。对应的源码如下：

```C
// db.c
// 寻找要执行写操作的key
robj *lookupKeyWriteWithFlags(redisDb *db, robj *key, int flags) {
    // 检查key是否过期，如果过期则删除
    expireIfNeeded(db,key);
    return lookupKey(db,key,flags);
}

// 寻找要执行读操作的key
robj *lookupKeyReadWithFlags(redisDb *db, robj *key, int flags) {
    robj *val;
    // 检查key是否过期，如果过期则删除
    if (expireIfNeeded(db,key) == 1) {
        // 略 ...
    }
    val = lookupKey(db,key,flags);
    if (val == NULL)
        goto keymiss;
    server.stat_keyspace_hits++;
    return val;
}
```

**周期删除**：顾明思议是通过一个定时任务，周期性的抽样部分过期的key，然后执行删除。

执行周期有两种：

- **SLOW模式：**Redis会设置一个定时任务`serverCron()`，按照`server.hz`的频率来执行过期key清理
- **FAST模式：**Redis的每个事件循环前执行过期key清理（事件循环就是NIO事件处理的循环）。

**SLOW**模式规则：

- ① 执行频率受`server.hz`影响，默认为10，即每秒执行10次，每个执行周期100ms。
- ② 执行清理耗时不超过一次执行周期的25%，即25ms.
- ③ 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
- ④ 如果没达到时间上限（25ms）并且过期key比例大于10%，再进行一次抽样，否则结束

**FAST**模式规则（过期key比例小于10%不执行）：

- ① 执行频率受`beforeSleep()`调用频率影响，但两次FAST模式间隔不低于2ms
- ② 执行清理耗时不超过1ms
- ③ 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
- ④ 如果没达到时间上限（1ms）并且过期key比例大于10%，再进行一次抽样，否则结束

## 5.2.内存淘汰策略

对于某些特别依赖于Redis的项目而言，仅仅依靠过期KEY清理是不够的，内存可能很快就达到上限。因此Redis允许设置内存告警阈值，当内存使用达到阈值时就会主动挑选部分KEY删除以释放更多内存。这叫做**内存淘汰**机制。

### 5.2.1.内存淘汰时机

那么问题来了，当内存达到阈值时执行内存淘汰，但问题是Redis什么时候会执去判断内存是否达到预警呢？

Redis每次执行任何命令时，都会判断内存是否达到阈值：

```C
// server.c中处理命令的部分源码
int processCommand(client *c) {
    // ... 略
    if (server.maxmemory && !server.lua_timedout) {
        // 调用performEvictions()方法尝试进行内存淘汰
        int out_of_memory = (performEvictions() == EVICT_FAIL);
        // ... 略
        if (out_of_memory && reject_cmd_on_oom) {
            // 如果内存依然不足，直接拒绝命令
            rejectCommand(c, shared.oomerr);
            return C_OK;
        }
    }
}
```

### 5.2.2.淘汰策略

好了，知道什么时候尝试淘汰了，那具体Redis是如何判断该淘汰哪些`Key`的呢？

Redis支持8种不同的内存淘汰策略：

- `noeviction`： 不淘汰任何key，但是内存满时不允许写入新数据，默认就是这种策略。
- `volatile``-ttl`： 对设置了TTL的key，比较key的剩余TTL值，TTL越小越先被淘汰
- `allkeys``-random`：对全体key ，随机进行淘汰。也就是直接从db->dict中随机挑选
- `volatile-random`：对设置了TTL的key ，随机进行淘汰。也就是从db->expires中随机挑选。
- `allkeys-lru`： 对全体key，基于LRU算法进行淘汰
- `volatile-lru`： 对设置了TTL的key，基于LRU算法进行淘汰
- `allkeys-lfu`： 对全体key，基于LFU算法进行淘汰
- `volatile-lfu`： 对设置了TTL的key，基于LFI算法进行淘汰

比较容易混淆的有两个算法：

- **LRU**（**`L`**`east `**`R`**`ecently `**`U`**`sed`），最近最久未使用。用当前时间减去最后一次访问时间，这个值越大则淘汰优先级越高。
- **LFU**（**`L`**`east `**`F`**`requently `**`U`**`sed`），最少频率使用。会统计每个key的访问频率，值越小淘汰优先级越高。

Redis怎么知道某个KEY的`最近一次访问时间`或者是`访问频率`呢？

还记不记得之前讲过的RedisObject的结构？

回忆一下：

![img](./assets/1770364511698-44.png)

其中的`lru`就是记录最近一次访问时间和访问频率的。当然，你选择`LRU`和`LFU`时的记录方式不同：

- **LRU**：以秒为单位记录最近一次访问时间，长度24bit
- **LFU**：高16位以分钟为单位记录最近一次访问时间，低8位记录逻辑访问次数

时间就不说了，那么逻辑访问次数又是怎么回事呢？8位无符号数字最大才255，访问次数超过255怎么办？

这就要聊起Redis的**逻辑访问次数**算法了，LFU的访问次数之所以叫做**逻辑访问次数**，是因为并不是每次key被访问都计数，而是通过运算：

- ① 生成`[0,1)`之间的随机数`R`
- ② 计算 `1/(``旧次数`` * lfu_log_factor + 1)`，记录为`P`， `lfu_log_factor`默认为10
- ③ 如果 `R` < `P `，则计数器 `+1`，且最大不超过255
- ④ 访问次数会随时间衰减，距离上一次访问时间每隔 `lfu_decay_time` 分钟(默认1) ，计数器`-1`

显然LFU的基于访问频率的统计更符合我们的淘汰目标，因此**官方推荐使用LFU算法。**

算法我们弄明白了，不过这里大家要注意一下：Redis中的`KEY`可能有数百万甚至更多，每个KEY都有自己访问时间或者逻辑访问次数。我们要找出时间最早的或者访问次数最小的，难道要把Redis中**所有数据排序**？

要知道Redis的内存淘汰是在每次执行命令时处理的。如果每次执行命令都先对全量数据做内存排序，那命令的执行时长肯定会非常长，这是不现实的。

所以Redis采取的是**抽样法**，即每次抽样一定数量（`maxmemory_smples`）的key，然后基于内存策略做排序，找出淘汰优先级最高的，删除这个key。这就导致Redis的算法并不是真正的**LRU**，而是一种基于抽样的**近似LRU算法**。

不过，在Redis3.0以后改进了这个算法，引入了一个淘汰候选池，抽样的key要与候选池中的key比较淘汰优先级，优先级更高的才会被放入候选池。然后在候选池中找出优先级最高的淘汰掉，这就使算法的结果更接近与真正的LRU算法了。特别是在抽样值较高的情况下（例如10），可以达到与真正的LRU接近的效果。

这也是官方给出的真正LRU与近似LRU的结果对比：

![img](./assets/1770364511698-45.png)

你可以在图表中看到三种颜色的点形成三个不同的带，每个点就是一个加入的`KEY`。

- 浅灰色带是被驱逐的对象
- 灰色带是没有被驱逐的对象
- 绿色带是被添加的对象

## 5.3.总结

**面试题**：**Redis如何判断KEY是否过期呢？**

**答**：在Redis中会有两个Dict，也就是HashTable，其中一个记录KEY-VALUE键值对，另一个记录KEY和过期时间。要判断一个KEY是否过期，只需要到记录过期时间的Dict中根据KEY查询即可。

**面试题**：**Redis何时删除过期KEY？如何删除？**

**答**：Redis的过期KEY处理有两种策略，分别是惰性删除和周期删除。

**惰性删除**是指在每次用户访问某个KEY时，判断KEY的过期时间：如果过期则删除；如果未过期则忽略。

**周期删除**有两种模式：

- **SLOW**模式：通过一个定时任务，定期的抽样部分带有TTL的KEY，判断其是否过期。默认情况下定时任务的执行频率是每秒10次，但每次执行不能超过25毫秒。如果执行抽样后发现时间还有剩余，并且过期KEY的比例较高，则会多次抽样。
- **FAST**模式：在Redis每次处理NIO事件之前，都会抽样部分带有TTL的KEY，判断是否过期，因此执行频率较高。但是每次执行时长不能超过1ms，如果时间充足并且过期KEY比例过高，也会多次抽样

**面试题**：**当Redis****内存****不足时会怎么做**？

**答**：这取决于配置的内存淘汰策略，Redis支持很多种内存淘汰策略，例如LRU、LFU、Random. 但默认的策略是直接拒绝新的写入请求。而如果设置了其它策略，则会在每次执行命令后判断占用内存是否达到阈值。如果达到阈值则会基于配置的淘汰策略尝试进行内存淘汰，直到占用内存小于阈值为止。

**面试题**：**那你能聊聊****LRU****和****LFU****吗**？

**答**：`LRU`是最近最久未使用。Redis的Key都是RedisObject，当启用LRU算法后，Redis会在Key的头信息中使用24个bit记录每个key的最近一次使用的时间`lru`。每次需要内存淘汰时，就会抽样一部分KEY，找出其中空闲时间最长的，也就是`now - lru`结果最大的，然后将其删除。如果内存依然不足，就重复这个过程。

由于采用了抽样来计算，这种算法只能说是一种近似LRU算法。因此在Redis4.0以后又引入了`LFU`算法，这种算法是统计最近最少使用，也就是按key的访问频率来统计。当启用LFU算法后，Redis会在key的头信息中使用24bit记录最近一次使用时间和逻辑访问频率。其中高16位是以分钟为单位的最近访问时间，后8位是逻辑访问次数。与LFU类似，每次需要内存淘汰时，就会抽样一部分KEY，找出其中逻辑访问次数最小的，将其淘汰。

**面试题**：**逻辑访问次数是如何计算的**？

**答**：由于记录访问次数的只有`8bit`，即便是无符号数，最大值只有255，不可能记录真实的访问次数。因此Redis统计的其实是逻辑访问次数。这其中有一个计算公式，会根据当前的访问次数做计算，结果要么是次数`+1`，要么是次数不变。但随着当前访问次数越大，`+1`的概率也会越低，并且最大值不超过255.

除此以外，逻辑访问次数还有一个衰减周期，默认为1分钟，即每隔1分钟逻辑访问次数会`-1`。这样逻辑访问次数就能基本反映出一个`key`的访问热度了。

# 6.缓存问题

Redis经常被用作缓存，而缓存在使用的过程中存在很多问题需要解决。例如：

- 缓存的数据一致性问题
- 缓存击穿
- 缓存穿透
- 缓存雪崩

## 6.1.缓存一致性

我们先看下目前企业用的最多的缓存模型。缓存的通用模型有三种：

- `Cache Aside`：有缓存调用者自己维护数据库与缓存的一致性。即：
  - 查询时：命中则直接返回，未命中则查询数据库并写入缓存
  - 更新时：更新数据库并删除缓存，查询时自然会更新缓存
- `Read/Write Through`：数据库自己维护一份缓存，底层实现对调用者透明。底层实现：
  - 查询时：命中则直接返回，未命中则查询数据库并写入缓存
  - 更新时：判断缓存是否存在，不存在直接更新数据库。存在则更新缓存，同步更新数据库
- `Write Behind Cahing`：读写操作都直接操作缓存，由线程异步的将缓存数据同步到数据库

目前企业中使用最多的就是`Cache Aside`模式，因为实现起来非常简单。但缺点也很明显，就是无法保证数据库与缓存的强一致性。为什么呢？我们一起来分析一下。

`Cache Aside`的写操作是要在更新数据库的同时删除缓存，那为什么不选择更新数据库的同时更新缓存，而是删除呢？

原因很简单，假如一段时间内无人查询，但是有多次更新，那这些更新都属于无效更新。采用删除方案也就是延迟更新，什么时候有人查询了，什么时候更新。

那到底是先更新数据库再删除缓存，还是先删除缓存再更新数据库呢？

现在假设有两个线程，一个来更新数据，一个来查询数据。我们分别分析两种策略的表现。

我们先分析策略1，先更新数据库再删除缓存：

**正常情况**

![img](./assets/1770364511699-46.png)

**异常情况**

![img](./assets/1770364511700-47.png)

异常情况说明：

- 线程1删除缓存后，还没来得及更新数据库，
- 此时线程2来查询，发现缓存未命中，于是查询数据库，写入缓存。由于此时数据库尚未更新，查询的是旧数据。也就是说刚才的删除白删了，缓存又变成旧数据了。
- 然后线程1更新数据库，此时数据库是新数据，缓存是旧数据

由于更新数据库的操作本身比较耗时，在期间有线程来查询数据库并更新缓存的概率非常高。因此不推荐这种方案。

再来看策略2，先更新数据库再删除缓存：

**正常情况**

![img](./assets/1770364511700-48.png)

**异常情况**

![img](./assets/1770364511701-49.png)

异常情况说明：

- 线程1查询缓存未命中，于是去查询数据库，查询到旧数据
- 线程1将数据写入缓存之前，线程2来了，更新数据库，删除缓存
- 线程1执行写入缓存的操作，写入旧数据

可以发现，异常状态发生的概率极为苛刻，线程1必须是查询数据库已经完成，但是缓存尚未写入之前。线程2要完成更新数据库同时删除缓存的两个操作。要知道线程1执行写缓存的速度在毫秒之间，速度非常快，在这么短的时间要完成数据库和缓存的操作，概率非常之低。

**综上**，添加缓存的目的是为了提高系统性能，而你要付出的代价就是缓存与数据库的强一致性。如果你要求数据库与缓存的强一致，那就需要加锁避免并行读写。但这就降低了性能，与缓存的目标背道而驰。

因此不管任何缓存同步方案最终的目的都是尽可能保证最终一致性，降低发生不一致的概率。我们采用先更新数据库再删除缓存的方案，已经将这种概率降到足够低，目的已经达到了。

同时我们还要给缓存加上过期时间，一旦发生缓存不一致，当缓存过期后会重新加载，数据最终还是能保证一致。这就可以作为一个兜底方案。

## 6.2.缓存穿透

![image-20260208113832384](./assets/image-20260208113832384.png)

什么是缓存穿透呢？

我们知道，当请求查询缓存未命中时，需要查询数据库以加载缓存。但是大家思考一下这样的场景：

> 如果我访问一个数据库中也不存在的数据。会出现什么现象？

由于数据库中不存在该数据，那么缓存中肯定也不存在。因此不管请求该数据多少次，缓存永远不可能建立，请求永远会直达数据库。

假如有不怀好意的人，开启很多线程频繁的访问一个数据库中也不存在的数据。由于缓存不可能生效，那么所有的请求都访问数据库，可能就会导致数据库因过高的压力而宕机。

解决这个问题有两种思路：

- 缓存空值
- 布隆过滤器

### 6.2.1.缓存空值

简单来说，就是当我们发现请求的数据即不存在与缓存，也不存在与数据库时，将空值缓存到Redis，避免频繁查询数据库。实现思路如下：

![img](./assets/1770364511701-50.png)

优点：

- 实现简单，维护方便

缺点：

- 额外的内存消耗

### 6.2.2.布隆过滤器

![image-20260208114216434](./assets/image-20260208114216434.png)

布隆过滤是一种数据统计的算法，用于检索一个元素是否存在一个集合中。

一般我们判断集合中是否存在元素，都会先把元素保存到类似于树、哈希表等数据结构中，然后利用这些结构查询效率高的特点来快速匹配判断。但是随着元素数量越来越多，这种模式对内存的占用也越来越大，检索的速度也会越来越慢。而布隆过滤的内存占用小，查询效率却很高。

布隆过滤首先需要一个很长的bit数组，默认数组中每一位都是0. 

![img](./assets/1770364511702-51.png)

然后还需要`K`个`hash`函数，将元素基于这些hash函数做运算的结果映射到bit数组的不同位置，并将这些位置置为1，例如现在k=3：

- `hello`经过运算得到3个角标：1、5、12
- `world`经过运算得到3个角标：8、17、21
- `java`经过运算得到3个角标：17、25、28

则需要将每个元素对应角标位置置为1：

![img](./assets/1770364511702-52.png)

此时，我们要判断元素是否存在，只需要再次基于`K`个`hash`函数做运算， 得到`K`个角标，判断每个角标的位置是不是1：

- 只要全是1，就证明元素存在
- 任意位置为0，就证明元素一定不存在

假如某个元素本身并不存在，也没添加到布隆过滤器过。但是由于存在hash碰撞的可能性，这就会出现这个元素计算出的角标已经被其它元素置为1的情况。那么这个元素也会被误判为已经存在。

因此，布隆过滤器的判断存在误差：

- 当布隆过滤器认为元素不存在时，它**肯定不存在**
- 当布隆过滤器认为元素存在时，它**可能存在，也可能不存在**

当`bit`数组越大、`Hash`函数`K`越复杂，`K`越大时，这个误判的概率也就越低。由于采用`bit`数组来标示数据，即便`4,294,967,296`个`bit`位，也只占`512mb`的空间

我们可以把数据库中的数据利用布隆过滤器标记出来，当用户请求缓存未命中时，先基于布隆过滤器判断。如果不存在则直接拒绝请求，存在则去查询数据库。尽管布隆过滤存在误差，但一般都在0.01%左右，可以大大减少数据库压力。

使用布隆过滤后的流程如下：

![img](./assets/1770364511702-53.png)

## 6.3.缓存雪崩

缓存雪崩是指在同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。

![img](./assets/1770364511702-54.png)

常见的解决方案有：

- 给不同的Key的TTL添加随机值，这样KEY的过期时间不同，不会大量KEY同时过期
- 利用Redis集群提高服务的可用性，避免缓存服务宕机
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存，比如先查询本地缓存，本地缓存未命中再查询Redis，Redis未命中再查询数据库。即便Redis宕机，也还有本地缓存可以抗压力

## 6.4.缓存击穿

**缓存击穿**问题也叫**热点Key**问题，就是一个被高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

由于我们采用的是`Cache Aside`模式，当缓存失效时需要下次查询时才会更新缓存。当某个key缓存失效时，如果这个key是热点key，并发访问量比较高。就会在一瞬间涌入大量请求，都发现缓存未命中，于是都会去查询数据库，尝试重建缓存。可能一瞬间就把数据库压垮了。

![img](./assets/1770364511702-55.png)

如上图所示：

- 线程1发现缓存未命中，准备查询数据库，重建缓存，但是因为数据比较复杂，导致查询数据库耗时较久
- 在这个过程中，一下次来了3个新的线程，就都会发现缓存未命中，都去查询数据库
- 数据库压力激增

常见的解决方案有两种：

- 互斥锁：给重建缓存逻辑加锁，避免多线程同时指向
- 逻辑过期：热点key不要设置过期时间，在活动结束后手动删除。

基于互斥锁的方案如图：

![img](./assets/1770364511702-56.png)

逻辑过期的思路如图：

![img](./assets/1770364511702-57.png)

## 6.5.面试总结

**面试题**：**如何保证缓存的****双写一致性**？

**答**：缓存的双写一致性很难保证强一致，只能尽可能降低不一致的概率，确保最终一致。我们项目中采用的是`Cache Aside`模式。简单来说，就是在更新数据库之后删除缓存；在查询时先查询缓存，如果未命中则查询数据库并写入缓存。同时我们会给缓存设置过期时间作为兜底方案，如果真的出现了不一致的情况，也可以通过缓存过期来保证最终一致。

**追问**：为什么不采用延迟双删机制？

**答**：延迟双删的第一次删除并没有实际意义，第二次采用延迟删除主要是解决数据库主从同步的延迟问题，我认为这是数据库主从的一致性问题，与缓存同步无关。既然主节点数据已经更新，Redis的缓存理应更新。而且延迟双删会增加缓存业务复杂度，也没能完全避免缓存一致性问题，投入回报比太低。

**面试题**：**如何解决缓存穿透问题**？

**答**：缓存穿透也可以说是穿透攻击，具体来说是因为请求访问到了数据库不存在的值，这样缓存无法命中，必然访问数据库。如果高并发的访问这样的接口，会给数据库带来巨大压力。

我们项目中都是基于布隆过滤器来解决缓存穿透问题的，当缓存未命中时基于布隆过滤器判断数据是否存在。如果不存在则不去访问数据库。

当然，也可以使用缓存空值的方式解决，不过这种方案比较浪费内存。

**面试题**：**如何解决缓存雪崩问题**？

**答**：缓存雪崩的常见原因有两个，第一是因为大量key同时过期。针对问这个题我们可以可以给缓存key设置不同的TTL值，避免key同时过期。

第二个原因是Redis宕机导致缓存不可用。针对这个问题我们可以利用集群提高Redis的可用性。也可以添加多级缓存，当Redis宕机时还有本地缓存可用。

**面试题**：**如何解决缓存击穿问题**？

**答**：缓存击穿往往是由热点Key引起的，当热点Key过期时，大量请求涌入同时查询，发现缓存未命中都会去访问数据库，导致数据库压力激增。解决这个问题的主要思路就是避免多线程并发去重建缓存，因此方案有两种。

第一种是基于互斥锁，当发现缓存未命中时需要先获取互斥锁，再重建缓存，缓存重建完成释放锁。这样就可以保证缓存重建同一时刻只会有一个线程执行。不过这种做法会导致缓存重建时性能下降严重。

第二种是基于逻辑过期，也就是不给热点Key设置过期时间，而是给数据添加一个过期时间的字段。这样热点Key就不会过期，缓存中永远有数据。

查询到数据时基于其中的过期时间判断key是否过期，如果过期开启独立新线程异步的重建缓存，而查询请求先返回旧数据即可。当然，这个过程也要加互斥锁，但由于重建缓存是异步的，而且获取锁失败也无需等待，而是返回旧数据，这样性能几乎不受影响。

需要注意的是，无论是采用哪种方式，在获取互斥锁后一定要再次判断缓存是否命中，做dubbo check. 因为当你获取锁成功时，可能是在你之前有其它线程已经重建缓存了。

# Redis是AP的还是CP的？

## 1. 单机版Redis：不适用CAP理论

单机版Redis不属于分布式系统，因此**不适用CAP理论**：

- 不存在网络分区(P)问题，CAP是分布式场景中的理论，如果单机Redis，那就没啥分布式可言了。P都没有了，还谈什么AP、CP呢？
- 在单机Redis中，因为只有一个实例，他的一致性是有保障的，具备强一致性，但单点故障时服务完全不可用。
- 本质上是CA系统，但因非分布式，严格来说不在CAP分类范围内。

## 2. Redis Cluster（分布式Redis）：AP系统

Redis Cluster在设计上明确属于**AP系统**（可用性和分区容忍性）：

### 为什么是AP系统？

1. 异步复制机制：
   - 主节点写入成功后立即响应客户端
   - 数据异步同步到从节点
   - 优先保证可用性，但存在短暂不一致
2. 网络分区行为：
   - 发生分区时，允许客户端继续向存活的主节点写入
   - 不会阻塞等待数据同步，明确选择可用性优先
   - 可能导致数据丢失（未同步的写入）
3. 最终一致性模型：
   - **通过异步复制逐步收敛数据**
   - **不保证强一致性（如故障时可能丢失未同步的写入）。**Redis没办法保证强一致性的主要原因是，因为它的分布式设计中采用的是异步复制，这导致在节点之间存在数据同步延迟和不一致的可能性。也就是说，当某个节点上的数据发生改变时，Redis会将这个修改操作发送给其他节点进行同步，但由于网络传输的延迟等原因，这些操作不一定会立即被其他节点接收到和执行，这就可能导致节点之间存在数据不一致的情况。
   - 除此之外，Redis的一致性还受到了节点故障的影响。当一个节点宕机时,这个节点上的数据可能无法同步到其他节点,这就可能导致数据在节点之间的不一致。虽然Redis通过复制和哨兵等机制可以提高系统的可用性和容错性，但是这些机制并不能完全解决数据一致性问题。
   - Redis的一致性模型是最终一致性，即在某个时间点读取的数据可能并不是最新的，但最终会达到一致的状态。

## 3. WAIT命令的局限性

虽然Redis提供了WAIT命令（要求主节点等待N个副本确认写入），但它**不能将Redis转变为CP系统**

这一点在Redis的官网中自已明确的说

![img](./assets/1706424748455-e8509337-9b7d-473c-9512-32b14a4950b1.png)

- 非原子性保证：副本确认后仍可能因故障回滚。
- 故障转移风险：旧主节点恢复后可能触发数据冲突
- Redis官方明确表示WAIT无法构建强一致系统，仅降低数据丢失概率
- 也就是说。客户端可以使用WAIT命令请求对特定数据进行同步复制。然而, WAIT 仅能确保数据在 Redis 实例中有指定数量的副本中被确认，它并不能将一组 Redis实例转变为具有强一致性的 CP 系统：在故障转移期间,已确认的写操作仍然可能会丢失,这取决于Redis持久化的具体配置。然而，使用 WAIT 后，在发生故障事件时丢失写操作的概率大大降低，只在某些难以触发的故障模式下才会发生。

## 4. 与真正CP系统的对比

| 特性         | Redis集群          | ZooKeeper(典型CP系统) |
| :----------- | :----------------- | :-------------------- |
| 数据同步方式 | 异步复制           | 原子广播(ZAB协议)     |
| 写入响应时机 | 主节点成功即返回   | 多数节点确认后返回    |
| 网络分区处理 | 继续服务，可能丢数 | 阻塞写入保一致性      |
| 一致性模型   | 最终一致           | 线性一致              |

## 5. 为什么会有"Redis是CP"的误解？

部分资料错误地将Redis归类为CP系统，主要原因：

1. **混淆单机与分布式**：单机Redis确实有强一致性，但不适用CAP理论
2. **误解主从复制**：认为主从复制保证了一致性，忽略了异步复制的本质
3. **混淆持久化与一致性**：将RDB/AOF持久化机制与分布式一致性混淆

## 6. 实际应用建议

- **缓存场景**：充分利用Redis的AP特性，接受短暂不一致换取高性能
- **关键业务**：强一致性需求建议结合关系型数据库，使用Redis作缓存层
- **分布式场景**：使用Redis Cluster时需设计容错机制，对一致性要求高的操作考虑WAIT命令（但需了解其局限性）

## 结论

Redis Cluster在CAP理论中明确属于**AP系统**，其设计优先保障高可用性和分区容忍性，通过异步复制实现最终一致性。开发者需要根据业务需求，在CAP之间找到合适的平衡点，必要时结合其他CP型组件构建混合架构。

## CAP理论回顾

CAP理论指出分布式系统最多只能同时满足以下三项中的两项：

- C (Consistency)：所有节点看到的数据是一致的
- A (Availability)：每个请求都能获得响应（非错误响应）
- P (Partition tolerance)：在网络分区时系统仍能继续运行

# Git 的代理配置

Git 的代理配置主要分为 **HTTP/HTTPS 代理**（最常用）和 **SOCKS 代理**（通常配合 Shadowsocks 或 Clash 等软件使用）。

以下是具体的配置方法：

### 1. 配置 HTTP/HTTPS 代理

#### 🎯 全局配置（对所有仓库生效）

如果你希望电脑上所有的 Git 操作都走代理，使用以下命令：

```
1# 设置 HTTP 代理
2git config --global http.proxy http://127.0.0.1:7890
3# 设置 HTTPS 代理
4git config --global https.proxy https://127.0.0.1:7890
```

#### 📂 局部配置（仅对当前仓库生效）

如果你只想让当前这个 `java_learning_note` 仓库走代理，**请在该仓库的文件夹中**运行（不加 `--global`）：

bash



```
1# 设置 HTTP 代理
2git config http.proxy http://127.0.0.1:7890
3# 设置 HTTPS 代理
4git config https.proxy https://127.0.0.1:7890
```

### 2. 配置 SOCKS5 代理

如果你使用的是 Clash、Shadowsocks 等软件，它们通常提供的是 SOCKS5 代理（端口通常是 `1080` 或 `7891`）。

bash



```
1# 全局配置 SOCKS5 代理
2git config --global http.proxy socks5://127.0.0.1:1080
3git config --global https.proxy socks5://127.0.0.1:1080
```

### 3. 取消代理配置

如果网络恢复了，或者代理导致了问题，可以随时取消配置：

bash



```
1# 取消全局代理
2git config --global --unset http.proxy
3git config --global --unset https.proxy
4
5# 取消局部代理（在仓库目录下执行）
6git config --unset http.proxy
7git config --unset https.proxy
```

### 4. 查看当前配置

随时可以通过以下命令查看是否配置成功：

```
1git config --global --get http.proxy
2git config --global --get https.proxy
```

### 💡 关键参数说明

- **`127.0.0.1`**：这是本地回环地址，代表代理软件运行在你自己的电脑上。

- `7890` / `1080`

  ：这是代理软件监听的端口号。

  - **Clash** 默认端口：HTTP 是 `7890`，SOCKS5 是 `7891`。
  - **Shadowsocks** 默认端口：通常是 `1080`。
  - *请根据你实际软件界面上显示的端口号进行修改。*

## 问题

这是一个非常关键的错误！🎉 恭喜你，**SSH 连接已经完全修复了**（因为这次报错是 GitHub 服务器明确拒绝了你的请求，而不是连接超时）。

这次失败的原因不是网络问题，而是**文件太大**。

### ❌ 错误原因

GitHub 对单个文件的大小有限制。

- **报错信息**：`File ... nacos.tar is 311.02 MB; this exceeds GitHub's file size limit of 100.00 MB`
- **核心问题**：你试图推送一个名为 `nacos.tar` 的文件，大小为 **311MB**。GitHub 禁止直接上传超过 100MB 的文件。

### ✅ 解决方案

你需要把这个大文件从 Git 的提交历史中移除，然后重新推送。

#### 第一步：从 Git 中删除大文件（保留本地文件）

在 Git Bash 中执行以下命令，将大文件从“待提交区”移除，但保留在你的电脑上（避免你辛辛苦苦下载的资料丢失）：

```bash
# 1. 从 Git 缓存中删除该文件（注意路径要和报错里的一致）
git rm --cached "springCloud/springCloud_note/springCloud_2_Heima_Mall/code/nacos.tar"

# 2. 提交这个“删除”操作
git commit -m "Remove large nacos.tar file"
```

#### 第二步：创建 .gitignore 文件（防止复发）

在你的项目根目录（`Java_Learning_Notes` 文件夹）下，创建一个名为 `.gitignore` 的文件（如果没有的话），并在其中添加以下内容，防止以后不小心把编译产物或压缩包再次传上去：

```gitignore
# 忽略所有 .tar, .zip, .jar 文件
*.tar
*.zip
*.jar
*.war
*.gz

# 忽略 IDE 配置
.idea/
*.iml
target/
```

#### 第三步：重新推送

```bash
git push origin master
```

这次因为去掉了那个 300MB 的巨无霸文件，数据量变小了，推送应该会非常快且成功。

------

### 💡 关于那个大文件的建议

那个 `nacos.tar` 看起来像是课程资料里的安装包或镜像。

- **不要**把它放在代码仓库里，它不属于代码。
- **建议**：将它移出项目文件夹，放在本地硬盘的“学习资料”或“安装包”目录中单独保存。代码仓库只应该存放文本类的代码、配置文件和文档（Markdown）。

# 如何使用SSH密钥

鉴于你刚才遇到的推送中断问题，切换到 **SSH 协议**确实是最好的选择。它不仅传输更稳定（基于端口 22，不像 HTTPS 容易受防火墙干扰），而且配置好之后**不需要每次输入账号密码**。

请按照以下 4 个步骤操作，大约 3 分钟即可完成：

### 第一步：检查是否已有 SSH 密钥

在 Git Bash 中运行以下命令，查看是否已经生成过密钥：

```bash
ls -al ~/.ssh/
```

- **如果看到 `id_rsa` 和 `id_rsa.pub`**：说明已有密钥，跳到**第二步**。
- **如果提示 "No such file"**：说明没有密钥，请执行**生成密钥**操作。

**生成密钥（如果没有的话）：**

```bash
ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

- `-t rsa`：指定加密类型。
- `-b 4096`：指定密钥长度，更安全。
- `-C`：注释，一般填邮箱。
- **操作**：连续按 **3 次回车**（第一次是确认保存路径，后两次是设置密码，直接回车代表不设密码，方便免密推送）。

### 第二步：复制公钥内容

密钥生成后，需要把**公钥**（`.pub` 文件）的内容复制到剪贴板。

**方法 A（命令行复制）：**

```bash
clip < ~/.ssh/id_rsa.pub
```

（这条命令会自动把内容复制到剪贴板）

**方法 B（手动复制）：**

1. 打开文件资源管理器，进入 `C:\Users\你的用户名\.ssh\` 文件夹。
2. 用记事本打开 `id_rsa.pub` 文件（注意是 **.pub** 结尾的）。
3. 全选并复制里面的所有内容。

### 第三步：添加公钥到 GitHub

1. 登录 GitHub 网站。
2. 点击右上角头像 -> **Settings** -> 左侧菜单的 **SSH and GPG keys**。
3. 点击绿色的 **New SSH key** 按钮。
4. **Title**：随便填，比如 "My Windows PC"。
5. **Key**：把刚才复制的公钥内容粘贴进去（**一定要粘贴完整**，从 `ssh-rsa` 开头，到你的邮箱结尾）。
6. 点击 **Add SSH key**。

### 第四步：修改本地仓库地址并测试

现在你的电脑和 GitHub 已经通过密钥“握手”成功了，但本地仓库还是指向 HTTPS 地址，需要改一下。

1. **修改远程地址为 SSH 格式**：
   在你的项目文件夹（Git Bash）中执行：

   ```bash
   git remote set-url origin git@github.com:Lucky666Prince/java_learning_note.git
   ```

2. **测试连接是否成功**：

   ```bash
   ssh -T git@github.com
   ```

   - 第一次连接会提示 `Are you sure you want to continue connecting?`，输入 `yes` 并回车。
   - 如果看到 `Hi Lucky666Prince! You've successfully authenticated...`，说明配置成功！

### 最后：再次推送

现在，你可以直接推送代码了，不需要输入密码：

```bash
git push origin master
```

**为什么推荐 SSH？**

- **稳定**：基于 TCP 22 端口，比 HTTPS (443端口) 更不容易被运营商干扰。
- **安全**：使用非对称加密，比 Token 更安全。
- **省事**：配置一次，终身免密（或者只在重启后输一次密码，如果你设置了密钥密码的话）。
  [(video_note_list_1)]

# 介绍一下Redis的集群模式？Could you please introduce the cluster mode of Redis?

Redis有三种主要的集群模式，用于在分布式环境中实现高可用性和数据复制。这些集群模式分别是：**主从复制（Master-Slave Replication）、哨兵模式（Sentinel）和Redis Cluster模式。**

Redis has three main cluster modes for achieving high availability and data replication in a distributed environment. These Cluster modes are respectively: Master-Slave Replication, Sentinel mode and Redis Cluster mode.

## 主从模式

主从复制是Redis最简单的集群模式。这个模式主要是为了**解决单点故障**的问题，所以将数据复制多个副本中，这样即使有一台服务器出现故障，其他服务器依然可以继续提供服务。

 主从模式中，包括一个主节点（Master）和一个或多个从节点（Slave）。主节点负责处理写操作和读操作，而从节点则复制主节点的数据，并且只能处理读操作。当主节点发生故障时，可以将一个从节点升级为主节点，实现故障转移（需要手动实现）。

Master-slave replication is the simplest cluster mode of Redis. This mode is mainly designed to address the issue of single point of failure. Therefore, data is replicated into multiple copies, so that even if one server fails, the others can still continue to provide services.In the master-slave mode, there is one Master node (Master) and one or more

![image-20260209124226744](./assets/image-20260209124226744.png)

**主从复制的优势在于简单易用，适用于读多写少的场景。**它提供了数据备份功能，并且可以有很好的扩展性，只要增加更多的从节点，就能让整个集群的读的能力不断提升。

**The advantage of master-slave replication lies in its simplicity and ease of use, making it suitable for scenarios where there are many reads and few writes.** It offers a data backup function and has excellent scalability. As long as more slave nodes are added, the reading capacity of the entire cluster can be continuously enhanced.

但是主从模式最大的缺点，就是不具备故障自动转移的能力，没有办法做容错和恢复。

However, the biggest drawback of the master-slave mode is that it lacks the ability to automatically transfer faults, making it impossible to perform fault tolerance and recovery.

主节点和从节点的宕机都会导致客户端部分读写请求失败，需要人工介入让节点恢复或者手动切换一台从节点服务器变成主节点服务器才可以。并且在主节点宕机时，如果数据没有及时复制到从节点，也会导致数据不一致。

## 哨兵模式

为了解决主从模式的无法自动容错及恢复的问题，Redis引入了一种哨兵模式的集群架构。

哨兵模式是在主从复制的基础上加入了哨兵节点。哨兵节点是一种特殊的Redis节点，用于监控主节点和从节点的状态。当主节点发生故障时，哨兵节点可以自动进行故障转移，选择一个合适的从节点升级为主节点，并通知其他从节点和应用程序进行更新。

在原来的主从架构中，引入哨兵节点，其作用是监控Redis主节点和从节点的状态。通常需要部署多个哨兵节点，以确保故障转移的可靠性。

 哨兵节点定期向所有主节点和从节点发送PING命令，如果在指定的时间内未收到PONG响应，哨兵节点会将该节点标记为主观下线。如果一个主节点被多数哨兵节点标记为主观下线，那么它将被标记为客观下线。

 当主节点被标记为客观下线时，哨兵节点会触发故障转移过程。它会从所有健康的从节点中选举一个新的主节点，并将所有从节点切换到新的主节点，实现自动故障转移。同时，哨兵节点会更新所有客户端的配置，指向新的主节点。

 哨兵节点通过发布订阅功能来通知客户端有关主节点状态变化的消息。客户端收到消息后，会更新配置，将新的主节点信息应用于连接池，从而使客户端可以继续与新的主节点进行交互。

 这个哨兵模式的优点就是为整个集群提供了一种故障转移和恢复的能力。

## Cluster模式

Redis Cluster是Redis中推荐的分布式集群解决方案。它将数据自动分片到多个节点上，每个节点负责一部分数据。

Redis Cluster is the recommended distributed cluster solution in Redis. It automatically shards the data to multiple nodes, with each node responsible for a portion of the data.

Redis Cluster采用主从复制模式来提高可用性。每个分片都有一个主节点和多个从节点。主节点负责处理写操作，而从节点负责复制主节点的数据并处理读请求。[翻译](Redis Cluster adopts a master-slave replication mode to enhance availability. Each shard has one master node and multiple slave nodes. The master node is responsible for handling write operations, while the slave nodes are responsible for replicating the data of the master node and processing read requests.)

Redis Cluster能够自动检测节点的故障。当一个主节点失去连接或不可达时，Redis Cluster会尝试将该节点标记为不可用，并从可用的从节点中提升一个新的主节点。

Redis Cluster can automatically detect faults in nodes. When a master node loses connection or is unreachable, Redis Cluster will attempt to mark that node as unavailable and promote a new master node from among the available slave nodes.

Redis Cluster是适用于大规模应用的解决方案，它提供了更好的横向扩展和容错能力。它自动管理数据分片和故障转移，减少了运维的负担。

 Cluster模式的特点是数据分片存储在不同的节点上，每个节点都可以单独对外提供读写服务。不存在单点故障的问题。

 关于分片的规则和细节，参考：

[什么是Redis的数据分片](#什么是Redis的数据分片)

关于 Cluster 中存在对事务和 lua 的限制，参考：

[Redis Cluster中使用事务和lua有什么限制](#Redis Cluster 中使用事务和 lua 有什么限制？)

# 什么是Redis的数据分片？

在Redis的Cluster 集群模式中，使用哈希槽（hash slot）的方式来进行数据分片，将整个数据集划分为多个槽，每个槽分配给一个节点。客户端访问数据时，先计算出数据对应的槽，然后直接连接到该槽所在的节点进行操作。Redis Cluster还提供了自动故障转移、数据迁移和扩缩容等功能，能够比较方便地管理一个大规模的Redis集群。

In the Cluster mode of Redis, hash slots are used for data sharding, dividing the entire dataset into multiple slots, and each slot is assigned to a node. When the client accesses data, it first calculates the slot corresponding to the data and then directly connects to the node where the slot is located for operation. Redis Cluster also offers functions such as automatic failover, data migration, and scaling, making it relatively convenient to manage a large-scale Redis cluster.

![img](./assets/1702293948603-17c6942a-6a69-493d-b233-a9c9e39ff8e6.png)

Redis Cluster将整个数据集划分为16384个槽，每个槽都有一个编号（0~16383），集群的每个节点可以负责多个hash槽，客户端访问数据时，先根据key计算出对应的槽编号，然后根据槽编号找到负责该槽的节点，向该节点发送请求。

在 Redis 的每一个节点上，都有这么两个东西，一个是槽（slot），它的的取值范围是：0-16383。还有一个就是 cluster，可以理解为是一个集群管理的插件。当我们在存取的 Key 的时候，Redis 会根据 CRC16 算法得出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

Redis Cluster中的数据分片具有以下特点：
  1. **提升性能和吞吐量**：通过在多个节点上分散数据，可以并行处理更多的操作，从而提升整体的性能和吞吐量。这在高流量场景下尤其重要，因为单个节点可能无法处理所有请求。
  2. **提高可扩展性**：分片使得Redis可以水平扩展。可以通过添加更多节点扩展数据库的容量和处理能力。
  3. **更好的资源利用**：分片允许更有效地利用服务器资源。每个节点只处理数据的一部分，这降低了单个节点的内存和计算需求。
  4. **避免单点故障**：在没有分片的情况下，如果唯一的Redis服务器发生故障，整个服务可能会停止。在分片的环境中，即使一个节点出现问题，其他节点仍然可以继续运行。
  5. **数据冗余和高可用性**：在某些分片策略中，如Redis集群，每个分片的数据都可以在集群内的其他节点上进行复制。这意味着即使一个节点失败，数据也不会丢失，从而提高了系统的可用性。

**扩展知识**

**Redis Cluster将整个数据集划分为16384个槽，为什么是16384呢，这个数字有什么特别的呢？**

这个问题在Github上有所讨论，Redis的作者也下场做过回复：https://github.com/redis/redis/issues/2576

16384这个数字是一个2的14次方（2^14），尽管crc16能得到2^16 -1=65535个值，但是并没有选择，主要从消息大小和集群规模等方面考虑的：

1、正常的心跳数据包携带了节点的完整配置，在更新配置的时候，可以以幂等方式进行替换。这意味着它们包含了节点的原始槽配置，对于包含16384个槽位的情况，使用2k的空间就够了，但如果使用65535个槽位，则需要使用8k的空间，这就有点浪费了。

2、由于其他设计权衡的原因，Redis Cluster不太可能扩展到超过1000个主节点，这种情况下，用65535的话会让每个节点上面的slot太多了，会导致节点的负载重并且数据迁移成本也比较高。而16384是相对比较好的选择，可以在1000个节点下使得slot均匀分布，每个分片平均分到的slot不至于太小。

除此之外，还有一些原因和优点供大家参考：

 1. 易于扩展：槽数量是一个固定的常数，这样就可以方便地进行集群的扩展和缩小。如果需要添加或删除节点，只需要将槽重新分配即可。
 2. 易于计算：哈希算法通常是基于槽编号计算的，将槽数量设置为2的幂次方，可以使用位运算等简单的算法来计算槽编号，从而提高计算效率。
 3. 负载均衡：槽数量的选择可以影响数据的负载均衡。如果槽数量太少，会导致某些节点负载过重；如果槽数量太多，会导致数据迁移的开销过大。16384这个数量在实践中被证明是一个比较合适的选择，能够在保证负载均衡的同时，减少数据迁移的开销。

CRC16**算法**

当我们在存取 Key 的时候，Redis 会根据 CRC16 算法得出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽。

 那么，什么是CRC16算法呢？

 CRC16（Cyclic Redundancy Check，循环冗余校验码）算法是一种广泛使用的校验算法，主要用于数据通信和数据存储等领域，例如网络通信中的错误检测和校正、数据存储中的文件校验和等。

 CRC16算法基于多项式除法，将输入数据按位进行多项式除法运算，最后得到一个16位的校验码。CRC16算法的计算过程包括以下几个步骤：

1. 初始化一个16位的寄存器为全1；

 2. 将输入数据的第一个字节与16位寄存器的低8位进行异或操作，结果作为新的16位寄存器的值；
 3. 将16位寄存器的高8位和低8位分别右移一位，丢弃掉最低位，即寄存器右移一位；
 4. 如果输入数据还没有处理完，转到第2步继续处理下一个字节；
 5. 如果输入数据已经处理完，将16位寄存器的值取反，得到CRC16校验码。

CRC16算法的多项式是一个固定的16位二进制数，不同的CRC16算法使用的多项式也不相同。例如，CRC-16/CCITT算法使用的多项式为0x1021，而Modbus CRC16算法使用的多项式为0xA001。

CRC16算法的优点是计算速度快，校验效果好，具有广泛的应用范围。缺点是只能检测错误，无法纠正错误。如果数据被修改，CRC校验值也会被修改，但无法确定是哪一位数据被修改。因此，在数据传输和存储中，通常需要与其它校验算法配合使用，以保证数据的完整性和正确性。

#  Redis 使用什么协议进行通信？

Redis 使用自己设计的一种文本协议进行客户端与服务端之间的通信——RESP（REdis Serialization Protocol），这种协议简单、高效，易于解析，被广泛使用。

RESP 协议基于 TCP 协议，采用请求/响应模式，每条请求由多个参数组成，以命令名称作为第一个参数。请求和响应都以行结束符（\r\n）作为分隔符，具体格式如下：

```
*<number of arguments>\r\n 
$<length of argument 1>\r\n 
<argument data>\r\n 
... 
$<length of argument N>\r\n <argument data>\r\n
```

其中，`<number of arguments>`表示参数个数，`<length of argument>` 表示参数数据的长度，`<argument data> `表示参数数据。参数可以是字符串、整数、数组等数据类型。

 例如，以下是一个 Redis 协议的示例请求和响应：
 请求：

```
*3\r\n 
$3\r\n 
SET\r\n 
$5\r\n 
mykey\r\n 
$7\r\n 
myvalue\r\n
```

响应：

```
+OK\r\n
```

上面的请求表示向 Redis 服务器设置一个名为 "mykey" 的键，值为 "myvalue"。响应返回 "+OK" 表示操作成功。

"$3\r\n"表示参数长度为3，即下一个参数是一个3个字符的字符串。它表示要执行的命令是"SET"，即设置键值对。

 "$5\r\n"表示参数长度为5，即下一个参数是一个5个字符的字符串。它表示要设置的键是"mykey"。

 "$7\r\n"表示参数长度为7，即下一个参数是一个7个字符的字符串。它表示要设置的值是"myvalue"。

 除了基本的 GET、SET 操作，Redis 还支持事务、Lua 脚本、管道等高级功能，这些功能都是通过 Redis 协议来实现的。

# Redis 与 Memcached 有什么区别？

Redis 和 Memcached 都是常见的缓存服务器，它们的主要区别包括以下几个方面：

# Redis为什么这么快？

- 内存存储：速度的基石
  - **所有数据驻留内存**：Redis将全部数据存储在内存中，内存访问速度在**几十纳秒级别**，而磁盘访问速度在**几毫秒到几十毫秒级别**，两者差距可达**10万倍以上**
  - **内存操作无I/O瓶颈**：传统数据库（如MySQL）需要频繁进行磁盘I/O操作，而Redis直接在内存中操作数据，避免了磁盘寻址和I/O等待时间。
  - **持久化不影响性能**：虽然Redis支持RDB和AOF持久化机制，但这些操作通过**后台子进程**执行，主进程仍专注于处理内存中的请求，不会阻塞正常读写
- 单线程模型：避免竞争与开销
  - **核心命令执行为单线程**：Redis使用单线程处理所有客户端请求（从Redis 6.0开始引入了I/O多线程，但**核心命令执行依然保持单线程**）
  - **避免上下文切换**：多线程环境下的线程切换会消耗CPU资源，单线程完全避免了这种开销，实测表明线程切换一次需1-5微秒，高并发下累积开销巨大
  - **无需锁机制**：单线程模型**天然避免了锁竞争**，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗。
  - **原子性保障**：所有命令顺序执行，确保了操作的原子性，如`INCR`命令无需额外锁即可保证原子性。

- I/O多路复用：高效处理并发
  - **多路复用技术**：Redis采用**epoll**（Linux）或**kqueue**（BSD）等I/O多路复用技术，让单个线程能**同时监听多个网络连接**
  - 工作原理：
    - 客户端连接 → 注册到事件循环 → 监听可读事件
    - 请求到达 → 事件就绪 → 单线程处理 → 返回响应
  - **避免资源浪费**：无需为每个连接分配线程，1个线程可处理数万连接，大大提高了系统吞吐量

- 高效数据结构：时间复杂度优化
  - 动态选择数据结构：Redis根据数据大小自动切换编码方式，实现空间与时间的平衡：
    - **String**：小字符串用`embstr`（直接存于redisObject），大字符串用`raw`（指针指向缓冲区）
    - **List**：小列表用`ziplist`（紧凑存储），大数据量转`quicklist`（链表+压缩列表）
    - **ZSet**：小数据用`ziplist`，大数据用`skiplist`（跳表），支持O(logN)范围查询
  - **时间复杂度优势**：多数操作为O(1)或O(log n)，如哈希表查找、跳表范围查询，确保了高效执行

- 零拷贝与协议优化：减少数据移动
  - **零拷贝技术**：Redis网络通信采用了**writev/sendfile**等零拷贝技术，尽可能减少内核态与用户态之间的数据拷贝
  - **高效协议**：使用**RESP**（Redis序列化协议）作为通信协议，二进制格式解析高效，减少网络传输量
  - **Pipeline批量操作**：客户端可批量发送命令，减少网络往返时间（RTT），例如Pipeline(100)可将QPS从50k提升至500k

- Redis 6.0多线程：针对性突破瓶颈
  - **I/O多线程设计**：Redis 6.0引入多线程**仅用于网络I/O处理**，**核心命令执行仍为单线程**
  - 工作流程：
    - 主线程接收连接 → 分配给I/O线程
    - I/O线程解析请求 → 主线程执行命令
    - I/O线程序列化响应 → 发送回客户端
  - **性能提升**：在高并发、大带宽场景下，QPS可提升**2-3倍**（如从10万提升至20-30万）
  - **配置建议**：默认不开启，需手动设置`io-threads-do-reads yes`和`io-threads`数量（4核机器建议2-3个）

- 其他优化因素
  - **C语言实现**：Redis使用C语言开发，直接操作内存，不依赖虚拟机或垃圾回收机制，执行效率更高。
  - **热点数据缓存**：作为缓存层前置数据库，高命中率避免访问慢源，极大提高系统整体吞吐。
  - **对象共享池**：整数对象共享池（0～9999）减少内存占用，提高效率。

# Redis Cluster 中使用事务和 lua 有什么限制？

[介绍一下Redis的集群模式？](#介绍一下Redis的集群模式？)

Redis Cluster采用主从复制模式来提高可用性。每个分片都有一个主节点和多个从节点。主节点负责处理写操作，而从节点负责复制主节点的数据并处理读请求。在Redis的Cluster 集群模式中，会对数据进行数据分片，将整个数据集分配给不同节点。

这个思想就和我们在MySQL 中做分库分表是一样的，都是通过一定的分片算法，把数据分散到不同的节点上进行存储。

**那么和 MySQL 对跨库事务支持存在限制一样，在 Redis Cluster 中使用事务和 Lua 脚本时，也是有一定的限制的。**

在 Redis Cluster 中，事务不能跨多个节点执行。事务中涉及的所有键必须位于同一节点上。如果尝试在一个事务中包含多个分片的键，事务将失败。另外，对 WATCH 命令也用同样的限制，要求他只能监视位于同一分片上的键。

和事务相同，执行 Lua 脚本时，脚本中访问的所有键也必须位于同一节点。Redis 不会在节点之间迁移数据来支持跨节点的脚本执行。Lua 脚本执行为原子操作，但是如果脚本因为某些键不在同一节点而失败，整个脚本将终止执行，可能会影响数据的一致性。

当我们要跨节点执行 lua 的时候，会报错提示：command keys must in same slot。（详见：https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/ ）

**如何解决**

[如何在RedisCluster中执行lua脚本](#如何在Redis Cluster中执行lua脚本)

# 如何在 Redis Cluster 中执行 lua 脚本？

[Redis Cluster 中使用事务和 lua 有什么限制？](#Redis Cluster 中使用事务和 lua 有什么限制？)

因为 Redis  Cluster 中，数据会被分片到多个节点上，跨节点的 lua 脚本是不支持的，所以就会失败。但是 Cluster 是很常见的场景，lua （以及事务）也是一个非常重要的用法，这个问题怎么解决呢？

**Hash Tag**

 在 Redis 的官方中，提到了一个 hash tags的功能：

![img](./assets/1716003544630-fb56821b-49a0-4f76-9fa6-88dd0d2fd617.png)

如果我们想要执行 lua 脚本或者事务的时候，就需要确保多个相关的键应该存储在同一个节点上以便执行原子操作，**默认情况下，Redis 使用键的哈希值来决定将数据存储在哪个节点**。而Redis 中的 hashtag 就是一种可以让我们干预 hash 结果的机制。

Redis 中的 hashtag 是键名中用大括号 {} 包裹的部分。Redis 对大括号内的字符串计算哈希值，并基于这个哈希值将键分配到特定的节点。只有键名中包含大括号，且大括号内有内容时，大括号内的部分才会被用来计算哈希值。如果大括号为空或不包含任何字符，Redis 将整个键名用于哈希计算。

有了这个特性，我们就可以在设计键名时，可以将共享相同逻辑或数据集的键包含相同的 hashtag。就和我们在 MySQL 的分库分表中的基因法其实是类似的概念。

例如，如果你有多个与用户 ID 相关的键，可以使用 user:{12345}:profile 和 user:{12345}:settings 这样的命名方式，确保它们都位于同一个节点。这样他只会用{12345}进行 hash 算法，这样虽然他们是不同的key，但是分片之后的结果就可以在同一个节点上。这样就能执行事务或者 lua 脚本了。

**其他方案**

 除了使用 Hash Tag 以外，还有一些其他的方案，也能实现，比如：

  1. 应用层处理：如果跨节点操作不可避免，可以在应用层通过分布式事务管理器或其他机制来协调多个节点的数据一致性。这通常需要复杂的逻辑和额外的开发工作。

  2. 拆分操作：尽量将需要事务处理的逻辑拆分成多个独立的、可以在单个节点上执行的小操作，从而避免跨节点事务的需求。


**扩展知识**

     allow-cross-slot-keys

 在 Redis 7.0.11中新增了一个命令：allow-cross-slot-keys

https://github.com/redis/redis-doc/pull/1893/files

开启这个配置，可以允许在单个命令中使用不同槽（slot）的键。例如，你可以在一个 MSET 命令中设置多个键，即使这些键属于不同的槽。

 但是，需要注意的是，即使启用了 allow-cross-slot-keys，事务中的所有键仍然必须位于同一个槽（即同一个节点）才能保证事务的原子性。如果事务中的键分布在不同的节点上，Redis 会拒绝执行这些命令。

 所以，网上有的文章说，使用 allow-cross-slot-keys就能做跨节点事务或者 lua 了，其实是不对的！

 但是需要注意的是，启用这个选项的命令可能会因为涉及多个节点的网络通信而导致性能降低，但这并不会让这些操作变成原子操作。

# Redis 支持哪几种数据类型？

Redis 中支持了多种数据类型，其中比较常用的有五种：

 1. 字符串（String）
 2. 哈希（Hash）
 3. 列表（List）
 4. 集合（Set）
 5. 有序集合（Sorted Set）

 另外，Redis中还支持一些高级的数据类型，如：Streams、Bitmap、Geospatial以及HyperLogLog

**字符串**

[Redis为什么要自已定义SDS?](#Redis为什么要自已定义SDS?)

**有序集合**

[Redis中的Zset是怎么实现的？](Redis中的Zset是怎么实现的？)

# Redis为什么要自已定义SDS?

Redis是一种KV的存储结构，他的key是字符串类型，值也支持字符串，所以字符串是redis中最常见的一个类型了。Redis自己本身是通过C语言实现的，但是他并没有直接使用C语言中的字符数组的方式来实现字符串，而是自己实现了一个SDS（Simple Dynamic Strings），即简单动态字符串，这是为什么呢？

首先，因为字符串在Redis中使用实在是太广泛了 ，所以对他的基本要求就有两点，第一就是要支持任意字符的存储，第二就是各种操作需要高效。

接着我们看看C语言中字符串的实现方式有什么问题呢？很多人可能都忘了，我帮大家回忆一下，C语言中，字符串是通过字符数组实现的，底层呢是开辟了一块连续的空间，依次存放字符串中的每一个字符。为了表示字符串的结束，他会在字符数组的最后一个字符处记录﻿\0﻿，

也就是说，在C语言中，当识别到字符数组中的﻿\0﻿字符的时候，就认为字符串结束了，那么这么做会带来哪些问题呢？

**就是这样实现的字符串中就不能保存任意内容了，至少﻿\0﻿就不行，因为遇到他的时候就直接截断了，这肯定是接受不了的。**

 **还有就是因为C中的字符串以﻿\0﻿作为识别字符串结束的方式，所以他的字符串长度判断、字符串追加等操作，都需要从头开始遍历，一直遍历到﻿\0﻿的时候再返回长度或者做追加。这就使得字符串相关的操作效率都很低。**

那么，想要解决上面的两个问题要怎么办呢？那就是**在用字符数组表示字符串的同时，在这个字符串中增加一个表示分配给该字符数组的总长度的﻿alloc﻿字段，和一个表示字符串现有长度的﻿len﻿字段。这样在获取长度的时候就不依赖﻿\0﻿了，直接返回﻿len﻿的值就行了。**

 还有呢，就是在做追加操作的时候，只需要判断新追加的部分的﻿len﻿加上已有的﻿len﻿是否大于﻿alloc﻿，如果超过就重新再申请新空间，如果没超过，就直接进行追加就行了。

 还有很多其他操作，比如复制、比较等都可以使用类似的思想高效的操作。

[Redis中的Zset是怎么实现的？](#Redis中的Zset是怎么实现的？)

# Redis中的Zset是怎么实现的？

ZSet（也称为Sorted Set）是Redis中的一种特殊的数据结构，它内部维护了一个有序的字典，这个字典的元素中既包括了一个成员（member），也包括了一个double类型的分值(score)。这个结构可以帮助用户实现记分类型的排行榜数据，比如游戏分数排行榜，网站流行度排行等。

**Redis中的ZSet在实现中，有多种结构，大类的话有两种，分别是ziplist(压缩列表)和skiplist(跳跃表)，但是这只是以前，在Redis 5.0中新增了一个listpack（紧凑列表）的数据结构，这种数据结构就是为了替代ziplist的，而在之后Redis 7.0的发布中，在Zset的实现中，已经彻底不再使用zipList了。**

当ZSet的元素数量比较少时，Redis会采用ZipList（ListPack）来存储ZSet的数据。ZipList（ListPack）是一种紧凑的列表结构，它通过连续存储元素来节约内存空间。当ZSet的元素数量增多时，Redis会自动将ZipList（ListPack）转换为SkipList，以保持元素的有序性和支持范围查询操作。

![image-20260212211837058](./assets/image-20260212211837058.png)



# Zset为什么在数据量少的时候用zipList，而在数据量大·的时候转成SkipList?

[Redis中的Zset是怎么实现的？](#Redis中的Zset是怎么实现的？)

通过上文我们知道，在 Redis 中，ZSet在特定条件下会使用ZipList作为其内部表示。这通常发生在有序集合较小的时候，默认情况下，当元素数量少于128，每个元素的长度都小于64字节的时候，使用ZipList（ListPack），否则，使用SkipList！

Redis 之所以在数据量少的时候使用ZipList（包括后来的ListPack），而在数据量大时转成SkipList（跳表），是出于 内存优化 和 性能考量 的双重目的。

内存对比

 首先ZipList是一个压缩的数据结构，它的每个元素都是连续存储的（和数组有点像），因此内存的使用非常紧凑。与其他数据结构相比，ZipList在小规模数据存储时显著减少了内存占用。

而在内存方面，SkipList 相比于 ZipList 内存开销要大得多，因为它是通过多个链表层级来实现的，每个元素需要额外的指针来维护层级结构。

 这就涉及到ZipList的一个缺点了。而在内存方面，SkipList 相比于 ZipList 内存开销要大得多，因为它是通过多个链表层级来实现的，每个元素需要额外的指针来维护层级结构。

 那么按理说，ZipList的内存占用要比Skip要小的，所以他更适合用来存储数据，所以ZSet会使用ZipList，那为啥不全都用，而只有在数据量小的额时候采用呢？

 这就涉及到ZipList的一个缺点了。

性能对比

 前面说过了ZipList 是一个紧凑的线性结构，所以如果在 ZipList 中查找一个元素时，可能需要遍历整个列表，同理插入和删除操作也是线性的。所以它的插入、删除和查找操作的时间复杂度通常是 O(N)。

 而SkipList 采用多级索引结构，它的查找、插入、删除操作的时间复杂度是 O(log N)，远远优于 ZipList 的 O(N)。

总结

 所以，ZipList的存储更节省空间，而SkipList的操作性能会更好。

 所以，对于少量数据，ZipList更好，因为它的内存开销很小，而且性能也可以接受（N越小，logN和N的差别更小）。但是当数据量大到一定程度时，SkipList的 O(log N) 性能会显著优于ZipList 的 O(N) 性能，尤其是涉及到范围查询和顺序遍历时。

 所以，默认情况下，当元素数量少于128，每个元素的长度都小于64字节的时候，ZSet使用ZipList（ListPack），否则，使用SkipList！

![tongyi-mermaid-2026-02-12-221419](./assets/tongyi-mermaid-2026-02-12-221419.png)

