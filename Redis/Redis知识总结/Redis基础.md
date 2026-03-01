# Redis基础快速入门

# 参考网址

- 官网地址：
  - 英文：https://redis.io/
  - 中文：
    - http://www.redis.cn/
    - https://www.redis.com.cn/documentation.html

- Redis源码地址：https://github.com/redis/redis
- Redis从发布至今，已经有十余年的时光了，一直遵循着自己的命名规则： 版本号第二位如果是奇数，则为非稳定版本 如2.7、2.9.3.1 版本号第二位如果是偶数，则为稳定版本 如2.6、2.8. 3.0.3.2 当前奇数版本就是下一个稳定版本的开发版本,如2.9版本是3.0版本的开发版本 我们可以通过redis.lo官网来下载自己感兴趣的版本进行源码阅读：历史发布版本的源码：https://download.redis.io/releases/
- Redis新特性：https://github.com/redis/redis/releases

- Redis在线测试：https://try.redis.io/

- Redis命令参考：http://doc.redisfans.com/

- Redis之父的github：https://github.com/antirez

# 1.初识Redis

Redis是一种**键值型的NoSql数据库**，这里有两个关键字：

- 键值型

- NoSql

其中**键值型**，是指Redis中存储的数据都是以key、value对的形式存储，而value的形式多种多样，可以是字符串、数值、甚至json：

![image-20220502190959608](assets/6U1Rhxo.png)

而NoSql则是相对于传统关系型数据库而言，有很大差异的一种数据库。

## 1.1.认识NoSQL

**NoSql**可以翻译做Not Only Sql（不仅仅是SQL），或者是No Sql（非Sql的）数据库。是相对于传统关系型数据库而言，有很大差异的一种特殊的数据库，因此也称之为**非关系型数据库**。



### 1.1.1.结构化与非结构化

传统关系型数据库是结构化数据，每一张表都有严格的约束信息：字段名、字段数据类型、字段约束等等信息，插入的数据必须遵守这些约束：

![](assets/4tUgFo6.png)



而NoSql则对数据库格式没有严格约束，往往形式松散，自由。

可以是键值型：

![](assets/GdqOSsj.png)

也可以是文档型：

![](assets/zBBQfcc.png)



甚至可以是图格式：

![](assets/zBnKxWf.png)



### 1.1.2.关联和非关联

传统数据库的表与表之间往往存在关联，例如外键：

![](assets/tXYSl5x.png)



而非关系型数据库不存在关联关系，要维护关系要么靠代码中的业务逻辑，要么靠数据之间的耦合：

```json
{
  id: 1,
  name: "张三",
  orders: [
    {
       id: 1,
       item: {
	 id: 10, title: "荣耀6", price: 4999
       }
    },
    {
       id: 2,
       item: {
	 id: 20, title: "小米11", price: 3999
       }
    }
  ]
}
```

此处要维护“张三”的订单与商品“荣耀”和“小米11”的关系，不得不冗余的将这两个商品保存在张三的订单文档中，不够优雅。还是建议用业务来维护关联关系。



### 1.1.3.查询方式

传统关系型数据库会基于Sql语句做查询，语法有统一标准；

而不同的非关系数据库查询语法差异极大，五花八门各种各样。

![](assets/AzaHOTF.png)



### 1.1.4.事务

传统关系型数据库能满足事务ACID的原则。

![](assets/J1MqOJM.png)



而非关系型数据库往往不支持事务，或者不能严格保证ACID的特性，只能实现基本的一致性。



### 1.1.5.总结

除了上述四点以外，在存储方式、扩展性、查询性能上关系型与非关系型也都有着显著差异，总结如下：

![](assets/kZP40dQ.png)

- 存储方式
  - 关系型数据库基于磁盘进行存储，会有大量的磁盘IO，对性能有一定影响
  - 非关系型数据库，他们的操作更多的是依赖于内存来操作，内存的读写速度会非常快，性能自然会好一些

* 扩展性
  * 关系型数据库集群模式一般是主从，主从数据一致，起到数据备份的作用，称为垂直扩展。
  * 非关系型数据库可以将数据拆分，存储在不同机器上，可以保存海量数据，解决内存大小有限的问题。称为水平扩展。
  * 关系型数据库因为表之间存在关联关系，如果做水平扩展会给数据查询带来很多麻烦




## 1.2.Redis的特征

Redis诞生于2009年全称是**Re**mote  **D**ictionary **S**erver 远程词典服务器，是一个**基于内存的键值型NoSQL数据库**。

**特征**：

**1.速度快**

正常情况下，Redis执行命令的速度非常快，官方给出的数字是读写性能可以达到10万/秒，当然这也取决于机器的性能，但这里先不讨论机器性能上的差异，只分析一下是什么造就了Redis除此之快的速度，可以大致归纳为以下四点：

- Redis的所有数据都是存放在**内存**中
- Redis是用**C语言**实现的，一般来说C语言实现的程序“距离”操作系统更近，执行速度相对会更快。
- Redis使用了**单线程架构**，预防了多线程可能产生的竞争问题。

**2.基于键值对的数据结构服务器**

**3.丰富的功能**

除了5种数据结构，Redis还提供了许多额外的功能：

- 提供了键过期功能，可以用来实现缓存。
- 提供了发布订阅功能，可以用来实现消息系统。
- 支持Lua脚本功能，可以利用Lua创造出新的Redis命令。
- 提供了简单的事务功能，能在一定程度上保证事务特性。
- 提供了流水线(Pipeline)功能，这样客户端能将一批命令一次性传到Redis，减少了网络的开销。

**4.简单稳定**

Redis的简单主要表现在三个方面。首先，Redis的源码很少，早期版本的代码只有2万行左右，3.0版本以后由于添加了集群特性，代码增至5万行左右，相对于很多NoSQL数据库来说代码量相对要少很多，也就意味着普通的开发和运维人员完全可以“吃透”它。其次，Redis使用单线程模型，这样不仅使得Redis服务端处理模型变得简单，而且也使得客户端开发变得简单。最后，Redis不需要依赖于操作系统中的类库（例如Memcache需要依赖libevent这样的系统类库），Redis自己实现了事件处理的相关功能。

**5.客户端语言多**

Redis提供了简单的TCP通信协议，很多编程语言可以很方便地接入到Redis，并且由于Redis受到社区和各大公司的广泛认可，所以支持Redis的客户端语言也非常多，几乎涵盖了主流的编程语言，例如Java、PHP、Python、C、C++、Nodejs等

**6.持久化**

Redis提供了两种持久化方式：RDB和AOF，即可以用两种策略将内存的数据保存到硬盘中

**7.主从复制**

Redis提供了复制功能，实现了多个相同数据的Redis副本（如图1-2所示），复制功能是分布式Redis的基础。

![image-20251121111222793](images/image-20251121111222793.png)

**8.高可用和分布式**

Redis从2.8版本正式提供了高可用实现Redis Sentinel，它能够保证Redis节点的故障发现和故障自动转移。Redis从3.0版本正式提供了分布式实现Redis Cluster，它是Redis真正的分布式实现，提供了高可用、读写和容量的扩展性。

## 1.3.安装Redis

大多数企业都是基于Linux服务器来部署项目，而且Redis官方也没有提供Windows版本的安装包。因此课程中我们会基于Linux系统来安装Redis.

此处选择的Linux版本为CentOS 7.

### 1.3.1.依赖库

Redis是基于C语言编写的，因此首先需要安装Redis所需要的gcc依赖：

```sh
yum install -y gcc tcl
```

### 1.3.2.在Linux上安装Redis并解压

(1)下载Redis指定版本的源码压缩包到当前目录

```
wget http://download.redis.io/releases/redis-6.2.6.tar.gz
```

(2)解压缩Redis源码压缩包。：

```sh
tar -xzf redis-6.2.6.tar.gz
```

解压后：

![image-20211211080339076](assets/8V6zvCD.png)

(3)进入redis目录：

```sh
cd redis-6.2.6
```

(4)运行编译命令：（编译之前确保操作系统已经安装gcc）

```sh
make && make install
```

如果没有出错，应该就安装成功了。

该目录已经默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- **redis-cli：是redis提供的命令行客户端**
- **redis-server：是redis的服务端启动脚本**
- **redis-sentinel：是redis的哨兵启动脚本**
- **redis-benchmark：Redis基准测试工具**
- **redis-check-aof：RedisAOF持久化文件检测和修复工具**
- **redis-check-dump：RedisRDB持久化文件检测和修复工具**

## 1.4.Redis桌面客户端

安装完成Redis，我们就可以操作Redis，实现数据的CRUD了。这需要用到Redis客户端，包括：

- 命令行客户端
- 图形化桌面客户端
- 编程客户端

### 1.4.1.Redis命令行客户端

Redis安装完成后就自带了命令行客户端：redis-cli，使用方式如下：

```sh
redis-cli [options] [commonds]
```

其中常见的options有：

- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a 123321`：指定redis的访问密码 
  - systemctl cat redis.service | grep ExecStart
  - sudo grep "^requirepass" /etc/redis/redis.conf检查密码

其中的commonds就是Redis的操作命令，例如：

- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`

不指定commond时，会进入`redis-cli`的交互控制台：

![](assets/OYYWPNo.png)

# **Windows 上连接 Linux（或虚拟机/云服务器）上的 Redis**

## 前提条件

| 条件                                          | 如何检查/设置                                              |
| --------------------------------------------- | ---------------------------------------------------------- |
| 1. Redis 正在 Linux 上运行                    | `systemctl status redis`                                   |
| 2. Redis 监听外网 IP（非 127.0.0.1）          | 配置 `bind 0.0.0.0`                                        |
| 3. Redis 设置了密码（强烈建议）               | `requirepass YourStrongPass`                               |
| 4. Linux 防火墙开放 6379 端口                 | `firewall-cmd --add-port=6379/tcp --permanent && --reload` |
| 5. 云服务器安全组开放 6379（如阿里云/腾讯云） | 控制台 → 安全组 → 放行 TCP 6379                            |
| 6. 你知道 Linux 的 **IP 地址**                | 在 Linux 执行 `ip addr` 或 `hostname -I`                   |

## 步骤1：redis.conf配置文件

编辑 Redis 配置文件（通常为 `/etc/redis/redis.conf`）

```
sudo vim /etc/redis/redis.conf
```

基本操作：

- 按 `i` 进入插入模式（可编辑）
- 按 `Esc` 退出编辑模式
- 输入 `:wq` 保存并退出
- 输入 `:q!` 不保存强制退出
- 输入搜索命令：`/关键词`

确保以下配置：

```
# 允许所有 IP 连接（测试用），生产环境建议限制 IP
bind 0.0.0.0

# 设置强密码（必须！）
requirepass YourStrongPassword123! (111111`)

# 关闭保护模式（因为 bind 0.0.0.0）
protected-mode no

# 端口默认 6379，一般不用改
port 6379

#后台运行
daemonize yes
```

改完后确保生效记得重启，重启 Redis

```
sudo systemctl restart redis
```

------

## 步骤 2：开放 Linux 防火墙

CentOS / RHEL（firewalld）

```
sudo firewall-cmd --add-port=6379/tcp --permanent
sudo firewall-cmd --reload
```

Ubuntu / Debian（ufw）

```
sudo ufw allow 6379/tcp
```

------

## 步骤 3：获取 Linux 的 IP 地址

```
ip addr show
# 或
hostname -I
```

输出示例：

```
192.168.14.131
```

> ✅ 这个 `192.168.14.131` 就是你在 Windows 上要连接的地址。

> ⚠️ 如果是云服务器（如阿里云），用**公网 IP**；如果是本地虚拟机，用**内网 IP**。

------

## Redis启动方式详解

### 1.直接启动

```
redis-server
```

- **特点**：会占用当前终端窗口

- **使用场景**：调试时用，但不能继续在终端里输入其他命令

- **小技巧**：如果想在前台运行并看到详细日志，可以加`--loglevel verbose`

  ```
  redis-server --loglevel verbose
  ```

- --port 6380：指定端口

### 2.后台启动（推荐！）

```
redis-server --daemonize yes
```

- **特点**：Redis在后台运行，终端可以继续使用
- **优点**：不会占用你的终端，适合日常使用
- **备选方案**：在配置文件`redis.conf`中设置`daemonize yes`，然后直接用`redis-server redis.conf`启动
-  **重要提示**：`--daemonize yes`是后台运行的关键参数，如果没有这个，Redis会前台运行。

### 3.指定配置文件启动（启动多个Redis实例）

复制配置文件

```
cp redis.conf redis6379.conf
cp redis.conf redis6380.conf
```

编辑新配置文件（正确方式）

```
# 使用vim编辑
vim redis6379.conf
vim redis6380.conf
```

这种方式通过指定配置文件启动Redis，可以灵活设置各种参数。

```bash
# 设置监听端口
port 6380

# 设置数据存储目录（确保目录存在）
dir /var/lib/redis/6380

# 设置PID文件（每个实例必须不同）
pidfile /var/run/redis_6380.pid

# 设置日志文件（每个实例必须不同）
logfile "/var/log/redis/redis_6380.log"

# 设置持久化文件（每个实例必须不同）
dbfilename dump_6380.rdb

# 允许所有IP连接（测试用，生产环境建议限制IP）
bind 0.0.0.0

# 设置强密码（生产环境必须设置）
requirepass YourStrongPassword123!

# 关闭保护模式（因为bind 0.0.0.0）
protected-mode no

# 后台运行
daemonize yes

#从机访问主机的通行密码masterauth，必须
#从机设置
replicaof 192.168.111 6379 #访问的master的IP和端口
masterauth "111111"
```

启动两个实例

```
# 启动6379（默认配置）
redis-server redis6379.conf

# 启动6380（新配置）
redis-server redis6380.conf
```

如何验证启动并连接

```
# 连接到6379
redis-cli -p 6379 ping
# 应该返回 PONG

# 连接到6380
redis-cli -p 6380 -a YourStrongPassword123!
# 应该返回 PONG
```

停止服务（正确方式使用systemd或redis-cli）：

```sh
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -a 来指定密码
# -p指定端口，不写-p默认是6379
# 单实例关闭 方式一（安全）：
redis-cli -a 111111 shutdown

# 多实例关闭
redis-cli -p 6379 shutdown

# 方式二：
sudo systemctl stop redis

# 方式三：
#查看占用6379端口的进程
# Linux/Mac
lsof -i :6379

#查看Redis进程
ps -ef | grep redis
#直接使用 kill -9 会强制终止进程，可能导致数据丢失
kill -9 12821   
```

**特点**：

- 可以使用自定义配置文件
- **适合多实例部署（如6379和6380同时运行）**
- 配置文件中可以设置端口、密码、日志等



### 4.开机自启（推荐）

- 首先，新建一个系统服务文件：

```
vi /etc/systemd/system/redis.service
```

- 添加内容：

```
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
User=redis
Group=redis
Type=forking
ExecStart=/root/redis-7.2.4/src/redis-server /root/redis-7.2.4/redis.conf
ExecStop=/root/redis-7.2.4/src/redis-cli -p 6379 shutdown
Restart=always
RestartSec=5
PrivateTmp=true
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

- 设置开机自启

  ```
  # 重新加载 systemd 配置
  sudo systemctl daemon-reload
  
  # 启动 Redis 服务
  sudo systemctl start redis
  
  # 设置开机自启
  sudo systemctl enable redis
  
  # 验证开机自启
  sudo systemctl is-enabled redis
  # 应该返回 "enabled"
  
  ```

- 服务管理命令

  ```
  # 启动服务
  sudo systemctl start redis
  
  # 停止服务
  sudo systemctl stop redis
  
  # 重启服务
  sudo systemctl restart redis
  
  # 查看服务状态
  sudo systemctl status redis
  
  # 查看服务日志
  sudo journalctl -u redis -f
  ```

- 验证：

  ```
  sudo systemctl is-enabled redis
  # 应该返回"enabled"
  ```

- **启动redis命令行客户端**

  ```
  [root@192 redis-7.2.4]# systemctl restart redis
  [root@192 redis-7.2.4]# redis-cli
  127.0.0.1:6379> auth 111111
  OK
  127.0.0.1:6379>quit
  ```

## 步骤 4：在 Windows 上连接 Redis

### 方法 1：使用命令行 `redis-cli`（需安装 Redis for Windows）

1. 下载 Windows 版 Redis 命令行工具：
   - Redis for Windows
2. 解压后将目录添加到系统 PATH

3. 安装后，在 CMD 或 PowerShell 中执行：

```
redis-cli -h 192.168.14.131 -p 6379
```

4. 然后输入密码：

```
192.168.14.131:6379> AUTH YourStrongPassword123!
OK
192.168.14.131:6379> PING
PONG
```

------

### 方法 2：使用图形化工具（推荐，更方便）

1. 推荐工具：
   - **Another Redis Desktop Manager**（免费开源，跨平台）
     GitHub: https://github.com/qishibo/AnotherRedisDesktopManager

2. 建立连接

   - 点击左上角的`连接到Redis服务器`按钮：

     ![image-20251102113525504](images/image-20251102113525504.png)

   - 在弹出的窗口中填写Redis服务信息：

     ![image-20251102113600917](images/image-20251102113600917.png)

     连接配置：

| 字段 | 值                                |
| ---- | --------------------------------- |
| Host | `192.168.14.131`（你的 Linux IP） |
| Port | `6379`                            |
| Auth | `YourStrongPassword123!`          |

3. 点击确定后，在左侧菜单会出现这个链接：

   ![image-20251102113739683](images/image-20251102113739683.png)

   点击即可建立连接了。

   ![image-20251102113758358](images/image-20251102113758358.png)

   Redis默认有16个仓库，编号从0至15.  通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称。

# 2.Redis常见命令

Redis是典型的key-value数据库，**key一般是字符串，而value包含很多不同的数据类型：**

![Redis 数据类型概览](images/redis-overview-of-data-types-2023-09-28.jpg)



Redis为了方便我们学习，将操作不同数据类型的命令也做了分组，在官网（ [https://redis.io/commands ](https://redis.io/commands)）可以查看到不同的命令：

https://www.redis.cn/commands.html

![](assets/5Lcr3BE.png)

**不同类型的命令称为一个group，我们也可以通过help命令来查看各种不同group的命令：**

![](assets/suevOIR.png)

接下来，我们就学习常见的五种基本数据类型的相关命令。



## 2.1.Redis通用命令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

- KEYS：查看符合模板的所有key。KEYS heima:*
- DEL：删除一个指定的key。DEL heima:user:1
- EXISTS：判断key是否存在。EXISTS heima:user:1
- EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除。EXPIRE heima:user:1 3600
- TTL：查看一个KEY的剩余有效期。TTL heima:user:1
- move key dbindex【0-15】。将当前数据库的key移动到给定的数据库db当中
- select dbindex。切换到数据库【0-15】，默认为0
- dbsize。查看当前数据库key的数量
- flushdb。清空当前库。
- flushall。通杀全部库。
- type key：键的数据结构类型

通过help [command] 可以查看一个命令的具体用法，例如：

```sh
# 查看keys命令的帮助信息：
127.0.0.1:6379> help keys

KEYS pattern
summary: Find all keys matching the given pattern
since: 1.0.0
group: generic
```

![image-20251103151757643](images/image-20251103151757643.png)

![image-20251103151828954](images/image-20251103151828954.png)

### 单个键管理

- `rename key newkey`：键重命名

  - 下面操作将键python重命名为java：

    ```
    127.0.0.1:6379> set python jedis
    OK
    127.0.0.1:6379> rename python java
    OK
    127.0.0.1:6379> get python
    (nil)
    127.0.0.1:6379> get java
    "jedis"
    ```

    如果在rename之前，键java已经存在，那么它的值也将被覆盖

    ```
    127.0.0.1:6379> set java 123
    OK
    127.0.0.1:6379> set python 89
    OK
    127.0.0.1:6379> rename python java
    OK
    127.0.0.1:6379> get java
    "89"
    ```

  - **为了防止被强行rename，Redis提供了renamenx命令，确保只有newKey不存在时候才被覆盖**

  - 在使用重命名命令时，有两点需要注意：

    - 由于重命名键期间会执行del命令删除旧的键，**如果键对应的值比较大，会存在阻塞Redis的可能性**，这点不要忽视。

    - 如果rename和renamenx中的key和newkey如果是相同的，在Redis3.2和之前版本返回结果略有不同。

    - Redis3.2中会返回OK：

      ```
      127.0.0.1:6379> rename key key
      OK
      ```

    - Redis3.2之前的版本会提示错误：

      ```
      127.0.0.1:6379> rename key key
      (error) ERR source and destination objects are the same
      ```

- `randomkey`：随机返回一个键

- 键过期：它可以自动将带有过期时间的键删除，在许多应用场景都非常有帮助。除了expire、ttl命令以外，Redis还提供了expireat、pexpire、pexpireat、pttl、persist等一系列命令，下面分别进行说明：

  - expire key seconds：键在seconds秒后过期。

  - expireat key timestamp：键在秒级时间戳timestamp后过期。

    - expireat命令可以设置键的秒级过期时间戳，例如如果需要将键hello在2016-08-0100：00：00（秒级时间戳为1469980800）过期，可以执行如下操作：

      ```
      127.0.0.1:6379> expireat hello 1469980800
      (integer) 1
      ```

  - ttl命令和pttl都可以查询键的剩余过期时间，但是pttl精度更高可以达到毫秒级别，有3种返回值：

    - 大于等于0的整数：键剩余的过期时间（ttl是秒，pttl是毫秒）。
    - -1：键没有设置过期时间
    - -2：键不存在

  - Redis2.6版本后提供了毫秒级的过期方案：

    - pexpire key milliseconds：键在milliseconds毫秒后过期。
    - pexpireat key milliseconds-timestamp键在毫秒级时间戳timestamp后过期。

  - 在使用Redis相关过期命令时，需要注意以下几点。

    - 1)如果expire key的键不存在，返回结果为0：

      ```
      127.0.0.1:6379> expire not_exist_key 30
      (integer) 0
      ```

    - 2)如果过期时间为负值，键会立即被删除，犹如使用del命令一样：

      ```
      127.0.0.1:6379> set hello world
      OK
      127.0.0.1:6379> expire hello -2
      (integer) 1
      127.0.0.1:6379> get hello
      (nil)
      ```

    - 3)**persist命令可以将键的过期时间清除**：

      ```
      127.0.0.1:6379> hset key f1 v1
      (integer) 1
      127.0.0.1:6379> expire key 50
      (integer) 1
      127.0.0.1:6379> ttl key
      (integer) 46
      127.0.0.1:6379> persist key
      (integer) 1
      127.0.0.1:6379> ttl key
      (integer) -1
      ```

    - 4)对于字符串类型键，执行set命令会去掉过期时间，这个问题很容易在开发中被忽视。如下是Redis源码中，set命令的函数setKey，可以看到最后执行了removeExpire(db，key)函数去掉了过期时间：

      ```
      void setKey(redisDb *db, robj *key, robj *val) {
          if (lookupKeyWrite(db,key) == NULL) {
              dbAdd(db,key,val);
          } else {
              dbOverwrite(db,key,val);
          }
          incrRefCount(val);
      // 去掉过期时间
          removeExpire(db,key);
          signalModifiedKey(db,key);
      }
      ```

      下面的例子证实了set会导致过期时间失效，因为ttl变为-1：

      ```
      127.0.0.1:6379> expire hello 50
      (integer) 1
      127.0.0.1:6379> ttl hello
      (integer) 46
      127.0.0.1:6379> set hello world
      OK
      127.0.0.1:6379> ttl hello
      (integer) -1
      ```

    - 5)**Redis不支持二级数据结构（例如哈希、列表）内部元素的过期功能，例如不能对列表类型的一个元素做过期时间设置。**

    - 6)**setex命令作为set+expire的组合，不但是原子执行，同时减少了一次网络通讯的时间**。

- 迁移键：迁移键功能非常重要，因为有时候我们只想把部分数据由一个Redis迁移到另一个Redis（例如从生产环境迁移到测试环境），Redis发展历程中提供了move、dump+restore、migrate三组迁移键的方法

  - `move key db`：move key db就是把指定的键从源数据库移动到目标数据库中

  - `dump+restore`：dump+restore可以实现在不同的Redis实例之间进行数据迁移的功能，整个迁移的过程分为两步：

    ```
    dump key
    restore key ttl value
    ```

    - 1)在源Redis上，dump命令会将键值序列化，格式采用的是RDB格式。

    - 2)在目标Redis上，restore命令将上面序列化的值进行复原，其中ttl参数代表过期时间，如果ttl=0代表没有过期时间。

    - 有关dump+restore有两点需要注意：**第一，整个迁移过程并非原子性的，而是通过客户端分步完成的。第二，迁移过程是开启了两个客户端连接，所以dump的结果不是在源Redis和目标Redis之间进行传输**，下面用一个例子演示完整过程。

    - 1)在源Redis上执行dump：

      ````
      redis-source> set hello world
      OKre
      redis-source> dump hello
      "\x00\x05world\x06\x00\x8f<T\x04%\xfcNQ"
      ````

    - 2)在目标Redis上执行restore：

      ```
      redis-target> get hello
      (nil)
      redis-target> restore hello 0 "\x00\x05world\x06\x00\x8f<T\x04%\xfcNQ"
      OK
      redis-target> get hello
      "world"
      ```

      

    - ![image-20251121224814826](images/image-20251121224814826.png)

  - migrate命令也是用于在Redis实例间进行数据迁移的，实际上migrate命令就是将dump、restore、del三个命令进行组合，从而简化了操作流程。

    ```
    migrate host port key|"" destination-db timeout [copy] [replace] [keys key [key …]]
    ```

    实现过程和dump+restore基本类似，但是有3点不太相同：**第一，整个过程是原子执行的，不需要在多个Redis实例上开启客户端的，只需要在源Redis上执行migrate命令即可。第二，migrate命令的数据传输直接在源Redis和目标Redis上完成的。第三，目标Redis完成restore后会发送OK给源Redis，源Redis接收后会根据migrate对应的选项来决定是否在源Redis上删除对应的键。**

    ![image-20251121230121480](images/image-20251121230121480.png)

    下面对migrate的参数进行逐个说明：

    - host：目标Redis的IP地址。
    - port：目标Redis的端口。
    - key|""：在Redis3.0.6版本之前，migrate只支持迁移一个键，所以此处是要迁移的键，但Redis3.0.6版本之后支持迁移多个键，如果当前需要迁移多个键，此处为空字符串""。
    - destination-db：目标Redis的数据库索引，例如要迁移到0号数据库，这里就写0。
    - timeout：迁移的超时时间（单位为毫秒）。
    - [copy]：如果添加此选项，迁移后并不删除源键。
    - [replace]：如果添加此选项，migrate不管目标Redis是否存在该键都会正常迁移进行数据覆盖。
    - [keys key[key…]]：迁移多个键，例如要迁移key1、key2、key3，此处填写“keys key1 key2 key3”。

  - 下面用示例演示migrate命令，为了方便演示源Redis使用6379端口，目标Redis使用6380端口，现要将源Redis的键hello迁移到目标Redis中，会分为如下几种情况：

    - 情况1：源Redis有键hello，目标Redis没有：

    ```
    127.0.0.1:6379> migrate 127.0.0.1 6380 hello 0 1000 auth 111111
    OK
    ```

    - 情况2：源Redis和目标Redis都有键hello：

    ```
    127.0.0.1:6379> get hello
    "world"
    127.0.0.1:6380> get hello
    "redis"
    ```

    如果migrate命令没有加replace选项会收到错误提示，如果加了replace会返回OK表明迁移成功：

    ```
    127.0.0.1:6379> migrate 127.0.0.1 6379 hello 0 1000
    (error) ERR Target instance replied with error: BUSYKEY Target key name already exists.
    127.0.0.1:6379> migrate 127.0.0.1 6379 hello 0 1000 replace
    OK
    ```

    - 情况3：源Redis没有键hello。如下所示，此种情况会收到nokey的提示：

    ```
    127.0.0.1:6379> migrate 127.0.0.1 6380 hello 0 1000
    NOKEY
    ```

    - 情况4：迁移多个键

      ```
      127.0.0.1:6379> mset key1 value1 key2 value2 key3 value3
      OK
      127.0.0.1:6379> migrate 127.0.0.1 6380 "" 0 5000 keys key1 key2 key3
      OK
      ```

- move、dump+restore、migrate三个命令比较

  ![image-20251122194050339](images/image-20251122194050339.png)

### 遍历键

- `keys pattern`：全量遍历键

  - pattern的几种情况：

    - *代表匹配任意字符。
    - 代表匹配一个字符。
    - []代表匹配部分字符，例如[1，3]代表匹配1，3，[1-10]代表匹配1到10的任意数字。
    - \x用来做转义，例如要匹配星号、问号需要进行转义。

  - 示例1：

    ```
    127.0.0.1:6379> dbsize
    (integer) 0
    127.0.0.1:6379> mset hello world redis best jedis best hill high
    OK
    127.0.0.1:6379> keys *
    1) "hill"
    2) "jedis"
    3) "redis"
    4) "hello"
    ```

  - 示例2：

    ```
    127.0.0.1:6379> keys [j,r]edis
    1) "jedis"
    2) "redis"
    127.0.0.1:6379> keys hll*
    1) "hill"
    2) "hello"
    ```

  - 当需要遍历所有键时（例如检测过期或闲置时间、寻找大对象等），keys是一个很有帮助的命令，例如想删除所有以video字符串开头的键，可以执行如下操作：

    ```
    redis-cli keys video* | xargs redis-cli del
    ```

    但是如果考虑到Redis的单线程架构就不那么美妙了，**如果Redis包含了大量的键，执行keys命令很可能会造成Redis阻塞，所以一般建议不要在生产环境下使用keys命令**。但有时候确实有遍历键的需求该怎么办，可以在以下三种情况使用：

    - 在一个不对外提供服务的Redis从节点上执行，这样不会阻塞到客户端的请求，但是会影响到主从复制，有关主从复制我们将在第6章进行详细介绍。
    - 如果确认键值总数确实比较少，可以执行该命令。
    - 使用下面要介绍的scan命令渐进式的遍历所有键，可以有效防止阻塞。

- `scan`：渐进式遍历

  - 和keys命令执行时会遍历所有键不同，scan采用**渐进式遍历**的方式来**解决keys命令可能带来的阻塞问题**，**每次scan命令的时间复杂度是O(1)**，但是要真正实现keys的功能，需要执行多次scan。Redis存储键值对实际使用的是**hashtable的数据结构**

    ![image-20251122204317162](images/image-20251122204317162.png)

  - 那么每次执行scan，可以想象成只扫描一个字典中的一部分键，直到将字典中的所有键遍历完毕。scan的使用方法如下：

    ```
    scan cursor [match pattern] [count number]
    ```

    - cursor是必需参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值，直到游标值为0，表示遍历结束。

    - match pattern是可选参数，它的作用的是做模式的匹配，这点和keys的模式匹配很像。

    - count number是可选参数，它的作用是表明每次要遍历的键个数，默认值是10，此参数可以适当增大。

    - 现有一个Redis有26个键（英文26个字母），现在要遍历所有的键，使用scan命令效果的操作如下。第一次执行scan0，返回结果分为两个部分：第一个部分6就是下次scan需要的cursor，第二个部分是10个键：

      ```
      127.0.0.1:6379> scan 0
      1) "6"
      2)  1) "w"
          2) "i"
          3) "e"
          4) "x"
          5) "j"
          6) "q"
          7) "y"
          8) "u"
          9) "b"
         10) "o"
      ```

      使用新的cursor="6"，执行scan6：

      ```
      127.0.0.1:6379> scan 6
      1) "11"
      2)  1) "h"
          2) "n"
          3) "m"
          4) "t"
          5) "c"
          6) "d"
          7) "g"
          8) "p"
          9) "z"
         10) "a"
      ```

      这次得到的cursor="11"，继续执行scan11得到结果cursor变为0，说明所有的键已经被遍历过了：

      ```
      127.0.0.1:6379> scan 11
      1) "0"
      2)  1) "s"
          2) "f"
          3) "r"
          4) "v"
          5) "k"
          6) "l"
      ```

  - 除了scan以外，Redis提供了面向哈希类型、集合类型、有序集合的扫描遍历命令，解决诸如hgetall、smembers、zrange可能产生的阻塞问题，对应的命令分别是hscan、sscan、zscan，它们的用法和scan基本类似
  - 渐进式遍历可以有效的解决keys命令可能产生的阻塞问题，但是scan并非完美无瑕，**如果在scan的过程中如果有键的变化（增加、删除、修改），那么遍历效果可能会碰到如下问题：新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan并不能保证完整的遍历出来所有的键**，这些是我们在开发时需要考虑的。

- 数据库管理
  - `select dnIndex`：切换数据库
  - `flushdb`：只清楚当前数据库
  - `flushsall`：会清楚所有数据库
  - flushdb/flushall命令可以非常方便的清理数据，但是也带来两个问题：
    - flushdb/flushall命令会将所有数据清除，一旦误操作后果不堪设想，rename-command配置规避这个问题，以及如何在误操作后快速恢复数据。
    - **如果当前数据库键值数量比较多，flushdb/flushall存在阻塞Redis的可能性**。所以在使用flushdb/flushall一定要小心谨慎。

## 2.2.String类型

### **特点**

String类型，也就是字符串类型，是Redis中最简单的存储类型。

其value是字符串，不过根据字符串的格式不同，又可以分为3类：

- string：普通字符串
- int：整数类型，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m.

![](assets/VZqpv73.png)

### **常用命令**

- 设置键值

  - `set key value [EX seconds] [PX milliseconds] [NX|XX]`

    - EX：key在多少秒之后过期
    - PX：key在多少毫秒之后过期
    - NX：当key不存在的时候，才创建key，效果等同于setnx
    - XX：与nx相反，键必须存在，才可以设置成功，用于更新。

  - setex/setnx

    - 示例：

      ```
      当前键hello不存在：
      127.0.0.1:6379> exists hello
      (integer) 0
      
      设置键为hello，值为world的键值对：
      127.0.0.1:6379> set hello world
      OK
      
      因为键hello已存在，所以setnx失败，返回结果为0：
      127.0.0.1:6379> setnx hello redis
      (integer) 0
      
      因为键hello已存在，所以set xx成功，返回结果为OK：
      127.0.0.1:6379> set hello jedis xx
      OK
      ```

      setnx和setxx在实际使用中有什么应用场景吗？以setnx命令为例子，由于Redis的单线程命令处理机制，如果有多个客户端同时执行setnx key value，根据setnx的特性只有一个客户端能设置成功，setnx可以作为分布式锁的一种实现方案，Redis官方给出了使用setnx实现分布式锁的方法：http://redis.io/topics/distlock。

- 获取值：`get key`

- 批量设置值：`mset key value [key value …]`

- 批量获取值：`mget key [key …]`。如果有些键不存在，那么它的值为nil（空），结果是按照传入键的顺序返回

- 数值增减：一定要是数字才能进行加减

  - `INCR key`：递增数字
  - `INCEBY key increament`：增加指定的整数
  - `DECR key`：递减数值
  - `DECRBY key decrement`：减少指定的整数

- 获取字符串长度和内容追加

  - `STRLEN key`

    - 示例

      ```
      127.0.0.1:6379> get key
      "redisworld"
      127.0.0.1:6379> strlen key
      (integer) 10
      ```

  - `APPEND key value`

- getset（先GET后SET）：将给定key的值设为value，并返回key的旧值（old value）

  - 示例：

    ```
    127.0.0.1:6379> set k1 haha
    OK
    127.0.0.1:6379> get k1
    "haha"
    127.0.0.1:6379> getset k1 nini
    "haha"
    127.0.0.1:6379> get k1
    "nini"
    ```

- 设置指定位置的字符：`setrange key offeset value`

  - 示例：

  ```
  127.0.0.1:6379> set redis pest
  OK
  127.0.0.1:6379> setrange redis 0 b
  (integer) 4
  127.0.0.1:6379> get redis
  "best"
  ```

- 获取部分字符串：`getrange key start end`

### **key命名规范（避免冲突）**

Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢？

例如，需要存储用户、商品信息到redis，有一个用户id是1，有一个商品id恰好也是1，此时如果使用id作为key，那就会冲突了，该怎么办？

我们可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范：

Redis的key允许有多个单词形成层级结构，多个单词之间用':'隔开，格式如下：

```
	项目名:业务名:类型:id
```

这个格式并非固定，也可以根据自己的需求来删除或添加词条。这样以来，我们就可以把不同类型的数据区分开了。从而避免了key的冲突问题。

例如我们的项目名称叫 heima，有user和product两种不同类型的数据，我们可以这样定义key：

- user相关的key：**heima:user:1**

- product相关的key：**heima:product:1**



如果Value是一个Java对象，例如一个User对象，则可以将对象序列化为JSON字符串后存储：

| **KEY**         | **VALUE**                                    |
| --------------- | -------------------------------------------- |
| heima:user:1    | '{"id":1,  "name": "Jack", "age": 21}'       |
| heima:product:1 | '{"id":1,  "name": "小米11", "price": 4999}' |

并且，在Redis的桌面客户端中，还会以相同前缀作为层级结构，让数据看起来层次分明，关系清晰：

![](assets/InWMfeD.png)



## 2.3.Hash类型

### 特点

Hash类型，也叫散列，其value是一个无序字典，类似于Java中的HashMap结构。

String结构是将对象序列化为JSON字符串后存储，当需要修改对象某个字段时很不方便：

![](assets/x2zDBjf.png)



Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD：

![](assets/VF2EPt0.png)

### Hash的常见命令有：

- 设置值：`hset key field value`

  - 示例

    ```
    下面为user：1添加一对field-value：
    
    127.0.0.1:6379> hset user:1 name tom
    (integer) 1
    ```

  - 如果设置成功会返回1，反之会返回0。此外Redis提供了hsetnx命令，它们的关系就像set和setnx命令一样，只不过作用域由键变为field。

- 获取值：`hget key field`

- hdel会删除一个或多个field，返回结果为成功删除field的个数：`hdel key field [field …]`

- 计算field个数：`hlen key`

  - 示例

    ```
    127.0.0.1:6379> hset user:1 name tom
    (integer) 1
    127.0.0.1:6379> hset user:1 age 23
    (integer) 1
    127.0.0.1:6379> hset user:1 city tianjin
    (integer) 1
    127.0.0.1:6379> hlen user:1
    (integer) 3
    ```

- 批量设置或获取field-value

  ```
  hmget key field [field …]
  hmset key field value [field value …]
  ```

- 判断field是否存在：`hexists key field`

- 获取所有field：`hkeys key`

- 获取所有value：`hvals key`

- 获取所有的field-value：`hgetall key`

  - 示例：下面操作获取user：1所有的field-value：

    ```
    127.0.0.1:6379> hgetall user:1
    1) "name"
    2) "mike"
    3) "age"
    4) "12"
    5) "city"
    6) "tianjin"
    ```

  - 在使用hgetall时，如果哈希元素个数比较多，会存在阻塞Redis的可能。如果开发人员只需要获取部分field，可以使用hmget，如果一定要获取全部field-value，可以使用hscan命令，该命令会渐进式遍历哈希类型

- hincrby和hincrbyfloat，就像incrby和incrbyfloat命令一样，但是它们的作用域是filed。`hincrby key field increment`/`hincrbyfloat key field increment`

- 计算value的字符串长度：`hstrlen key field`

  - 示例：例如hget user:1 name的value是tom，那么hstrlen的返回结果是3

    ```
    127.0.0.1:6379> hstrlen user:1 name
    (integer) 3
    ```

## 2.4.List类型

### 特点

Redis中的List类型与Java中的LinkedList类似，可以看做是一个**双向链表结构**。既可以支持正向检索和也可以支持反向检索。

特征也与LinkedList类似：

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。



### List的常见命令有：

- `lpush/rpush/lrange`：从左侧插入元素/从右侧插入元素/获取指定范围元素

  - 示例：

    ```
    127.0.0.1:6379> lpush list1 1 2 3 4 5
    (integer) 5
    127.0.0.1:6379> rpush list1 6 7 8 9
    (integer) 9
    127.0.0.1:6379> type list1
    list
    127.0.0.1:6379> lrange list1 0 -1
    1) "5"
    2) "4"
    3) "3"
    4) "2"
    5) "1"
    6) "6"
    7) "7"
    8) "8"
    9) "9"
    ```

- `lpop/rpop`：移除并返回左侧第一个元素/移除并返回右侧第一个元素

  - 示例：

    ```bash
    127.0.0.1:6379> lrange list1 0 -1
    1) "5"
    2) "4"
    3) "3"
    4) "2"
    5) "1"
    6) "6"
    7) "7"
    8) "8"
    9) "9"
    127.0.0.1:6379> lpop list1
    "5"
    127.0.0.1:6379> lrange list1 0 -1
    1) "4"
    2) "3"
    3) "2"
    4) "1"
    5) "6"
    6) "7"
    7) "8"
    8) "9"
    127.0.0.1:6379> rpop list1
    "9"
    127.0.0.1:6379> lrange list1 0 -1
    1) "4"
    2) "3"
    3) "2"
    4) "1"
    5) "6"
    6) "7"
    7) "8"
    ```

- `LINDEX`：通过索引获取列表中的元素

  - 示例：

    ```
    127.0.0.1:6379> lrange list1 0 -1
    1) "4"
    2) "3"
    3) "2"
    4) "1"
    5) "6"
    6) "7"
    7) "8"
    127.0.0.1:6379> lindex list1 0
    "4"
    127.0.0.1:6379> lindex list1 3
    "1"
    ```

- `llen`：获取列表中元素的个数

- `lrem key 数字N 给定值v1`：（删除N个值等于v1的元素），从left往right删除2个值等于v1的元素，返回的值为实际删除的数量

  - 示例：`lrem list3 2 v1`（删除2个重复v1）

    ```
    127.0.0.1:6379> lpush list3 v1 v1 v1 v2 v3 v4 v5
    (integer) 7
    127.0.0.1:6379> lrange list3 0 -1
    1) "v5"
    2) "v4"
    3) "v3"
    4) "v2"
    5) "v1"
    6) "v1"
    7) "v1"
    127.0.0.1:6379> lrem list3 2 v1
    (integer) 2
    127.0.0.1:6379> lrange list3 0 -1
    1) "v5"
    2) "v4"
    3) "v3"
    4) "v2"
    5) "v1"
    ```

- `ltrim key`：开始index结束index，截取指定范围的值后再赋值给key

- `rpoplpush 源列表 目的列表`：移除列表的最后一个元素，并将该元素添加到另一个列表并返回

  - 示例：

    ```
    127.0.0.1:6379> lrange list1 0 -1
    1) "4"
    2) "3"
    3) "2"
    4) "1"
    5) "6"
    6) "7"
    7) "8"
    127.0.0.1:6379> lrange list2 0 -1
    (empty array)
    127.0.0.1:6379> lrange list3 0 -1
    1) "v5"
    2) "v4"
    3) "v3"
    4) "v2"
    5) "v1"
    127.0.0.1:6379> rpoplpush list1 list3
    "8"
    127.0.0.1:6379> lrange list1 0 -1
    1) "4"
    2) "3"
    3) "2"
    4) "1"
    5) "6"
    6) "7"
    127.0.0.1:6379> lrange list3 0 -1
    1) "8"
    2) "v5"
    3) "v4"
    4) "v3"
    5) "v2"
    6) "v1"
    ```

- `lset key index value`

  - 示例

    ```
    127.0.0.1:6379> lrange list3 0 -1
    1) "8"
    2) "v5"
    3) "v4"
    4) "v3"
    5) "v2"
    6) "v1"
    127.0.0.1:6379> lset list3 3 mysql
    OK
    127.0.0.1:6379> lrange list3 0 -1
    1) "8"
    2) "v5"
    3) "v4"
    4) "mysql"
    5) "v2"
    6) "v1"
    ```

- linsert key before/after：已有值插入的新值

- 阻塞式弹出如下：

  ```
  blpop key [key …] timeout
  brpop key [key …] timeout
  ```

  blpop和brpop是lpop和rpop的阻塞版本，它们除了弹出方向不同，使用方法基本相同，所以下面以brpop命令进行说明，brpop命令包含两个参数：

  - key[key…]：多个列表的键。
  - timeout：阻塞时间（单位：秒）。

1)列表为空：如果timeout=3，那么客户端要等到3秒后返回，如果timeout=0，那么客户端一直阻塞等下去：

```
127.0.0.1:6379> brpop list:test 3
(nil)
(3.10s)
127.0.0.1:6379> brpop list:test 0
…阻塞…
```

如果此期间添加了数据element1，客户端立即返回：

```
127.0.0.1:6379> brpop list:test 3
1) "list:test"
2) "element1"
(2.06s)
```

2)列表不为空：客户端会立即返回。

```
127.0.0.1:6379> brpop list:test 0
1) "list:test"
2) "element1"
```

在使用brpop时，有两点需要注意。

第一点，如果是多个键，那么brpop会从左至右遍历键，一旦有一个键能弹出元素，客户端立即返回：

```
127.0.0.1:6379> brpop list:1 list:2 list:3 0
..阻塞..
```

此时另一个客户端分别向list：2和list：3插入元素：

```
client-lpush> lpush list:2 element2
(integer) 1
client-lpush> lpush list:3 element3
(integer) 1
```

客户端会立即返回list：2中的element2，因为list：2最先有可以弹出的元素：

```
127.0.0.1:6379> brpop list:1 list:2 list:3 0
1) "list:2"
2) "element2_1"
```

第二点，如果多个客户端对同一个键执行brpop，那么最先执行brpop命令的客户端可以获取到弹出的值。

客户端1：

```
client-1> brpop list:test 0
…阻塞…
```

客户端2：

```
client-2> brpop list:test 0
…阻塞…
```

客户端3：

```
client-3> brpop list:test 0
…阻塞…
```

此时另一个客户端lpush一个元素到list：test列表中：

```
client-lpush> lpush list:test element
(integer) 1
```

那么客户端1最会获取到元素，因为客户端1最先执行brpop，而客户端2和客户端3继续阻塞：

```
client> brpop list:test 0
1) "list:test"
2) "element"
```



## 2.5.Set类型

### 特点

Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：

- 无序

- 元素不可重复

- 查找快

- 支持交集、并集、差集等功能



### Set的常见命令有：

- `sadd key element [element …]`：添加元素

- `srem key element [element …]`：删除元素

- `scard key`：计算元素个数

  - scard的时间复杂度为O(1)，它不会遍历集合所有元素，而是直接用Redis内部的变量

  - 示例

    ```
    127.0.0.1:6379> scard myset
    (integer) 1
    ```

- `sismember key element`：判读元素是否在集合中

- `srandmember key [count]`：随机从集合返回指定个数元素

  - [count]是可选参数，如果不写默认为1

  - 示例

    ```
    127.0.0.1:6379> srandmember myset 2
    1) "a"
    2) "c"
    127.0.0.1:6379> srandmember myset
    "d"
    ```

- `spop key`：从集合随机弹出元素

- `smembers key`：获取所有元素

- 集合间的操作

  - `sinter key [key …]`：求多个集合的交集

    - 示例

      现在有两个集合，它们分别是user：1：follow和user：2：follow：

      ```
      127.0.0.1:6379> sadd user:1:follow it music his sports
      (integer) 4
      127.0.0.1:6379> sadd user:2:follow it news ent sports
      (integer) 4
      ```

      例如下面代码是求user：1：follow和user：2：follow两个集合的交集，返回结果是sports、it：

      ```
      127.0.0.1:6379> sinter user:1:follow user:2:follow
      1) "sports"
      2) "it"
      ```

  - `sunion key [key …]`：求多个集合的并集

    - 例如下面代码是求user：1：follow和user：2：follow两个集合的并集，返回结果是sports、it、his、news、music、ent：

      ```
      127.0.0.1:6379> sunion user:1:follow user:2:follow
      1) "sports"
      2) "it"
      3) "his"
      4) "news"
      5) "music"
      6) "ent"
      ```

  - `sdiff key [key …]`：求多个集合的差集

    - 例如下面代码是求user：1：follow和user：2：follow两个集合的差集，返回结果是music和his：

      ```
      127.0.0.1:6379> sdiff user:1:follow user:2:follow
      1) "music"
      2) "his"
      ```

  - 将交集、并集、差集的结果保存

    ```
    sinterstore destination key [key …]
    suionstore  destination key [key …]
    sdiffstore  destination key [key …]
    ```

    集合间的运算在元素较多的情况下会比较耗时，所以Redis提供了上面三个命令（原命令+store）将集合间交集、并集、差集的结果保存在destination key中，例如下面操作将user：1：follow和user：2：follow两个集合的交集结果保存在user：1_2：inter中，user：1_2：inter本身也是集合类型：

    ```
    127.0.0.1:6379> sinterstore user:1_2:inter user:1:follow user:2:follow
    (integer) 2
    127.0.0.1:6379> type user:1_2:inter
    set
    127.0.0.1:6379> smembers user:1_2:inter
    1) "it"
    2) "sports"
    ```

    

  

  

  







例如两个集合：s1和s2:

![](assets/ha8x86R.png)

求交集：SINTER s1 s2

求s1与s2的不同：SDIFF s1 s2

![](assets/L9vTv2X.png)

## 2.6.SortedSet类型

### 特点

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

SortedSet具备下列特性：

- 可排序
- 元素不重复
- 查询速度快

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。

### SortedSet的常见命令有：

- `zadd key score member [score member …]`：添加成员

  - 下面操作向有序集合user：ranking添加用户tom和他的分数251：

    ```
    127.0.0.1:6379> zadd user:ranking 251 tom
    (integer) 1
    127.0.0.1:6379> zadd user:ranking 1 kris 91 mike 200 frank 220 tim 250 martin
    (integer) 5
    ```

  - 有关zadd命令有两点需要注意：

    - Redis3.2为zadd命令添加了nx、xx、ch、incr四个选项：
    - nx：member必须不存在，才可以设置成功，用于添加。
    - xx：member必须存在，才可以设置成功，用于更新。
    - ch：返回此次操作后，有序集合元素和分数发生变化的个数
    - incr：对score做增加，相当于后面介绍的zincrby。
    - 有序集合相比集合提供了排序字段，但是也产生了代价，zadd的时间复杂度为O(log(n))，sadd的时间复杂度为O(1)。

- `zcard key`：计算成员个数

- `zscore key member`：计算某个成员的分数

  - tom的分数为251，如果成员不存在则返回nil：

    ```
    127.0.0.1:6379> zscore user:ranking tom
    "251"
    127.0.0.1:6379> zscore user:ranking test
    (nil)
    ```

- 计算成员的排名

  ```
  zrank key member
  zrevrank key member
  ```

  zrank是从分数从低到高返回排名，zrevrank反之。例如下面操作中，tom在zrank和zrevrank分别排名第5和第0（排名从0开始计算）。

  ```
  127.0.0.1:6379> zrank user:ranking tom
  (integer) 5
  127.0.0.1:6379> zrevrank user:ranking tom
  (integer) 0
  ```

- `zrem key member [member …]`：删除成员

  下面操作将成员mike从有序集合user：ranking中删除。

  ```
  127.0.0.1:6379> zrem user:ranking mike
  (integer) 1
  ```

  返回结果为成功删除的个数。

- `zincrby key increment member`：增加成员的分数

  - 下面操作给tom增加了9分，分数变为了260分：

    ```
    127.0.0.1:6379> zincrby user:ranking 9 tom
    "260"
    ```

- 返回指定排名范围的成员

  ```
  zrange    key start end [withscores]
  zrevrange key start end [withscores]
  ```

  有序集合是按照分值排名的，zrange是从低到高返回，zrevrange反之。下面代码返回排名最低的是三个成员，如果加上withscores选项，同时会返回成员的分数：

  ```
  127.0.0.1:6379> zrange user:ranking 0 2 withscores
  1) "kris"
  2) "1"
  3) "frank"
  4) "200"
  5) "tim"
  6) "220"
  127.0.0.1:6379> zrevrange user:ranking 0 2 withscores
  1) "tom"
  2) "260"
  3) "martin"
  4) "250"
  5) "tim"
  6) "220"
  ```

- 返回指定分数范围的成员

  ```
  zrangebyscore    key min max [withscores] [limit offset count]
  zrevrangebyscore key max min [withscores] [limit offset count]
  ```

  其中zrangebyscore按照分数从低到高返回，zrevrangebyscore反之。例如下面操作从低到高返回200到221分的成员，withscores选项会同时返回每个成员的分数。[limit offset count]选项可以限制输出的起始位置和个数：

  ```
  127.0.0.1:6379> zrangebyscore user:ranking 200 tinf withscores
  1) "frank"
  2) "200"
  3) "tim"
  4) "220"
  127.0.0.1:6379> zrevrangebyscore user:ranking 221 200 withscores
  1) "tim"
  2) "220"
  3) "frank"
  4) "200"
  ```

  同时min和max还支持开区间（小括号）和闭区间（中括号），-inf和+inf分别代表无限小和无限大：

  ```
  127.0.0.1:6379> zrangebyscore user:ranking (200 +inf withscores
  1) "tim"
  2) "220"
  3) "martin"
  4) "250"
  5) "tom"
  6) "260"
  ```

- `zcount key min max`：返回指定分数范围成员个数

  - 下面操作返回200到221分的成员的个数：

    ```
    127.0.0.1:6379> zcount user:ranking 200 221
    (integer) 2
    ```

- `zremrangebyrank key start end`：删除指定排名内的升序元素

  - 下面操作删除第start到第end名的成员：

    ```
    127.0.0.1:6379> zremrangebyrank user:ranking 0 2
    (integer) 3
    ```

- `zremrangebyscore key min max`：删除指定分数范围的成员

  - 下面操作将250分以上的成员全部删除，返回结果为成功删除的个数：

    ```
    下面操作将250分以上的成员全部删除，返回结果为成功删除的个数：
    ```

- 集合间的操作

  ```
  127.0.0.1:6379> zadd user:ranking:1 1 kris 91 mike 200 frank 220 tim 250 martin 251 tom
  (integer) 6
  127.0.0.1:6379> zadd user:ranking:2 8 james 77 mike 625 martin 888 tom
  (integer) 4
  ```

  - 交集

    ```
    zinterstore destination numkeys key [key …] [weights weight [weight …]]
      [aggregate sum|min|max]
    ```

    这个命令参数较多，下面分别进行说明：

    - destination：交集计算结果保存到这个键。
    - numkeys：需要做交集计算键的个数。
    - key[key…]：需要做交集计算的键。
    - weights weight[weight…]：每个键的权重，在做交集计算时，每个键中的每个member会将自己分数乘以这个权重，每个键的权重默认是1。
    - aggregate sum|min|max：计算成员交集后，分值可以按照sum（和）、min（最小值）、max（最大值）做汇总，默认值是sum。\

    下面操作对user：ranking：1和user：ranking：2做交集，weights和aggregate使用了默认配置，可以看到目标键user：ranking：1_inter_2对分值做了sum操作：

    ```
    127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1
        user:ranking:2
    (integer) 3
    127.0.0.1:6379> zrange user:ranking:1_inter_2 0 -1 withscores
    1) "mike"
    2) "168"
    3) "martin"
    4) "875"
    5) "tom"
    6) "1139"
    ```

    如果想让user：ranking：2的权重变为0.5，并且聚合效果使用max，可以执行如下操作：

    ```
    127.0.0.1:6379> zinterstore user:ranking:1_inter_2 2 user:ranking:1
      user:ranking:2 weights 1 0.5 aggregate max
    (integer) 3
    127.0.0.1:6379> zrange user:ranking:1_inter_2 0 -1 withscores
    1) "mike"
    2) "91"
    3) "martin"
    4) "312.5"
    5) "tom"
    6) "444"
    ```

  - 并集

    ```
    zunionstore destination numkeys key [key …] [weights weight [weight …]]
      [aggregate sum|min|max]
    ```

    - 该命令的所有参数和zinterstore是一致的，只不过是做并集计算，例如下面操作是计算user：ranking：1和user：ranking：2的并集，weights和aggregate使用了默认配置，可以看到目标键user：ranking：1_union_2对分值做了sum操作：

      ````
      127.0.0.1:6379> zunionstore user:ranking:1_union_2 2 user:ranking:1
          user:ranking:2
      (integer) 7
      127.0.0.1:6379> zrange user:ranking:1_union_2 0 -1 withscores
       1) "kris"
       2) "1"
       3) "james"
       4) "8"
       5) "mike"
       6) "168"
       7) "frank"
       8) "200"
       9) "tim"
      10) "220"
      11) "martin"
      12) "875"
      13) "tom"
      14) "1139"
      ````

## 慢查询分析

许多存储系统（例如MySQL）提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。**所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息（例如：发生时间，耗时，命令的详细信息）记录下来**，Redis也提供了类似的功能。

Redis客户端执行一条命令分为如下4个部分：

1)发送命令

2)命令排队

3)命令执行**（仅此阶段计入慢查询）**

4)返回结果

需要注意，慢查询只统计步骤3)的时间，所以没有慢查询并不代表客户端没有超时问题。

![image-20251122211216113](images/image-20251122211216113.png)

- Redis提供了**slowlog-log-slower-than和slowlog-max-len**配置来解决这两个问题。从字面意思就可以看出，slowlog-log-slower-than就是那个预设阀值，它的单位是**微秒**（1秒=1000毫秒=1000000微秒），默认值是10000，假如执行了一条“很慢”的命令（例如keys*），如果它的执行时间超过了10000微秒，那么它将被记录在慢查询日志中。

- 如果slowlog-log-slower-than=0会记录所有的命令，slowlog-log-slower-than<0对于任何命令都不会进行记录。

- 从字面意思看，slowlog-max-len只是说明了慢查询日志最多存储多少条，并没有说明存放在哪里？实际上**Redis使用了一个列表来存储慢查询日志，slowlog-max-len就是列表的最大长度**。一个新的命令满足慢查询条件时被插入到这个列表中，当慢查询日志列表已处于其最大长度时，最早插入的一个命令将从列表中移出，例如slowlog-max-len设置为5，当有第6条慢查询插入的话，那么队头的第一条数据就出列，第6条慢查询就会入列。

- 在Redis中有两种修改配置的方法，一种是修改配置文件，另一种是使用config set命令动态修改。例如下面使用config set命令将slowlog-log-slower-than设置为20000微秒，slowlog-max-len设置为1000：

  ```
  config set slowlog-log-slower-than 20000
  config set slowlog-max-len 1000
  config rewrite   如果要Redis将配置持久化到本地配置文件，需要执行config rewrite命令
  ```

- 虽然慢查询日志是存放在Redis内存列表中的，但是Redis并没有暴露这个列表的键，而是通过一组命令来实现对慢查询日志的访问和管理。

  - `slowlog get [n]`：获取慢查询日志

    - 下面操作返回当前Redis的慢查询，参数n可以指定条数：

      ```
      127.0.0.1:6379> slowlog get
       1) 1) (integer) 666
          2) (integer) 1456786500
          3) (integer) 11615
          4) 1) "BGREWRITEAOF"
       2) 1) (integer) 665
          2) (integer) 1456718400
          3) (integer) 12006
          4) 1) "SETEX"
             2) "video_info_200"
             3) "300"
             4) "2"
      …
      ```

      可以看到每个慢查询日志有4个属性组成，分别是慢查询日志的**标识id、发生时间戳、命令耗时、执行命令和参数**

      ![image-20251122212828198](images/image-20251122212828198.png)

  - `slowlog len`：获取慢查询日志列表当前的长度

  - `slowlog reset`：慢查询日志重置

- 慢查询功能可以有效地帮助我们找到Redis可能存在的瓶颈，但在实际使用过程中要注意以下几点：
  - slowlog-max-len配置建议：线上建议调大慢查询列表，记录慢查询时Redis会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除的可能，例如线上可设置为1000以上。
  - slowlog-log-slower-than配置建议：默认值超过10毫秒判定为慢查询，需要根据Redis并发量调整该值。由于Redis采用单线程响应命令，对于高流量的场景，如果命令执行时间在1毫秒以上，那么Redis最多可支撑OPS不到1000。因此对于高OPS场景的Redis建议设置为1毫秒。
  - **慢查询只记录命令执行时间，并不包括命令排队和网络传输时间。**因此客户端执行命令的时间会大于命令实际执行时间。因为命令执行排队机制，慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询，从而分析出是否为慢查询导致的命令级联阻塞。
  - 由于慢查询日志是一个**先进先出的队列**，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行slow get命令将慢查询日志持久化到其他存储中（例如MySQL），然后可以制作可视化界面进行查询，第13章介绍的Redis私有云CacheCloud提供了这样的功能，好的工具可以让问题排查事半功倍。

## Pipeline

### Pipeline概念

Pipeline（流水线）机制能将一组Redis命令进行组装，通过一次RTT传输给Redis，再将这组Redis命令的执行结果按顺序返回给客户端

![image-20251122215529947](images/image-20251122215529947.png)

使用Pipeline执行了n次命令，整个过程需要1次RTT

#### 示例

![image-20251220163405638](./assets/image-20251220163405638.png)





### 性能测试

- Pipeline执行速度一般比逐条执行要快。
- 客户端和服务端的网络延时越大，Pipeline的效果越明显。

![image-20251122215834045](images/image-20251122215834045.png)

### 原生批量命令与Pipeline对比

可以使用Pipeline模拟出批量操作的效果，但是在使用时要注意它与原生批量命令的区别，具体包含以下几点：

- 原生批量命令是原子的，Pipeline是非原子的。
- 原生批量命令一次只能执行一种命令，pipeline支持批量执行不同命令
- 原生批量命令是Redis服务端支持实现的，而Pipeline需要服务端和客户端的共同实现。

### Pipeline与事务对比

- 事务具有原子性,管道不具有原子性 
- 管道一次性将多条命令发送到服务器,事务是一条一条的发,事务只有在接收到exec命令后才会执行,
- 管道不会执行事务时会阻塞其他命令的执行,而执行管道中的命令时不会

### 使用pipeline

- pipeline缓冲的指令只是会依次执行,不保证原子性,如果执行中指令发生异常,将会继续执行后续的指令 
- 使用pipeline组装的命令个数不能太多,不然数据量过大客户端阻塞的时间可能过久,同时服务端此时也被迫回复一个队列答复,占用很多内存,

## 事务与Lua

### multi和exec/discard

**事务表示一组动作，要么全部执行，要么全部不执行。**例如在社交网站上用户A关注了用户B，那么需要在用户A的关注表中加入用户B，并且在用户B的粉丝表中添加用户A，这两个行为要么全部执行，要么全部不执行，否则会出现数据不一致的情况。

Redis提供了简单的事务功能，将一组需要一起执行的命令放到multi和exec两个命令之间。**multi命令代表事务开始，exec命令代表事务结束**，**它们之间的命令是原子顺序执行的**，例如下面操作实现了上述用户关注问题。

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> sadd user:a:follow user:b
QUEUED
127.0.0.1:6379> sadd user:b:fans user:a
QUEUED
```

可以看到sadd命令此时的返回结果是QUEUED，代表命令并没有真正执行，而是暂时保存在Redis中

如果此时另一个客户端执行sismember user：a：follow user：b返回结果应该为0。

```
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 0
```

只有当exec执行后，用户A关注用户B的行为才算完成，如下所示返回的两个结果对应sadd命令。

```
127.0.0.1:6379> exec
1) (integer) 1
2) (integer) 1
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 1
```

**如果要停止事务的执行，可以使用discard命令代替exec命令即可。**

```
127.0.0.1:6379> discard
OK
127.0.0.1:6379> sismember user:a:follow user:b
(integer) 0
```

### 事物错误，Redis处理机制（Watch)

- 命令错误
  - 例如操作错将set写成了sett，属于语法错误，会造成整个事务无法执行

- 运行时错误

  - 例如用户B在添加粉丝列表时，误把sadd命令写成了zadd命令，这种就是运行时命令，因为语法是正确的：

    ```
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> sadd user:a:follow user:b
    QUEUED
    127.0.0.1:6379> zadd user:b:fans 1 user:a
    QUEUED
    127.0.0.1:6379> exec
    1) (integer) 1
    2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
    127.0.0.1:6379> sismember user:a:follow user:b
    (integer) 1
    ```

    可以看到**Redis并不支持回滚功能**，sadd user​：a：​follow user：b命令已经执行成功，开发人员需要自己修复这类问题。有些应用场景需要在事务之前，确保事务中的key没有被其他客户端修改过，才执行事务，否则不执行（类似乐观锁）。**Redis提供了watch命令来解决这类问题**

    ![image-20251123100809731](images/image-20251123100809731.png)

    可以看到“客户端-1”在执行multi之前执行了watch命令，“客户端-2”在“客户端-1”执行exec之前修改了key值，造成事务没有执行（exec结果为nil），整个代码如下所示：

    ```
    #T1：客户端1
    127.0.0.1:6379> set key "java"
    OK
    #T2：客户端1
    127.0.0.1:6379> watch key
    OK
    #T3：客户端1
    127.0.0.1:6379> multi 
    OK
    #T4：客户端2
    127.0.0.1:6379> append key python
    (integer) 11
    #T5：客户端1
    127.0.0.1:6379> append key jedis
    QUEUED
    #T6：客户端1
    127.0.0.1:6379> exec
    (nil)
    #T7：客户端1
    127.0.0.1:6379> get key
    "javapython"
    ```

### Lua用法简述

Lua的官方网站(http://www.lua.org/)进行学习

- 数据类型及其逻辑处理

  Lua语言提供了如下几种数据类型：booleans（布尔）、numbers（数值）、strings（字符串）、tables（表格），和许多高级语言相比，相对简单。下面将结合例子对Lua的基本数据类型和逻辑处理进行说明。

  - 字符串

    下面定义一个字符串类型的数据：

    ```
    local strings val = "world"
    ```

    其中，local代表val是一个局部变量，如果没有local代表是全局变量。print函数可以打印出变量的值，例如下面代码将打印world，其中"--"是Lua语言的注释。

    ```lua
    -- 结果是"world"
    print(hello)
    ```

  - 数组

    在Lua中，如果要使用类似数组的功能，可以用tables类型，下面代码使用定义了一个tables类型的变量myArray，但和大多数编程语言不同的是，Lua的数组下标从1开始计算：

    ```
    local tables myArray = {"redis", "jedis", true, 88.0}
    --true
    print(myArray[3])
    ```

  - for

    下面代码会计算1到100的和，关键字for以end作为结束符：

    ```
    local int sum = 0
    for i = 1, 100
    do
        sum = sum + i
    end
    -- 输出结果为5050
    print(sum)
    ```

    要遍历myArray，首先需要知道tables的长度，只需要在变量前加一个#号即可：

    ```
    for i = 1, #myArray
    do
        print(myArray[i])
    end
    ```

    除此之外，Lua还提供了内置函数ipairs，使用for index，value ipairs(tables)可以遍历出所有的索引下标和值：

    ```
    for index,value in ipairs(myArray)
    do
        print(index)
        print(value)
    end
    ```

  - while

    ```
    local int sum = 0
    local int i = 0
    while i <= 100
    do
        sum = sum +i
        i = i + 1
    end
    --输出结果为5050
    print(sum)
    ```

  - if else

    要确定数组中是否包含了jedis，有则打印true，注意if以end结尾，if后紧跟then：

    ```
    local tables myArray = {"redis", "jedis", true, 88.0}
    for i = 1, #myArray
    do
        if myArray[i] == "jedis"
        then
            print("true")
            break
        else
            --do nothing
        end
    end
    ```

  - 哈希

    如果要使用类似哈希的功能，同样可以使用tables类型，例如下面代码定义了一个tables，每个元素包含了key和value，其中strings1..string2是将两个字符串进行连接：

    ```
    local tables user_1 = {age = 28, name = "tome"}
    --user_1 age is 28
    print("user_1 age is " .. user_1["age"])
    ```

    如果要遍历user_1，可以使用Lua的内置函数pairs：

    ```
    for key,value in pairs(user_1)
    do print(key .. value)
    end
    ```

  - 函数定义

    在Lua中，函数以function开头，以end结尾，funcName是函数名，中间部分是函数体：

    ```
    function funcName()
        …
    end
    contact函数将两个字符串拼接：
    function contact(str1, str2)
        return str1 .. str2
    end
    --"hello world"
    print(contact("hello ", "world"))
    ```

### Redis与Lua

- 在Redis中执行Lua脚本有两种方法：eval和evalsha。

  - eval

    ```
    eval 脚本内容 key个数 key列表 参数列表
    ```

    下面例子使用了key列表和参数列表来为Lua脚本提供更多的灵活性：

    ```
    127.0.0.1:6379> eval 'return "hello " .. KEYS[1] .. ARGV[1]' 1 redis world
    "hello redisworld"
    ```

    此时KEYS[1]="redis"，ARGV[1]="world"，所以最终的返回结果是"hello redisworld"。如果Lua脚本较长，还可以使用redis-cli--eval直接执行文件。

    eval命令和--eval参数本质是一样的，客户端如果想执行Lua脚本，首先在客户端编写好Lua脚本代码，然后把脚本作为字符串发送给服务端，服务端会将执行结果返回给客户端

    ![image-20251123103055628](images/image-20251123103055628.png)

  - evalsha

    首先要将Lua脚本加载到Redis服务端，得到该脚本的SHA1校验和，evalsha命令使用SHA1作为参数可以直接执行对应Lua脚本，避免每次发送Lua脚本的开销。这样客户端就不需要每次执行脚本内容，而脚本也会常驻在服务端，脚本功能得到了复用。

    ![image-20251123103553878](images/image-20251123103553878.png)

    加载脚本：script load命令可以将脚本内容加载到Redis内存中，例如下面将lua_get.lua加载到Redis中，得到SHA1为："7413dc2440db1fea7c0a0bde841fa68eefaf149c"

    ```
    # redis-cli script load "$(cat lua_get.lua)"
    "7413dc2440db1fea7c0a0bde841fa68eefaf149c"
    ```

    执行脚本：evalsha的使用方法如下，参数使用SHA1值，执行逻辑和eval一致。

    ```
    evalsha 脚本SHA1值 key个数 key列表 参数列表
    ```

    所以只需要执行如下操作，就可以调用lua_get.lua脚本：

    ```
    127.0.0.1:6379> evalsha 7413dc2440db1fea7c0a0bde841fa68eefaf149c 1 redis world
    "hello redisworld"
    ```

- Lua可以使用redis.call函数实现对Redis的访问，例如下面代码是Lua使用redis.call调用了Redis的set和get操作：

  ```
  redis.call("set", "hello", "world")
  redis.call("get", "hello")
  ```

  放在Redis的执行效果如下：

  ```
  127.0.0.1:6379> eval 'return redis.call("get", KEYS[1])' 1 hello
  "world"
  ```

  Redis3.2提供了Lua Script Debugger功能用来调试复杂的Lua脚本，具体可以参考：http://redis.io/topics/ldb。

### 案例

Lua脚本功能为Redis开发和运维人员带来如下三个好处：

- Lua脚本在Redis中是原子执行的，执行过程中间不会插入其他命令。
- Lua脚本可以帮助开发和运维人员创造出自己定制的命令，并可以将这些命令常驻在Redis内存中，实现复用的效果。
- Lua脚本可以将多条命令一次性打包，有效地减少网络开销。

下面以一个例子说明Lua脚本的使用，当前列表记录着热门用户的id，假设这个列表有5个元素，如下所示：

```
127.0.0.1:6379> lrange hot:user:list 0 -1
1) "user:1:ratio"
2) "user:8:ratio"
3) "user:3:ratio"
4) "user:99:ratio"
5) "user:72:ratio"
```

user：{id}：ratio代表用户的热度，它本身又是一个字符串类型的键：

```
127.0.0.1:6379> mget user:1:ratio user:8:ratio user:3:ratio user:99:ratio
    user:72:ratio
1) "986"
2) "762"
3) "556"
4) "400"
5) "101"
```

现要求将列表内所有的键对应热度做加1操作，并且保证是原子执行，此功能可以利用Lua脚本来实现。

1)将列表中所有元素取出，赋值给mylist：

```
local mylist = redis.call("lrange", KEYS[1], 0, -1)
```

2)定义局部变量count=0，这个count就是最后incr的总次数：

```
local count = 0
```

3)遍历mylist中所有元素，每次做完count自增，最后返回count：

```
for index,key in ipairs(mylist)
do
    redis.call("incr",key)
    count = count + 1
end
return count
```

将上述脚本写入lrange_and_mincr.lua文件中，并执行如下操作，返回结果为5。

```
redis-cli --eval lrange_and_mincr.lua  hot:user:list
(integer) 5
```

执行后所有用户的热度自增1：

```
127.0.0.1:6379> mget user:1:ratio user:8:ratio user:3:ratio user:99:ratio
    user:72:ratio
1) "987"
2) "763"
3) "557"
4) "401"
5) "102"
```



## Redis Shell

### redis-cli详解

- -r(repeat)选项代表将命令执行多次，例如下面操作将会执行三次ping命令：

  ```
  redis-cli -r 3 ping
  PONG
  PONG
  PONG
  ```

- -i(interval)选项代表每隔几秒执行一次命令，但是-i选项必须和-r选项一起使用，下面的操作会每隔1秒执行一次ping命令，一共执行5次：

  ```
  $ redis-cli -r 5 -i 1 ping
  PONG
  PONG
  PONG
  PONG
  PONG
  ```

  注意-i的单位是秒，不支持毫秒为单位，但是如果想以每隔10毫秒执行一次，可以用-i0.01，例如：

  ```
  $ redis-cli -r 5 -i 0.01 ping
  PONG
  PONG
  PONG
  PONG
  PONG
  ```

- -c(cluster)选项是连接Redis Cluster节点时需要使用的，-c选项可以防止moved和ask异常，有关Redis Cluster

- 如果Redis配置了密码，可以用-a(auth)选项，有了这个选项就不需要手动输入auth命令。

- --scan选项和--pattern选项用于扫描指定模式的键，相当于使用scan命令。

- 

### redis-server详解

redis-server除了启动Redis外，还有一个--test-memory选项。**redis-server--test-memory可以用来检测当前操作系统能否稳定地分配指定容量的内存给Redis**，通过这种检测可以有效避免因为内存问题造成Redis崩溃，例如下面操作检测当前操作系统能否提供1G的内存给Redis：

```
redis-server --test-memory 1024
```

整个内存检测的时间比较长。当输出passed this test时说明内存检测完毕，最后会提示--test-memory只是简单检测，如果有质疑可以使用更加专业的内存检测工具：

```
Please keep the test running several minutes per GB of memory.
Also check http:// www.memtest86.com/ and http:// pyropus.ca/software/memtester/
…………….忽略检测细节…………….
Your memory passed this test.
Please if you are still in doubt use the following two tools:
1) memtest86: http:// www.memtest86.com/
2) memtester: http:// pyropus.ca/software/memtester/
```

通常无需每次开启Redis实例时都执行--test-memory选项，该功能更偏向于调试和测试，例如，想快速占满机器内存做一些极端条件的测试，这个功能是一个不错的选择。

### redis-benchmark详解

redis-benchmark可以为Redis做基准性能测试

- -c(clients)选项代表客户端的并发数量（默认是50）。

- -n(num)选项代表客户端请求总量（默认是100000）。例如redis-benchmark -c100 -n20000代表100各个客户端同时请求Redis，一共执行20000次。redis-benchmark会对各类数据结构的命令进行测试，并给出性能指标：

  ```
  ====== GET ======
      20000 requests completed in 0.27 seconds
      100 parallel clients
      3 bytes payload
      keep alive: 1
  99.11% <= 1 milliseconds
  100.00% <= 1 milliseconds    
  73529.41 requests per second
  ```

  例如上面一共执行了20000次get操作，在0.27秒完成，每个请求数据量是3个字节，99.11%的命令执行时间小于1毫秒，Redis每秒可以处理73529.41次get请求。

- -q选项仅仅显示redis-benchmark的requests per second信息，例如：

  ```
  $redis-benchmark -c 100 -n 20000 -q
  PING_INLINE: 74349.45 requests per second
  PING_BULK: 68728.52 requests per second
  SET: 71174.38 requests per second…
  LRANGE_500 (first 450 elements): 11299.44 requests per second
  LRANGE_600 (first 600 elements): 9319.67 requests per second
  MSET (10 keys): 70671.38 requests per second
  ```

- 在一个空的Redis上执行了redis-benchmark会发现只有3个键：

  ```
  127.0.0.1:6379> dbsize
  (integer) 3
  127.0.0.1:6379> keys *
  1) "counter:__rand_int__"
  2) "mylist"
  3) "key:__rand_int__"
  ```

  如果想向Redis插入更多的键，可以执行使用-r(random)选项，可以向Redis插入更多随机的键。

  ```
  $redis-benchmark -c 100 -n 20000 -r 10000
  ```

  -r选项会在key、counter键上加一个12位的后缀，-r10000代表只对后四位做随机处理（-r不是随机数的个数）。例如上面操作后，key的数量和结果结构如下：

  ```
  127.0.0.1:6379> dbsize
  (integer) 18641
  127.0.0.1:6379> scan 0
  1) "14336"
  2)  1) "key:000000004580"
      2) "key:000000004519"
  …
     10) "key:000000002113"
  ```

- -P选项代表每个请求pipeline的数据量（默认为1）。

- -k选项代表客户端是否使用keepalive，1为使用，0为不使用，默认值为1。

- -t选项可以对指定命令进行基准测试。

  ```
  redis-benchmark -t get,set -q
  SET: 98619.32 requests per second
  GET: 97560.98 requests per second
  ```

- --csv选项会将结果按照csv格式输出，便于后续处理，如导出到Excel等。

  ```
  redis-benchmark -t get,set --csv
  "SET","81300.81"
  "GET","79051.38"
  ```

  

## Bitmaps（位图）

### 特点

- 本质是**String字符串类型**，但可以对字符串的位进行操作
- 每个位只能存储 0 或 1，相当于一个**以位为单位的数组**，数组的下标在Bitmaps中叫做偏移量。
- 非常节省内存，1 个字节 = 8 位

### 常用命令

- `setbit key offset value`：设置值

  设置键的第offset个位的值（从0算起），假设现在有20个用户，userid=0，5，11，15，19的用户对网站进行了访问

  ![image-20251123105406186](images/image-20251123105406186.png)

  具体操作过程如下，unique：users：2016-04-05代表2016-04-05这天的独立访问用户的Bitmaps：

  ```
  127.0.0.1:6379> setbit unique:users:2016-04-05 0 1
  (integer) 0
  127.0.0.1:6379> setbit unique:users:2016-04-05 5 1
  (integer) 0
  127.0.0.1:6379> setbit unique:users:2016-04-05 11 1
  (integer) 0
  127.0.0.1:6379> setbit unique:users:2016-04-05 15 1
  (integer) 0
  127.0.0.1:6379> setbit unique:users:2016-04-05 19 1
  (integer) 0
  ```

- `gitbit key offset`：获取值

  - 获取键的第offset位的值（从0开始算），下面操作获取id=8的用户是否在2016-04-05这天访问过，返回0说明没有访问过：

    ```
    127.0.0.1:6379> getbit unique:users:2016-04-05 8
    (integer) 0
    ```

- `bitcount [start][end]`：获取Bitmaps指定范围值为1的个数

  - [start]和[end]代表起始和结束字节数，下面操作计算用户id在第1个字节到第3个字节之间的独立访问用户数，对应的用户id是11，15，19。

    ```
    127.0.0.1:6379> bitcount unique:users:2016-04-05 1 3
    (integer) 3
    ```

- `bitop op destkey key[key...]`：Bitmaps间的运算

  - bitop是一个复合操作，它可以做多个Bitmaps的and（交集）、or（并集）、not（非）、xor（异或）操作并将结果保存在destkey中。假设2016-04-04访问网站的userid=1，2，5，9

    ![image-20251123110115334](images/image-20251123110115334.png)

  - 下面操作计算出2016-04-04和2016-04-03两天都访问过网站的用户数量

    ```
    127.0.0.1:6379> bitop and unique:users:and:2016-04-04_03 unique: users:2016-04-03
        unique:users:2016-04-03
    (integer) 2
    127.0.0.1:6379> bitcount unique:users:and:2016-04-04_03
    (integer) 2
    ```

    如果想算出2016-04-04和2016-04-03任意一天都访问过网站的用户数量（例如月活跃就是类似这种），可以使用or求并集，具体命令如下：

    ```
    127.0.0.1:6379> bitop or unique:users:or:2016-04-04_03 unique:
        users:2016-04-03 unique:users:2016-04-03
    (integer) 2
    127.0.0.1:6379> bitcount unique:users:or:2016-04-04_03
    (integer) 6
    ```

    ![image-20251123110526057](images/image-20251123110526057.png)

- `bitpos key targetBit [start] [end]`：计算Bitmaps中第一个值为targetBit的偏移量

  - bitops有两个选项[start]和[end]，分别代表起始字节和结束字节，例如计算第0个字节到第1个字节之间，第一个值为0的偏移量

    ```
    127.0.0.1:6379> bitpos unique:users:2016-04-04 0 0 1
    (integer) 0
    ```

    

## Bitfield（位域）

### 特点

- 位域是对 Bitmaps 的扩展，可以操作多个位字段
- 允许对字符串中指定的位范围进行操作
- 可以设置和获取多字节的整数值

### 常用命令

```
BITFIELD key [GET type offset] [SET type offset value] ...
```

## HyperLogLog（基数统计）

### 特点

- HyperLogLog并不是一种新的数据结构（实际类型为字符串类型），而是一种**基数算法**，通过HyperLogLog可以利用极小的内存空间完成独立总数的统计，数据集可以是IP、Email、ID等。
- 用于估算集合中不重复元素的数量（基数）
- 使用固定内存空间（约 12KB），非常节省内存
- 估算结果有约 1% 的误差
- HyperLogLog的算法是由Philippe Flajolet(https://en.wikipedia.org/wiki/Philippe_Flajolet)在The analysis of a near-optimal cardinality estimation algorithm这篇论文中提出，读者如果有兴趣可以自行阅读。

### 常用命令

- `pfadd key element [element ...]`：添加

  - pfadd用于向HyperLogLog添加元素，如果添加成功返回1：

    ```
    127.0.0.1:6379> pfadd 2016_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
    (integer) 1
    ```

- `pfcount key [key ...]`：计算独立用户数

  - pfcount用于计算一个或多个HyperLogLog的独立总数，例如2016_03_06：unique：ids的独立总数为4：

    ```
    127.0.0.1:6379> pfcount 2016_03_06:unique:ids
    (integer) 4
    ```

  - 当前这个例子内存节省的效果还不是很明显，下面使用脚本向HyperLogLog插入100万个id，插入前记录一下info memory：

    ```
    127.0.0.1:6379> info memory
    # Memory
    used_memory:835144
    used_memory_human:815.57K
    …向2016_05_01:unique:ids插入100万个用户，每次插入1000条：
    elements=""
    key="2016_05_01:unique:ids"
    for i in `seq 1 1000000`
    do
        elements="${elements} uuid-"${i}
        if [[ $((i%1000))  == 0 ]];
        then
            redis-cli pfadd ${key} ${elements}
            elements=""
        fi
    done
    ```

    当上述代码执行完成后，可以看到内存只增加了15K左右：

    ```
    127.0.0.1:6379> info memory
    # Memory
    used_memory:850616
    used_memory_human:830.68K
    ```

    但是，同时可以看到pfcount的执行结果并不是100万：

    ```
    127.0.0.1:6379> pfcount 2016_05_01:unique:ids
    (integer) 1009838
    ```

    可以对100万个uuid使用集合类型进行测试，代码如下：

    ```
    elements=""
    key="2016_05_01:unique:ids:set"
    for i in `seq 1 1000000`
    do
        elements="${elements} "${i}
        if [[ $((i%1000))  == 0 ]];
        then
            redis-cli sadd ${key} ${elements}
            elements="" 
        fi
    done
    ```

    可以看到内存使用了84MB：

    ```
    127.0.0.1:6379> info memory # Memory used_memory:88702680 used_memory_human:84.59M
    ```

    但独立用户数为100万：

    ```
    127.0.0.1:6379> scard 2016_05_01:unique:ids:set
    (integer) 1000000
    ```

    ![image-20251123112139829](images/image-20251123112139829.png)

    可以看到，HyperLogLog内存占用量小得惊人，但是用如此小空间来估算如此巨大的数据，必然不是100%的正确，其中一定存在误差率。Redis官方给出的数字是0.81%的失误率

- `pfmerge destkey sourcekey [sourcekey …]`：合并

  - pfmerge可以求出多个HyperLogLog的并集并赋值给destkey，例如要计算2016年3月5日和3月6日的访问独立用户数，可以按照如下方式来执行，可以看到最终独立用户数是7：

    ```
    127.0.0.1:6379> pfadd 2016_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
    (integer) 1
    127.0.0.1:6379> pfadd 2016_03_05:unique:ids "uuid-4" "uuid-5" "uuid-6" "uuid-7"
    (integer) 1
    127.0.0.1:6379> pfmerge 2016_03_05_06:unique:ids 2016_03_05:unique:ids 
        2016_03_06:unique:ids
    OK
    127.0.0.1:6379> pfcount 2016_03_05_06:unique:ids
    (integer) 7
    ```

    HyperLogLog内存占用量非常小，但是存在错误率，开发者在进行数据结构选型时只需要确认如下两条即可：·只为了计算独立总数，不需要获取单条数据。·可以容忍一定误差率，毕竟HyperLogLog在内存的占用量上有很大的优势。

## 发布订阅

Redis提供了基于“发布/订阅”模式的消息机制，此种模式下，消息发布者和订阅者不进行直接通信，**发布者客户端向指定的频道(channel)发布消息，订阅该频道的每个客户端都可以收到该消息**

![image-20251123113222142](images/image-20251123113222142.png)

### 命令

- `publish channel message`：发布消息

  - 下面操作会向channel：sports频道发布一条消息“Tim won the championship”，返回结果为订阅者个数，因为此时没有订阅，所以返回结果为0：

    ```
    127.0.0.1:6379> publish channel:sports "Tim won the championship"
    (integer) 0
    ```

- `subscribe channel [channel ...]`：订阅消息

  - 订阅者可以订阅一个或多个频道，下面操作为当前客户端订阅了channel：sports频道：

    ```
    127.0.0.1:6379> subscribe channel:sports
    Reading messages… (press Ctrl-C to quit)
    1) "subscribe"
    2) "channel:sports"
    3) (integer) 1
    ```

    此时另一个客户端发布一条消息

    ```
    127.0.0.1:6379> publish channel:sports "James lost the championship"
    (integer) 1
    ```

    当前订阅者客户端会收到如下消息：

    ```
    127.0.0.1:6379> subscribe channel:sports
    Reading messages… (press Ctrl-C to quit)
    …
    1) "message"
    2) "channel:sports"
    3) "James lost the championship"
    ```

    订阅的客户端每次可以收到一个3个参数的消息：消息的种类，始发频道的名称，实际的消息内容

    如果有多个客户端同时订阅了channel：sports，整个过程如图

    ![image-20251123114318989](images/image-20251123114318989.png)

  - 有关订阅命令有两点需要注意：

    - 客户端在执行订阅命令之后进入了订阅状态，只能接收subscribe、psubscribe、unsubscribe、punsubscribe的四个命令。
    - 新开启的订阅客户端，无法收到该频道之前的消息，因为Redis不会对发布的消息进行持久化。

- `unsubscribe [channel [channel ...]]`：取消订阅

  - 客户端可以通过unsubscribe命令取消对指定频道的订阅，取消成功后，不会再收到该频道的发布消息：

    ```
    127.0.0.1:6379> unsubscribe channel:sports
    1) "unsubscribe"
    2) "channel:sports"
    3) (integer) 0
    ```

- 按照模式订阅和取消订阅

  ```
  psubscribe pattern [pattern…]
  punsubscribe [pattern [pattern …]]
  ```

  - 除了subcribe和unsubscribe命令，Redis命令还支持glob风格的订阅命令psubscribe和取消订阅命令punsubscribe，例如下面操作订阅以it开头的所有频道：

    ```
    127.0.0.1:6379> psubscribe it*
    Reading messages… (press Ctrl-C to quit)
    1) "psubscribe"
    2) "it*"
    3) (integer) 1
    ```

- 查询订阅

  - `pubsub channels [pattern]`：查看活跃的频道

    - 所谓活跃的频道是指当前频道至少有一个订阅者，其中[pattern]是可以指定具体的模式：

      ```
      127.0.0.1:6379> pubsub channels
      1) "channel:sports"
      2) "channel:it"
      3) "channel:travel"
      127.0.0.1:6379> pubsub channels channel:*r*
      1) "channel:sports"
      2) "channel:travel"
      ```

  - `pubsub numsub [channel ...]`：查看频道订阅数

    - 当前channel：sports频道的订阅数为2：

      ```
      127.0.0.1:6379> pubsub numsub channel:sports
      1) "channel:sports"
      2) (integer) 2
      ```

  - `pubsub numpat`：查看模式订阅

    当前只有一个客户端通过模式来订阅：

    ```
    127.0.0.1:6379> pubsub numpat
    (integer) 1
    ```




## Stream（流）

### 特点

- Redis 5.0 新增的数据类型
- 用于实现消息队列，支持多消费者组
- 消息持久化存储，可以回溯消费
- 类似 Kafka 的消息队列功能

### 常用命令

```
XADD key * field value [field value ...]  # 添加消息
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key1 key2 ...  # 读取消息
XGROUP CREATE groupname consumername [IDLE ms] [CLAIM]  # 创建消费者组
XREADGROUP GROUP groupname consumername [COUNT count] [BLOCK milliseconds] STREAMS key1 key2 ...  # 消费消息
```

## Geospatial（地理空间）

### 特点

- 用于存储和处理地理位置相关的数据
- 支持经纬度坐标，可以计算距离、范围查询
- 使用 ZSET 实现，但有专门的地理空间命令

### 常用命令

- 增加地理位置信息

  ```
  geoadd key longitude latitude member [longitude latitude member …]
  ```

  longitude、latitude、member分别是该地理位置的经度、纬度、成员

  - cities：locations是上面5个城市地理位置信息的集合，现向其添加北京的地理位置信息：

    ```
    127.0.0.1:6379> geoadd cities:locations 116.28 39.55 beijing
    (integer) 1
    ```

    返回结果代表添加成功的个数，如果cities：locations没有包含beijing，那么返回结果为1，如果已经存在则返回0

    如果需要更新地理位置信息，仍然可以使用geoadd命令，虽然返回结果为0。geoadd命令可以同时添加多个地理位置信息：

    ```
    127.0.0.1:6379> geoadd cities:locations 117.12 39.08 tianjin 114.29 38.02
        shijiazhuang 118.01 39.38 tangshan 115.29 38.51 baoding
    (integer) 4
    ```

- `geopos key member [member …]`：获取地理位置信息

  - 下面操作会获取天津的经维度：

    ```
    127.0.0.1:6379> geopos cities:locations tianjin
    1) 1) "117.12000042200088501"
       2) "39.0800000535766543"
    ```

- `geodist key member1 member2 [unit]`：获取两个地理位置的距离

  - 其中unit代表返回结果的单位，包含以下四种：

    - m(meters)代表米。
    - km(kilometers)代表公里。
    - mi(miles)代表英里。·ft(feet)代表尺。

  - 下面操作用于计算天津到北京的距离，并以公里为单位：

    ```
    127.0.0.1:6379> geodist cities:locations tianjin beijing km
    "89.2061"
    ```

- 获取指定位置范围内的地理信息位置集合

  ```
  georadius key longitude latitude radiusm|km|ft|mi [withcoord] [withdist]
      [withhash] [COUNT count] [asc|desc] [store key] [storedist key]
  georadiusbymember key member     radiusm|km|ft|mi [withcoord] [withdist] 
      [withhash] [COUNT count] [asc|desc] [store key] [storedist key]
  ```

  - georadius和georadiusbymember两个命令的作用是一样的，都是以一个地理位置为中心算出指定半径内的其他地理信息位置，不同的是georadius命令的中心位置给出了具体的经纬度，georadiusbymember只需给出成员即可。其中radiusm|km|ft|mi是必需参数，指定了半径（带单位），这两个命令有很多可选参数，如下所示：
    - withcoord：返回结果中包含经纬度。
    - withdist：返回结果中包含离中心节点位置的距离。
    - withhash：返回结果中包含geohash，有关geohash后面介绍。
    - COUNT count：指定返回结果的数量。
    - asc|desc：返回结果按照离中心节点的距离做升序或者降序。
    - store key：将返回结果的地理位置信息保存到指定键。·storedist key：将返回结果离中心节点的距离保存到指定键。

  - 下面操作计算五座城市中，距离北京150公里以内的城市：

    ```
    127.0.0.1:6379> georadiusbymember cities:locations beijing 150 km
    1) "beijing"
    2) "tianjin"
    3) "tangshan"
    4) "baoding"
    ```

- `geohash key member [member …]`：获取geohash

  - Redis使用**geohash将二维经纬度转换为一维字符串**，下面操作会返回beijing的geohash值。

    ```
    127.0.0.1:6379> geohash cities:locations beijing
    1) "wx4ww02w070"
    ```

  - geohash有如下特点：

    - GEO的数据类型为zset，Redis将所有地理位置信息的geohash存放在zset中。

      ```
      127.0.0.1:6379> type cities:locations
      zset
      ```

    - 字符串越长，表示的位置更精确，表3-8给出了字符串长度对应的精度，例如geohash长度为9时，精度在2米左右。

    - 两个字符串越相似，它们之间的距离越近，Redis利用字符串前缀匹配算法实现相关的命令。

    - geohash编码和经纬度是可以相互转换的。Redis正是使用有序集合并结合geohash的特性实现了GEO的若干命令。

- `zrem key member`：删除地理位置信息

# 3.Redis的Java客户端

## 客服端通信协议

几乎所有的主流编程语言都有Redis的客户端(http://redis.io/clients)，不考虑Redis非常流行的原因，如果站在技术的角度看原因还有两个：

第一，客户端与服务端之间的通信协议是在TCP协议之上构建的。

第二，Redis制定了RESP（REdis Serialization Protocol，Redis序列化协议）实现客户端与服务端的正常交互，这种协议简单高效，既能够被机器解析，又容易被人类识别。例如客户端发送一条set hello world命令给服务端，按照RESP的标准，客户端需要将其封装为如下格式（每行用\r\n分隔）：

```
*3
$3
SET
$5
hello
$5
world
```

这样Redis服务端能够按照RESP将其解析为set hello world命令，执行后回复的格式如下：

```
+OK
```

可以看到除了命令(set hello world)和返回结果(OK)本身还包含了一些特殊字符以及数字，下面将对这些格式进行说明。

- 发送命令格式

  - RESP的规定一条命令的格式如下，CRLF代表"\r\n"。

    ```
    *<参数数量> CRLF
    $<参数1的字节数量> CRLF
    <参数1> CRLF
    …
    $<参数N的字节数量> CRLF
    <参数N> CRLF
    ```

    依然以set hell world这条命令进行说明。参数数量为3个，因此第一行为：

    ```
    *3       参数数量为3个，因此第一行为
    $3       参数字节数分别是355，因此后面几行为：
    SET 
    $5
    hello
    $5
    world
    ```

    有一点要注意的是，上面只是格式化显示的结果，实际传输格式为如下代码

    ```
    *3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$5\r\nworld\r\n
    ```

- 返回结果格式

  - Redis的返回结果类型分为以下五种，如图4-2所示：

    - 状态回复：在RESP中第一个字节为"+"。

    - 错误回复：在RESP中第一个字节为"-"。

    - 整数回复：在RESP中第一个字节为"："。

    - 字符串回复：在RESP中第一个字节为"$"。

    - 多条字符串回复：在RESP中第一个字节为"*"。

      ![image-20251123122253446](images/image-20251123122253446.png)

      ![image-20251123122322295](images/image-20251123122322295.png)

- redis-cli只能看到最终的执行结果，那是因为redis-cli本身就是按照RESP进行结果解析的，所以看不到中间结果，redis-cli.c源码对命令结果的解析结构如下：

  ```
  static sds cliFormatReplyTTY(redisReply *r, char *prefix) {
      sds out = sdsempty();
      switch (r->type) {
      case REDIS_REPLY_ERROR:
  // 处理错误回复
      case REDIS_REPLY_STATUS:
  // 处理状态回复
      case REDIS_REPLY_INTEGER:
  // 处理整数回复
      case REDIS_REPLY_STRING:
  // 处理字符串回复
      case REDIS_REPLY_NIL:
  // 处理空
      case REDIS_REPLY_ARRAY:
  // 处理多条字符串回复
      return out;
  }
  ```

  例如执行set hello world，返回结果是OK，并不能看到加号：

  ```
  127.0.0.1:6379> set hello world
  OK
  ```

  为了看到Redis服务端返回的“真正”结果，可以使用nc命令、telnet命令、甚至写一个socket程序进行模拟。下面以nc命令进行演示，首先使用nc127.0.0.16379连接到Redis：

  ```
  nc 127.0.0.1 6379
  ```

  - 状态回复：set hello world的返回结果为+OK：

    ```
    set hello world
    +OK
    ```

  - 错误回复：由于sethx这条命令不存在，那么返回结果就是"-"号加上错误消息：

    ```
    sethx
    -ERR unknown command 'sethx'
    ```

  - 整数回复：当命令的执行结果是整数时，返回结果就是整数回复，例如incr、exists、del、dbsize返回结果都是整数，例如执行incr counter返回结果就是“：”加上整数：

    ```
    incr counter
    :1
    ```

  - 字符串回复：当命令的执行结果是字符串时，返回结果就是字符串回复。例如get、hget返回结果都是字符串，例如get hello的结果为“$5\r\nworld\r\n”：

    ```
    get hello
    $5
    world
    ```

  - 多条字符串回复：当命令的执行结果是多条字符串时，返回结果就是多条字符串回复。例如mget、hgetall、lrange等命令会返回多个结果，例如下面操作：

    - 首先使用mset设置多个键值对：

      ```
      mset java jedis python redis-py
      +OK
      ```

    - 然后执行mget命令返回多个结果，第一个*2代表返回结果的个数，后面的格式是和字符串回复一致的：

      ```
      mget java python
      *2
      $5
      jedis
      $8
      redis-py
      ```

  - 有一点需要注意，无论是字符串回复还是多条字符串回复，如果有nil值，那么会返回$-1。例如，对一个不存在的键执行get操作，返回结果为：

    ```
    get not_exist_key
    $-1
    ```

    如果批量操作中包含一条为nil值的结果，那么返回结果如下：

    ```
    mget hello not_exist_key java
    *3
    $5
    world
    $-1
    $5
    jedis
    ```

    

  

  

  



在Redis官网中提供了各种语言的客户端，地址：https://redis.io/docs/clients/

![](assets/9f68ivq.png)

其中Java客户端也包含很多：

![image-20220609102817435](assets/image-20220609102817435-165735883948534.png)

标记为*的就是推荐使用的java客户端，包括：

- Jedis和Lettuce：这两个主要是提供了Redis命令对应的API，方便我们操作Redis，而SpringDataRedis又对这两种做了抽象和封装，因此我们后期会直接以SpringDataRedis来学习。
- Redisson：是在Redis基础上实现了分布式的可伸缩的java数据结构，例如Map、Queue等，而且支持跨进程的同步机制：Lock、Semaphore等待，比较适合用来实现特殊的功能需求。



## 3.1.Jedis客户端

Jedis的官网地址： https://github.com/redis/jedis

### 3.1.1.快速入门

我们先来个快速入门：

1）引入依赖：

```xml
<!--jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.7.0</version>
</dependency>
<!--单元测试-->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```



2）建立连接

新建一个单元测试类，内容如下：

```java
private Jedis jedis;

@BeforeEach
void setUp() {
    // 1.建立连接
    // jedis = new Jedis("192.168.150.101", 6379);
    jedis = JedisConnectionFactory.getJedis();
    // 2.设置密码
    jedis.auth("123321");
    // 3.选择库
    jedis.select(0);
}
```



3）测试：

```java
@Test
void testString() {
    // 存入数据
    String result = jedis.set("name", "虎哥");
    System.out.println("result = " + result);
    // 获取数据
    String name = jedis.get("name");
    System.out.println("name = " + name);
}

@Test
void testHash() {
    // 插入hash数据
    jedis.hset("user:1", "name", "Jack");
    jedis.hset("user:1", "age", "21");

    // 获取
    Map<String, String> map = jedis.hgetAll("user:1");
    System.out.println(map);
}
```



4）释放资源

```java
@AfterEach
void tearDown() {
    if (jedis != null) {
        jedis.close();
    }
}
```



### 3.1.2.连接池

Jedis的直连方式，所谓直连是指Jedis每次都会新建TCP连接，使用后再断开连接，对于频繁访问Redis的场景显然不是高效的使用方式

![image-20251123124541648](images/image-20251123124541648.png)

因此生产环境中一般使用连接池的方式对Jedis连接进行管理，如图4-4所示，所有Jedis对象预先放在池子中(JedisPool)，每次要连接Redis，只需要在池子中借，用完了在归还给池子。

![image-20251123124609138](images/image-20251123124609138.png)



客户端连接Redis使用的是TCP协议，直连的方式每次需要建立TCP连接，而连接池的方式是可以预先初始化好Jedis连接，所以每次只需要从Jedis连接池借用即可，而借用和归还操作是在本地进行的，只有少量的并发同步开销，远远小于新建TCP连接的开销。另外直连的方式无法限制Jedis对象的个数，在极端情况下可能会造成连接泄露，而连接池的形式可以有效的保护和控制资源的使用。但是直连的方式也并不是一无是处

![image-20251123124508740](images/image-20251123124508740.png)

Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此我们推荐大家使用Jedis连接池代替Jedis的直连方式。

Jedis提供了JedisPool这个类作为对Jedis的连接池，同时使用了Apache的通用对象池工具common-pool作为资源的管理工具，下面是使用JedisPool操作Redis的代码示例：

1)Jedis连接池（通常JedisPool是单例的）：

```
// common-pool连接池配置，这里使用默认配置，后面小节会介绍具体配置说明
GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
// 初始化Jedis连接池
JedisPool jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379);
```

2)获取Jedis对象不再是直接生成一个Jedis对象进行直连，而是从连接池直接获取，代码如下：

```
Jedis jedis = null;
try {
// 1. 从连接池获取jedis对象
    jedis = jedisPool.getResource();
// 2. 执行操作
    jedis.get("hello");
} catch (Exception e) {
    logger.error(e.getMessage(),e);
} finally {
    if (jedis != null) {
// 如果使用JedisPool，close操作不是关闭连接，代表归还连接池
        jedis.close();
    }
}
```

这里可以看到在finally中依然是jedis.close()操作，为什么会把连接关闭呢，这不和连接池的原则违背了吗？但实际上Jedis的close()实现方式如下：

```
public void close() {
// 使用Jedis连接池
    if (dataSource != null) {
        if (client.isBroken()) {
            this.dataSource.returnBrokenResource(this);
        } else {
            this.dataSource.returnResource(this);
        }
// 直连
    } else {
        client.close();
    }
}
```

参数说明：

- dataSource！=null代表使用的是连接池，所以jedis.close()代表归还连接给连接池，而且Jedis会判断当前连接是否已经断开。

- dataSource=null代表直连，jedis.close()代表关闭连接。前面GenericObjectPoolConfig使用的是默认配置，实际它提供有很多参数，例如池子中最大连接数、最大空闲连接数、最小空闲连接数、连接活性检测，等等，例如下面代码：

  ```
  GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
  // 设置最大连接数为默认值的5倍
  poolConfig.setMaxTotal(GenericObjectPoolConfig.DEFAULT_MAX_TOTAL * 5);
  // 设置最大空闲连接数为默认值的3倍
  poolConfig.setMaxIdle(GenericObjectPoolConfig.DEFAULT_MAX_IDLE * 3);
  // 设置最小空闲连接数为默认值的2倍
  poolConfig.setMinIdle(GenericObjectPoolConfig.DEFAULT_MIN_IDLE * 2);
  // 设置开启jmx功能
  poolConfig.setJmxEnabled(true);
  // 设置连接池没有连接后客户端的最大等待时间(单位为毫秒)
  poolConfig.setMaxWaitMillis(3000);
  ```

  ![image-20251123125016873](images/image-20251123125016873.png)

### Redis中Pipeline的使用方法

Jedis支持Pipeline特性，我们知道Redis提供了mget、mset方法，但是并没有提供mdel方法，如果想实现这个功能，可以借助Pipeline来模拟批量删除，虽然不会像mget和mset那样是一个原子命令，但是在绝大数场景下可以使用。下面代码是mdel删除的实现过程。

注意这里为了节省篇幅，没有写try catch finally，没有关闭jedis。

```
public void mdel(List<String> keys) {
    Jedis jedis = new Jedis("127.0.0.1");
    // 1)生成pipeline对象
    Pipeline pipeline = jedis.pipelined();
    // 2)pipeline执行命令，注意此时命令并未真正执行
    for (String key : keys) {
        pipeline.del(key);
    }
    // 3)执行命令
    pipeline.sync();
}
```

说明如下：

- 利用jedis对象生成一个pipeline对象，直接可以调用jedis.pipelined()。

- 将del命令封装到pipeline中，可以调用pipeline.del(String key)，这个方法和jedis.del(String key)的写法是完全一致的，只不过此时不会真正的执行命令。

- 使用pipeline.sync()完成此次pipeline对象的调用。

- 除了pipeline.sync()，还可以使用pipeline.syncAndReturnAll()将pipeline的命令进行返回，例如下面代码将set和incr做了一次pipeline操作，并顺序打印了两个命令的结果：

  ```
  Jedis jedis = new Jedis("127.0.0.1");
  Pipeline pipeline = jedis.pipelined();
  pipeline.set("hello", "world");
  pipeline.incr("counter");
  List<Object> resultList = pipeline.syncAndReturnAll();
  for (Object object : resultList) {
      System.out.println(object);
  }
  ```

  输出结果为：

  ```
  OK
  1
  ```

### 客户端API

#### client list

client list命令能列出与Redis服务端相连的所有客户端连接信息，例如下面代码是在一个Redis实例上执行client list的结果：

```
127.0.0.1:6379> client list
id=254487 addr=10.2.xx.234:60240 fd=1311 name= age=8888581 idle=8888581 flags=N 
    db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
id=300210 addr=10.2.xx.215:61972 fd=3342 name= age=8054103 idle=8054103 flags=N 
    db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
id=5448879 addr=10.16.xx.105:51157 fd=233 name= age=411281 idle=331077 flags=N 
    db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ttl
id=2232080 addr=10.16.xx.55:32886 fd=946 name= age=603382 idle=331060 flags=N 
    db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
id=7125108 addr=10.10.xx.103:33403 fd=139 name= age=241 idle=1 flags=N db=0 
    sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=del
id=7125109 addr=10.10.xx.101:58658 fd=140 name= age=241 idle=1 flags=N db=0 
    sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=del
…
```

输出结果的每一行代表一个客户端的信息，可以看到每行包含了十几个属性，它们是每个客户端的一些执行状态，理解这些属性对于Redis的开发和运维人员非常有帮助。下面将选择几个重要的属性进行说明，其余通过表格的形式进行展示。

##### (1)标识：id、addr、fd、name

这四个属性属于客户端的标识：·

id：客户端连接的唯一标识，这个id是随着Redis的连接自增的，重启Redis后会重置为0。

addr：客户端连接的ip和端口。

fd：socket的文件描述符，与lsof命令结果中的fd是同一个，如果fd=-1代表当前客户端不是外部客户端，而是Redis内部的伪装客户端。

name：客户端的名字，后面的client setName和client getName两个命令会对其进行说明。

##### (2)输入缓冲区：qbuf、qbuf-free

Redis为每个客户端分配了输入缓冲区，它的作用是**将客户端发送的命令临时保存，同时Redis从会输入缓冲区拉取命令并执行**，输入缓冲区为客户端发送命令到Redis执行命令提供了缓冲功能

client list中**qbuf和qbuf-free分别代表这个缓冲区的总容量和剩余容量**，Redis没有提供相应的配置来规定每个缓冲区的大小，输入缓冲区会根据输入内容大小的不同动态调整，只是要求每个客户端缓冲区的大小不能超过1G，超过后客户端将被关闭。下面是Redis源码中对于输入缓冲区的硬编码：

![image-20251123210756865](images/image-20251123210756865.png)

输入缓冲使用不当会产生两个问题：

- 一旦某个客户端的输入缓冲区超过1G，客户端将会被关闭。

- 输入缓冲区不受maxmemory控制，假设一个Redis实例设置了maxmemory为4G，已经存储了2G数据，但是如果此时输入缓冲区使用了3G，已经超过maxmemory限制，可能会产生数据丢失、键值淘汰、OOM等情况

  ![image-20251123211320070](images/image-20251123211320070.png)

- 执行效果如下：

  ````
  127.0.0.1:6390> info memory
  # Memory
  used_memory_human:5.00G
  …
  maxmemory_human:4.00G
  …. 
  ````

  上面已经看到，输入缓冲区使用不当造成的危害非常大，那么造成输入缓冲区过大的原因有哪些？输入缓冲区过大主要是因为Redis的处理速度跟不上输入缓冲区的输入速度，并且每次进入输入缓冲区的命令包含了大量bigkey，从而造成了输入缓冲区过大的情况。还有一种情况就是Redis发生了阻塞，短期内不能处理命令，造成客户端输入的命令积压在了输入缓冲区，造成了输入缓冲区过大。

- 那么如何快速发现和监控呢？监控输入缓冲区异常的方法有两种：

  - 通过定期执行client list命令，收集qbuf和qbuf-free找到异常的连接记录并分析，最终找到可能出问题的客户端。

  - 通过info命令的info clients模块，找到最大的输入缓冲区，例如下面命令中的其中client_biggest_input_buf代表最大的输入缓冲区，例如可以设置超过10M就进行报警：

    ```
    127.0.0.1:6379> info clients
    # Clients
    connected_clients:1414
    client_longest_output_list:0
    client_biggest_input_buf:2097152
    blocked_clients:0
    ```

    ![image-20251123212513519](images/image-20251123212513519.png)

##### (3)输出缓冲区：obl、oll、omem

Redis为每个客户端分配了输出缓冲区，它的作用是**保存命令执行的结果返回给客户端，为Redis和客户端交互返回结果提供缓冲**，

![image-20251123213024910](images/image-20251123213024910.png)

与输入缓冲区不同的是，输出缓冲区的容量可以通过参数client-output-buffer-limit来进行设置，并且输出缓冲区做得更加细致，按照客户端的不同分为三种：普通客户端、发布订阅客户端、slave客户端

![image-20251123213128824](images/image-20251123213128824.png)

对应的配置规则是：

```
client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
```

- <class>：客户端类型，分为三种。a)normal：普通客户端；b)slave：slave客户端，用于复制；c)pubsub：发布订阅客户端。
- <hard limit>：如果客户端使用的输出缓冲区大于<hard limit>，客户端会被立即关闭。
- <soft limit>和<soft seconds>：如果客户端使用的输出缓冲区超过了<soft limit>并且持续了<soft limit>秒，客户端会被立即关闭。

Redis的默认配置是：

```
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

和输入缓冲区相同的是，输出缓冲区也不会受到maxmemory的限制，如果使用不当同样会造成maxmemory用满产生的数据丢失、键值淘汰、OOM等情况。

实际上输出缓冲区由两部分组成：固定缓冲区(16KB)和动态缓冲区，其中固定缓冲区返回比较小的执行结果，而动态缓冲区返回比较大的结果，例如大的字符串、hgetall、smembers命令的结果等，通过Redis源码中redis.h的redisClient结构体（Redis3.2版本变为Client）可以看到两个缓冲区的实现细节：

```
typedef struct redisClient {
// 动态缓冲区列表
    list *reply;
// 动态缓冲区列表的长度(对象个数)
    unsigned long reply_bytes;
// 固定缓冲区已经使用的字节数
    int bufpos;
// 字节数组作为固定缓冲区
    char buf[REDIS_REPLY_CHUNK_BYTES];
} redisClient;
```

固定缓冲区使用的是字节数组，动态缓冲区使用的是列表。当固定缓冲区存满后会将Redis新的返回结果存放在动态缓冲区的队列中，队列中的每个对象就是每个返回结果

![image-20251123213610479](images/image-20251123213610479.png)

client list中的obl代表固定缓冲区的长度，oll代表动态缓冲区列表的长度，omem代表使用的字节数。例如下面代表当前客户端的固定缓冲区的长度为0，动态缓冲区有4869个对象，两个部分共使用了133081288字节=126M内存：

```
id=7 addr=127.0.0.1:56358 fd=6 name= age=91 idle=0 flags=O db=0 sub=0 psub=0 multi=-1
    qbuf=0 qbuf-free=0 obl=0 oll=4869 omem=133081288 events=rw cmd=monitor
```

监控输出缓冲区的方法依然有两种：

- 通过定期执行client list命令，收集obl、oll、omem找到异常的连接记录并分析，最终找到可能出问题的客户端。

- 通过info命令的info clients模块，找到输出缓冲区列表最大对象数，例如：

  ```
  127.0.0.1:6379> info clients
  # Clients
  connected_clients:502  
  client_longest_output_list:4869  
  client_biggest_input_buf:0  
  blocked_clients:0  
  ```

- 其中，client_longest_output_list代表输出缓冲区列表最大对象数，这两种统计方法的优劣势和输入缓冲区是一样的，这里就不再赘述了。相比于输入缓冲区，输出缓冲区出现异常的概率相对会比较大，那么如何预防呢？方法如下：·进行上述监控，设置阀值，超过阀值及时处理。·限制普通客户端输出缓冲区的，把错误扼杀在摇篮中，例如可以进行如下设置：

  - 进行上述监控，设置阀值，超过阀值及时处理。

  - 限制普通客户端输出缓冲区的，把错误扼杀在摇篮中，例如可以进行如下设置：

    ```
    client-output-buffer-limit normal 20mb 10mb 120
    ```

  - 适当增大slave的输出缓冲区的，如果master节点写入较大，slave客户端的输出缓冲区可能会比较大，一旦slave客户端连接因为输出缓冲区溢出被kill，会造成复制重连。

  - 限制容易让输出缓冲区增大的命令，例如，高并发下的monitor命令就是一个危险的命令。

  - 及时监控内存，一旦发现内存抖动频繁，可能就是输出缓冲区过大。

##### (4)客户端的存活状态

client list中的age和idle分别代表当前客户端已经连接的时间和最近一次的空闲时间：

```
id=2232080 addr=10.16.xx.55:32886 fd=946 name= age=603382 idle=331060 flags=N db=0
    sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

例如上面这条记录代表当期客户端连接Redis的时间为603382秒，其中空闲了331060秒：

```
id=254487 addr=10.2.xx.234:60240 fd=1311 name= age=8888581 idle=8888581 flags=N db=0
    sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

注意

为了与redis-cli的客户端区分，本次测试客户端IP地址：10.7.40.98。

1)在执行代码之前，client list只有一个客户端，也就是当前的redis-cli，下面为了节省篇幅忽略掉这个客户端。

```
127.0.0.1:6379> client list
id=45 addr=127.0.0.1:55171 fd=6 name= age=2 idle=0 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

2)使用Jedis生成了一个新的连接，并执行get操作，可以看到IP地址为10.7.40.98的客户端，最后执行的命令是get，age和idle分别是1秒和0秒：

```
127.0.0.1:6379> client list
id=46 addr=10.7.40.98:62908 fd=7 name= age=1 idle=0 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

3)休息10秒，此时Jedis客户端并没有关闭，所以age和idle一直在递增：

```
127.0.0.1:6379> client list
id=46 addr=10.7.40.98:62908 fd=7 name= age=9 idle=9 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

4)执行新的操作ping，发现执行后age依然在增加，而idle从0计算，也就是不再闲置：

```
127.0.0.1:6379> client list
id=46 addr=10.7.40.98:62908 fd=7 name= age=11 idle=0 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
```

5)休息5秒，观察age和idle增加：

```
127.0.0.1:6379> client list
id=46 addr=10.7.40.98:62908 fd=7 name= age=15 idle=5 flags=N db=0 sub=0 psub=0 
multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping 
```

6)关闭Jedis，Jedis连接已经消失：

```
redis-cli client list | grep "10.7.40.98”为空
```

##### (5)客户端的限制maxclients和timeout

Redis提供了maxclients参数来限制最大客户端连接数，一旦连接数超过maxclients，新的连接将被拒绝。maxclients默认值是10000，可以通过info clients来查询当前Redis的连接数：

```
127.0.0.1:6379> info clients
# Clients
connected_clients:1414
…
```

可以通过config set maxclients对最大客户端连接数进行动态设置：

```
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "10000"
127.0.0.1:6379> config set maxclients 50
OK
127.0.0.1:6379> config get maxclients
1) "maxclients"
2) "50"
```

一般来说maxclients=10000在大部分场景下已经绝对够用，但是某些情况由于业务方使用不当（例如没有主动关闭连接）可能存在大量idle连接，无论是从网络连接的成本还是超过maxclients的后果来说都不是什么好事，因此Redis提供了timeout（单位为秒）参数来限制连接的最大空闲时间，一旦客户端连接的idle时间超过了timeout，连接将会被关闭，例如设置timeout为30秒：

```
#Redis默认的timeout是0，也就是不会检测客户端的空闲
127.0.0.1:6379> config set timeout 30
OK
```

下面继续使用Jedis进行模拟，整个代码和上面是一样的，只不过第2)步骤休息了31秒：

```
String key = "hello";
// 1) 生成jedis，并执行get操作
Jedis jedis = new Jedis("127.0.0.1", 6379);
System.out.println(jedis.get(key));
// 2) 休息31秒
TimeUnit.SECONDS.sleep(31);
// 3) 执行get操作
System.out.println(jedis.get(key));
// 4) 休息5秒
TimeUnit.SECONDS.sleep(5);
// 5) 关闭jedis连接
jedis.close();
```

执行上述代码可以发现在执行完第2)步之后，client list中已经没有了Jedis的连接，也就是说timeout已经生效，将超过30秒空闲的连接关闭掉：

```
127.0.0.1:6379> client list
id=16 addr=10.7.40.98:63892 fd=6 name= age=19 idle=19 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
# 超过timeout后，Jedis连接被关闭
redis-cli client list | grep “10.7.40.98”为空
```

同时可以看到，在Jedis代码中的第3)步抛出了异常，因为此时客户端已经被关闭，所以抛出的异常是JedisConnectionException，并且提示Unexpected end of stream：

```
stream：
world
Exception in thread "main" redis.clients.jedis.exceptions.JedisConnectionException: 
    Unexpected end of stream.
```

如果将Redis的loglevel设置成debug级别，可以看到如下日志，也就是客户端被Redis关闭的日志：

```
12885:M 26 Aug 08:46:40.085 - Closing idle client
```

Redis源码中redis.c文件中clientsCronHandleTimeout函数就是针对timeout参数进行检验的，只不过在源码中timeout被赋值给了server.maxidletime：

```
int clientsCronHandleTimeout(redisClient *c) {
// 当前时间
    time_t now = server.unixtime;
    // server.maxidletime就是参数timeout
    if (server.maxidletime &&
// 很多客户端验证，这里就不占用篇幅，最重要的验证是下面空闲时间超过了maxidletime就会
// 被关闭掉客户端
        (now - c->lastinteraction > server.maxidletime))
    {
        redisLog(REDIS_VERBOSE,"Closing idle client");
// 关闭客户端
        freeClient(c);
    }
}
```

Redis的默认配置给出的timeout=0，在这种情况下客户端基本不会出现上面的异常，这是基于对客户端开发的一种保护。例如很多开发人员在使用JedisPool时不会对连接池对象做空闲检测和验证，如果设置了timeout>0，可能就会出现上面的异常，对应用业务造成一定影响，但是如果Redis的客户端使用不当或者客户端本身的一些问题，造成没有及时释放客户端连接，可能会造成大量的idle连接占据着很多连接资源，一旦超过maxclients；后果也是不堪设想。所在在实际开发和运维中，需要将timeout设置成大于0，例如可以设置为300秒，同时在客户端使用上添加空闲检测和验证等等措施，例如JedisPool使用common-pool提供的三个属性：minEvictableIdleTimeMillis、testWhileIdle、timeBetweenEvictionRunsMillis，4.2节已经进行了说明，这里就不再赘述。

##### (6)客户端类型

client list中的flag是用于标识当前客户端的类型，例如flag=S代表当前客户端是slave客户端、flag=N代表当前是普通客户端，flag=O代表当前客户端正在执行monitor命令，表4-4列出了11种客户端类型。

![image-20251123215355297](images/image-20251123215355297.png)

##### (7)其他

![image-20251123215455283](images/image-20251123215455283.png)

![image-20251123215507976](images/image-20251123215507976.png)

#### client setName和client getName

client setName用于给客户端设置名字，这样比较容易标识出客户端的来源，例如将当前客户端命名为test_client，可以执行如下操作：

```
127.0.0.1:6379> client setName test_client
OK
```

此时再执行client list命令，就可以看到当前客户端的name属性为test_client：

```
127.0.0.1:6379> client list
id=55 addr=127.0.0.1:55604 fd=7 name=test_client age=23 idle=0 flags=N db=0 sub=0 
    psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

如果想直接查看当前客户端的name，可以使用client getName命令，例如下面的操作：

```
127.0.0.1:6379> client getName
"test_client"
```

client getName和setName命令可以做为标识客户端来源的一种方式，但是通常来讲，在Redis只有一个应用方使用的情况下，IP和端口作为标识会更加清晰。当多个应用方共同使用一个Redis，那么此时client setName可以作为标识客户端的一个依据。

#### client kill

```
client kill ip:port
```

此命令用于杀掉指定IP地址和端口的客户端，例如当前客户端列表为：

```
127.0.0.1:6379> client list
id=49 addr=127.0.0.1:55593 fd=6 name= age=9 idle=0 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
id=50 addr=127.0.0.1:52343 fd=7 name= age=4 idle=4 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

如果想杀掉127.0.0.1：52343的客户端，可以执行：

```
127.0.0.1:6379> client kill 127.0.0.1:52343
OK
```

执行命令后，client list结果只剩下了127.0.0.1：55593这个客户端：

```
127.0.0.1:6379> client list
id=49 addr=127.0.0.1:55593 fd=6 name= age=9 idle=0 flags=N db=0 sub=0 psub=0 
    multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

由于一些原因（例如设置timeout=0时产生的长时间idle的客户端），需要手动杀掉客户端连接时，可以使用client kill命令。

#### client pause

```
client pause timeout(毫秒)
```

client pause命令用于阻塞客户端timeout毫秒数，在此期间客户端连接将被阻塞。

![image-20251123220035900](images/image-20251123220035900.png)

例如在一个客户端执行：

```
127.0.0.1:6379> client pause 10000
OK
```

在另一个客户端执行ping命令，发现整个ping命令执行了9.72秒（手动执行redis-cli，只为了演示，不代表真实执行时间）：

```
127.0.0.1:6379> ping
PONG
(9.72s)
```

该命令可以在如下场景起到作用：

- client pause只对普通和发布订阅客户端有效，对于主从复制（从节点内部伪装了一个客户端）是无效的，也就是此期间主从复制是正常进行的，所以此命令可以用来让主从复制保持一致。
- client pause可以用一种可控的方式将客户端连接从一个Redis节点切换到另一个Redis节点。需要注意的是在生产环境中，暂停客户端成本非常高。

#### monitor

monitor命令用于监控Redis正在执行的命令，如图4-11所示，我们打开了两个redis-cli，一个执行set get ping命令，另一个执行monitor命令。可以看到monitor命令能够监听其他客户端正在执行的命令，并记录了详细的时间戳。

![image-20251123220301601](images/image-20251123220301601.png)

monitor的作用很明显，如果开发和运维人员想监听Redis正在执行的命令，就可以用monitor命令，但事实并非如此美好，每个客户端都有自己的输出缓冲区，既然monitor能监听到所有的命令，一旦Redis的并发量过大，monitor客户端的输出缓冲会暴涨，可能瞬间会占用大量内存，图4-12展示了monitor命令造成大量内存使用。

![image-20251123220334604](images/image-20251123220334604.png)

### 客户端相关配置

- timeout：检测客户端空闲连接的超时时间，一旦idle时间达到了timeout，客户端将会被关闭，如果设置为0就不进行检测。
- maxclients：客户端最大连接数，4.4.1节中的客户端存活状态部分已经进行分析，这里不再赘述，但是这个参数会受到操作系统设置的限制，第12章Linux相关配置小节还会对这个参数进行介绍。
- tcp-keepalive：检测TCP连接活性的周期，默认值为0，也就是不进行检测，如果需要设置，建议为60，那么Redis会每隔60秒对它创建的TCP连接进行活性检测，防止大量死连接占用系统资源。
- tcp-backlog：TCP三次握手后，会将接受的连接放入队列中，tcp-backlog就是队列的大小，它在Redis中的默认值是511。通常来讲这个参数不需要调整，但是这个参数会受到操作系统的影响，例如在Linux操作系统中，如果/proc/sys/net/core/somaxconn小于tcp-backlog，那么在Redis启动时会看到如下日志，并建议将/proc/sys/net/core/somaxconn设置更大。

```
# WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/
    sys/net/core/somaxconn is set to the lower value of 128.
```

修改方法也非常简单，只需要执行如下命令：

```
echo 511 > /proc/sys/net/core/somaxconn
```

### 客户端统计片段

例如下面就是一次info clients的执行结果：

```
127.0.0.1:6379> info clients
# Clients
connected_clients:1414
client_longest_output_list:0
client_biggest_input_buf:2097152
blocked_clients:0
```

说明如下：

1)connected_clients：代表当前Redis节点的客户端连接数，需要重点监控，一旦超过maxclients，新的客户端连接将被拒绝。

2)client_longest_output_list：当前所有输出缓冲区中队列对象个数的最大值。

3)client_biggest_input_buf：当前所有输入缓冲区中占用的最大容量。

4)blocked_clients：正在执行阻塞命令（例如blpop、brpop、brpoplpush）的客户端个数。

除此之外info stats中还包含了两个客户端相关的统计指标，如下：127.0.0.1:6379> info stats

```
# Stats
total_connections_received:80
…
rejected_connections:0
```

参数说明：

total_connections_received：Redis自启动以来处理的客户端连接数总数。

rejected_connections：Redis自启动以来拒绝的客户端连接数，需要重点监控

## 持久化

### RDB

**RDB持久化是把当前进程数据生成快照保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发。**

#### 触发机制

手动触发分别对应save和bgsave命令：

- save命令：**阻塞当前Redis服务器，直到RDB过程完成为止**，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。运行save命令对应的Redis日志如下：

  ```
  * DB saved on disk
  ```

  - 示例：

    先删除dump_6379.rdb

    ```
    [root@localhost 6379]# rm -f dump_6379.rdb
    [root@localhost 6379]# ll
    总用量 0
    ```

    ```
    127.0.0.1:6379> set k1 v1
    OK
    127.0.0.1:6379> save
    OK
    ```

    生产了dump_6379.rdb

    ```
    [root@localhost 6379]# ll
    总用量 4
    -rw-r--r--. 1 root root 100 12月 18 11:33 dump_6379.rdb
    ```

- bgsave命令：**Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。**阻塞只发生在fork阶段，一般时间很短。运行bgsave命令对应的Redis日志如下：

  ```
  * Background saving started by pid 3151
  * DB saved on disk
  * RDB: 0 MB of memory used by copy-on-write
  * Background saving terminated with success
  ```

  显然bgsave命令是针对save阻塞问题做的优化。因此Redis内部所有的涉及RDB的操作都采用bgsave的方式，而save命令已经废弃。

  ![image-20251218113646114](./assets/image-20251218113646114.png)

  - 示例：`127.0.0.1:6379> BGSAVE` → 返回 `Background saving started` 表示子进程已启动，可通过 `INFO persistence` 查看快照进度。

- 除了执行命令手动触发之外，Redis内部还存在自动触发RDB的持久化机制，例如以下场景：

  - 1)使用save相关配置，如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。
  - 2)如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点，更多细节见复制原理。
  - 3)执行debug reload命令重新加载Redis时，也会自动触发save操作。
  - 4)默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave。

- 如何禁用快照？

  - 动态所有停止RDB保存规则的方法：`redis-cli config set save ""`

  - 快照禁用`save ""`

    ![image-20251218121345012](./assets/image-20251218121345012.png)

#### 流程说明

- bgsave流的触发RDB持久化方式
  - 1)执行bgsave命令，Redis父进程判断当前是否存在正在执行的子进程，如RDB/AOF子进程，如果存在bgsave命令直接返回。
  
  - 2)进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通过info stats命令查看latest_fork_usec选项，可以获取最近一个fork操作的耗时，单位为微秒。
  
  - 3)父进程fork完成后，bgsave命令返回“Background saving started”信息并不再阻塞父进程，可以继续响应其他命令。
  
  - 4)子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换。执行lastsave命令可以获取最后一次生成RDB的时间，对应info统计的rdb_last_save_time选项。
  
    ![image-20251218114016422](./assets/image-20251218114016422.png)
  
  - 5)进程发送信号给父进程表示完成，父进程更新统计信息，具体见info Persistence下的rdb_*相关选项。

![image-20251125182428637](images/image-20251125182428637.png)

#### RDB文件的处理

- 保存：RDB文件保存在dir配置指定的目录下，文件名通过dbfilename配置指定。可以通过执行config set dir{newDir}和config set dbfilename{newFileName}运行期动态执行，当下次运行时RDB文件会保存到新目录。

  ```
  127.0.0.1:6379> config get requirepass
  1) "requirepass"
  2) "111111"
  127.0.0.1:6379> config get port
  1) "port"
  2) "6379"
  127.0.0.1:6379> config get dir
  1) "dir"
  2) "/var/lib/redis/6379"
  ```

- ![image-20251218105449320](./assets/image-20251218105449320.png)

  - 如何恢复？**将 RDB 文件放到 `dir` 配置指定的目录下，并确保文件名与 `dbfilename` 一致，同时确认 AOF 未启用或已处理，最后以正确权限启动 Redis。**

    ```
    `\# 假设备份文件为 backup.rdb 
    sudo cp backup.rdb /var/lib/redis/6379/dump.rdb 
    # 注意：文件名必须与 dbfilename 一致！`
    ```

  - 验证恢复是否成功：`redis-cli info persistence`

    关注以下字段：

    - `loading:0` → 加载完成（若为 1 表示仍在加载）
    - `rdb_last_load_keys` → 成功加载的 key 数量

  - 备注：不可以把备份文件dump.rdb和生产redis服务器放在同一台机器，必须分开各自存储。以防止生产物理损坏后备份文件也挂了。

  - 执行flushall/flushdb命令也会产生dump.rdb文件，但里面是空的，无意义。

#### 关键配置参数rdbcompression、rdbchecksum

- 当遇到坏盘或磁盘写满等情况时，可以通过config set dir{newDir}在线修改文件路径到可用的磁盘路径，之后执行bgsave进行磁盘切换，同样适用于AOF持久化文件。

- 压缩：Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启，可以通过参数config set rdbcompression{yes|no}动态修改。虽然压缩RDB会消耗CPU，但可大幅降低文件的体积，方便保存到硬盘或通过网络发送给从节点，因此线上建议开启。

  配置文件压缩：

  ![image-20251218121755876](./assets/image-20251218121755876.png)

- `rdbchecksum yes`：是否对 RDB 文件进行 CRC64 校验（确保文件未损坏），默认 `yes`。开启（`yes`）：增加约 10% 快照时间，但能避免加载损坏文件；关闭（`no`）：仅用于测试环境

- `stop-writes-on-bgsave-error`：若 `BGSAVE` 执行失败（如磁盘满），是否禁止后续写操作，默认 `yes`。生产环境建议开启（`yes`）：避免数据只在内存中，无持久化；测试环境可关闭（`no`）

#### 恢复失败的典型场景与应当

| 错误日志              | 原因                             | 解决方案                                          |
| --------------------- | -------------------------------- | ------------------------------------------------- |
| `Bad RDB file format` | RDB 版本过高                     | 升级 Redis 或使用旧版 RDB                         |
| `RDB checksum error`  | **文件损坏（磁盘坏道、写中断）** | 使用 `redis-check-rdb --fix` 尝试修复；从备份恢复 |
| `Short read`          | 文件不完整（未正常关闭）         | 检查是否有未完成的 bgsave；使用备份               |
| `OOM loading DB`      | 内存不足                         | 增加内存；分片数据                                |

#### RDB的优缺点

- RDB的优点：
  - RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每6小时执行bgsave备份，并把RDB文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复。
  - Redis加载RDB恢复数据远远快于AOF的方式。

- RDB的缺点：
  - **在一定间隔时间做一次备份,所以如果redis意外down掉的话,就会丢失从当前至最近一次快照期间的数据,快照之间的数据会丢失。**如果您需要在 Redis 停止工作时（例如断电后）将数据丢失的可能性降到最低，那么 RDB 并不好，您可以配置生成RDB的不同保存点(例如，在对数据集至少5分钟和100次写入之后,您可以有多个保存点),但是,您通常会每五分钟或更长时间创读一次RDB快照,因此,如果Redis由于任何原因在没有正确关闭的情况下停止工作,您应该准备好丢失最新分钟的数据。
  
    ![image-20251218120120729](./assets/image-20251218120120729.png)
  
    
  
  - **RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。**如果数据集很大，fork()可能会很耗时，并且如果数据集很大并且CPU性能不好，可能会导致Redis停止为客户端服务几毫秒甚至一秒钟，AOF也需要fork()但频率较低，你可以调整要重写日志的频率，而不需要对持久化进行任何权衡
  
  - RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题。
  
  - 针对RDB不适合实时持久化的问题，Redis提供了AOF持久化方式来解决。

### AOF

**AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的。**AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。理解掌握好AOF持久化机制对我们兼顾数据安全性和性能非常有帮助。

#### 使用AOF

开启AOF功能需要设置配置：appendonly yes，默认不开启。AOF文件名通过appendfilename配置设置，默认文件名是appendonly.aof。保存路径同RDB持久化方式一致，通过dir配置指定。

AOF的工作流程操作：**命令写入(append)、文件同步(sync)、文件重写(rewrite)、重启加载(load)**

![image-20251125183736615](images/image-20251125183736615.png)

- 流程如下：
  - 1)所有的写入命令会追加到aof_buf（缓冲区）中。
  - 2)AOF缓冲区根据对应的策略向硬盘做同步操作。
  - 3)随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩的目的。
  - 4)当Redis服务器重启时，可以加载AOF文件进行数据恢复。

- 文件保存

  - dump.rdb文件保存在/myredis下。appendonly.aof文件保存在dir+appenddirname下

  ![image-20251218124958584](./assets/image-20251218124958584.png)

  - redis7.0 config

    - base：表示基础AOF，它一般由子进程通过重写产生，该文件最多只有一个；
    - incr：表示增量AOF，它一般会在AOFRW开始执行时被创建，该文件可能存在多个；
    - manifest：清单文件，用来跟踪、管理这些AOF文件，manifest文件单独保存在appenddirname配置路径。
    
    ![image-20251218125739881](./assets/image-20251218125739881.png)
    
    ![image-20251218130028158](./assets/image-20251218130028158.png)
  
  - 按照上面的操作开启AOF，并将dir配置为/myRedis时，我们执行Redis的写操作命令后，就会在myRedis/appendonlydir路径下生成AOF文件，如下图所示：
  
    可以发现，当我们执行写操作后，Redis就会在myRedis文件夹创建appendonlydir文件夹并生成三个AOF文件，当我们再次执行写操作后，三个AOF文件，只有incr的AOF发生变化。
  
    当发生宕机后，重启Redis服务，Redis就会根据AOF文件恢复数据。
  
  ![img](./assets/fb740a77292548b6b492dc7ae1744e41-1767436914639-4-1767436919034-6.png)

#### 命令写入文件格式（RESP协议）

AOF命令写入的内容直接是文本协议格式。例如set hello world这条命令，在AOF缓冲区会追加如下文本：

```
*3\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n
```

- 1)AOF为什么直接采用文本协议格式？可能的理由如下：
  - 文本协议具有很好的兼容性。
  - 开启AOF后，所有写入命令都包含追加操作，直接采用协议格式，避免了二次处理开销。
  - 文本协议具有可读性，方便直接修改和处理。

- 2)AOF为什么把命令追加到aof_buf中？Redis使用单线程响应命令，如果每次写AOF文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载。先写入缓冲区aof_buf中，还有另一个好处，Redis可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡。

#### 恢复

- 正常恢复

  - 启动：设置Yes，修改appendonly no，改为yes。重启redis6379.conf后配置生效

  - 示例1：

    ```
    [root@localhost 6379]# ll
    总用量 4
    drwxr-xr-x. 2 root root 103 12月 18 13:35 appendonlydir
    -rw-r--r--. 1 root root 121 12月 18 13:34 dump_6379.rdb
    [root@localhost 6379]# cd appendonlydir
    [root@localhost appendonlydir]# ll
    总用量 12
    -rw-r--r--. 1 root root 88 12月 18 13:35 appendonly.aof.1.base.rdb
    -rw-r--r--. 1 root root 52 12月 18 13:36 appendonly.aof.1.incr.aof
    -rw-r--r--. 1 root root 88 12月 18 13:35 appendonly.aof.manifest
    ```

    发现appendonly.aof.1.incr.aof为18  13:36

    执行

    ```
    127.0.0.1:6379> set k9 v9
    OK
    ```

    ```
    [root@localhost appendonlydir]# ll
    总用量 12
    -rw-r--r--. 1 root root  88 12月 18 13:35 appendonly.aof.1.base.rdb
    -rw-r--r--. 1 root root 168 12月 18 13:48 appendonly.aof.1.incr.aof
    -rw-r--r--. 1 root root  88 12月 18 13:35 appendonly.aof.manifest
    ```

    发现root root 168 12月 18 13:48 appendonly.aof.1.incr.aof发生改变

    执行vim appendonly.aof.1.incr.aof

    ```
    set
    $2
    k9
    $2
    v9
    ```

  - 示例2：

    flushall清空后

    ```
    127.0.0.1:6379> keys *
    1) "k2"
    2) "k1"
    127.0.0.1:6379> flushall
    OK
    127.0.0.1:6379> keys *
    (empty array)
    ```
    
    config get dir
    
    在dir下找到appenddirname
    
    执行vim appendonly.aof.1.incr.aof删除flushall后:wq！保存
    
    重启redis
    
    ```
    127.0.0.1:6379> shutdown
    not connected> quit
    [root@localhost redis-7.2.4]# redis-server redis6379.conf
    [root@localhost redis-7.2.4]# redis-cli -p 6379 -a 111111
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    127.0.0.1:6379> keys *
    1) "k2"
    2) "k1"
    ```
    
    发现已恢复
  
- 异常恢复

  - 故意乱写正常的AOF文件，模拟网络闪断文件写error，AOF 文件损坏，AOF 文件未完整写入
  - 重启Redis之后就会进行AOF文件的载入，发现启动都不行
  - 异常修复命令：redis-check-aof --fix进行修复，记得要带上--fix参数

  ![image-20251218140420350](./assets/image-20251218140420350.png)

- `aof-load-truncated yes`（默认开启）：文件结尾不完整



#### 文件同步

Redis提供了多种AOF缓冲区同步文件策略，由参数appendfsync控制

- appendfsync always：

  - 策略逻辑：每执行一条写命令，立即将缓冲区中的命令写入磁盘（调用操作系统 `fsync` 函数，确保数据落盘）；
  - 数据可靠性：**零数据丢失**（除非磁盘硬件故障）
  - 性能影响：磁盘 IO 频率极高（每写一次触发一次 `fsync`），单机 QPS 会从 10 万 + 降至几千，仅适合对数据可靠性要求极高的场景（如金融核心交易）

- appendfsync everysec：

  - 策略逻辑：每秒将缓冲区中的命令批量写入磁盘（由 Redis 后台线程执行 fsync）；

  - 数据可靠性：最多丢失 1 秒 内的数据（若 Redis 进程崩溃，缓冲区中未刷盘的命令丢失；若操作系统崩溃，已写入内核缓冲区但未刷盘的命令丢失）；

  - 性能影响：磁盘 IO 频率适中（每秒一次 fsync），单机 QPS 可维持在几万，是 生产环境默认且推荐的策略；

- appendfsync no：

  - 策略逻辑：Redis 仅将命令写入操作系统内核缓冲区，刷盘时机由操作系统决定（如内核缓冲区满、系统空闲时）；

  - 数据可靠性：数据丢失风险高（可能丢失几秒到几分钟的数据）；

  - 性能影响：Redis 性能最优（无 fsync 开销），但仅适合非核心业务（如临时缓存），生产环境 禁止使用。
    

![image-20251218123731186](./assets/image-20251218123731186.png)



系统调用write和fsync说明：

- write操作会触发延迟写(delayed write)机制。Linux在内核提供页缓冲区用来提高硬盘IO性能。write操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，例如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失。
- fsync针对单个文件操作（比如AOF文件），做强制硬盘同步，fsync将阻塞直到写入硬盘完成后返回，保证了数据持久化。

除了write、fsync，Linux还提供了sync、fdatasync操作，具体API说明参见：http://linux.die.net/man/2/write，http://linux.die.net/man/2/fsync，http://linux.die.net/man/2/sync，http://linux.die.net/man/2/fdatasync。



#### 重写机制

- 随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入AOF重写机制压缩文件体积。AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程。重写后的AOF文件为什么可以变小？有如下原因：
  - 1)进程内已经超时的数据不再写入文件。
  - 2)旧的AOF文件含有无效命令，重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。举个例子：比如有个key —开始你 set k1 v1 然后改成 set k1 v2 最后改成 set k1 v3 如果不重写，那么这3条语句都在aof文件中，内容占空间不说启动的时候都要执行一遍，共计3条命令；但是，我们实际效果只需要set k1 v3这一条，所以， 开启重写后，只需要保存set k1 v3就可以了只需要保留最后一次修改值，相当于给aof文件瘦身减肥，性能更好。 AOF重写不仅降低了文件的占用空间，同时更小的AQF也可以更快地被Redis加我。
  - 3)多条写命令可以合并为一个，如：lpush list a、lpush list b、lpush list c可以转化为：lpush list a b c。为了防止单条命令过大造成客户端缓冲区溢出，对于list、set、hash、zset等类型操作，以64个元素为界拆分为多条。

- AOF重写降低了文件占用空间，除此之外，另一个目的是：更小的AOF文件可以更快地被Redis加载。

- AOF重写过程可以手动触发和自动触发：
  - 手动触发：直接调用bgrewriteaof命令。
  
    - 示例
  
      ![image-20251220153028856](./assets/image-20251220153028856.png)
      
      - 查看进度：通过 `INFO persistence` 中的 `aof_rewrite_in_progress`（1 表示正在重写，0 表示未重写）。
  
  - 自动触发：根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数确定自动触发时机。
  
    - auto-aof-rewrite-min-size：表示运行AOF重写时文件最小体积，默认为64MB。
  
    - auto-aof-rewrite-percentage：代表当前AOF文件空间(aof_current_size)和上一次重写后AOF文件空间(aof_base_size)的比值。
  
    - 自动触发时机=aof_current_size>auto-aof-rewrite-min-size&&(aof_current_size-aof_base_size)/aof_base_size>=auto-aof-rewrite-percentage其中aof_current_size和aof_base_size可以在info Persistence统计信息中查看。
  
    - 例如：
  
      ![image-20251220152901012](./assets/image-20251220152901012.png)
  
      ![image-20251220152913895](./assets/image-20251220152913895.png)
  
      
  
- 当触发AOF重写时，内部做了哪些事呢？

  ![image-20251125185024521](images/image-20251125185024521.png)

  流程说明：

  - 1)执行AOF重写请求。如果当前进程正在执行AOF重写，请求不执行并返回如下响应：

    ```
    ERR Background append only file rewriting already in progress
    ```

    如果当前进程正在执行bgsave操作，重写命令延迟到bgsave完成之后再执行，返回如下响应：

    ````
    Background append only file rewriting scheduled
    ````

  - 2)父进程执行fork创建子进程，开销等同于bgsave过程。

  - 3.1)主进程fork操作完成后，继续响应其他命令。所有修改命令依然写入AOF缓冲区并根据appendfsync策略同步到硬盘，保证原有AOF机制正确性。3.2)由于fork操作运用写时复制技术，子进程只能共享fork操作时的内存数据。由于父进程依然响应命令，Redis使用“AOF重写缓冲区”保存这部分新数据，防止新AOF文件生成期间丢失这部分数据。

  - 4)子进程根据内存快照，按照命令合并规则写入到新的AOF文件。每次批量写入硬盘数据量由配置aof-rewrite-incremental-fsync控制，默认为32MB，防止单次刷盘数据过多造成硬盘阻塞。

  - 5.1)新AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息，具体见info persistence下的aof_*相关统计。5.2)父进程把AOF重写缓冲区的数据写入到新的AOF文件。5.3)使用新AOF文件替换老文件，完成AOF重写。

#### 重启加载

AOF和RDB文件都可以用于服务器重启时的数据恢复。Redis持久化文件加载流程。

![image-20251125185417737](images/image-20251125185417737.png)

- 流程说明：

  - 1)AOF持久化开启且存在AOF文件时，优先加载AOF文件，打印如下日志：

    ```
    * DB loaded from append only file: 5.841 seconds
    ```

  - 2)AOF关闭或者AOF文件不存在时，加载RDB文件，打印如下日志：

    ```
    * DB loaded from disk: 5.586 seconds
    ```

  - 3)加载AOF/RDB文件成功后，Redis启动成功。

  - 4)AOF/RDB文件存在错误时，Redis启动失败并打印错误信息。

#### 文件校验

加载损坏的AOF文件时会拒绝启动，并打印如下日志：

```
# Bad file format reading the append only file: make a backup of your AOF file,
    then use ./redis-check-aof --fix <filename>
```

对于错误格式的AOF文件，先进行备份，然后采用redis-check-aof--fix命令进行修复，修复后使用diff-u对比数据的差异，找出丢失的数据，有些可以人工修改补全。

AOF文件可能存在结尾不完整的情况，比如机器突然掉电导致AOF尾部文件命令写入不全。Redis为我们提供了aof-load-truncated配置来兼容这种情况，默认开启。加载AOF时，当遇到此问题时会忽略并继续启动，同时打印如下警告日志：

```
# !!! Warning: short read while loading the AOF file !!!
# !!! Truncating the AOF at offset 397856725 !!!
# AOF loaded anyway because aof-load-truncated is enabled
```

#### AOF优势

优势

- 使用 AOF Redis更加持久：您可以有不同的 fsync 策略：根本不fsync，每秒fsync，每次查询时 fsync，使用每秒 fsync 的默认策略，写入性能仍然很棒，fsync 是使用后台任程执行的，当没有 fsync 正在进行时，主线程将努力执行写入，因此您只能丢失一秒神的写入。 
- AOF 日志是一个仅附加日志，因此不会出现寻道问题，也不会在断电时出现损坏问题，即便由于某种原因（磁盘已满或其他原因）日志以写一半的命令结尾，redis-check-aof 工典也能够轻松修复它。
- 当AOF 变得太大时，Redis 能够在后台自动重写 AOF，重写是完全安全的，因为当 Redis 继续附加到旧文件时，会使用创建当前数据集所需的最少操作集生成一个全新的文件，一旦第二个文件准备就绪，Redis 就会切换两者并开始附加到新的那一个。 
- AOF 以易于理解和解析的格式依次包含所有操作的日志，您甚至可以轻松导出 AOF 文件，例如，即使您不小心使用该FLUSHALL命令刷新了所有内容，只要在此期间没有执行日志重写，您 仍然可以通过停止服务器，删除最新命令并重新启动 Redis 来保存您的数据集

缺点

- AOF 文件通常比相同数据集的等效 RDB 文件大。 
- 根据确切的fsync策略, AOF可能比RDB慢,一般来说,将fsyne设置为每秒性能仍然本常高,并且在禁用foync的情况下，即使在高负载下它也应该与RDB一样快，即使在巨大的写入负载的情况下，RDB仍然能够提供关于最大延迟的更多保证。

#### RDB和AOF

![image-20251220154408319](./assets/image-20251220154408319.png)

1. 开启混合方式设置

设置aof-rdb-preamble的值为yes，yes表示开启

2. RDB+AOF混合方式 --------》结论：RDB镜像做全量持久化，AOF镜像做增量持久化

先使用RDB进行快照存储,然后使用AOF持久化记录所有的写操作,当重写策略满足或手动触发重写的时候,将最新的数据存储为新的RDB记录,这样的话,重启服务的时候会从RDB和AOF两部分恢复数据,既保证了数据完整性,又提高了恢复数据的性能,简单来说:混合持久化方式产生的文件一部分是RDB格式,一部分是AOF格式, -------》AOF包格了RDB头部+AOF混写

![image-20251220155106772](./assets/image-20251220155106772.png)

3. 纯内存模式--同时关闭RDB+AOF

`save ""`：禁用rdb，禁用rdb持久化模式下，我们仍然可以使用命令save，bgsave生成rdb文件

`appendonly no`：禁用aof，禁用aof持久化模式下，我们仍然可以使用命令bgrewriteaof生产aof文件



### 问题定位与优化

#### fork操作

当Redis做RDB或AOF重写时，一个必不可少的操作就是执行fork操作创建子进程，对于大多数操作系统来说fork是个重量级错误。虽然fork创建的子进程不需要拷贝父进程的物理内存空间，但是会复制父进程的空间内存页表。例如对于10GB的Redis进程，需要复制大约20MB的内存页表，因此fork操作耗时跟进程总内存量息息相关，如果使用虚拟化技术，特别是Xen虚拟机，fork操作会更耗时。

fork耗时问题定位：对于高流量的Redis实例OPS可达5万以上，如果fork操作耗时在秒级别将拖慢Redis几万条命令执行，对线上应用延迟影响非常明显。正常情况下fork耗时应该是每GB消耗20毫秒左右。可以在info stats统计中查latest_fork_usec指标获取最近一次fork操作耗时，单位微秒。

如何改善fork操作的耗时：

- 1)优先使用物理机或者高效支持fork操作的虚拟化技术，避免使用Xen。
- 2)控制Redis实例最大可用内存，fork耗时跟内存量成正比，线上建议每个Redis实例内存控制在10GB以内。
- 3)合理配置Linux内存分配策略，避免物理内存不足导致fork失败，具体细节见12.1节“Linux配置优化”。
- 4)降低fork操作的频率，如适度放宽AOF自动触发时机，避免不必要的全量复制等。

#### 子进程开销监控和优化

子进程负责AOF或者RDB文件的重写，它的运行过程主要涉及CPU、内存、硬盘三部分的消耗。

##### 1.CPU

- CPU开销分析。子进程负责把进程内的数据分批写入文件，这个过程属于CPU密集操作，通常子进程对单核CPU利用率接近90%.

- CPU消耗优化。Redis是CPU密集型服务，不要做绑定单核CPU操作。由于子进程非常消耗CPU，会和父进程产生单核资源竞争。
- 不要和其他CPU密集型服务部署在一起，造成CPU过度竞争。
- 如果部署多个Redis实例，尽量保证同一时刻只有一个子进程执行重写工作，具体细节见5.4节多实例部署”。

##### 2.内存

- 内存消耗分析。子进程通过fork操作产生，占用内存大小等同于父进程，理论上需要两倍的内存来完成持久化操作，但Linux有写时复制机制(copy-on-write)。父子进程会共享相同的物理内存页，当父进程处理写请求时会把要修改的页创建副本，而子进程在fork操作过程中共享整个父进程内存快照。

- 内存消耗监控。RDB重写时，Redis日志输出容如下：

  ```
  * Background saving started by pid 7692
  * DB saved on disk
  * RDB: 5 MB of memory used by copy-on-write
  * Background saving terminated with success
  ```

  如果重写过程中存在内存修改操作，父进程负责创建所修改内存页的副本，从日志中可以看出这部分内存消耗了5MB，可以等价认为RDB重写消耗了5MB的内存。

  AOF重写时，Redis日志输出容如下：

  ```
  * Background append only file rewriting started by pid 8937
  * AOF rewrite child asks to stop sending diffs.
  * Parent agreed to stop sending diffs. Finalizing AOF…
  * Concatenating 0.00 MB of AOF diff received from parent.
  * SYNC append only file rewrite performed
  * AOF rewrite: 53 MB of memory used by copy-on-write
  * Background AOF rewrite terminated with success
  * Residual parent diff successfully flushed to the rewritten AOF (1.49 MB)
  * Background AOF rewrite finished successfully
  ```

  父进程维护页副本消耗同RDB重写过程类似，不同之处在于AOF重写需要AOF重写缓冲区，因此根据以上日志可以预估内存消耗为：53MB+1.49MB，也就是AOF重写时子进程消耗的内存量。

- 编写shell脚本根据Redis日志可快速定位子进程重写期间内存过度消耗情况。内存消耗优化：
  - 1)同CPU优化一样，如果部署多个Redis实例，尽量保证同一时刻只有一个子进程在工作。
  - 2)避免在大量写入时做子进程重写操作，这样将导致父进程维护大量页副本，造成内存消耗。
  - Linux kernel在2.6.38内核增加了Transparent Huge Pages(THP)，支持huge page(2MB)的页分配，默认开启。当开启时可以降低fork创建子进程的速度，但执行fork之后，如果开启THP，复制页单位从原来4KB变为2MB，会大幅增加重写期间父进程内存消耗。建议设置“sudo echo never>/sys/kernel/mm/transparent_hugepage/enabled”关闭THP。更多THP细节和配置见12.1节Linux配置优化”。

##### 3.硬盘

- 硬盘开销分析。子进程主要职责是把AOF或者RDB文件写入硬盘持久化。势必造成硬盘写入压力。根据Redis重写AOF/RDB的数据量，结合系统工具如sar、iostat、iotop等，可分析出重写期间硬盘负载情况。
- 硬盘开销优化。优化方法如下：
  - a)不要和其他高硬盘负载的服务部署在一起。如：存储服务、消息队列服务等。
  - b)AOF重写时会消耗大量硬盘IO，可以开启配置no-appendfsync-on-rewrite，默认关闭。表示在AOF重写期间不做fsync操作。
  - c)当开启AOF功能的Redis用于高流量写入场景时，如果使用普通机械磁盘，写入吞吐一般在100MB/s左右，这时Redis实例的瓶颈主要在AOF同步硬盘上。
  - d)对于单机配置多个Redis实例的情况，可以配置不同实例分盘存储AOF文件，分摊硬盘写入压力。
  - 配置no-appendfsync-on-rewrite=yes时，在极端情况下可能丢失整个AOF重写期间的数据，需要根据数据安全性决定是否配置。

#### AOF追加阻塞

当开启AOF持久化时，常用的同步硬盘的策略是everysec，用于平衡性能和数据安全性。对于这种方式，Redis使用另一条线程每秒执行fsync同步硬盘。当系统硬盘资源繁忙时，会造成Redis主线程阻塞

![image-20251125190757810](images/image-20251125190757810.png)

- 阻塞流程分析：
  - 1)主线程负责写入AOF缓冲区。
  - 2)AOF线程负责每秒执行一次同步磁盘操作，并记录最近一次同步时间。
  - 3)主线程负责对比上次AOF同步时间：
    - 如果距上次同步成功时间在2秒内，主线程直接返回。
    - 如果距上次同步成功时间超过2秒，主线程将会阻塞，直到同步操作完成。

- 通过对AOF阻塞流程可以发现两个问题：
  - 1)everysec配置最多可能丢失2秒数据，不是1秒。
  - 2)如果系统fsync缓慢，将会导致Redis主线程阻塞影响效率。

- AOF阻塞问题定位：

  - 1)发生AOF阻塞时，Redis输出如下日志，用于记录AOF fsync阻塞导致拖慢Redis服务的行为：

    ```
    Asynchronous AOF fsync is taking too long (disk is busy). Writing the AOF buffer
        without waiting for fsync to complete, this may slow down Redis
    ```

  - 2)每当发生AOF追加阻塞事件发生时，在info Persistence统计中，aof_delayed_fsync指标会累加，查看这个指标方便定位AOF阻塞问题。
  - 3)AOF同步最多允许2秒的延迟，当延迟发生时说明硬盘存在高负载问题，可以通过监控工具如iotop，定位消耗硬盘IO资源的进程。
  - 优化AOF追加阻塞问题主要是优化系统硬盘负载，优化方式见上一节。

### 多实例部署

Redis单线程架构导致无法充分利用CPU多核特性，通常的做法是在一台机器上部署多个Redis实例。当多个实例开启AOF重写后，彼此之间会产生对CPU和IO的竞争。本节主要介绍针对这种场景的分析和优化。

上一节介绍了持久化相关的子进程开销。对于单机多Redis部署，如果同一时刻运行多个子进程，对当前系统影响将非常明显，因此需要采用一种措施，把子进程工作进行隔离。Redis在info Persistence中为我们提供了监控子进程运行状况的度量指标，如表5-2所示。

info Persistence片段度量指标

![image-20251125191151038](images/image-20251125191151038.png)

我们基于以上指标，可以通过外部程序轮询控制AOF重写操作的执行，整个过程如图5-6所示。

![image-20251125191229305](images/image-20251125191229305.png)

流程说明：

1)外部程序定时轮询监控机器(machine)上所有Redis实例。

2)对于开启AOF的实例，查看(aof_current_size-aof_base_size)/aof_base_size确认增长率。

3)当增长率超过特定阈值（如100%），执行bgrewriteaof命令手动触发当前实例的AOF重写。

4)运行期间循环检查aof_rewrite_in_progress和aof_current_rewrite_time_sec指标，直到AOF重写结束。

5)确认实例AOF重写完成后，再检查其他实例并重复2)~4)步操作。从而保证机器内每个Redis实例AOF重写串行化执行。

## **3.2.SpringDataRedis**客户端

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，官网地址：https://spring.io/projects/spring-data-redis

- 提供了对不同Redis客户端的整合（Lettuce和Jedis）
- 提供了RedisTemplate统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现



SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

![](assets/UFlNIV0.png)



### 3.2.1.快速入门

SpringBoot已经提供了对SpringDataRedis的支持，使用非常简单。



首先，新建一个maven项目，然后按照下面步骤执行：

#### 1）引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.7</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.heima</groupId>
    <artifactId>redis-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redis-demo</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--common-pool-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!--Jackson依赖-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 2）配置Redis

在application.yml配置Redis信息

```yaml
spring:
  redis:
    host: 192.168.150.101
    port: 6379
    password: 123321
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: 100ms
```

#### 3）注入RedisTemplate

因为有了SpringBoot的自动装配，我们可以拿来就用：

```java
@SpringBootTest
class RedisStringTests {

    @Autowired
    private RedisTemplate redisTemplate;
}
```



#### 4）编写测试

```java
@SpringBootTest
class RedisStringTests {

    @Autowired
    private RedisTemplate edisTemplate;

    @Test
    void testString() {
        // 写入一条String数据
        redisTemplate.opsForValue().set("name", "虎哥");
        // 获取string数据
        Object name = stringRedisTemplate.opsForValue().get("name");
        System.out.println("name = " + name);
    }
}
```



### 3.2.2.自定义序列化

RedisTemplate可以接收任意Object作为值写入Redis：

![](assets/OEMcbuu.png)





只不过写入前会把Object序列化为字节形式，默认是采用JDK序列化，得到的结果是这样的：

![](assets/5FjtWk5.png)



缺点：

- 可读性差
- 内存占用较大





我们可以自定义RedisTemplate的序列化方式，代码如下：

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = 
            							new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
```



这里采用了JSON序列化来代替默认的JDK序列化方式。最终结果如图：

![](assets/XOAq3cN.png)

整体可读性有了很大提升，并且能将Java对象自动的序列化为JSON字符串，并且查询时能自动把JSON反序列化为Java对象。不过，其中记录了序列化时对应的class名称，目的是为了查询时实现自动反序列化。这会带来额外的内存开销。





### 3.2.3.StringRedisTemplate

为了节省内存空间，我们可以不使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的key和value。当需要存储Java对象时，手动完成对象的序列化和反序列化。

![](assets/Ip9TKSY.png)

因为存入和读取时的序列化及反序列化都是我们自己实现的，SpringDataRedis就不会将class信息写入Redis了。



这种用法比较普遍，因此SpringDataRedis就提供了RedisTemplate的子类：StringRedisTemplate，它的key和value的序列化方式默认就是String方式。

![](assets/zXH6Qn6.png)



省去了我们自定义RedisTemplate的序列化方式的步骤，而是直接使用：

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;
// JSON序列化工具
private static final ObjectMapper mapper = new ObjectMapper();

@Test
void testSaveUser() throws JsonProcessingException {
    // 创建对象
    User user = new User("虎哥", 21);
    // 手动序列化
    String json = mapper.writeValueAsString(user);
    // 写入数据
    stringRedisTemplate.opsForValue().set("user:200", json);

    // 获取数据
    String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
    // 手动反序列化
    User user1 = mapper.readValue(jsonUser, User.class);
    System.out.println("user1 = " + user1);
}

```

## 主从复制

### 配置

#### 建立复制

- 参与复制的Redis实例划分为**主节点(master)和从节点(slave)**。
- 默认情况下，Redis都是主节点。**每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。**
- **复制的数据流是单向的，只能由主节点复制到从节点，master以写为主，Slave以读为主。**

- **主从复制的优势在于简单易用,适用于读多写少的场景。**它提供了数据备份功能，并且可以有很好的扩展性，只要增加更多的从节点，就能让整个集群的读的能力不断提升。 

- 但是主从模式最大的缺点，就是**不具备故障自动转移**的能力，没有办法做容错和恢复。 

- 主节点和从节点的宕机都会导致客户端部分读写请求失败,需要人工介入让节点恢复或者手动切换一台从节点服务器变成主节点服务器才可以。并且在主节点宕机时,如果数据没有及时复制到从节点,也会导致数据不一致。

配置复制的方式有以下三种：

- 1)在配置文件中加入slaveof{masterHost}{masterPort}随Redis启动生效。
- 2)在redis-server启动命令后加入--slaveof{masterHost}{masterPort}生效。
- 3)直接使用命令：slaveof{masterHost}{masterPort}生效。

综上所述，slaveof命令在使用时，可以运行期动态配置，也可以提前写到配置文件中。例如本地启动两个端口为6379和6380的Redis节点，在127.0.0.1：6380执行如下命令：

```
127.0.0.1:6380>slaveof 127.0.0.1 6379
```

slaveof配置都是在从节点发起，这时6379作为主节点，6380作为从节点。复制关系建立后执行如下命令测试：

```
127.0.0.1:6379>set hello redis
OK
127.0.0.1:6379>get hello
"redis"
127.0.0.1:6380>get hello
"redis"
```

主从节点复制成功建立后，可以使用info replication命令查看复制相关状态，如下所示。

1)主节点6379复制状态信息：

```
127.0.0.1:6379>info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6379,state=online,offset=43,lag=0
….
```

2)从节点6380复制状态信息：

````
127.0.0.1:6380>info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:4
master_sync_in_progress:0
…
````

#### 断开复制

- slaveof命令不但可以建立复制，还可以**在从节点执行slaveof no one来断开与主节点复制关系**。例如在6380节点上执行slaveof no one来断开复制

- 断开复制主要流程：
  - 1)断开与主节点复制关系。
  - 2)从节点晋升为主节点。

- **从节点断开复制后并不会抛弃原有数据，只是无法再获取主节点上的数据变化。**

- 通过slaveof命令还可以实现切主操作，所谓切主是指把当前从节点对主节点的复制切换到另一个主节点。执行slaveof{newMasterIp}{newMasterPort}命令即可，例如把6380节点从原来的复制6379节点变为复制6381节点

- 切主操作流程如下：1)断开与旧主节点复制关系。2)与新主节点建立复制关系。3)删除从节点当前所有数据。4)对新主节点进行复制操作。

- 切主后从节点会清空之前所有的数据，线上人工操作时小心slaveof在错误的节点上执行或者指向错误的主节点。

#### 安全性

对于数据比较重要的节点，主节点会通过设置requirepass参数进行密码验证，这时所有的客户端访问必须使用auth命令实行校验。从节点与主节点的复制连接是通过一个特殊标识的客户端来完成，因此需要配置从节点的masterauth参数与主节点密码保持一致，这样从节点才可以正确地连接到主节点并发起复制流程。

#### 只读

默认情况下，从节点使用slave-read-only=yes配置为只读模式。由于复制只能从主节点到从节点，对于从节点的任何修改主节点都无法感知，修改从节点会造成主从数据不一致。因此建议线上不要修改从节点的只读模式。

#### 传输延迟

主从节点一般部署在不同机器上，复制时的网络延迟就成为需要考虑的问题，Redis为我们提供了repl-disable-tcp-nodelay参数用于控制是否关闭TCP_NODELAY，默认关闭，说明如下：

- 当关闭时，主节点产生的命令数据无论大小都会及时地发送给从节点，这样主从之间延迟会变小，但增加了网络带宽的消耗。适用于主从之间的网络环境良好的场景，如同机架或同机房部署。
- 当开启时，主节点会合并较小的TCP数据包从而节省带宽。默认发送时间间隔取决于Linux的内核，一般默认为40毫秒。这种配置节省了带宽但增大主从之间的延迟。适用于主从网络环境复杂或带宽紧张的场景，如跨机房部署。
- 部署主从节点时需要考虑网络延迟、带宽使用率、防灾级别等因素，如要求低延迟时，建议同机架或同机房部署并关闭repl-disable-tcp-nodelay；如果考虑高容灾性，可以同城跨机房部署并开启repl-disable-tcp-nodelay。

#### 主从问题演示

![img](./assets/1690621505299-413de817-a5c0-4f03-8f15-231cbca0735d-1769505974236-3.png)

| 角色             | 职责                          | 关键特性                                              |
| ---------------- | ----------------------------- | ----------------------------------------------------- |
| 主节点 (Master)  | 处理写请求 + 同步数据到从节点 | 唯一写入点，可挂载多个从节点                          |
| 从节点 (Replica) | 复制主节点数据 + 处理读请求   | Redis 5.0+ 命令已更名为 `replica`（`slaveof` 仍兼容） |
| 数据流           | 单向同步：Master → Replica    | 不可反向同步（避免数据冲突）                          |

- **重要澄清**：**"Slave" 术语已废弃**：Redis 5.0+ 官方文档全面使用 **`replica`**（配置项 `replicaof`）

  - 配置replicao

    | 方式           | 命令/配置                      | 生效时机     | 适用场景      | 风险                   |
    | :------------- | :----------------------------- | :----------- | :------------ | :--------------------- |
    | **配置文件**   | `replicaof 192.168.1.100 6379` | Redis启动时  | 永久复制关系  | 修改需重启             |
    | **启动参数**   | `redis-server --replicaof ...` | 启动时       | 临时测试      | 进程级生效             |
    | **运行时命令** | `REPLICAOF 192.168.1.100 6379` | **立即生效** | 动态切主/运维 | **误操作高危！**       |
    | **断开复制**   | `REPLICAOF NO ONE`             | 立即生效     | 故障隔离      | 从节点变主（保留数据） |

![image-20251220181755675](./assets/image-20251220181755675.png)

- 从机可以执行写命令吗？
  - 从机默认只可以读取，不可以写。主机可以读/写

- **Slave复制起点：全量 or 增量？**
  - **首次连接/偏移量失效** → **全量同步**（从头复制）
    （发送 `PSYNC ? -1`，主节点生成RDB快照传输）
  - **断线重连且偏移量有效** → **部分同步**（切入点续传）
    （发送 `PSYNC <runid> <offset>`，仅同步缓冲区缺失命令）
  - **关键依赖**：`复制积压缓冲区`（`repl-backlog-size`）大小决定能否增量同步

- **Master宕机，Slave会自动上位吗？❌ 不会**
  - **基础主从模式无自动故障转移能力**
  - "需要人工介入手动切换"、"Redis不具备自动容错和恢复功能"
  - **解决方案**：需部署 **Sentinel哨兵**，实现自动选主
- 主机shutdown后，从机会上位吗？

![image-20251220180223244](./assets/image-20251220180223244.png)

- **Master重启后主从关系是否保留？保留（配置文件方式）**
  - **配置文件设置**（`slaveof`写入redis.conf）→ 重启后自动恢复连接
    （"重启服务器后，还是会回到之前一主两从的结构，因为配置文件里面是这样设置的"）
  - **注意**：若Master重启导致`runid`变更，Slave会触发**全量同步**

- 某台从机down后，master继续，从机重启后它能跟上大部队吗？可以
  - 重连后根据offerset和runid判断：
    - 偏移量在缓冲区内 → **部分同步**（高效追平）
    - 偏移量失效/`runid`变更 → **全量同步**
      （8]详细描述PSYNC机制）

- **命令设置的主从关系重启后是否保留？ 不保留**
  - 用命令使用的话，2台从机重启后，关系还在吗？不在。**命令方式**：仅当前会话有效，重启后失效（恢复独立Master）
  - 配置VS命令的区别？配置持久稳定，命令当次生效。**配置文件方式**：持久化生效，重启自动恢复

### 拓扑

Redis的复制拓扑结构可以支持单层或多层复制关系，根据拓扑复杂性可以分为以下三种：一主一从、一主多从、树状主从结构，下面分别介绍。

#### 1.一主一从结构

一主一从结构是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持。当应用写命令并发量较高且需要持久化时，可以只在从节点上开启AOF，这样既保证数据安全性同时也避免了持久化对主节点的性能干扰。但需要注意的是，当主节点关闭持久化功能时，如果主节点脱机要避免自动重启操作。因为主节点之前没有开启持久化功能自动重启后数据集为空，这时从节点如果继续复制主节点会导致从节点数据也被清空的情况，丧失了持久化的意义。安全的做法是在从节点上执行slaveof no one断开与主节点的复制关系，再重启主节点从而避免这一问题。

![image-20251126125402185](images/image-20251126125402185.png)

#### 2.一主多从结构

一主多从结构（又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离（见图6-5）。对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力。同时在日常开发中如果需要执行一些比较耗时的读命令，如：keys、sort等，可以在其中一台从节点上执行，防止慢查询对主节点造成阻塞从而影响线上服务的稳定性。对于写并发量较高的场景，多个从节点会导致主节点写命令的多次发送从而过度消耗网络带宽，同时也加重了主节点的负载影响服务稳定性。

![image-20251126125448912](images/image-20251126125448912-1764132890446-1.png)

#### 3.树状主从结构

树状主从结构（又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。如图6-6所示，数据写入节点A后会同步到B和C节点，B节点再把数据同步到D和E节点，数据实现了一层一层的向下复制。当主节点需要挂载多个从节点时为了避免对主节点的性能干扰，可以采用树状主从结构降低主节点压力。

![image-20251126125538383](images/image-20251126125538383.png)

### 原理

#### 复制过程

在从节点执行slaveof命令后，复制过程便开始运作，下面详细介绍建立复制的完整流程。从图中可以看出复制过程大致分为6个过程：

![tongyi-mermaid-2026-01-27-174545](./assets/tongyi-mermaid-2026-01-27-174545.png)

![image-20251126130147411](images/image-20251126130147411.png)

##### **1）保存主节点(master)信息。**

执行slaveof后从节点只保存主节点的地址信息便直接返回，这时建立复制流程还没有开始，在从节点6380执行info replication可以看到如下信息：

```
# 立即查看状态（连接尚未建立）
master_host:127.0.0.1
master_port:6379
master_link_status:down # 关键：状态为down
```

从统计信息可以看出，主节点的ip和port被保存下来，但是主节点的连接状态(master_link_status)是下线状态。执行slaveof后Redis会打印如下日志：

```
SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=65 addr=127.0.0.1:58090
    fd=5 name= age=11 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free= 
    32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
```

通过该日志可以帮助运维人员定位发送slaveof命令的客户端，方便追踪和发现问题。

关键配置：

```
repl-timeout 60          # 连接/同步超时（秒），默认60
repl-ping-replica-period 10  # 从节点向主节点PING间隔（秒）
```

故障排查：

```
# 查看断连时长
redis-cli -p 6380 INFO replication | grep master_link_down_since_seconds

# 检查网络连通性
telnet 127.0.0.1 6379
```

##### 2）建立网络连接（重试机制）

**从节点(slave)内部通过每秒运行的定时任务维护复制相关逻辑，当定时任务发现存在新的主节点后，会尝试与该节点建立网络连接**

![image-20251126130304394](images/image-20251126130304394.png)

从节点会建立一个socket套接字，例如图6-8中从节点建立了一个端口为24555的套接字，专门用于接受主节点发送的复制命令。从节点连接成功后打印如下日志：

```
# 从节点日志（连接成功）
* Connecting to MASTER 127.0.0.1:6379
* MASTER <-> SLAVE sync started
```

如果从节点无法建立连接，定时任务会无限重试直到连接成功或者执行slaveof no one取消复制。关于连接失败，可以在从节点执行info replication查看master_link_down_since_seconds指标，它会记录与主节点连接失败的系统时间。从节点连接主节点失败时也会每秒打印如下日志，方便运维人员发现问题：

```
# 连接失败日志（每秒重试）
# Error condition on socket for SYNC: {socket_error_reason}
```

##### **3)发送ping命令**（健康探测）

连接建立成功后从节点发送ping请求进行首次通信，ping请求主要目的如下：

- 检测主从之间网络套接字是否可用。

- 检测主节点当前是否可接受处理命令。（避免连接到阻塞中的主节点）

- **触发主节点记录从节点信息**（为后续PSYNC做准备）


如果发送ping命令后，从节点没有收到主节点的pong回复或者超时，比如网络超时或者主节点正在阻塞无法响应命令，从节点会断开复制连接，下次定时任务会发起重连。

**超时控制**：`repl-timeout` 同时控制PING响应超时（默认60秒）

![image-20251126130543478](images/image-20251126130543478.png)

![image-20251126130602228](images/image-20251126130602228.png)

从节点发送的ping命令成功返回，Redis打印如下日志，并继续后续复制流程：

```
# 从节点日志（成功）
Master replied to PING, replication can continue…

# 失败场景
# 主节点阻塞（如DEL BigKey）→ 无PONG → 从节点断连重试
```

##### **4)权限验证（安全屏障）**

如果主节点设置了requirepass参数，则需要密码验证，从节点必须配置masterauth参数保证与主节点相同的密码才能通过验证；如果验证失败复制将终止，从节点重新发起复制流程。

```
# 主节点配置
requirepass "Master@2024!"

# 从节点配置（必须匹配）
masterauth "Master@2024!"
```

- **认证流程**：
  从节点发送 `AUTH <masterauth>` → 主节点验证 → 返回 `+OK` 或 `-NOAUTH`

- **失败后果**：

  ```
  # 从节点日志
  NOAUTH Authentication failed
  * Reconnecting to MASTER 127.0.0.1:6379 after failure
  ```

##### **5)同步数据集（PSYNC2协议核心）**

主从复制连接正常通信后，对于首次建立复制的场景，主节点会把持有的数据全部发送给从节点，这部分操作是耗时最长的步骤。Redis在2.8版本以后采用新复制命令psync进行数据同步，原来的sync命令依然支持，保证新旧版本的兼容性。新版同步划分两种情况：全量同步和部分同步，下一节将重点介绍。

##### **6)命令持续复制（异步追平）**

当主节点把当前的数据同步给从节点后，便完成了复制的建立流程。接下来主节点会持续地把写命令发送给从节点，保证主从数据一致性。

```
# 主节点写入
127.0.0.1:6379> SET user:1001 "Alice"
OK

# 从节点自动同步（无需手动操作）
127.0.0.1:6380> GET user:1001
"Alice"
```

- **底层机制**：

  - 主节点将写命令写入 **复制缓冲区（replication buffer）** → 异步发送给从节点（与AOF缓冲区独立，避免相互阻塞）

- **延迟监控**：

  ```
  # 从节点查看延迟（秒）
  redis-cli -p 6380 INFO replication | grep master_last_io_seconds_ago
  # 输出：master_last_io_seconds_ago:0  # 0=实时同步
  ```

#### 数据同步 PSYNC2协议三大核心组件（同步决策基石）

Redis在2.8及以上版本使用psync命令完成主从数据同步，同步过程分为：全量复制和部分复制。

- 全量复制：一般用于初次复制场景，Redis早期支持的复制功能只有全量复制，它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。
- 部分复制：用于处理在主从复制中因网络闪断等原因造成的数据丢失场景，当从节点再次连上主节点后，如果条件允许，主节点会补发丢失数据给从节点。因为补发的数据远远小于全量数据，可以有效避免全量复制的过高开销。

psync命令运行需要以下组件支持：

- 主从节点各自复制偏移量。
- 主节点复制积压缓冲区。
- 主节点运行id。

##### 1.复制偏移量

参与复制的主从节点都会维护自身复制偏移量。主节点(master)在处理完写入命令后，会把命令的字节长度做累加记录，统计信息在info relication中的master_repl_offset指标中：

```
# 主节点视角
127.0.0.1:6379> info replication
# Replication
role:master
…
master_repl_offset:1055130  # 主节点累计写入字节数
```

从节点(slave)每秒钟上报自身的复制偏移量给主节点，因此主节点也会保存从节点的复制偏移量，统计指标如下：

```
127.0.0.1:6379> info replication
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=1055214,lag=1
…
```

从节点在接收到主节点发送的命令后，也会累加记录自身的偏移量。统计信息在info relication中的slave_repl_offset指标中：

```
# 从节点视角
127.0.0.1:6380> info replication
# Replication
role:slave
…
slave_repl_offset:1055214  # 从节点已接收字节数
master_sync_in_progress:0   # 0=同步完成
```

![image-20251126164027135](images/image-20251126164027135.png)

通过对比主从节点的复制偏移量，可以判断主从节点数据是否一致。

可以通过主节点的统计信息，计算出master_repl_offset-slave_offset字节量，判断主从节点复制相差的数据量，根据这个差值判定当前复制的健康度。如果主从之间复制偏移量相差较大，则可能是网络延迟或命令阻塞等原因引起。

- 主从通过偏移量差值判断数据一致性：`lag = master_repl_offset - slave_repl_offset`
- **关键阈值**：`lag > repl-backlog-size` → 触发全量同步

##### 2.复制积压缓冲区

复制积压缓冲区是保存在主节点上的一个固定长度的队列，默认大小为1MB，当主节点有连接的从节点(slave)时被创建，这时主节点(master)响应写命令时，不但会把命令发送给从节点，还会写入复制积压缓冲区

![image-20251126164156695](images/image-20251126164156695.png)

由于缓冲区本质上是先进先出的定长队列，所以能实现保存最近已复制数据的功能，用于部分复制和复制命令丢失的数据补救。复制缓冲区相关统计信息保存在主节点的info replication中：

```
127.0.0.1:6379> info replication
# Replication
role:master
…
repl_backlog_active:1                   // 开启复制缓冲区
repl_backlog_size:1048576                       // 缓冲区最大长度
repl_backlog_first_byte_offset:7479     // 起始偏移量，计算当前缓冲区可用范围
repl_backlog_histlen:1048576            // 已保存数据的有效长度。当前有效数据长度
```

根据统计指标，可算出复制积压缓冲区内的可用偏移量范围：[repl_backlog_first_byte_offset，repl_backlog_first_byte_offset+repl_backlog_histlen]。

*从节点offset在此范围内 → 可触发部分复制*

##### 3.主节点运行ID

每个Redis节点启动后都会动态分配一个40位的十六进制字符串作为运行ID。运行ID的主要作用是用来唯一识别Redis节点，比如从节点保存主节点的运行ID识别自己正在复制的是哪个主节点。如果只使用ip+port的方式识别主节点，那么主节点重启变更了整体数据集（如替换RDB/AOF文件），从节点再基于偏移量复制数据将是不安全的，因此当运行ID变化后从节点将做全量复制。可以运行info server命令查看当前节点的运行ID：

```
127.0.0.1:6379> info server
# Server
redis_version:3.0.7
…
run_id:545f7c76183d0798a327591395b030000ee6def9
```

需要注意的是Redis关闭再启动后，运行ID会随之改变，例如执行如下命令：

```
# redis-cli -p 6379 info server | grep run_id
run_id:545f7c76183d0798a327591395b030000ee6def9
# redis-cli -p shutdown
# redis-server redis-6379.conf
# redis-cli -p 6379 info server | grep run_id
run_id:2b2ec5f49f752f35c2b2da4d05775b5b3aaa57ca
```

如何在不改变运行ID的情况下重启呢？

当需要调优一些内存相关配置，例如：hash-max-ziplist-value等，这些配置需要Redis重新加载才能优化已存在的数据，这时可以使用debug reload命令重新加载RDB并保持运行ID不变，从而有效避免不必要的全量复制。命令如下：

```
# redis-cli -p 6379 info server | grep run_id
run_id:2b2ec5f49f752f35c2b2da4d05775b5b3aaa57ca
# redis-cli debug reload
OK
# redis-cli -p 6379 info server | grep run_id
run_id:2b2ec5f49f752f35c2b2da4d05775b5b3aaa57ca
```

debug reload命令会阻塞当前Redis节点主线程，阻塞期间会生成本地RDB快照并清空数据之后再加载RDB文件。因此对于大数据量的主节点和无法容忍阻塞的应用场景，谨慎使用。

##### 4.psync命令

从节点使用psync命令完成部分复制和全量复制功能，命令格式：psync{runId}{offset}，参数含义如下：

- runId：从节点所复制主节点的运行id。
- offset：当前从节点已复制的数据偏移量。

psync命令运行流程如图

![image-20251126164553843](images/image-20251126164553843.png)

流程说明：

- 1)从节点(slave)发送psync命令给主节点，参数runId是当前从节点保存的主节点运行ID，如果没有则默认值为，参数offset是当前从节点保存的复制偏移量，如果是第一次参与复制则默认值为-1。

- 2)主节点(master)根据psync参数和自身数据情况决定响应结果：
  - 如果回复+FULLRESYNC{runId}{offset}，那么从节点将触发全量复制流程。
  - 如果回复+CONTINUE，从节点将触发部分复制流程。
  - 如果回复+ERR，说明主节点版本低于Redis2.8，无法识别psync命令，从节点将发送旧版的sync命令触发全量复制流程。

#### 全量复制

全量复制是Redis最早支持的复制方式，也是主从第一次建立复制时必须经历的阶段。触发全量复制的命令是sync和psync，它们的的对应版本redis<2.8是用sync，redis>=2.8是用psync

##### 流程说明：

1)发送psync命令进行数据同步，由于是第一次进行复制，从节点没有复制偏移量和主节点的运行ID，所以发送psync-1。

2)主节点根据psync-1解析出当前为全量复制，回复+FULLRESYNC响应。

3)从节点接收主节点的响应数据保存运行ID和偏移量offset，执行到当前步骤时从节点打印如下日志：

```
Partial resynchronization not possible (no cached master)
Full resync from master: 92d1cb14ff7ba97816216f7beb839efe036775b2:216789
```

![image-20251126180929111](images/image-20251126180929111.png)

4)主节点执行bgsave保存RDB文件到本地，bgsave操作细节和开销见5.1节。主节点bgsave相关日志如下：

```
M * Full resync requested by slave 127.0.0.1:6380
M * Starting BGSAVE for SYNC with target: disk
C * Background saving started by pid 32618
C * RDB: 0 MB of memory used by copy-on-write
M * Background saving terminated with success
```

##### 6步深度拆解（含缓冲区交互）

| 步骤              | 主节点动作                                                   | 从节点动作                                         | 关键日志                                                     | 风险点                       |
| ----------------- | ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
| 1. PSYNC请求      | 接收`PSYNC ? -1`                                             | 发送首次同步请求                                   | `Full resync requested by slave`                             | -                            |
| 2. FULLRESYNC响应 | 返回`+FULLRESYNC <runid> <offset>`                           | 保存runid/offset                                   | `Full resync from master: abc123:216789`                     | Run ID变化导致全量           |
| 3. BGSAVE生成RDB  | Fork子进程 → 生成RDB • 新命令写入复制缓冲区(per replica) • 同时写入积压缓冲区(global) | 等待RDB传输                                        | `Starting BGSAVE for SYNC` `RDB: X MB of memory used by copy-on-write` | COW内存暴涨 BigKey导致fork慢 |
| 4. 传输RDB        | 通过socket发送RDB • 无盘复制：直传socket • 传统模式：先写磁盘再传输 | 接收RDB文件                                        | `Background saving terminated with success`                  | 网络带宽打满                 |
| 5. 加载RDB        | 持续写入复制缓冲区                                           | 清空旧数据 → 加载RDB • 期间`loading:1`（服务阻塞） | `Loading RDB...`                                             | 从节点服务中断               |
| 6. 追平增量       | 发送复制缓冲区中所有命令                                     | 执行增量命令 → 追平主节点                          | `MASTER <-> SLAVE sync: Finished`                            | 缓冲区溢出导致重连           |

- 双缓冲区协同：
  - **复制缓冲区**（Replication Buffer）：每个从节点独立，RDB生成期间暂存新命令（由`client-output-buffer-limit replica`控制）
  - **积压缓冲区**（Replication Backlog）：全局环形缓冲区，用于后续部分复制



##### 全量复制四大致命风险（实测数据）

| 风险点     | 100MB实例实测            | 根本原因                         | 解决方案                                |
| ---------- | ------------------------ | -------------------------------- | --------------------------------------- |
| Fork阻塞   | 耗时28秒（普通key 0.5s） | COW内存复制（BigKey加剧）        | `repl-diskless-sync yes` + 避免高峰操作 |
| 网络打满   | 1Gbps链路占满10秒        | RDB文件传输                      | 限流 + 业务低峰期                       |
| 从节点阻塞 | 服务不可用15秒           | RDB加载期间`loading:1`           | 监控加载状态 + 渐进式删除               |
| 缓冲区溢出 | 从节点被踢出             | `client-output-buffer-limit`超限 | 调大缓冲区：`512mb 1gb 60`              |

#####  全量复制优化配置（生产必备）

```
# 主节点 
repl-diskless-sync yes          # 无盘复制（避免磁盘IO瓶颈） repl-diskless-sync-delay 5      # 等待多从节点连接（秒） 
repl-timeout 300                # 大实例同步延长超时 
client-output-buffer-limit replica 512mb 1gb 60  # 防复制缓冲区溢出 

# 从节点 
repl-timeout 300 
replica-serve-stale-data yes    # 主挂时仍提供旧数据（按需）
```



#### 部分复制

部分复制主要是Redis针对全量复制的过高开销做出的一种优化措施，使用psync{runId}{offset}命令实现。当从节点(slave)正在复制主节点(master)时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向主节点要求补发丢失的命令数据，如果主节点的复制积压缓冲区内存在这部分数据则直接发送给从节点，这样就可以保持主从节点复制的一致性。补发的这部分数据一般远远小于全量数据，所以开销很小。

![tongyi-mermaid-2026-01-27-194804](./assets/tongyi-mermaid-2026-01-27-194804.png)



![image-20251126182402815](images/image-20251126182402815.png)

##### 流程说明

1）当主从节点之间网络出现中断时，如果超过repl-timeout时间，主节点会认为从节点故障并中断复制连接，打印如下日志：

```
M # Disconnecting timedout slave: 127.0.0.1:6380
M # Connection with slave 127.0.0.1:6380 lost.
```

如果此时从节点没有宕机，也会打印与主节点连接丢失日志：

```
S # Connection with master lost.
S * Caching the disconnected master state.
```

2）主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内部存在的复制积压缓冲区，依然可以保存最近一段时间的写命令数据，默认最大缓存1MB。

3）当主从节点网络恢复后，从节点会再次连上主节点，打印如下日志：

```
S * Connecting to MASTER 127.0.0.1:6379
S * MASTER <-> SLAVE sync started
S * Non blocking connect for SYNC fired the event.
S * Master replied to PING, replication can continue…
```

4）当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作。该行为对应从节点日志如下：

```
S * Trying a partial resynchronization (request 2b2ec5f49f752f35c2b2da4d05775b5
    b3aaa57ca:49768480).
```

5）主节点接到psync命令后首先核对参数runId是否与自身一致，如果一致，说明之前复制的是当前主节点；之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+CONTINUE响应，表示可以进行部分复制。从节点接到回复后打印如下日志：

```
S * Successful partial resynchronization with master.
S * MASTER <-> SLAVE sync: Master accepted a Partial Resynchronization.
```

6）主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。发送的数据量可以在主节点的日志获取，如下所示：

```
M * Slave 127.0.0.1:6380 asks for synchronization
M * Partial resynchronization request from 127.0.0.1:6380 accepted. Sending 78 
    bytes of backlog starting from offset 49769216.
```

从日志中可以发现这次部分复制只同步了78字节，传递的数据远远小于全量数据。

##### 6步深度拆解（含缓冲区交互）

| 步骤          | 主节点动作               | 从节点动作                   | 关键日志                                                     | 技术要点                                        |
| ------------- | ------------------------ | ---------------------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| 1. 连接中断   | 超时后断开连接           | 记录断连状态                 | `Connection with slave lost` `Caching the disconnected master state` | `repl-timeout`控制超时                          |
| 2. 持续写入   | 新命令写入积压缓冲区     | 无                           | `repl_backlog_histlen`增长                                   | 环形缓冲区覆盖旧数据                            |
| 3. 重连恢复   | 接受新连接               | 发送`PSYNC <runid> <offset>` | `Trying a partial resynchronization`                         | 从节点保存runid/offset                          |
| 4. 主节点决策 | 校验runid + offset范围   | 等待响应                     | `Partial resynchronization request accepted`                 | 核心：offset ∈ [first_byte, first_byte+histlen] |
| 5. 补发命令   | 从积压缓冲区提取缺失命令 | 接收命令                     | `Sending 78 bytes of backlog starting from offset 49769216`  | 仅传输缺失部分                                  |
| 6. 持续同步   | 发送新命令               | 执行命令 → 追平              | `Successful partial resynchronization`                       | 进入正常复制状态                                |

- 积压缓冲区科学计算

  ```
  repl-backlog-size ≥ (主节点每秒写入字节数 × 最大容忍断连时间) × 1.5
  ```

  - **示例**：500KB/s × 60s × 1.5 = **45MB** → 配置 **64MB**

  - **监控验证**：

    ```
    redis-cli INFO replication | awk -F: '
      /repl_backlog_histlen/ {h= $ 2}
      /repl_backlog_size/ {s= $ 2; printf "缓冲区使用率: %.1f%%\n", h/s*100}
    '
    ```

- 78字节真相解析

  ```
  M * Sending 78 bytes of backlog starting from offset 49769216.
  ```

  - 组成：

    - 70字节：缺失命令原始数据（二进制协议格式）
    - 8字节：协议控制字符（`\r\n`等）

  - **对比全量**：100MB实例 → 节省 **1,282,051倍** 流量！

  - **验证方法**：

    ```
    # 检查同步前后偏移量差值
    BEFORE= $ (redis-cli -p 6380 INFO replication | grep slave_repl_offset | cut -d: -f2)
    # 等待同步完成...
    AFTER= $ (redis-cli -p 6380 INFO replication | grep slave_repl_offset | cut -d: -f2)
    echo "同步数据量:  $ ((AFTER - BEFORE)) 字节"  # 应≈78
    ```

#### 心跳

主从节点在建立复制后，它们之间维护着长连接并彼此发送心跳命令

![image-20251126182825663](images/image-20251126182825663.png)



主从心跳判断机制：

- 1)主从节点彼此都有心跳检测机制，各自模拟成对方的客户端进行通信，通过client list命令查看复制相关客户端信息，主节点的连接状态为flags=M，从节点连接状态为flags=S。
- 2)主节点默认每隔10秒对从节点发送ping命令，判断从节点的存活性和连接状态。可通过参数repl-ping-slave-period控制发送频率。
- 3)从节点在主线程中每隔1秒发送replconf ack{offset}命令，给主节点上报自身当前的复制偏移量。replconf命令主要作用如下：
  - 实时监测主从节点网络状态。
  - 上报自身复制偏移量，检查复制数据是否丢失，如果从节点数据丢失，再从主节点的复制缓冲区中拉取丢失数据。
  - 实现保证从节点的数量和延迟性功能，通过min-slaves-to-write、min-slaves-max-lag参数配置定义。

主节点根据replconf命令判断从节点超时时间，体现在info replication统计中的lag信息中，lag表示与从节点最后一次通信延迟的秒数，正常延迟应该在0和1之间。如果超过repl-timeout配置的值（默认60秒），则判定从节点下线并断开复制客户端连接。即使主节点判定从节点下线后，如果从节点重新恢复，心跳检测会继续进行。

为了降低主从延迟，一般把Redis主从节点部署在相同的机房/同城机房，避免网络延迟和网络分区造成的心跳中断等情况。

#### 异步复制

主节点不但负责数据读写，还负责把写命令同步给从节点。写命令的发送过程是异步完成，也就是说主节点自身处理完写命令后直接返回给客户端，并不等待从节点复制完成

![image-20251126183204377](images/image-20251126183204377.png)

主节点复制流程：

1)主节点6379接收处理命令。

2)命令处理完之后返回响应结果。

3)对于修改命令异步发送给6380从节点，从节点在主线程中执行复制的命令。

由于主从复制过程是异步的，就会造成从节点的数据相对主节点存在延迟。具体延迟多少字节，我们可以在主节点执行info replication命令查看相关指标获得。如下：

```
slave0:ip=127.0.0.1,port=6380,state=online,offset=841,lag=1
master_repl_offset:841
```

在统计信息中可以看到从节点slave0信息，分别记录了从节点的ip和port，从节点的状态，offset表示当前从节点的复制偏移量，master_repl_offset表示当前主节点的复制偏移量，两者的差值就是当前从节点复制延迟量。Redis的复制速度取决于主从之间网络环境，repl-disable-tcp-nodelay，命令处理速度等。正常情况下，延迟在1秒以内。

### 缺点

- 复制延时，信号衰减：由于所有的写操作都是先在Master上操作,然后同步更新到Slave上,所以从Master同步到Slave机器有一定的延迟,当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。
- master挂了如何办？默认情况下，不会在slave节点中自动重选一个master。那每次都要干预？哨兵

### 开发与运维中的问题

#### 读写分离

对于读占比较高的场景，可以通过把一部分读流量分摊到从节点(slave)来减轻主节点(master)压力，同时需要注意永远只对主节点执行写操作

![image-20251126183512741](images/image-20251126183512741-1764153313521-3.png)

当使用从节点响应读请求时，业务端可能会遇到如下问题：

- 复制数据延迟。
- 读到过期数据。
- 从节点故障。

##### 1.数据延迟

Redis复制数据的延迟由于异步复制特性是无法避免的，延迟取决于网络带宽和命令阻塞情况，比如刚在主节点写入数据后立刻在从节点上读取可能获取不到。需要业务场景允许短时间内的数据延迟。对于无法容忍大量延迟场景，可以编写外部监控程序监听主从节点的复制偏移量，当延迟较大时触发报警或者通知客户端避免读取延迟过高的从节点

![image-20251126183635643](images/image-20251126183635643-1764153396344-5.png)

说明如下：

1)监控程序(monitor)定期检查主从节点的偏移量，主节点偏移量在info replication的master_repl_offset指标记录，从节点偏移量可以查询主节点的slave0字段的offset指标，它们的差值就是主从节点延迟的字节量。

2)当延迟字节量过高时，比如超过10MB。监控程序触发报警并通知客户端从节点延迟过高。可以采用Zookeeper的监听回调机制实现客户端通知。

3)客户端接到具体的从节点高延迟通知后，修改读命令路由到其他从节点或主节点上。当延迟恢复后，再次通知客户端，恢复从节点的读命令请求。这种方案的成本比较高，需要单独修改适配Redis的客户端类库。如果涉及多种语言成本将会扩大。客户端逻辑需要识别出读写请求并自动路由，还需要维护故障和恢复的通知。采用此方案视具体的业务而定，如果允许不一致性或对延迟不敏感的业务可以忽略，也可以采用Redis集群方案做水平扩展。

##### 2.读到过期数据

当主节点存储大量设置超时的数据时，如缓存数据，Redis内部需要维护过期数据删除策略，删除策略主要有两种：**惰性删除和定时删除**

惰性删除：主节点每次处理读取命令时，都会检查键是否超时，如果超时则执行del命令删除键对象，之后del命令也会异步发送给从节点。需要注意的是为了保证复制的一致性，从节点自身永远不会主动删除超时数据

![image-20251126183827753](images/image-20251126183827753.png)

定时删除：Redis主节点在内部定时任务会循环采样一定数量的键，当发现采样的键过期时执行del命令，之后再同步给从节点，

![image-20251126183851573](images/image-20251126183851573.png)

如果此时数据大量超时，主节点采样速度跟不上过期速度且主节点没有读取过期键的操作，那么从节点将无法收到del命令。这时在从节点上可以读取到已经超时的数据。Redis在3.2版本解决了这个问题，从节点读取数据之前会检查键的过期时间来决定是否返回数据，可以升级到3.2版本来规避这个问题。

##### 3.从节点故障问题

对于从节点的故障问题，需要在客户端维护可用从节点列表，当从节点故障时立刻切换到其他从节点或主节点上。这个过程类似上文提到的针对延迟过高的监控处理，需要开发人员改造客户端类库。

综上所出，使用Redis做读写分离存在一定的成本。Redis本身的性能非常高，开发人员在使用额外的从节点提升读性能之前，尽量在主节点上做充分优化，比如解决慢查询，持久化阻塞，合理应用数据结构等，当主节点优化空间不大时再考虑扩展。笔者建议大家在做读写分离之前，可以考虑使用Redis Cluster等分布式解决方案，这样不止扩展了读性能还可以扩展写性能和可支撑数据规模，并且一致性和故障转移也可以得到保证，对于客户端的维护逻辑也相对容易。

#### 主从配置不一致

主从配置不一致是一个容易忽视的问题。对于有些配置主从之间是可以不一致，比如：主节点关闭AOF在从节点开启。但对于内存相关的配置必须要一致，比如maxmemory，hash-max-ziplist-entries等参数。当配置的maxmemory从节点小于主节点，如果复制的数据量超过从节点maxmemory时，它会根据maxmemory-policy策略进行内存溢出控制，此时从节点数据已经丢失，但主从复制流程依然正常进行，复制偏移量也正常。修复这类问题也只能手动进行全量复制。当压缩列表相关参数不一致时，虽然主从节点存储的数据一致但实际内存占用情况差异会比较大。更多压缩列表细节见8.3节“内存管理”。

#### 规避全量复制

全量复制是一个非常消耗资源的操作，前面做了具体说明。因此如何规避全量复制是需要重点关注的运维点。下面我们对需要进行全量复制的场景逐个分析：

- 第一次建立复制：由于是第一次建立复制，从节点不包含任何主节点数据，因此必须进行全量复制才能完成数据同步。对于这种情况全量复制无法避免。当对数据量较大且流量较高的主节点添加从节点时，建议在低峰时进行操作，或者尽量规避使用大数据量的Redis节点。
- 节点运行ID不匹配：当主从复制关系建立后，从节点会保存主节点的运行ID，如果此时主节点因故障重启，那么它的运行ID会改变，从节点发现主节点运行ID不匹配时，会认为自己复制的是一个新的主节点从而进行全量复制。对于这种情况应该从架构上规避，比如提供故障转移功能。当主节点发生故障后，手动提升从节点为主节点或者采用支持自动故障转移的哨兵或集群方案。
- 复制积压缓冲区不足：当主从节点网络中断后，从节点再次连上主节点时会发送psync{offset}{runId}命令请求部分复制，如果请求的偏移量不在主节点的积压缓冲区内，则无法提供给从节点数据，因此部分复制会退化为全量复制。针对这种情况需要根据网络中断时长，写命令数据量分析出合理的积压缓冲区大小。网络中断一般有闪断、机房割接、网络分区等情况。这时网络中断的时长一般在分钟级(net_break_time)。写命令数据量可以统计高峰期主节点每秒info replication的master_repl_offset差值获取(write_size_per_minute)。积压缓冲区默认为1MB，对于大流量场景显然不够，这时需要增大积压缓冲区，保证repl_backlog_size>net_break_time*write_size_per_minute，从而避免因复制积压缓冲区不足造成的全量复制。

#### 规避复制风暴

复制风暴是指大量从节点对同一主节点或者对同一台机器的多个主节点短时间内发起全量复制的过程。复制风暴对发起复制的主节点或者机器造成大量开销，导致CPU、内存、带宽消耗。因此我们应该分析出复制风暴发生的场景，提前采用合理的方式规避。规避方式有如下几个。

##### 1.单主节点复制风暴

单主节点复制风暴一般发生在主节点挂载多个从节点的场景。当主节点重启恢复后，从节点会发起全量复制流程，这时主节点就会为从节点创建RDB快照，如果在快照创建完毕之前，有多个从节点都尝试与主节点进行全量同步，那么其他从节点将共享这份RDB快照。这点Redis做了优化，有效避免了创建多个快照。但是，同时向多个从节点发送RDB快照，可能使主节点的网络带宽消耗严重，造成主节点的延迟变大，极端情况会发生主从节点连接断开，导致复制失败。解决方案首先可以减少主节点(master)挂载从节点(slave)的数量，或者采用树状复制结构，加入中间层从节点用来保护主节点

![image-20251126184415732](images/image-20251126184415732.png)

从节点采用树状树非常有用，网络开销交给位于中间层的从节点，而不必消耗顶层的主节点。但是这种树状结构也带来了运维的复杂性，增加了手动和自动处理故障转移的难度。

##### 2.单机器复制风暴

由于Redis的单线程架构，通常单台机器会部署多个Redis实例。当一台机器(machine)上同时部署多个主节点(master)时

![image-20251126184455546](images/image-20251126184455546-1764153896153-7.png)

如果这台机器出现故障或网络长时间中断，当它重启恢复后，会有大量从节点(slave)针对这台机器的主节点进行全量复制，会造成当前机器网络带宽耗尽。

如何避免？方法如下：

- 应该把主节点尽量分散在多台机器上，避免在单台机器上部署过多的主节点。
- 当主节点所在机器故障后提供故障转移机制，避免机器恢复后进行密集的全量复制。

## Redis的噩梦：阻塞

Redis是典型的单线程架构，所有的读写操作都是在一条主线程中完成的。当Redis用于高并发场景时，这条线程就变成了它的生命线。如果出现阻塞，哪怕是很短时间，对于我们的应用来说都是噩梦。导致阻塞问题的场景大致分为内在原因和外在原因：

- 内在原因包括：不合理地使用API或数据结构、CPU饱和、持久化阻塞等。
- 外在原因包括：CPU竞争、内存交换、网络问题等。

### 发现阻塞

当Redis阻塞时，线上应用服务应该最先感知到，这时应用方会收到大量Redis超时异常，比如Jedis客户端会抛出JedisConnectionException异常。常见的做法是在应用方加入异常统计并通过邮件/短信/微信报警，以便及时发现通知问题。开发人员需要处理如何统计异常以及触发报警的时机。何时触发报警一般根据应用的并发量决定，如1分钟内超过10个异常触发报警。在实现异常统计时要注意，由于Redis调用API会分散在项目的多个地方，每个地方都监听异常并加入监控代码必然难以维护。这时可以借助于日志系统，如Java语言可以使用logback或log4j。当异常发生时，异常信息最终会被日志系统收集到Appender（输出目的地），默认的Appender一般是具体的日志文件，开发人员可以自定义一个Appender，用于专门统计异常和触发报警逻辑

![image-20251126221243082](images/image-20251126221243082.png)

以Java的logback为例，实现代码如下：

```
public class Redis Appender extends AppenderBase<ILoggingEvent> {
// 使用guava的AtomicLongMap,用于并发计数
    public static final AtomicLongMap<String> ATOMIC_LONG_MAP = AtomicLongMap.create();
    static {
// 自定义Appender加入到logback的rootLogger中
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        Logger rootLogger = loggerContext.getLogger(Logger.ROOT_LOGGER_NAME);
        ErrorStatisticsAppender errorStatisticsAppender = new ErrorStatisticsAppender();
        errorStatisticsAppender.setContext(loggerContext);
        errorStatisticsAppender.start();
        rootLogger.addAppender(errorStatisticsAppender);
}
// 重写接收日志事件方法
    protected void append(ILoggingEvent event) {
// 只监控error级别日志
        if (event.getLevel() == Level.ERROR) {
            IThrowableProxy throwableProxy = event.getThrowableProxy();
// 确认抛出异常
            if (throwableProxy != null) {
// 以每分钟为key，记录每分钟异常数量
                String key = DateUtil.formatDate(new Date(), "yyyyMMddHHmm");
                long errorCount = ATOMIC_LONG_MAP.incrementAndGet(key);
                if (errorCount > 10) {
// 超过10次触发报警代码
                }
// 清理历史计数统计，防止极端情况下内存泄露
                for (String oldKey : ATOMIC_LONG_MAP.asMap().keySet()) {
                    if (!StringUtils.equals(key, oldKey)) {
                        ATOMIC_LONG_MAP.remove(oldKey);
                    }
                }
            }
        }
    }
```

借助日志系统统计异常的前提是，需要项目必须使用日志API进行异常统一输出，比如所有的异常都通过logger.error打印，这应该作为开发规范推广。其他编程语言也可以采用类似的日志系统实现异常统计报警。应用方加入异常监控之后还存在一个问题，当开发人员接到异常报警后，通常会去线上服务器查看错误日志细节。这时如果应用操作的是多个Redis节点（比如使用Redis集群），如何决定是哪一个节点超时还是所有的节点都有超时呢？这是线上很常见的需求，但绝大多数的客户端类库并没有在异常信息中打印ip和port信息，导致无法快速定位是哪个Redis节点超时。不过修改Redis客户端成本很低，比如Jedis只需要修改Connection类下的connect、sendCommand、readProtocolWithCheckingBroken方法专门捕获连接，发送命令，协议读取事件的异常。由于客户端类库都会保存ip和port信息，当异常发生时很容易打印出对应节点的ip和port，辅助我们快速定位问题节点。

除了在应用方加入统计报警逻辑之外，还可以借助Redis监控系统发现阻塞问题，当监控系统检测到Redis运行期的一些关键指标出现不正常时会触发报警。Redis相关的监控系统开源的方案有很多，一些公司内部也会自己开发监控系统。一个可靠的Redis监控系统首先需要做到对关键指标全方位监控和异常识别，辅助开发运维人员发现定位问题。如果Redis服务没有引入监控系统作辅助支撑，对于线上的服务是非常不负责任和危险的。这里推荐笔者团队开源的CacheCloud系统，它内部的统计监控模块能够很好地辅助工程师发现定位问题。监控系统所监控的关键指标有很多，如命令耗时、慢查询、持久化阻塞、连接拒绝、CPU/内存/网络/磁盘使用过载等。当出现阻塞时如果相关人员不能深刻理解这些关键指标的含义和背后的原理，会严重影响解决问题的速度。后面的内容将围绕引起Redis阻塞的原因做重点说明。

### 内在原因

定位到具体的Redis节点异常后，首先应该排查是否是Redis自身原因导致，围绕以下几个方面排查：

- API或数据结构使用不合理。
- CPU饱和的问题。
- 持久化相关的阻塞。

#### API或数据结构使用不合理

通常Redis执行命令速度非常快，但也存在例外，**如对一个包含上万个元素的hash结构执行hgetall操作，由于数据量比较大且命令算法复杂度是O(n)，这条命令执行速度必然很慢。**这个问题就是典型的不合理使用API和数据结构。**对于高并发的场景我们应该尽量避免在大对象上执行算法复杂度超过O(n)的命令**

##### 1.如何发现慢查询

Redis原生提供慢查询统计功能，执行slowlog get{n}命令可以获取最近的n条慢查询命令，默认对于执行超过10毫秒的命令都会记录到一个定长队列中，线上实例建议设置为1毫秒便于及时发现毫秒级以上的命令。如果命令执行时间在毫秒级，则实例实际OPS只有1000左右。慢查询队列长度默认128，可适当调大。慢查询更多细节见第3章。慢查询本身只记录了命令执行时间，不包括数据网络传输时间和命令排队时间，因此客户端发生阻塞异常后，可能不是当前命令缓慢，而是在等待其他命令执行。需要重点比对异常和慢查询发生的时间点，确认是否有慢查询造成的命令阻塞排队。

发现慢查询后，开发人员需要作出及时调整。可以按照以下两个方向去调整：

1)修改为低算法度的命令，如hgetall改为hmget等，禁用keys、sort等命令。

2)调整大对象：缩减大对象数据或把大对象拆分为多个小对象，防止一次命令操作过多的数据。大对象拆分过程需要视具体的业务决定，如用户好友集合存储在Redis中，有些热点用户会关注大量好友，这时可以按时间或其他维度拆分到多个集合中。

##### 2.如何发现大对象

Redis本身提供发现大对象的工具，对应命令：redis-cli-h{ip}-p{port}bigkeys。内部原理采用分段进行scan操作，把历史扫描过的最大对象统计出来便于分析优化，运行效果如下：

```
# redis-cli --bigkeys
# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type. You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).
[00.00%] Biggest string found so far 'ptc:-571805194744395733' with 17 bytes
[00.00%] Biggest string found so far 'RVF#2570599,1' with 3881 bytes
[00.01%] Biggest hash found so far 'pcl:8752795333786343845' with 208 fields
[00.37%] Biggest string found so far 'RVF#1224557,1' with 3882 bytes
[00.75%] Biggest string found so far 'ptc:2404721392920303995' with 4791 bytes
[04.64%] Biggest string found so far 'pcltm:614' with 5176729 bytes
[08.08%] Biggest string found so far 'pcltm:8561' with 11669889 bytes
[21.08%] Biggest string found so far 'pcltm:8598' with 12300864 bytes
..忽略更多输出…
-------- summary -------
Sampled 3192437 keys in the keyspace!
Total key length in bytes is 78299956 (avg len 24.53)
Biggest string found 'pcltm:121' has 17735928 bytes
Biggest hash found 'pcl:3650040409957394505' has 209 fields
2526878 strings with 954999242 bytes (79.15% of keys, avg size 377.94)
0 lists with 0 items (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
665559 hashs with 19013973 fields (20.85% of keys, avg size 28.57)
0 zsets with 0 members (00.00% of keys, avg size 0.00)
```

根据结果汇总信息能非常方便地获取到大对象的键，以及不同类型数据结构的使用情况。

#### CPU饱和

单线程的Redis处理命令时只能使用一个CPU。而CPU饱和是指Redis把单核CPU使用率跑到接近100%。使用top命令很容易识别出对应Redis进程的CPU使用率。CPU饱和是非常危险的，将导致Redis无法处理更多的命令，严重影响吞吐量和应用方的稳定性。对于这种情况，首先判断当前Redis的并发量是否达到极限，建议使用统计命令redis-cli-h{ip}-p{port}--stat获取当前Redis使用情况，该命令每秒输出一行统计信息，运行效果如下：

```
# redis-cli --stat
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests               connections
3789785    3.20G    507     0       8867955607 (+0)        555894
3789813    3.20G    507     0       8867959511 (+63904)    555894
3789822    3.20G    507     0       8867961602 (+62091)    555894
3789831    3.20G    507     0       8867965049 (+63447)    555894
3789842    3.20G    507     0       8867969520 (+62675)    555894
3789845    3.20G    507     0       8867971943 (+62423)    555894
```

以上输出是一个接近饱和的Redis实例的统计信息，它每秒平均处理6万+的请求。对于这种情况，垂直层面的命令优化很难达到效果，这时就需要做集群化水平扩展来分摊OPS压力。如果只有几百或几千OPS的Redis实例就接近CPU饱和是很不正常的，有可能使用了高算法复杂度的命令。还有一种情况是过度的内存优化，这种情况有些隐蔽，需要我们根据info commandstats统计信息分析出命令不合理开销时间，例如下面的耗时统计：

```
cmdstat_hset:calls=198757512,usec=27021957243,usec_per_call=135.95
```

查看这个统计可以发现一个问题，hset命令算法复杂度只有O(1)但平均耗时却达到135微秒，显然不合理，正常情况耗时应该在10微秒以下。这是因为上面的Redis实例为了追求低内存使用量，过度放宽ziplist使用条件（修改了hash-max-ziplist-entries和hash-max-ziplist-value配置）。进程内的hash对象平均存储着上万个元素，而针对ziplist的操作算法复杂度在O(n)到O(n2)之间。虽然采用ziplist编码后hash结构内存占用会变小，但是操作变得更慢且更消耗CPU。ziplist压缩编码是Redis用来平衡空间和效率的优化手段，不可过度使用。关于ziplist编码细节见第8章的8.3节“内存优化”。

#### 持久化阻塞

对于开启了持久化功能的Redis节点，需要排查是否是持久化导致的阻塞。持久化引起主线程阻塞的操作主要有：fork阻塞、AOF刷盘阻塞、HugePage写操作阻塞。

1.fork阻塞

fork操作发生在RDB和AOF重写时，Redis主线程调用fork操作产生共享内存的子进程，由子进程完成持久化文件重写工作。如果fork操作本身耗时过长，必然会导致主线程的阻塞。...

### 外在原因

排查Redis自身原因引起的阻塞原因之后，如果还没有定位问题，需要排查是否由外部原因引起。围绕以下三个方面进行排查：

- CPU竞争
- 内存交换
- 网络问题

#### CPU竞争

CPU竞争问题如下：

- 进程竞争：Redis是典型的CPU密集型应用，不建议和其他多核CPU密集型服务部署在一起。当其他进程过度消耗CPU时，将严重影响Redis吞吐量。可以通过top、sar等命令定位到CPU消耗的时间点和具体进程，这个问题比较容易发现，需要调整服务之间部署结构。

- 绑定CPU：部署Redis时为了充分利用多核CPU，通常一台机器部署多个实例。常见的一种优化是把Redis进程绑定到CPU上，用于降低CPU频繁上下文切换的开销。这个优化技巧正常情况下没有问题，但是存在例外情况，如图7-2所示。

  ![image-20251127153830747](images/image-20251127153830747.png)

  当Redis父进程创建子进程进行RDB/AOF重写时，如果做了CPU绑定，会与父进程共享使用一个CPU。子进程重写时对单核CPU使用率通常在90%以上，父进程与子进程将产生激烈CPU竞争，极大影响Redis稳定性。因此对于开启了持久化或参与复制的主节点不建议绑定CPU。

#### 内存交换

内存交换(swap)对于Redis来说是非常致命的，Redis保证高性能的一个重要前提是所有的数据在内存中。如果操作系统把Redis使用的部分内存换出到硬盘，由于内存与硬盘读写速度差几个数量级，会导致发生交换后的Redis性能急剧下降。识别Redis内存交换的检查方法如下：

1)查询Redis进程号：

```
# redis-cli -p 6383 info server | grep process_id
process_id:4476
```

2)根据进程号查询内存交换信息：

```
# cat /proc/4476/smaps | grep Swap
Swap: 0 kB
Swap: 0 kB
Swap: 4 kB
Swap: 0 kB
Swap: 0 kB
…..
```

如果交换量都是0KB或者个别的是4KB，则是正常现象，说明Redis进程内存没有被交换。预防内存交换的方法有：

- 保证机器充足的可用内存。
- 确保所有Redis实例设置最大可用内存(maxmemory)，防止极端情况下Redis内存不可控的增长。
- 降低系统使用swap优先级，如echo10>/proc/sys/vm/swappiness，具体细节见12.1节“Linux配置优化”。

#### 网络问题

网络问题经常是引起Redis阻塞的问题点。常见的网络问题主要有：连接拒绝、网络延迟、网卡软中断等。

##### 1.连接拒绝

当出现网络闪断或者连接数溢出时，客户端会出现无法连接Redis的情况。我们需要区分这三种情况：网络闪断、Redis连接拒绝、连接溢出。

第一种情况：网络闪断。一般发生在网络割接或者带宽耗尽的情况，对于网络闪断的识别比较困难，常见的做法可以通过sar-n DEV查看本机历史流量是否正常，或者借助外部系统监控工具（如Ganglia）进行识别。具体问题定位需要更上层的运维支持，对于重要的Redis服务需要充分考虑部署架构的优化，尽量避免客户端与Redis之间异地跨机房调用。

第二种情况：Redis连接拒绝。Redis通过maxclients参数控制客户端最大连接数，默认10000。当Redis连接数大于maxclients时会拒绝新的连接进入，info stats的rejected_connections统计指标记录所有被拒绝连接的数量：

```
# redis-cli -p 6384 info Stats | grep rejected_connections
rejected_connections:0
```

Redis使用多路复用IO模型可支撑大量连接，但是不代表可以无限连接。客户端访问Redis时尽量采用NIO长连接或者连接池的方式。

当Redis用于大量分布式节点访问且生命周期比较短的场景时，如比较典型的在Map/Reduce中使用Redis。因为客户端服务存在频繁启动和销毁的情况且默认Redis不会主动关闭长时间闲置连接或检查关闭无效的TCP连接，因此会导致Redis连接数快速消耗且无法释放的问题。这种场景下建议设置tcp-keepalive和timeout参数让Redis主动检查和关闭无效连接。

第三种情况：连接溢出。这是指操作系统或者Redis客户端在连接时的问题。这个问题的原因比较多，下面就分别介绍两种原因：进程限制、backlog队列溢出。

(1)进程限制

客户端想成功连接上Redis服务需要操作系统和Redis的限制都通过才可以

![image-20251127154419280](images/image-20251127154419280.png)

操作系统一般会对进程使用的资源做限制，其中一项是对进程可打开最大文件数控制，通过ulimit-n查看，通常默认1024。由于Linux系统对TCP连接也定义为一个文件句柄，因此对于支撑大量连接的Redis来说需要增大这个值，如设置ulimit-n65535，防止Too many open files错误。

(2)backlog队列溢出

系统对于特定端口的TCP连接使用backlog队列保存。Redis默认的长度为511，通过tcp-backlog参数设置。如果Redis用于高并发场景为了防止缓慢连接占用，可适当增大这个设置，但必须大于操作系统允许值才能生效。当Redis启动时如果tcp-backlog设置大于系统允许值将以系统值为准，Redis打印如下警告日志：

```
# WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/
    net/core/somaxconn is set to the lower value of 128.
```

系统的backlog默认值为128，使用echo511>/proc/sys/net/core/somaxconn命令进行修改。可以通过netstat-s命令获取因backlog队列溢出造成的连接拒绝统计，如下：

```
# netstat -s | grep overflowed
663 times the listen queue of a socket overflowed
```

如果怀疑是backlog队列溢出，线上可以使用cron定时执行netstat-s|grep overflowed统计，查看是否有持续增长的连接拒绝情况。

##### 2.网络延迟

网络延迟取决于客户端到Redis服务器之间的网络环境。主要包括它们之间的物理拓扑和带宽占用情况。常见的物理拓扑按网络延迟由快到慢可分为：同物理机>同机架>跨机架>同机房>同城机房>异地机房。但它们容灾性正好相反，同物理机容灾性最低而异地机房容灾性最高。Redis提供了测量机器之间网络延迟的工具，在redis-cli-h{host}-p{port}命令后面加入如下参数进行延迟测试：

- --latency：持续进行延迟测试，分别统计：最小值、最大值、平均值、采样次数。
- --latency-history：统计结果同--latency，但默认每15秒完成一行统计，可通过-i参数控制采样时间。
- --latency-dist：使用统计图的形式展示延迟统计，每1秒采样一次。

网络延迟问题经常出现在跨机房的部署结构上，对于机房之间延迟比较严重的场景需要调整拓扑结构，如把客户端和Redis部署在同机房或同城机房等。带宽瓶颈通常出现在以下几个方面：·机器网卡带宽。·机架交换机带宽。·机房之间专线带宽。带宽占用主要根据当时使用率是否达到瓶颈有关，如频繁操作Redis的大对象对于千兆网卡的机器很容易达到网卡瓶颈，因此需要重点监控机器流量，及时发现网卡打满产生的网络延迟或通信中断等情况，而机房专线和交换机带宽一般由上层运维监控支持，通常出现瓶颈的概率较小。

##### 3.网卡软中断

网卡软中断是指由于单个网卡队列只能使用一个CPU，高并发下网卡数据交互都集中在同一个CPU，导致无法充分利用多核CPU的情况。网卡软中断瓶颈一般出现在网络高流量吞吐的场景，如下使用“top+数字1”命令可以很明显看到CPU1的软中断指标(si)过高：

```
# top
Cpu0 : 15.3%us, 0.3%sy, 0.0%ni, 84.4%id, 0.0%wa, 0.0%hi, 0.0%si, 0.0%st
Cpu1 : 16.6%us, 2.0%sy, 0.0%ni, 47.1%id, 3.3%wa, 0.0%hi, 31.0%si, 0.0%st
Cpu2 : 13.3%us, 0.7%sy, 0.0%ni, 86.0%id, 0.0%wa, 0.0%hi, 0.0%si, 0.0%st
Cpu3 : 14.3%us, 1.7%sy, 0.0%ni, 82.4%id, 1.0%wa, 0.0%hi, 0.7%si, 0.0%st
…..
Cpu15 : 10.3%us, 8.0%sy, 0.0%ni, 78.7%id, 1.7%wa, 0.3%hi, 1.0%si, 0.0%st
```

Linux在内核2.6.35以后支持Receive Packet Steering(RPS)，实现了在软件层面模拟硬件的多队列网卡功能。如何配置多CPU分摊软中断已超出本书的范畴，具体配置见Torvalds的GitHub文档：https://github.com/torvalds/linux/blob/master/Documentation/networking/scaling.txt。

### 内存优化

Redis所有的数据都在内存中，而内存又是非常宝贵的资源。如何优化内存的使用一直是Redis用户非常关注的问题。

#### redisObject对象

**Redis存储的所有值对象在内部定义为redisObject结构体**

![image-20251128105531732](images/image-20251128105531732.png)

Redis存储的数据都使用redisObject来封装，包括string、hash、list、set、zset在内的所有数据类型。理解redisObject对内存优化非常有帮助，下面针对每个字段做详细说明：

- type字段：表示当前对象使用的数据类型，Redis主要支持5种数据类型：string、hash、list、set、zset。可以使用type{key}命令查看对象所属类型，type命令返回的是值对象类型，键都是string类型。
- encoding字段：表示Redis内部编码类型，encoding在Redis内部使用，代表当前对象内部采用哪种数据结构实现。理解Redis内部编码方式对于优化内存非常重要，同一个对象采用不同的编码实现内存占用存在明显差异。
- lru字段：记录对象最后一次被访问的时间，当配置了maxmemory和maxmemory-policy=volatile-lru或者allkeys-lru时，用于辅助LRU算法删除键数据。可以使用object idletime{key}命令在不更新lru字段情况下查看当前键的空闲时间。

可以使用scan+object idletime命令批量查询哪些键长时间未被访问，找出长时间不访问的键进行清理，可降低内存占用。

- refcount字段：记录当前对象被引用的次数，用于通过引用次数回收内存，当refcount=0时，可以安全回收当前对象空间。使用object refcount{key}获取当前对象引用。当对象为整数且范围在[0-9999]时，Redis可以使用共享对象的方式来节省内存。具体细节见之后8.3.3节“共享对象池”部分。

- *ptr字段：与对象的数据内容相关，如果是整数，直接存储数据；否则表示指向数据的指针。Redis在3.0之后对值对象是字符串且长度<=39字节的数据，内部编码为embstr类型，字符串sds和redisObject一起分配，从而只要一次内存操作即可。

高并发写入场景中，在条件允许的情况下，建议字符串长度控制在39字节以内，减少创建redisObject内存分配次数，从而提高性能。

#### 缩减键值对象

降低Redis内存使用最直接的方式就是缩减键(key)和值(value)的长度。

- key长度：如在设计键时，在完整描述业务情况下，键值越短越好。如user：{uid}：friends：notify：{fid}可以简化为u：{uid}：fs：nt：{fid}。

- value长度：值对象缩减比较复杂，常见需求是把业务对象序列化成二进制数组放入Redis。首先应该在业务上精简业务对象，去掉不必要的属性避免存储无效数据。其次在序列化工具选择上，应该选择更高效的序列化工具来降低字节数组大小。以Java为例，内置的序列化方式无论从速度还是压缩比都不尽如人意，这时可以选择更高效的序列化工具，如：protostuff、kryo等，图8-7是Java常见序列化工具空间压缩对比

  ![image-20251128105938020](images/image-20251128105938020.png)

  其中java-built-in-serializer表示Java内置序列化方式，更多数据见jvm-serializers项目：https://github.com/eishay/jvm-serializers/wiki，其他语言也有各自对应的高效序列化工具。

  值对象除了存储二进制数据之外，通常还会使用通用格式存储数据比如：json、xml等作为字符串存储在Redis中。这种方式优点是方便调试和跨语言，但是同样的数据相比字节数组所需的空间更大，在内存紧张的情况下，可以使用通用压缩算法压缩json、xml后再存入Redis，从而降低内存占用，例如使用GZIP压缩后的json可降低约60%的空间。

  当频繁压缩解压json等文本数据时，开发人员需要考虑压缩速度和计算开销成本，这里推荐使用Google的Snappy压缩工具，在特定的压缩率情况下效率远远高于GZIP等传统压缩工具，且支持所有主流语言环境。

#### 共享对象池

共享对象池是指Redis内部维护[0-9999]的整数对象池。创建大量的整数类型redisObject存在内存开销，每个redisObject内部结构至少占16字节，甚至超过了整数自身空间消耗。所以Redis内存维护一个[0-9999]的整数对象池，用于节约内存。除了整数值对象，其他类型如list、hash、set、zset内部元素也可以使用整数对象池。因此开发中在满足需求的前提下，尽量使用整数对象以节省内存。整数对象池在Redis中通过变量REDIS_SHARED_INTEGERS定义，不能通过配置修改。可以通过object refcount命令查看对象引用数验证是否启用整数对象池技术，如下：

```
redis> set foo 100
OK
redis> object refcount foo
(integer) 2
redis> set bar 100
OK
redis> object refcount bar
(integer) 3
```

设置键foo等于100时，直接使用共享池内整数对象，因此引用数是2，再设置键bar等于100时，引用数又变为3

![image-20251128110318854](images/image-20251128110318854.png)

使用整数对象池究竟能降低多少内存？让我们通过测试来对比对象池的内存优化效果。否使用整数对象池内存对比

![image-20251128110343722](images/image-20251128110343722.png)

本章所有测试环境都保持一致，信息如下：服务器信息：cpu=Intel-Xeon E5606@2.13GHz memory=32GBRedis版本：Redis server v=3.0.7sha=00000000：0malloc=jemalloc-3.6.0bits=64使用共享对象池后，相同的数据内存使用降低30%以上。可见当数据大量使用[0-9999]的整数时，共享对象池可以节约大量内存。需要注意的是对象池并不是只要存储[0-9999]的整数就可以工作。当设置maxmemory并启用LRU相关淘汰策略如：volatile-lru，allkeys-lru时，Redis禁止使用共享对象池，测试命令如下：

```
redis> set key:1 99
OK              // 设置key:1=99
redis> object refcount key:1
(integer) 2     // 使用了对象共享,引用数为2
redis> config set maxmemory-policy volatile-lru
OK              // 开启LRU淘汰策略
redis> set key:2 99
OK              // 设置key:2=99
redis> object refcount key:2
(integer) 3     // 使用了对象共享,引用数变为3
redis> config set maxmemory 1GB
OK              // 设置最大可用内存
redis> set key:3 99
OK              // 设置key:3=99
redis> object refcount key:3
(integer) 1     // 未使用对象共享,引用数为1
redis> config set maxmemory-policy volatile-ttl
OK              // 设置非LRU淘汰策略
redis> set key:4 99
OK              // 设置key:4=99
redis> object refcount key:4
(integer) 4     // 又可以使用对象共享,引用数变为4
```

为什么开启maxmemory和LRU淘汰策略后对象池无效？LRU算法需要获取对象最后被访问时间，以便淘汰最长未访问数据，每个对象最后访问时间存储在redisObject对象的lru字段。对象共享意味着多个引用共享同一个redisObject，这时lru字段也会被共享，导致无法获取每个对象的最后访问时间。如果没有设置maxmemory，直到内存被用尽Redis也不会触发内存回收，所以共享对象池可以正常工作。综上所述，共享对象池与maxmemory+LRU策略冲突，使用时需要注意。对于ziplist编码的值对象，即使内部数据为整数也无法使用共享对象池，因为ziplist使用压缩且内存连续的结构，对象共享判断成本过高，ziplist编码细节后面内容详细说明。

为什么只有整数对象池？首先整数对象池复用的几率最大，其次对象共享的一个关键操作就是判断相等性，Redis之所以只有整数对象池，是因为整数比较算法时间复杂度为O(1)，只保留一万个整数为了防止对象池浪费。如果是字符串判断相等性，时间复杂度变为O(n)，特别是长字符串更消耗性能（浮点数在Redis内部使用字符串存储）。对于更复杂的数据结构如hash、list等，相等性判断需要O(n2)。对于单线程的Redis来说，这样的开销显然不合理，因此Redis只保留整数共享对象池。

#### 字符串优化

字符串对象是Redis内部最常用的数据类型。所有的键都是字符串类型，值对象数据除了整数之外都使用字符串存储。比如执行命令：lpush cache：type"redis""memcache""tair""levelDB"，Redis首先创建"cache：type"键字符串，然后创建链表对象，链表对象内再包含四个字符串对象，排除Redis内部用到的字符串对象之外至少创建5个字符串对象。可见字符串对象在Redis内部使用非常广泛，因此深刻理解Redis字符串对于内存优化非常有帮助。

##### 1.字符串结构

Redis没有采用原生C语言的字符串类型而是自己实现了字符串结构，内部简单动态字符串(simple dynamic string，SDS)。

![image-20251128110655541](images/image-20251128110655541.png)

Redis自身实现的字符串结构有如下特点：

- O(1)时间复杂度获取：字符串长度、已用长度、未用长度。
- 可用于保存字节数组，支持安全的二进制数据存储。
- 内部实现空间预分配机制，降低内存再分配次数。
- 惰性删除机制，字符串缩减后的空间不释放，作为预分配空间保留。

##### 2.预分配机制

因为字符串(SDS)存在预分配机制，日常开发中要小心预分配带来的内存浪费

![image-20251128110803016](images/image-20251128110803016.png)

从测试数据可以看出，同样的数据追加后内存消耗非常严重，下面我们结合图来分析这一现象。阶段1每个字符串对象空间占用如图8-10所示。

![image-20251128110820525](images/image-20251128110820525.png)

阶段1插入新的字符串后，free字段保留空间为0，总占用空间=实际占用空间+1字节，最后1字节保存‘\0’标示结尾，这里忽略int类型len和free字段消耗的8字节。在阶段1原有字符串上追加60字节数据空间占用如图8-11所示。

![image-20251128110836980](images/image-20251128110836980.png)

追加操作后字符串对象预分配了一倍容量作为预留空间，而且大量追加操作需要内存重新分配，造成内存碎片率(mem_fragmentation_ratio)上升。直接插入与阶段2相同数据的空间占用，如图8-12所示。

![image-20251128110908752](images/image-20251128110908752.png)

阶段3直接插入同等数据后，相比阶段2节省了每个字符串对象预分配的空间，同时降低了碎片率。字符串之所以采用预分配的方式是防止修改操作需要不断重分配内存和字节数据拷贝。但同样也会造成内存的浪费。字符串预分配每次并不都是翻倍扩容，空间预分配规则如下：1)第一次创建len属性等于数据实际大小，free等于0，不做预分配。2)修改后如果已有free空间不够且数据小于1M，每次预分配一倍容量。如原有len=60byte，free=0，再追加60byte，预分配120byte，总占用空间：60byte+60byte+120byte+1byte。3)修改后如果已有free空间不够且数据大于1MB，每次预分配1MB数据。如原有len=30MB，free=0，当再追加100byte，预分配1MB，总占用空间：1MB+100byte+1MB+1byte。

尽量减少字符串频繁修改操作如append、setrange，改为直接使用set修改字符串，降低预分配带来的内存浪费和内存碎片化。

##### 3.字符串重构

字符串重构：指不一定把每份数据作为字符串整体存储，像json这样的数据可以使用hash结构，使用二级结构存储也能帮我们节省内存。同时可以使用hmget、hmset命令支持字段的部分读取修改，而不用每次整体存取。例如下面的json数据：

```
{
    "vid": "413368768",
    "title": "搜狐屌丝男士",
    "videoAlbumPic":"http://photocdn.sohu.com/60160518/vrsa_ver8400079_ae433_pic26.jpg",
    "pid": "6494271",
    "type": "1024",
    "playlist": "6494271",
    "playTime": "468"
}
```

分别使用字符串和hash结构测试内存表现

![image-20251128111011993](images/image-20251128111011993.png)

根据测试结构，第一次默认配置下使用hash类型，内存消耗不但没有降低反而比字符串存储多出2倍，而调整hash-max-ziplist-value=66之后内存降低为535.60M。因为json的videoAlbumPic属性长度是65，而hash-max-ziplist-value默认值是64，Redis采用hashtable编码方式，反而消耗了大量内存。调整配置后hash类型内部编码方式变为ziplist，相比字符串更省内存且支持属性的部分操作。下一节将具体介绍ziplist编码优化细节。

#### 编码优化

##### 1.了解编码

Redis对外提供了string、list、hash、set、zet等类型，但是Redis内部针对不同类型存在编码的概念，所谓编码就是具体使用哪种底层数据结构来实现。编码不同将直接影响数据的内存占用和读写效率。使用object encoding{key}命令获取编码类型。如下所示：

```
redis> set str:1 hello
OK
redis> object encoding str:1
"embstr"                // embstr编码字符串
redis> lpush list:1 1 2 3
(integer) 3
redis> object encoding list:1
"ziplist"               // ziplist编码列表
```

Redis针对每种数据类型(type)可以采用至少两种编码方式来实现，表8-5表示type和encoding的对应关系

![image-20251128111513763](images/image-20251128111513763.png)

了解编码和类型对应关系之后，我们不禁疑惑Redis为什么对一种数据结构实现多种编码方式？主要原因是Redis作者想通过不同编码实现效率和空间的平衡。比如当我们的存储只有10个元素的列表，当使用双向链表数据结构时，必然需要维护大量的内部字段如每个元素需要：前置指针，后置指针，数据指针等，造成空间浪费，如果采用连续内存结构的压缩列表(ziplist)，将会节省大量内存，而由于数据长度较小，存取操作时间复杂度即使为O(n2)性能也可满足需求。

##### 2.控制编码类型

编码类型转换在Redis写入数据时自动完成，这个转换过程是不可逆的，转换规则只能从小内存编码向大内存编码转换。例如：

```
redis> lpush list:1 a b c d
(integer) 4     // 存储4个元素
redis> object encoding list:1
"ziplist"               // 采用ziplist压缩列表编码
redis> config set list-max-ziplist-entries 4
OK              // 设置列表类型ziplist编码最大允许4个元素
redis> lpush list:1 e
(integer) 5     // 写入第5个元素e
redis> object encoding list:1
"linkedlist"    // 编码类型转换为链表
redis> rpop list:1
"a"             // 弹出元素a
redis> llen list:1
(integer) 4     // 列表此时有4个元素
redis> object encoding list:1
"linkedlist"    // 编码类型依然为链表，未做编码回退
```

以上命令体现了list类型编码的转换过程，其中Redis之所以不支持编码回退，主要是数据增删频繁时，数据向压缩编码转换非常消耗CPU，得不偿失。以上示例用到了list-max-ziplist-entries参数，这个参数用来决定列表长度在多少范围内使用ziplist编码。当然还有其他参数控制各种数据类型的编码，如表8-6所示。

![image-20251128111602390](images/image-20251128111602390-1764299762866-1.png)

![image-20251128111616260](images/image-20251128111616260.png)

掌握编码转换机制，对我们通过编码来优化内存使用非常有帮助。下面以hash类型为例，介绍编码转换的运行流程，如图8-13所示。

![image-20251128111631971](images/image-20251128111631971.png)

理解编码转换流程和相关配置之后，可以使用config set命令设置编码相关参数来满足使用压缩编码的条件。对于已经采用非压缩编码类型的数据如hashtable、linkedlist等，设置参数后即使数据满足压缩编码条件，Redis也不会做转换，需要重启Redis重新加载数据才能完成转换。



##### 3.ziplist编码



ziplist编码主要目的是为了节约内存，因此所有数据都是采用线性连续的内存结构。ziplist编码是应用范围最广的一种，可以分别作为hash、list、zset类型的底层数据结构实现。首先从ziplist编码结构开始分析，它的内部结构类似这样：<zlbytes><zltail><zllen><entry-1><entry-2><….><entry-n><zlend>。一个ziplist可以包含多个entry（元素），每个entry保存具体的数据（整数或者字节数组），内部结构如图8-14所示。

![image-20251128111717751](images/image-20251128111717751.png)

ziplist结构字段含义：1)zlbytes：记录整个压缩列表所占字节长度，方便重新调整ziplist空间。类型是int-32，长度为4字节。2)zltail：记录距离尾节点的偏移量，方便尾节点弹出操作。类型是int-32，长度为4字节。3)zllen：记录压缩链表节点数量，当长度超过216-2时需要遍历整个列表获取长度，一般很少见。类型是int-16，长度为2字节。4)entry：记录具体的节点，长度根据实际存储的数据而定。

a)prev_entry_bytes_length：记录前一个节点所占空间，用于快速定位上一个节点，可实现列表反向迭代。b)encoding：标示当前节点编码和长度，前两位表示编码类型：字符串/整数，其余位表示数据长度。c)contents：保存节点的值，针对实际数据长度做内存占用优化。5)zlend：记录列表结尾，占用一个字节。

根据以上对ziplist字段说明，可以分析出该数据结构特点如下：·内部表现为数据紧凑排列的一块连续内存数组。·可以模拟双向链表结构，以O(1)时间复杂度入队和出队。·新增删除操作涉及内存重新分配或释放，加大了操作的复杂性。·读写操作涉及复杂的指针移动，最坏时间复杂度为O(n2)。·适合存储小对象和长度有限的数据。下面通过测试展示ziplist编码在不同类型中内存和速度的表现，如表8-7所示。

![image-20251128111809491](images/image-20251128111809491.png)

测试数据采用100W个36字节数据，划分为1000个键，每个类型长度统一为1000。从测试结果可以看出：1)使用ziplist可以分别作为hash、list、zset数据类型实现。2)使用ziplist编码类型可以大幅降低内存占用。3)ziplist实现的数据类型相比原生结构，命令操作更加耗时，不同类型耗时排序：list<hash<zset。ziplist压缩编码的性能表现跟值长度和元素个数密切相关，正因为如此Redis提供了{type}-max-ziplist-value和{type}-max-ziplist-entries相关参数来做控制ziplist编码转换。最后再次强调使用ziplist压缩编码的原则：追求空间和时间的平衡。

针对性能要求较高的场景使用ziplist，建议长度不要超过1000，每个元素大小控制在512字节以内。命令平均耗时使用info Commandstats命令获取，包含每个命令调用次数、总耗时、平均耗时，单位为微秒。

##### 4.intset编码

intset编码是集合(set)类型编码的一种，内部表现为存储有序、不重复的整数集。当集合只包含整数且长度不超过set-max-intset-entries配置时被启用。执行以下命令查看intset表现：

```
redis> sadd set:test 3 4 2 6 8 9 2
(integer) 6             // 乱序写入6个整数
Redis> object encoding set:test
"intset"                        // 使用intset编码
Redis> smembers set:test
"2" "3" "4" "6" "8" "9"         // 排序输出整数结合
redis> config set set-max-intset-entries 6
OK                              // 设置intset最大允许整数长度
redis> sadd set:test 5
(integer) 1                     // 写入第7个整数 5
redis> object encoding set:test
"hashtable"                     // 编码变为hashtable
redis> smembers set:test
"8" "3" "5" "9" "4" "2" "6"     // 乱序输出
```

以上命令可以看出intset对写入整数进行排序，通过O(log(n))时间复杂度实现查找和去重操作，intset编码结构如图8-15所示。

![image-20251128111913344](images/image-20251128111913344.png)

intset的字段结构含义：1)encoding：整数表示类型，根据集合内最长整数值确定类型，整数类型划分为三种：int-16、int-32、int-64。2)length：表示集合元素个数。3)contents：整数数组，按从小到大顺序保存。intset保存的整数类型根据长度划分，当保存的整数超出当前类型时，将会触发自动升级操作且升级后不再做回退。升级操作将会导致重新申请内存空间，把原有数据按转换类型后拷贝到新数组。

使用intset编码的集合时，尽量保持整数范围一致，如都在int-16范围内。防止个别大整数触发集合升级操作，产生内存浪费。下面通过测试查看ziplist编码的集合内存和速度表现，如表8-8所示。

![image-20251128111937892](images/image-20251128111937892.png)

根据以上测试结果发现intset表现非常好，同样的数据内存占用只有不到hashtable编码的十分之一。intset数据结构插入命令复杂度为O(n)，查询命令为O(log(n))，由于整数占用空间非常小，所以在集合长度可控的基础上，写入命令执行速度也会非常快，因此当使用整数集合时尽量使用intset编码。表8-8测试第三行把ziplist-hash类型也放入其中，主要因为intset编码必须存储整数，当集合内保存非整数数据时，无法使用intset实现内存优化。这时可以使用ziplist-hash类型对象模拟集合类型，hash的field当作集合中的元素，value设置为1字节占位符即可。使用ziplist编码的hash类型依然比使用hashtable编码的集合节省大量内存。

#### 控制键的数量

当使用Redis存储大量数据时，通常会存在大量键，过多的键同样会消耗大量内存。Redis本质是一个数据结构服务器，它为我们提供多种数据结构，如hash、list、set、zset等。使用Redis时不要进入一个误区，大量使用get/set这样的API，把Redis当成Memcached使用。对于存储相同的数据内容利用Redis的数据结构降低外层键的数量，也可以节省大量内存。如图8-16所示，通过在客户端预估键规模，把大量键分组映射到多个hash结构中降低键的数量。

![image-20251128112040757](images/image-20251128112040757.png)

hash结构降低键数量分析：·根据键规模在客户端通过分组映射到一组hash对象中，如存在100万个键，可以映射到1000个hash中，每个hash保存1000个元素。·hash的field可用于记录原始key字符串，方便哈希查找。·hash的value保存原始值对象，确保不要超过hash-max-ziplist-value限制。下面测试这种优化技巧的内存表现

![image-20251128112100902](images/image-20251128112100902.png)

通过这个测试数据，可以说明：·同样的数据使用ziplist编码的hash类型存储比string类型节约内存。·节省内存量随着value空间的减少越来越明显。·hash-ziplist类型比string类型写入耗时，但随着value空间的减少，耗时逐渐降低。使用hash重构后节省内存量效果非常明显，特别对于存储小对象的场景，内存只有不到原来的1/5。下面分析这种内存优化技巧的关键点：

1)hash类型节省内存的原理是使用ziplist编码，如果使用hashtable编码方式反而会增加内存消耗。2)ziplist长度需要控制在1000以内，否则由于存取操作时间复杂度在O(n)到O(n2)之间，长列表会导致CPU消耗严重，得不偿失。3)ziplist适合存储小对象，对于大对象不但内存优化效果不明显还会增加命令操作耗时。4)需要预估键的规模，从而确定每个hash结构需要存储的元素数量。5)根据hash长度和元素大小，调整hash-max-ziplist-entries和hash-max-ziplist-value参数，确保hash类型使用ziplist编码。

关于hash键和field键的设计：1)当键离散度较高时，可以按字符串位截取，把后三位作为哈希的field，之前部分作为哈希的键。如：key=1948480哈希key=group：hash：1948，哈希field=480。2)当键离散度较低时，可以使用哈希算法打散键，如：使用crc32(key)&10000函数把所有的键映射到“0-9999”整数范围内，哈希field存储键的原始值。3)尽量减少hash键和field的长度，如使用部分键内容。

使用hash结构控制键的规模虽然可以大幅降低内存，但同样会带来问题，需要提前做好规避处理。如下所示：·客户端需要预估键的规模并设计hash分组规则，加重客户端开发成本。·hash重构后所有的键无法再使用超时(expire)和LRU淘汰机制自动删除，需要手动维护删除。·对于大对象，如1KB以上的对象，使用hash-ziplist结构控制键数量反而得不偿失。

不过瑕不掩瑜，对于大量小对象的存储场景，非常适合使用ziplist编码的hash类型控制键的规模来降低内存。

使用ziplist+hash优化keys后，如果想使用超时删除功能，开发人员可以存储每个对象写入的时间，再通过定时任务使用hscan命令扫描数据，找出hash内超时的数据项删除即可。本节主要讲解Redis内存优化技巧，Redis的数据特性是“all in memory”，优化内存将变得非常重要。对于内存优化建议读者先要掌握Redis内存存储的特性比如字符串、压缩编码、整数集合等，再根据数据规模和所用命令需求去调整，从而达到空间和效率的最佳平衡。建议使用Redis存储大量数据时，把内存优化环节加入到前期设计阶段，否则数据大幅增长后，开发人员需要面对重新优化内存所带来开发和数据迁移的双重成本。当Redis内存不足时，首先考虑的问题不是加机器做水平扩展，应该先尝试做内存优化，当遇到瓶颈时，再去考虑水平扩展。即使对于集群化方案，垂直层面优化也同样重要，避免不必要的资源浪费和集群化后的管理成本。

## 哨兵

- 吹哨人巡查监控后台master主机是否故障,如果故障了根据投票数自动将某一个从库转换为新主库，继续对外服务

- 哨兵的作用：
  - 监控主从redis运行状态，包括master和slave。
  - 哨兵可以将故障转移的结构发给客户端
  - 如果master异常，则会进行主从切换，将其中一个Slave作为新Master
  - 客户端通过连续哨兵来获得当前Redis服务的主节点地址

- 官网理论 https://redis.io/docs/manual/sentinel/

### 基本概念

Redis Sentinel相关名词解释

![image-20251128182750376](images/image-20251128182750376.png)

![image-20251128182806431](images/image-20251128182806431.png)

Redis Sentinel是Redis的高可用实现方案，在实际的生产环境中，对提高整个系统的高可用性是非常有帮助的，本节首先会回顾主从复制模式下故障处理可能产生的问题，而后引出高可用的概念，最后重点分析Redis Sentinel的基本架构、优势，以及是如何实现高可用的。

#### 主从复制问题

Redis的主从复制模式可以将主节点的数据改变同步给从节点，这样从节点就可以起到两个作用：

- 第一，作为主节点的一个备份，一旦主节点出了故障不可达的情况，从节点可以作为后备“顶”上来，并且保证数据尽量不丢失（主从复制是最终一致性）。

- 第二，从节点可以扩展主节点的读能力，一旦主节点不能支撑住大并发量的读操作，从节点可以在一定程度上帮助主节点分担读压力。

但是主从复制也带来了以下问题：

- 一旦主节点出现故障，需要手动将一个从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点，整个过程都需要人工干预。
- 主节点的写能力受到单机的限制
- 主节点的存储能力受到单机的限制。



#### 高可用

Redis主从复制模式下，一旦主节点出现了故障不可达，需要人工干预进行故障转移，无论对于Redis的应用方还是运维方都带来了很大的不便。对于应用方来说无法及时感知到主节点的变化，必然会造成一定的写数据丢失和读数据错误，甚至可能造成应用方服务不可用。对于Redis的运维方来说，整个故障转移的过程是需要人工来介入的，故障转移实时性和准确性上都无法得到保障，图9-1到图9-5展示了一个1主2从的Redis主从复制模式下的主节点出现故障后，是如何进行故障转移的，过程如下所示。

1)如图9-1所示，主节点发生故障后，客户端(client)连接主节点失败，两个从节点与主节点连接失败造成复制中断。

2)如图9-2所示，如果主节点无法正常启动，需要选出一个从节点(slave-1)，对其执行slaveof no one命令使其成为新的主节点。

![image-20251128183348466](images/image-20251128183348466.png)

3)如图9-3所示，原来的从节点(slave-1)成为新的主节点后，更新应用方的主节点信息，重新启动应用方。

从节点执行slaveof no one晋级为主节点

![image-20251128183406442](images/image-20251128183406442.png)

应用方连接新的主节点

![image-20251128183432835](images/image-20251128183432835.png)

如图9-4所示，客户端命令另一个从节点(slave-2)去复制新的主节点(new-master)

5)如图9-5所示，待原来的主节点恢复后，让它去复制新的主节点。

![image-20251128183520834](images/image-20251128183520834.png)

其余从节点复制新的主节点

![image-20251128183544545](images/image-20251128183544545.png)

上述处理过程就可以认为整个服务或者架构的设计不是高可用的，因为整个故障转移的过程需要人介入。考虑到这点，有些公司把上述流程自动化了，但是仍然存在如下问题：第一，判断节点不可达的机制是否健全和标准。第二，如果有多个从节点，怎样保证只有一个被晋升为主节点。第三，通知客户端新的主节点机制是否足够健壮。Redis Sentinel正是用于解决这些问题。

#### Redis Sentine的高可用性

当主节点出现故障时，Redis Sentinel能自动完成故障发现和故障转移，并通知应用方，从而实现真正的高可用。

Redis Sentinel是一个分布式架构，其中包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其余Sentinel节点进行监控，当它发现节点不可达时，会对节点做下线标识。如果被标识的是主节点，它还会和其他Sentinel节点进行“协商”，当大多数Sentinel节点都认为主节点不可达时，它们会选举出一个Sentinel节点来完成自动故障转移的工作，同时会将这个变化实时通知给Redis应用方。整个过程完全是自动的，不需要人工来介入，所以这套方案很有效地解决了Redis的高可用问题。

这里的分布式是指：Redis数据节点、Sentinel节点集合、客户端分布在多个物理节点的架构，不要与第10章介绍的Redis Cluster分布式混淆。

Redis Sentinel与Redis主从复制模式只是多了若干Sentinel节点，所以Redis Sentinel并没有针对Redis节点做了特殊处理，这里是很多开发和运维人员容易混淆的。

Redis主从复制与Redis Sentinel架构的区别

![image-20251128183935193](images/image-20251128183935193.png)



从逻辑架构上看，Sentinel节点集合会定期对所有节点进行监控，特别是对主节点的故障实现自动转移。

下面以1个主节点、2个从节点、3个Sentinel节点组成的Redis Sentinel为例子进行说明，拓扑结构如图9-7所示。

![image-20251128184017742](images/image-20251128184017742.png)

整个故障转移的处理逻辑有下面4个步骤：

1)如图9-8所示，主节点出现故障，此时两个从节点与主节点失去连接，主从复制失败。

![image-20251128184051023](images/image-20251128184051023.png)

2)如图9-9所示，每个Sentinel节点通过定期监控发现主节点出现了故障。

![image-20251128184115930](images/image-20251128184115930.png)

3)如图9-10所示，多个Sentinel节点对主节点的故障达成一致，选举出sentinel-3节点作为领导者负责故障转移。

![image-20251128184141737](images/image-20251128184141737.png)

4)如图9-11所示，Sentinel领导者节点执行了故障转移，整个过程和9.1.2节介绍的是完全一致的，只不过是自动化完成的。

![image-20251128184304299](images/image-20251128184304299.png)

5)故障转移后整个Redis Sentinel的拓扑结构图9-12所示。

![image-20251128184329481](images/image-20251128184329481.png)



通过上面介绍的Redis Sentinel逻辑架构以及故障转移的处理，可以看出Redis Sentinel具有以下几个功能：

- 监控：Sentinel节点会定期检测Redis数据节点、其余Sentinel节点是否可达。
- 通知：Sentinel节点会将故障转移的结果通知给应用方。
- 主节点故障转移：实现从节点晋升为主节点并维护后续正确的主从关系。
- 配置提供者：在Redis Sentinel结构中，客户端在初始化的时候连接的是Sentinel节点集合，从中获取主节点信息。

同时看到，Redis Sentinel包含了若个Sentinel节点，这样做也带来了两个好处：

- 对于节点的故障判断是由多个Sentinel节点共同完成，这样可以有效地防止误判。
- Sentinel节点集合是由若干个Sentinel节点组成的，这样即使个别Sentinel节点不可用，整个Sentinel节点集合依然是健壮的。

但是Sentinel节点本身就是独立的Redis节点，只不过它们有一些特殊，它们不存储数据，只支持部分命令。下一节将完整介绍Redis Sentinel的部署过程，相信在安装和部署完Redis Sentinel后，读者能更清晰地了解Redis Sentinel的整体架构。

### 安装和部署

#### 部署拓扑结构

下面将以3个Sentinel节点、1个主节点、2个从节点组成一个Redis Sentinel进行说明

![image-20251128184606028](images/image-20251128184606028.png)

具体的物理部署如表9-2所示。Redis Sentinel物理结构

![image-20251128184632690](images/image-20251128184632690.png)

#### 部署Redis数据节点

##### 1.启动主节点

配置：

```
redis-6379.conf
port 6379  
daemonize yes  
logfile "6379.log"
dbfilename "dump-6379.rdb"  
dir "/opt/soft/redis/data/" 
```

启动主节点：

```
redis-server redis-6379.conf
```

确认是否启动。一般来说只需要ping命令检测一下就可以，确认Redis数据节点是否已经启动。

```
$ redis-cli -h 127.0.0.1 -p 6379 ping
PONG
```

此时拓扑结构如图9-14所示。

![image-20251128184851051](images/image-20251128184851051.png)



启动主节点

##### 2.启动两个从节点

配置：两个从节点的配置是完全一样的，下面以一个从节点为例子进行说明，和主节点的配置不一样的是添加了slaveof配置。

```
redis-6380.conf
port 6380  
daemonize yes  
logfile "6380.log"  
dbfilename "dump-6380.rdb"  
dir "/opt/soft/redis/data/" 
slaveof 127.0.0.1 6379
```

启动两个从节点：

```
redis-server redis-6380.conf
redis-server redis-6381.conf
```

验证：

```
$ redis-cli -h 127.0.0.1 -p 6380 ping
PONG
$ redis-cli -h 127.0.0.1 -p 6381 ping
PONG
```

##### 3.确认主从关系

主节点的视角，它有两个从节点，分别是127.0.0.1：6380和127.0.0.1：6381：

```
$ redis-cli -h 127.0.0.1 -p 6379 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=281,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=281,lag=0
……………..
```

从节点的视角，它的主节点是127.0.0.1：6379：

```
$ redis-cli -h 127.0.0.1 -p 6380 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
……………..
```

此时拓扑结构如图

![image-20251128185111113](images/image-20251128185111113.png)



#### 部署sentinal节点

3个Sentinel节点的部署方法是完全一致的（端口不同），下面以sentinel-1节点的部署为例子进行说明。

cp sentinel.conf  /myredis/

ifconfig

##### 1.配置Sentinel节点

![image-20251221103028230](./assets/image-20251221103028230.png)

```
redis-sentinel-26379.conf
bind 0.0.0.0
protected-mode no
pidfile 
port 26379  
daemonize yes  
logfile "26379.log"   #日志
dir /opt/soft/redis/data  
sentinel monitor mymaster 127.0.0.1 6379 2  

sentinel down-after-milliseconds mymaster 30000  #指定多少毫秒之后，主节点没有应答哨兵，此时哨兵主观上认为主节点下线

sentinel parallel-syncs mymaster 1  #表示允许并行同步的slave个数,当Master挂了后,哨兵会选出新的Master,此时,剩余的slave会向新的master发起同步数据

sentinel failover-timeout mymaster 180000  #故障转移的超时时间，进行故障转移时，如果超过设置的毫秒，表示故障转移失败

sentinel auth-pass <master-name> <password>  #master设置了密码，连接master服务的密码
```

1)Sentinel节点的默认端口是26379。

2)sentinel monitor mymaster127.0.0.163792配置代表sentinel-1节点需要监控127.0.0.1：6379这个主节点，2代表判断主节点失败至少需要2个Sentinel节点同意，mymaster是主节点的别名，其余Sentinel配置将在下一节进行详细说明。

##### 2.启动Sentinel节点

Sentinel节点的启动方法有两种：

方法一，使用redis-sentinel命令：

```
redis-sentinel redis-sentinel-26379.conf
```

方法二，使用redis-server命令加--sentinel参数：

```
redis-server redis-sentinel-26379.conf --sentinel
```

两种方法本质上是一样的。

##### 3.确认

Sentinel节点本质上是一个特殊的Redis节点，所以也可以通过info命令来查询它的相关信息，从下面info的Sentinel片段来看，Sentinel节点找到了主节点127.0.0.1：6379，发现了它的两个从节点，同时发现Redis Sentinel一共有3个Sentinel节点。这里只需要了解Sentinel节点能够彼此感知到对方，同时能够感知到Redis数据节点就可以了：



```
$ redis-cli -h 127.0.0.1 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

当三个Sentinel节点都启动后，整个拓扑结构如图9-16所示。

至此Redis Sentinel已经搭建起来了，整体上还是比较容易的，但是有2点需要强调一下：

1)生产环境中建议Redis Sentinel的所有节点应该分布在不同的物理机上。

2)Redis Sentinel中的数据节点和普通的Redis数据节点在配置上没有任何区别，只不过是添加了一些Sentinel节点对它们进行监控。

Redis Sentinel最终拓扑结构

![image-20251128185659072](images/image-20251128185659072.png)

#### 配置优化

了解每个配置的含义有助于更加合理地使用Redis Sentinel，因此本节将对每个配置的使用和优化进行详细介绍。Redis安装目录下有一个sentinel.conf，是默认的Sentinel节点配置文件，下面就以它作为例子进行说明。

##### 1.配置说明和优化

```
port 26379
dir /opt/soft/redis/data        
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
#sentinel auth-pass <master-name> <password>
#sentinel notification-script <master-name> <script-path>
#sentinel client-reconfig-script <master-name> <script-path>
```

port和dir分别代表Sentinel节点的端口和工作目录，下面重点对sentinel相关配置进行详细说明。

(1)sentinel monitor

配置如下：

```
sentinel monitor <master-name> <ip> <port> <quorum>  
```

Sentinel节点会定期监控主节点，所以从配置上必然也会有所体现，本配置说明Sentinel节点要监控的是一个名字叫做<master-name>，ip地址和端口为<ip><port>的主节点。<quorum>代表要判定主节点最终不可达所需要的票数。但实际上Sentinel节点会对所有节点进行监控，但是在Sentinel节点的配置中没有看到有关从节点和其余Sentinel节点的配置，那是因为Sentinel节点会从主节点中获取有关从节点以及其余Sentinel节点的相关信息，有关这部分是如何实现的，将在9.5节介绍。例如某个Sentinel初始节点配置如下：

```
port 26379
daemonize yes
logfile "26379.log"
dir /opt/soft/redis/data
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

当所有节点启动后，配置文件中的内容发生了变化，体现在三个方面：·Sentinel节点自动发现了从节点、其余Sentinel节点。·去掉了默认配置，例如parallel-syncs、failover-timeout参数。·添加了配置纪元相关参数。

启动后变化为：

````
port 26379
daemonize yes
logfile "26379.log"
dir "/opt/soft/redis/data"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
#发现两个slave节点
sentinel known-slave mymaster 127.0.0.1 6380
sentinel known-slave mymaster 127.0.0.1 6381
#发现两个sentinel节点
sentinel known-sentinel mymaster 127.0.0.1 26380 282a70ff56c36ed56e8f7ee6ada741
    24140d6f53
sentinel known-sentinel mymaster 127.0.0.1 26381 f714470d30a61a8e39ae031192f1fe
    ae7eb5b2be
sentinel current-epoch 0
````

<quorum>参数用于故障发现和判定，例如将quorum配置为2，代表至少有2个Sentinel节点认为主节点不可达，那么这个不可达的判定才是客观的。对于<quorum>设置的越小，那么达到下线的条件越宽松，反之越严格。一般建议将其设置为Sentinel节点的一半加1。同时<quorum>还与Sentinel节点的领导者选举有关，至少要有max(quorum，num(sentinels)/2+1)个Sentinel节点参与选举，才能选出领导者Sentinel，从而完成故障转移。例如有5个Sentinel节点，quorum=4，那么至少要有max(quorum，num(sentinels)/2+1)=4个在线Sentinel节点才可以进行领导者选举。

(2)sentinel down-after-milliseconds

配置如下：

```
sentinel down-after-milliseconds <master-name> <times> 
```

每个Sentinel节点都要通过定期发送ping命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过了down-after-milliseconds配置的时间且没有有效的回复，则判定节点不可达，<times>（单位为毫秒）就是超时时间。这个配置是对节点失败判定的重要依据。

优化说明：down-after-milliseconds越大，代表Sentinel节点对于节点不可达的条件越宽松，反之越严格。条件宽松有可能带来的问题是节点确实不可达了，那么应用方需要等待故障转移的时间越长，也就意味着应用方故障时间可能越长。条件严格虽然可以及时发现故障完成故障转移，但是也存在一定的误判率。

down-after-milliseconds虽然以<master-name>为参数，但实际上对Sentinel节点、主节点、从节点的失败判定同时有效。

(3)sentinel parallel-syncs

```
sentinel parallel-syncs <master-name> <nums> 
```

当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，原来的从节点会向新的主节点发起复制操作，parallel-syncs就是用来限制在一次故障转移之后，每次向新的主节点发起复制操作的从节点个数。如果这个参数配置的比较大，那么多个从节点会向新的主节点同时发起复制操作，尽管复制操作通常不会阻塞主节点，但是同时向主节点发起复制，必然会对主节点所在的机器造成一定的网络和磁盘IO开销。图9-17展示parallel-syncs=3和parallel-syncs=1的效果，parallel-syncs=3会同时发起复制，parallel-syncs=1时从节点会轮询发起复制。

(4)sentinel failover-timeout

```
sentinel failover-timeout <master-name> <times> 
```

![image-20251128190311100](images/image-20251128190311100.png)

failover-timeout通常被解释成故障转移超时时间，但实际上它作用于故障转移的各个阶段：a)选出合适从节点。b)晋升选出的从节点为主节点。c)命令其余从节点复制新的主节点。d)等待原主节点恢复后命令它去复制新的主节点。failover-timeout的作用具体体现在四个方面：1)如果Redis Sentinel对一个主节点故障转移失败，那么下次再对该主节点做故障转移的起始时间是failover-timeout的2倍。2)在b)阶段时，如果Sentinel节点向a)阶段选出来的从节点执行slaveof no one一直失败（例如该从节点此时出现故障），当此过程超过failover-timeout时，则故障转移失败。3)在b)阶段如果执行成功，Sentinel节点还会执行info命令来确认a)阶段选出来的节点确实晋升为主节点，如果此过程执行时间超过failover-timeout时，则故障转移失败。4)如果c)阶段执行时间超过了failover-timeout（不包含复制时间），则故障转移失败。注意即使超过了这个时间，Sentinel节点也会最终配置从节点去同步最新的主节点。

(5)sentinel auth-pass

```
sentinel auth-pass <master-name> <password>
```

如果Sentinel监控的主节点配置了密码，sentinel auth-pass配置通过添加主节点的密码，防止Sentinel节点对主节点无法监控。

(6)sentinel notification-script

```
sentinel notification-script <master-name> <script-path>
```

sentinel notification-script的作用是在故障转移期间，当一些警告级别的Sentinel事件发生（指重要事件，例如-sdown：客观下线、-odown：主观下线）时，会触发对应路径的脚本，并向脚本发送相应的事件参数。例如在/opt/redis/scripts/下配置了notification.sh，该脚本会接收每个Sentinel节点传过来的事件参数，可以利用这些参数作为邮件或者短信报警依据：

```
#!/bin/sh
#获取所有参数
msg=$*
#报警脚本或者接口，将msg作为参数
exit 0
```

如果需要该功能，就可以在Sentinel节点添加如下配置(<master-name>=mymaster)：

```
sentinel notification-script mymaster /opt/redis/scripts/notification.sh
```

例如下面就是某个Sentinel节点对主节点做了主观下线（有关主观下线的概念将在9.5节进行详细介绍）后脚本收到的参数：

```
sentinel client-reconfig-script <master-name> <script-path>
```

sentinel client-reconfig-script的作用是在故障转移结束后，会触发对应路径的脚本，并向脚本发送故障转移结果的相关参数。和notification-script类似，可以在/opt/redis/scripts/下配置了client-reconfig.sh，该脚本会接收每个Sentinel节点传过来的故障转移结果参数，并触发类似短信和邮件报警：

```
#!/bin/sh
#获取所有参数
msg=$*
#报警脚本或者接口，将msg作为参数
exit 0
```

如果需要该功能，就可以在Sentinel节点添加如下配置(<master-name>=mymaster)：

```
sentinel client-reconfig-script mymaster /opt/redis/scripts/client-reconfig.sh
```

当故障转移结束，每个Sentinel节点会将故障转移的结果发送给对应的脚本，具体参数如下：

```
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
```

<master-name>：主节点名。·<role>：Sentinel节点的角色，分别是leader和observer，leader代表当前Sentinel节点是领导者，是它进行的故障转移；observer是其余Sentinel节点。·<from-ip>：原主节点的ip地址。·<from-port>：原主节点的端口。·<to-ip>：新主节点的ip地址。·<to-port>：新主节点的端口。

例如以下内容分别是三个Sentinel节点发送给脚本的，其中一个是leader，另外两个是observer：

```
mymaster leader start 127.0.0.1 6379 127.0.0.1 6380
mymaster observer start 127.0.0.1 6379 127.0.0.1 6380
mymaster observer start 127.0.0.1 6379 127.0.0.1 6380
```

有关sentinel notification-script和sentinel client-reconfig-script有几点需要注意：·<script-path>必须有可执行权限。·<script-path>开头必须包含shell脚本头（例如#！/bin/sh），否则事件发生时Redis将无法执行脚本产生如下错误：

```
-script-error /opt/sentinel/notification.sh 0 2
```

·Redis规定脚本的最大执行时间不能超过60秒，超过后脚本将被杀掉。·如果shell脚本以exit 1结束，那么脚本稍后重试执行。如果以exit 2或者更高的值结束，那么脚本不会重试。正常返回值是exit 0。·如果需要运维的Redis Sentinel比较多，建议不要使用这种脚本的形式来进行通知，这样会增加部署的成本。

##### 2.如何监控多个主节点

Redis Sentinel可以同时监控多个主节点，具体拓扑图类似于图9-18。配置方法也比较简单，只需要指定多个masterName来区分不同的主节点即可，例如下面的配置监控monitor master-business-1(10.10.xx.1：6379)和monitor master-business-2(10.10.xx.2：6379)两个主节点：

```
sentinel monitor master-business-1 10.10.xx.1 6379 2
sentinel down-after-milliseconds master-business-1 60000 
sentinel failover-timeout master-business-1 180000 
sentinel parallel-syncs master-business-1 1
sentinel monitor master-business-2 10.16.xx.2 6380 2 
sentinel down-after-milliseconds master-business-2 10000 
sentinel failover-timeout master-business-2 180000 
sentinel parallel-syncs master-business-2 1
```

![image-20251128190718169](images/image-20251128190718169.png)

##### 3.调整配置

和普通的Redis数据节点一样，Sentinel节点也支持动态地设置参数，而且和普通的Redis数据节点一样并不是支持所有的参数，具体使用方法如下：

```
sentinel set <param> <value>
```

sentinel set命令支持的参数

![image-20251128190825246](images/image-20251128190825246.png)

有几点需要注意一下：1)sentinel set命令只对当前Sentinel节点有效。2)sentinel set命令如果执行成功会立即刷新配置文件，这点和Redis普通数据节点设置配置需要执行config rewrite刷新到配置文件不同。3)建议所有Sentinel节点的配置尽可能一致，这样在故障发现和转移时比较容易达成一致。4)表9-3中为sentinel set支持的参数，具体可以参考源码中的sentinel.c的sentinelSetCommand函数。5)Sentinel对外不支持config命令。

#### 部署技巧

到现在有关Redis Sentinel的配置和部署方法相信读者已经基本掌握了，但在实际生产环境中都有哪些部署的技巧？本节将总结一下。1)Sentinel节点不应该部署在一台物理“机器”上。这里特意强调物理机是因为一台物理机做成了若干虚拟机或者现今比较流行的容器，它们虽然有不同的IP地址，但实际上它们都是同一台物理机，同一台物理机意味着如果这台机器有什么硬件故障，所有的虚拟机都会受到影响，为了实现Sentinel节点集合真正的高可用，请勿将Sentinel节点部署在同一台物理机器上。

2)部署至少三个且奇数个的Sentinel节点。3个以上是通过增加Sentinel节点的个数提高对于故障判定的准确性，因为领导者选举需要至少一半加1个节点，奇数个节点可以在满足该条件的基础上节省一个节点。有关Sentinel节点如何判断节点失败，如何选举出一个Sentinel节点进行故障转移将在9.5节进行介绍。

4)只有一套Sentinel，还是每个主节点配置一套Sentinel？Sentinel节点集合可以只监控一个主节点，也可以监控多个主节点，也就意味着部署拓扑可能是图9-19和图9-20两种情况。

一套Sentinel节点集合

![image-20251128190944190](images/image-20251128190944190.png)

多套Sentine节点集合

![image-20251128191004020](images/image-20251128191004020.png)

那么在实际生产环境中更偏向于哪一种部署方式呢，下面分别分析两种方案的优缺点。方案一：一套Sentinel，很明显这种方案在一定程度上降低了维护成本，因为只需要维护固定个数的Sentinel节点，集中对多个Redis数据节点进行管理就可以了。但是这同时也是它的缺点，如果这套Sentinel节点集合出现异常，可能会对多个Redis数据节点造成影响。还有如果监控的Redis数据节点较多，会造成Sentinel节点产生过多的网络连接，也会有一定的影响。方案二：多套Sentinel，显然这种方案的优点和缺点和上面是相反的，每个Redis主节点都有自己的Sentinel节点集合，会造成资源浪费。但是优点也很明显，每套Redis Sentinel都是彼此隔离的。运维提示如果Sentinel节点集合监控的是同一个业务的多个主节点集合，那么使用方案一、否则一般建议采用方案二。

#### API

##### 1.sentinel masters

展示所有被监控的主节点状态以及相关的统计信息，例如：

一套Sentinel集合监控多个主从结构

![image-20251128191400413](images/image-20251128191400413.png)

```
127.0.0.1:26379> sentinel masters
1)  1) "name"
    2) "mymaster-2"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6382"
………忽略…………
2)  1) "name"
    2) "mymaster-1"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6379"
………忽略…………
```

##### 2.sentinel master<master name>

展示指定<master name>的主节点状态以及相关的统计信息，例如：

```
127.0.0.1:26379> sentinel master mymaster-1
 1) "name"
 2) "mymaster-1"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
………忽略…………
```

##### 3.sentinel slaves<master name>

展示指定<master name>的从节点状态以及相关的统计信息，例如：

```
127.0.0.1:26379> sentinel slaves mymaster-1
1)  1) "name"
    2) "127.0.0.1:6380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6380"
………忽略…………
2)  1) "name"
    2) "127.0.0.1:6381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6381"
………忽略…………
```

##### 4.sentinel sentinels<master name>

展示指定<master name>的Sentinel节点集合（不包含当前Sentinel节点），例如：

```
127.0.0.1:26379> sentinel sentinels mymaster-1
1)  1) "name"
    2) "127.0.0.1:26380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26380"
………忽略…………
2)  1) "name"
    2) "127.0.0.1:26381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "26381"
………忽略…………
```

##### 5.sentinel get-master-addr-by-name<master name>

返回指定<master name>主节点的IP地址和端口，例如：

```
127.0.0.1:26379> sentinel get-master-addr-by-name mymaster-1
1) "127.0.0.1"
2) "6379"
```

##### 6.sentinel reset<pattern>

当前Sentinel节点对符合<pattern>（通配符风格）主节点的配置进行重置，包含清除主节点的相关状态（例如故障转移），重新发现从节点和Sentinel节点。例如sentinel-1节点对mymaster-1节点重置状态如下：

```
127.0.0.1:26379> sentinel reset mymaster-1
(integer) 1
```

##### 7.sentinel failover<master name>

对指定<master name>主节点进行强制故障转移（没有和其他Sentinel节点“协商”），当故障转移完成后，其他Sentinel节点按照故障转移的结果更新自身配置，这个命令在Redis Sentinel的日常运维中非常有用，将在9.6节进行详细介绍。例如，对mymaster-2进行故障转移：

```
127.0.0.1:26379> sentinel  failover mymaster-2
OK
```

执行命令前，mymaster-2是127.0.0.1：6382

```
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:2
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster-2,status=ok,address=127.0.0.1:6382,slaves=2,sentinels=3
master1:name=mymaster-1,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

执行命令后：mymaster-2由原来的一个从节点127.0.0.1：6383代替。

```
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:2
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster-2,status=ok,address=127.0.0.1:6383,slaves=2,sentinels=3
master1:name=mymaster-1,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

##### 8.sentinel ckquorum<master name>

检测当前可达的Sentinel节点总数是否达到<quorum>的个数。例如quorum=3，而当前可达的Sentinel节点个数为2个，那么将无法进行故障转移，Redis Sentinel的高可用特性也将失去。例如：

```
127.0.0.1:26379> sentinel ckquorum mymaster-1
OK 3 usable Sentinels. Quorum and failover authorization can be reached
```

##### 9.sentinel flushconfig

将Sentinel节点的配置强制刷到磁盘上，这个命令Sentinel节点自身用得比较多，对于开发和运维人员只有当外部原因（例如磁盘损坏）造成配置文件损坏或者丢失时，这个命令是很有用的。例如：

```
127.0.0.1:26379> sentinel flushconfig
OK
```

##### 10.sentinel remove<master name>

取消当前Sentinel节点对于指定<master name>主节点的监控。例如sentinel-1当前对mymaster-1进行了监控：

```
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:2
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster-2,status=ok,address=127.0.0.1:6382,slaves=2,sentinels=3
master1:name=mymaster-1,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

例如下面，sentinel-1节点取消对mymaster-1节点的监控，但是要注意这个命令仅仅对当前Sentinel节点有效。

```
127.0.0.1:26379> sentinel remove mymaster-1
OK
```

再执行info sentinel命令，发现sentinel-1已经失去对mymaster-1的监控：

```
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster-2,status=ok,address=127.0.0.1:6383,slaves=2,sentinels=3
```

##### 11.sentinel monitor<master name><ip><port><quorum>

这个命令和配置文件中的含义是完全一样的，只不过是通过命令的形式来完成Sentinel节点对主节点的监控。例如命令sentinel-1节点重新监控mymaster-1节点：

```
127.0.0.1:26379> sentinel monitor mymaster-1 127.0.0.1 6379 2
OK
```

命令执行后，发现sentinel-1节点重新对mymaster-1节点进行监控：

```
# Sentinel
sentinel_masters:2
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
master0:name=mymaster-2,status=ok,address=127.0.0.1:6383,slaves=2,sentinels=3
master1:name=mymaster-1,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

##### 12.sentinel set<master name>

动态修改Sentinel节点配置选项，这个命令已经在9.2.4小节进行了说明，这里就不赘述了。

##### 13.sentinel is-master-down-by-addrSentinel

节点之间用来交换对主节点是否下线的判断，根据参数的不同，还可以作为Sentinel领导者选举的通信方式，具体细节9.5节会介绍。

### 客户端连接

通过前面的学习，相信读者对Redis Sentinel有了一定的了解，本节将介绍应用方如何正确地连接Redis Sentinel。有人会说这有什么难的，已经知道了主节点的ip地址和端口，用对应编程语言的客户端连接主节点不就可以了吗？但试想一下，如果这样使用客户端，客户端连接Redis Sentinel和主从复制的Redis又有什么区别呢，如果主节点挂掉了，虽然Redis Sentinel可以完成故障转移，但是客户端无法获取这个变化，那么使用Redis Sentinel的意义就不大了，所以各个语言的客户端需要对Redis Sentinel进行显式的支持。

#### Redis Sentinel的客户端

Sentinel节点集合具备了监控、通知、自动故障转移、配置提供者若干功能，也就是说实际上最了解主节点信息的就是Sentinel节点集合，而各个主节点可以通过<master-name>进行标识的，所以，无论是哪种编程语言的客户端，如果需要正确地连接Redis Sentinel，必须有Sentinel节点集合和masterName两个参数。

#### Redis Sentinel客户端基本实现原理

实现一个Redis Sentinel客户端的基本步骤如下：1)遍历Sentinel节点集合获取一个可用的Sentinel节点，后面会介绍Sentinel节点之间可以共享数据，所以从任意一个Sentinel节点获取主节点信息都是可以的，如图9-22所示。2)通过sentinel get-master-addr-by-name master-name这个API来获取对应主节点的相关信息，如图9-23所示。3)验证当前获取的“主节点”是真正的主节点，这样做的目的是为了防止故障转移期间主节点的变化，如图9-24所示。

获取一个可用的Sentinel节点

![image-20251128205009539](images/image-20251128205009539.png)

![image-20251128205021396](images/image-20251128205021396.png)

利用sentinel get-master-addr-by-name返回主节点信息

4)保持和Sentinel节点集合的“联系”，时刻获取关于主节点的相关“信息”，如图9-25所示。

![image-20251128205056092](images/image-20251128205056092.png)

从上面的模型可以看出，Redis Sentinel客户端只有在初始化和切换主节点时需要和Sentinel节点集合进行交互来获取主节点信息，所以在设计客户端时需要将Sentinel节点集合考虑成配置（相关节点信息和变化）发现服务。上述过程只是从客户端设计的角度进行分析，在开发客户端时要考虑的细节还有很多，但是这些问题并不需要深究，下面将介绍如何使用Java的Redis客户端操作Redis Sentinel，并结合本节的内容分析一下相关源码。

#### Java操作Redis Sentinel

### 实现原理

#### 三个定时监控任务

一套合理的监控机制是Sentinel节点判定节点不可达的重要保证，Redis Sentinel通过三个定时监控任务完成对各个节点发现和监控：

1)每隔10秒，每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构

![image-20251218101326602](./assets/image-20251218101326602.png)

例如下面就是在一个主节点上执行info replication的结果片段：

```
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=4917,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=4917,lag=1
```



Sentinel节点通过对上述结果进行解析就可以找到相应的从节点。



这个定时任务的作用具体可以表现在三个方面：

- 通过向主节点执行info命令，获取从节点的信息，这也是为什么Sentinel节点不需要显式配置监控从节点。
- 当有新的从节点加入时都可以立刻感知出来。
- 节点不可达或者故障转移后，可以通过info命令实时更新节点拓扑信息。



2)每隔2秒，每个Sentinel节点会向Redis数据节点的__sentinel__：hello频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息，同时每个Sentinel节点也会订阅该频道，来了解其他Sentinel节点以及它们对主节点的判断，所以这个定时任务可以完成以下两个工作：

![image-20251218101802650](./assets/image-20251218101802650.png)

- 发现新的Sentinel节点：通过订阅主节点的__sentinel__：hello了解其他的Sentinel节点信息，如果是新加入的Sentinel节点，将该Sentinel节点信息保存起来，并与该Sentinel节点创建连接。
- Sentinel节点之间交换主节点的状态，作为后面客观下线以及领导者选举的依据。

Sentinel节点publish的消息格式如下：

```
<Sentinel节点IP> <Sentinel节点端口> <Sentinel节点runId> <Sentinel节点配置版本>
    <主节点名字> <主节点Ip> <主节点端口> <主节点配置版本>
```

3)每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达。通过上面的定时任务，Sentinel节点对主节点、从节点、其余Sentinel节点都建立起连接，实现了对每个节点的监控，这个定时任务是节点失败判定的重要依据。

![image-20251218101715629](./assets/image-20251218101715629.png)

#### 主观下线和客观下线

##### 1.主观下线

每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测，当这些节点超过down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。从字面意思也可以很容易看出主观下线是当前Sentinel节点的一家之言，存在误判的可能

![image-20251221110143185](./assets/image-20251221110143185.png)

![image-20251220170015876](./assets/image-20251220170015876-1766221216459-2.png)

##### 2.客观下线

当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is-master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过<quorum>个数，Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定，这样客观下线的含义是比较明显了，也就是大部分Sentinel节点都对主节点的下线做了同意的判定，那么这个判定就是客观的

![image-20251220170159264](./assets/image-20251220170159264.png)

注意

从节点、Sentinel节点在主观下线后，没有后续的故障转移操作。这里有必要对sentinel is-master-down-by-addr命令做一个介绍，它的使用方法如下：

```
sentinel is-master-down-by-addr <ip> <port> <current_epoch> <runid>
```

- ip：主节点IP。
- port：主节点端口。
- current_epoch：当前配置纪元。
- runid：此参数有两种类型，不同类型决定了此API作用的不同。

当runid等于“*”时，作用是Sentinel节点直接交换对主节点下线的判定。

当runid等于当前Sentinel节点的runid时，作用是当前Sentinel节点希望目标Sentinel节点同意自己成为领导者的请求，有关Sentinel领导者选举，后面会进行介绍。

例如sentinel-1节点对主节点做主观下线后，会向其余Sentinel节点（假设sentinel-2和sentinel-3节点）发送该命令：

```
sentinel is-master-down-by-addr 127.0.0.1 6379 0 *
```

返回结果包含三个参数，如下所示：

- down_state：目标Sentinel节点对于主节点的下线判断，1是下线，0是在线。
- leader_runid：当leader_runid等于“*”时，代表返回结果是用来做主节点是否不可达，当leader_runid等于具体的runid，代表目标节点同意runid成为领导者。
- leader_epoch：领导者纪元。

#### 领导者Sentinel节点选举

假如Sentinel节点对于主节点已经做了客观下线，那么是不是就可以立即进行故障转移了？当然不是，实际上故障转移的工作只需要一个Sentinel节点来完成即可，所以Sentinel节点之间会做一个领导者选举的工作，选出一个Sentinel节点作为领导者进行故障转移的工作。Redis使用了Raft算法实现领导者选举，因为Raft算法相对比较抽象和复杂，以及篇幅所限，所以这里给出一个Redis Sentinel进行领导者选举的大致思路：

1)每个在线的Sentinel节点都有资格成为领导者，当它确认主节点主观下线时候，会向其他Sentinel节点发送sentinel is-master-down-by-addr命令，要求将自己设置为领导者。2)收到命令的Sentinel节点，如果没有同意过其他Sentinel节点的sentinel is-master-down-by-addr命令，将同意该请求，否则拒绝。3)如果该Sentinel节点发现自己的票数已经大于等于max(quorum，num(sentinels)/2+1)，那么它将成为领导者。4)如果此过程没有选举出领导者，将进入下一次选举。

![image-20251220171600465](./assets/image-20251220171600465.png)

1)s1(sentinel-1)最先完成了客观下线，它会向s2(sentinel-2)和s3(sentinel-3)发送sentinel is-master-down-by-addr命令，s2和s3同意选其为领导者。2)s1此时已经拿到2张投票，满足了大于等于max(quorum，num(sentinels)/2+1)=2的条件，所以此时s1成为领导者。

由于每个Sentinel节点只有一票，所以当s2向s1和s3索要投票时，只能获取一票，而s3由于最后完成主观下线，当s3向s1和s2索要投票时一票都得不到

s2节点收到s1节点的同意票，s3节点的拒绝票

![image-20251220171655446](./assets/image-20251220171655446.png)

s2节点收到s1节点的同意票，s3节点的拒绝票

![image-20251220171731063](./assets/image-20251220171731063.png)

实际上Redis Sentinel实现会更简单一些，因为一旦有一个Sentinel节点获得了max(quorum，num(sentinels)/2+1)的票数，其他Sentinel节点再去确认已经没有意义了，因为每个Sentinel节点只有一票，如果读者有兴趣的话，可以修改sentinel.c源码，在Sentinel的执行命令列表中添加monitor命令：

```
struct redisCommand sentinelcmds[] = {
    {"monitor",monitorCommand,1,"",0,NULL,0,0,0,0,0},
    {"ping",pingCommand,1,"",0,NULL,0,0,0,0,0},
    {"sentinel",sentinelCommand,-2,"",0,NULL,0,0,0,0,0},
…
}
```

重新编译部署Redis Sentinel测试环境，在3个Sentinel节点上执行monitor命令：1)可以看到sentinel is-master-down-by-addr命令，此命令的执行过程并没有在Redis的日志中有所体现，monitor监控类似如下命令：

```
// 因为最后参数是"*"，所以此时是Sentinel节点之间交换对主节点的失败判定
[0 127.0.0.1:38440] "SENTINEL" "is-master-down-by-addr" "127.0.0.1" "6379" "0" "*"
// 因为最后参数是具体的runid，所以此时代表runid="2f4430bb62c039fb125c5771d7cde2571a7
    a5ab4"的节点希望目标Sentinel节点同意自己成为领导者。
[0 127.0.0.1:38440] "SENTINEL" "is-master-down-by-addr" "127.0.0.1" "6379" "1" 
    "2f4430bb62c039fb125c5771d7cde2571a7a5ab4"
```

2)选举的过程非常快，基本上谁先完成客观下线，谁就是领导者。3)一旦Sentinel得到足够的票数，不存在图9-32和图9-33的过程。注意有关Raft算法可以参考其GitHub主页https://raft.github.io/。

##### Raft算法

![image-20251221111150383](./assets/image-20251221111150383.png)

#### 故障转移

领导者选举出的Sentinel节点负责故障转移，具体步骤如下：

1)在从节点列表中选出一个节点作为新的主节点，选择方法如下：

a)过滤：“不健康”（主观下线、断线）、5秒内没有回复过Sentinel节点ping响应、与主节点失联超过down-after-milliseconds*10秒。

b)选择slave-priority（从节点优先级）最高的从节点列表，如果存在则返回，不存在则继续。

![image-20251221112554252](./assets/image-20251221112554252.png)

c)选择复制偏移量最大的从节点（复制的最完整），如果存在则返回，不存在则继续。

d)选择runid最小的从节点。

![image-20251221112443098](./assets/image-20251221112443098.png)



![image-20251220171953662](./assets/image-20251220171953662.png)

2)Sentinel领导者节点会对第一步选出来的从节点执行slaveof no one命令让其成为主节点。

3)Sentinel领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点，复制规则和parallel-syncs参数有关。

4)Sentinel节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点。

### 开发与运维中的问题

#### 故障转移日志分析

##### 1.Redis Sentinel拓扑结构

Redis Sentinel拓扑表

![image-20251220201229948](./assets/image-20251220201229948.png)

因为故障转移涉及节点关系的变化，所以下面说明中用端口号代表节点。

##### 2.开始故障转移测试

模拟故障的方法有很多，比较典型的方法有以下几种：

- 方法一，强制杀掉对应节点的进程号，这样可以模拟出宕机的效果。
- 方法二，使用Redis的debug sleep命令，让节点进入睡眠状态，这样可以模拟阻塞的效果。
- 方法三，使用Redis的shutdown命令，模拟正常的停掉Redis。

本次我们使用方法一进行测试，因为从实际经验来看，数百上千台机器偶尔宕机一两台是会不定期出现的，为了方便分析日志行为，这里记录一下操作的时间和命令。用kill-9使主节点的进程宕机，操作时间2016-07-2409：40：35：

```
$ kill -9 19661
```

3.观察效果

6380节点晋升为主节点，6381节点成为6380节点的从节点。

4.故障转移分析

相信故障转移的效果和预想的一样，这里重点分析相应节点的日志。(1)6379节点日志两个复制请求，分别来自端口为6380和6381的从节点：

```
19661:M 24 Jul 09:22:16.907 * Slave 127.0.0.1:6380 asks for synchronization
19661:M 24 Jul 09:22:16.907 * Full resync requested by slave 127.0.0.1:6380
…
19661:M 24 Jul 09:22:16.919 * Synchronization with slave 127.0.0.1:6380 succeeded
19661:M 24 Jul 09:22:23.396 * Slave 127.0.0.1:6381 asks for synchronization
19661:M 24 Jul 09:22:23.396 * Full resync requested by slave 127.0.0.1:6381
…
19661:M 24 Jul 09:22:23.432 * Synchronization with slave 127.0.0.1:6381 succeeded
```

09：40：35做了kill-9操作，由于模拟的是宕机效果，所以6379节点没有看到任何日志（这点和shutdown操作不太相同）。

(2)6380节点日志

6380节点在09：40：35之后发现它与6379节点已经失联：

```
19667:S 24 Jul 09:40:35.788 # Connection with master lost.
19667:S 24 Jul 09:40:35.788 * Caching the disconnected master state.
19667:S 24 Jul 09:40:35.974 * Connecting to MASTER 127.0.0.1:6379
19667:S 24 Jul 09:40:35.974 * MASTER <-> SLAVE sync started
19667:S 24 Jul 09:40:35.975 # Error condition on socket for SYNC: Connection 
refused
…
```

09：41：06时它接到Sentinel节点的命令：清理原来缓存的主节点状态，Sentinel节点将6380节点晋升为主节点，并重写配置：

```
19667:M 24 Jul 09:41:06.161 * Discarding previously cached master state.
19667:M 24 Jul 09:41:06.161 * MASTER MODE enabled (user request from 'id=7 
    addr=127.0.0.1:46759 fd=10 name=sentinel-7044753f-cmd age=1111 idle=0 
    flags=x db=0 sub=0 psub=0 multi=3 qbuf=0 qbuf-free=32768 obl=36 oll=0 
    omem=0 events=rw cmd=exec')
19667:M 24 Jul 09:41:06.161 # CONFIG REWRITE executed with success.
```

6381节点发来了复制请求：

```
19667:M 24 Jul 09:41:07.499 * Slave 127.0.0.1:6381 asks for synchronization
19667:M 24 Jul 09:41:07.499 * Full resync requested by slave 127.0.0.1:6381
…
19667:M 24 Jul 09:41:07.548 * Background saving terminated with success
19667:M 24 Jul 09:41:07.548 * Synchronization with slave 127.0.0.1:6381 succeeded
```

(3)6381节点日志6381节点同样与6379节点失联：

```
19685:S 24 Jul 09:40:35.788 # Connection with master lost.
19685:S 24 Jul 09:40:35.788 * Caching the disconnected master state.
19685:S 24 Jul 09:40:36.425 * Connecting to MASTER 127.0.0.1:6379
19685:S 24 Jul 09:40:36.425 * MASTER <-> SLAVE sync started
19685:S 24 Jul 09:40:36.425 # Error condition on socket for SYNC: Connection refused
…
```

后续操作如下：1)09：41：06时它接到Sentinel节点的命令，清理原来缓存的主节点状态，让它去复制新的主节点（6380节点）：

```
19685:S 24 Jul 09:41:06.497 # Error condition on socket for SYNC: Connection refused
19685:S 24 Jul 09:41:07.008 * Discarding previously cached master state.
19685:S 24 Jul 09:41:07.008 * SLAVE OF 127.0.0.1:6380 enabled (user request 
    from 'id=7 addr=127.0.0.1:55872 fd=10 name=sentinel-7044753f-cmd age=1111 
    idle=0 flags=x db=0 sub=0 psub=0 multi=3 qbuf=133 qbuf-free=32635 obl=36 
    oll=0 omem=0 events=rw cmd=exec')
19685:S 24 Jul 09:41:07.008 # CONFIG REWRITE executed with success.
```

2)向新的主节点（6380节点）发起复制操作：

```
19685:S 24 Jul 09:41:07.498 * Connecting to MASTER 127.0.0.1:6380
…
19685:S 24 Jul 09:41:07.549 * MASTER <-> SLAVE sync: Finished with success
```

(4)sentinel-1节点日志09：41：05对6379节点作了主观下线(+sdown)，注意这个时间正好是kill-9后的30秒，和down-after-milliseconds的配置是一致的。Sentinel节点更新自己的配置纪元(new-epoch)：

```
19697:X 24 Jul 09:41:05.850 # +sdown master mymaster 127.0.0.1 6379
19697:X 24 Jul 09:41:05.928 # +new-epoch 1
```

后续操作如下：

1)投票给sentinel-3节点：

```
19697:X 24 Jul 09:41:05.929 # +vote-for-leader 7044753f564e42b1578341acf4c49dca
    3681151c 1
19697:X 24 Jul 09:41:06.913 # +odown master mymaster 127.0.0.1 6379 #quorum 3/2
```

2)更新状态：从sentinel-3节点（领导者）得知：故障转移后6380节点变为主节点，并发现了两个从节点6381和6379，并在30秒后对(09：41：07~09：41：37)6379节点做了主观下线：

```
19697:X 24 Jul 09:41:06.913 # Next failover delay: I will not start a failover
     before Sun Jul 24 09:47:06 2016
19697:X 24 Jul 09:41:07.008 # +config-update-from sentinel 127.0.0.1:26381 
    127.0.0.1 26381 @ mymaster 127.0.0.1 6379
19697:X 24 Jul 09:41:07.008 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
19697:X 24 Jul 09:41:07.008 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ 
    mymaster 127.0.0.1 6380
19697:X 24 Jul 09:41:07.008 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ 
    mymaster 127.0.0.1 6380
19697:X 24 Jul 09:41:37.060 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ 
    mymaster 127.0.0.1 6380
```

(5)sentinel-2节点日志

整个过程和sentinel-1节点是一样的，这里就不占用篇幅分析了。

(6)sentinel-3节点日志

从sentinel-1节点和sentinel-2节点的日志来看，sentinel-3节点是领导者，所以分析sentinel-3节点的日志至关重要。后续操作如下。

1)达到了客观下线的条件：

```
19713:X 24 Jul 09:41:05.854 # +sdown master mymaster 127.0.0.1 6379
19713:X 24 Jul 09:41:05.909 # +odown master mymaster 127.0.0.1 6379 #quorum 2/2
19713:X 24 Jul 09:41:05.909 # +new-epoch 1
```

2)sentinel-3节点被选为领导者：

```
19713:X 24 Jul 09:41:05.909 # +try-failover master mymaster 127.0.0.1 6379
19713:X 24 Jul 09:41:05.911 # +vote-for-leader 7044753f564e42b1578341acf4c49dca
    3681151c 1
19713:X 24 Jul 09:41:05.929 # 127.0.0.1:26379 voted for 7044753f564e42b1578341a
    cf4c49dca3681151c 1
19713:X 24 Jul 09:41:05.930 # 127.0.0.1:26380 voted for 7044753f564e42b1578341a
    cf4c49dca3681151c 1
19713:X 24 Jul 09:41:06.001 # +elected-leader master mymaster 127.0.0.1 6379
```

展示了3个Sentinel节点完成客观下线的时间点，从时间点可以看到sentinel-3节点最先完成客观下线。

![image-20251220202002905](./assets/image-20251220202002905.png)

3)故障转移。每一步都可以通过发布订阅来获取，对于每个字段的说明可以参考表9-6。

寻找合适的从节点作为新的主节点：

```
19713:X 24 Jul 09:41:06.001 # +failover-state-select-slave master mymaster
    127.0.0.1 6379
```

选出了合适的从节点（6380节点）：

```
19713:X 24 Jul 09:41:06.077 # +selected-slave slave 127.0.0.1:6380 127.0.0.1
    6380 @ mymaster 127.0.0.1 6379
```

命令6380节点执行slaveof no one，使其成为主节点：

```
19713:X 24 Jul 09:41:06.077 * +failover-state-send-slaveof-noone slave
    127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
```

等待6380节点晋升为主节点：

```
19713:X 24 Jul 09:41:06.161 * +failover-state-wait-promotion slave
    127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
```

确认6380节点已经晋升为主节点：

```
19713:X 24 Jul 09:41:06.927 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1
    6380 @ mymaster 127.0.0.1 6379
```

故障转移进入重新配置从节点阶段：

```
19713:X 24 Jul 09:41:06.927 # +failover-state-reconf-slaves master mymaster
    127.0.0.1 6379
```

命令6381节点复制新的主节点：

```
19713:X 24 Jul 09:41:07.008 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1
    6381 @ mymaster 127.0.0.1 6379
```

6381节点正在重新配置成为6380节点的从节点，但是同步过程尚未完成：

```
19713:X 24 Jul 09:41:07.955 * +slave-reconf-inprog slave 127.0.0.1:6381
    127.0.0.1 6381 @ mymaster 127.0.0.1 6379
```

6381节点完成对6380节点的同步：

```
19713:X 24 Jul 09:41:07.955 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1
    6381 @ mymaster 127.0.0.1 6379
```

故障转移顺利完成：

```
19713:X 24 Jul 09:41:08.045 # +failover-end master mymaster 127.0.0.1 6379
```

故障转移成功后，发布主节点的切换消息：

```
19713:X 24 Jul 09:41:08.045 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
```

表9-6记录了Redis Sentinel在故障转移一些重要的事件消息对应的频道。表9-6　Sentinel节点发布订阅频道

![image-20251220202322866](./assets/image-20251220202322866.png)

<instance details>格式如下：

```
<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>
```

5.原主节点后续处理

重新启动原来的6379节点：

```
redis-server redis-6379.conf操作时间: 2016-07-24 09:46:21
```

(1)6379节点

启动后接到Sentinel节点的命令，让它去复制6380节点：

```
22223:M 24 Jul 09:46:21.260 * The server is now ready to accept connections on
    port 6379
22223:S 24 Jul 09:46:31.323 * SLAVE OF 127.0.0.1:6380 enabled (user request 
    from 'id=2 addr=127.0.0.1:51187 fd=6 name=sentinel-94dde2f5-cmd age=10 
    idle=0 flags=x db=0 sub=0 psub=0 multi=3 qbuf=0 qbuf-free=32768 obl=36 
    oll=0 omem=0 events=rw cmd=exec')
        22223:S 24 Jul 09:46:31.323 # CONFIG REWRITE executed with success.
        …
```

(2)6380节点

接到6379节点的复制请求，做复制的相应处理：

```
19667:M 24 Jul 09:46:32.284 * Slave 127.0.0.1:6379 asks for synchronization
19667:M 24 Jul 09:46:32.284 * Full resync requested by slave 127.0.0.1:6379
…
19667:M 24 Jul 09:46:32.353 * Synchronization with slave 127.0.0.1:6379 succeeded
```

(3)sentinel-1节点日志

撤销对6379节点主观下线的决定：

```
19707:X 24 Jul 09:46:21.406 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @
    mymaster 127.0.0.1 6380
```

(4)sentinel-2节点日志

撤销对6379节点主观下线的决定：

```
19713:X 24 Jul 09:46:21.408 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @
    mymaster 127.0.0.1 6380
```

(5)sentinel-3节点日志

撤销对6379节点主观下线的决定，更新Sentinel节点配置：

```
19697:X 24 Jul 09:46:21.367 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @
    mymaster 127.0.0.1 6380
19697:X 24 Jul 09:46:31.322 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 
    6379 @ mymaster 127.0.0.1 6380
```

6.注意点

部署各个节点的机器时间尽量要同步，否则日志的时序性会混乱，例如可以给机器添加NTP服务来同步时间，具体可以参考第12章Linux配置章节。

#### 节点运维

##### 1.节点下线

在介绍如何进行节点下线之前，首先需要弄清两个概念：临时下线和永久下线。

- 临时下线：暂时将节点关掉，之后还会重新启动，继续提供服务。
- 永久下线：将节点关掉后不再使用，需要做一些清理工作，如删除配置文件、持久化文件、日志文件。

所以运维人员需要弄清楚本次下线操作是临时下线还是永久下线。通常来看，无论是主节点、从节点还是Sentinel节点，下线原因无外乎以下几种：

- 节点所在的机器出现了不稳定或者即将过保被回收。
- 节点所在的机器性能比较差或者内存比较小，无法支撑应用方的需求。
- 节点自身出现服务不正常情况，需要快速处理。

(1)主节点

如果需要对主节点进行下线，比较合理的做法是选出一个“合适”（例如性能更高的机器）的从节点，使用sentinel failover功能将从节点晋升主节点，sentinel failover已经在9.3节介绍过了，只需要在任意可用的Sentinel节点执行如下操作即可。

```
sentinel failover <master name>
```

在任意一个Sentinel节点上（例如26379端口节点）执行sentinel failover即可。

![image-20251221113852107](./assets/image-20251221113852107.png)

运维提示

Redis Sentinel存在多个从节点时，如果想将指定从节点晋升为主节点，可以将其他从节点的slavepriority配置为0，但是需要注意failover后，将slave-priority调回原值。

(2)从节点和Sentinel节点

如果需要对从节点或者Sentinel节点进行下线，只需要确定好是临时还是永久下线后执行相应操作即可。如果使用了读写分离，下线从节点需要保证应用方可以感知从节点的下线变化，从而把读取请求路由到其他节点。需要注意的是，Sentinel节点依然会对这些下线节点进行定期监控，这是由Redis Sentinel的设计思路所决定的。下面日志显示（需要设置loglevel=debug），6380节点下线后，Sentinel节点还是会定期对其监控，会造成一定的网络资源浪费。

```
-cmd-link slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
    #Connection refused
-pubsub-link slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379 
    #Connection refused
…
```

##### 2.节点上线

(1)添加从节点

添加从节点的场景大致有如下几种：

- 使用了读写分离，但现有的从节点无法支撑应用方的流量。
- 主节点没有可用的从节点，无法支持故障转移。
- 添加一个更强悍的从节点利用手动failover替换主节点。

添加方法：添加slaveof{masterIp}{masterPort}的配置，使用redis-server启动即可，它将被Sentinel节点自动发现。

(2)添加Sentinel节点

添加Sentinel节点的场景可以分为以下几种：

- 当前Sentinel节点数量不够，无法达到Redis Sentinel健壮性要求或者无法达到票数。
- 原Sentinel节点所在机器需要下线。

添加方法：添加sentinel monitor主节点的配置，使用redis-sentinel启动即可，它将被其余Sentinel节点自动发现。

(3)添加主节点

因为Redis Sentinel中只能有一个主节点，所以不需要添加主节点，如果需要替换主节点，可以使用Sentinel failover手动故障转移。

##### 3.节点配置

有关Redis数据节点和Sentinel节点配置修改以及优化的方法，前面的章节已经介绍过了，这里给出Sentinel节点配置时要注意的地方：

- Sentinel节点配置尽可能一致，这样在判断节点故障时会更加准确。
- Sentinel节点支持的命令非常有限，例如config命令是不支持的，而Sentinel节点也需要dir、loglevel之类的配置，所以尽量在一开始规划好，不过所幸Sentinel节点不存储数据，如果需要修改配置，重新启动即可。

运维提示

Sentinel节点只支持如下命令：ping、sentinel、subscribe、unsubscribe、psubscribe、punsubscribe、publish、info、role、client、shutdown。具体可以参考源码中sentinel.c。上面介绍了Redis Sentinel节点运维的场景和方法，但在实际运维中，故障的发生通常比较突然并且瞬息万变，影响的范围也很难预估，所以建议运维人员将上述场景提前做好预案，当事故发生时，可以用脚本或者可视化工具快速处理故障。

#### 高可用读写分离

##### 1.从节点的作用

从节点一般可以起到两个作用：第一，当主节点出现故障时，作为主节点的后备“顶”上来实现故障转移，Redis Sentinel已经实现了该功能的自动化，实现了真正的高可用。第二，扩展主节点的读能力，尤其是在读多写少的场景非常适用，通常的模型如图9-36所示。

一般的读写分离模型

![image-20251221114332275](./assets/image-20251221114332275.png)

但上述模型中，从节点不是高可用的，如果slave-1节点出现故障，首先客户端client-1将与其失联，其次Sentinel节点只会对该节点做主观下线，因为Redis Sentinel的故障转移是针对主节点的。所以很多时候，Redis Sentinel中的从节点仅仅是作为主节点一个热备，不让它参与客户端的读操作，就是为了保证整体高可用性，但实际上这种使用方法还是有一些浪费，尤其是在有很多从节点或者确实需要读写分离的场景，所以如何实现从节点的高可用是非常有必要的。

##### 2.Redis Sentinel读写分离设计思路

Redis Sentinel在对各个节点的监控中，如果有对应事件的发生，都会发出相应的事件消息（见表9-6），其中和从节点变动的事件有以下几个：

- +switch-master：切换主节点（原来的从节点晋升为主节点），说明减少了某个从节点。
- +convert-to-slave：切换从节点（原来的主节点降级为从节点），说明添加了某个从节点。
- +sdown：主观下线，说明可能某个从节点可能不可用（因为对从节点不会做客观下线），所以在实现客户端时可以采用自身策略来实现类似主观下线的功能。
- +reboot：重新启动了某个节点，如果它的角色是slave，那么说明添加了某个从节点。

所以在设计Redis Sentinel的从节点高可用时，只要能够实时掌握所有从节点的状态，把所有从节点看做一个资源池（如图9-37所示），无论是上线还是下线从节点，客户端都能及时感知到（将其从资源池中添加或者删除），这样从节点的高可用目标就达到了。

![image-20251221114453177](./assets/image-20251221114453177.png)

# 集群

- 作用
  - Redis集群支持多个Master，每个Master又可以挂载多个Slave
    - 读写分离
    - 支持数据的高可用
    - 支持海量数据的读写存储操作
  - 由于Cluster自带Sentinel的故障转移机制，内置了高可用的支持，无需再去使用哨兵功能
  - 客户端与Redis的节点连接，不再需要连接集群中所有的节点，只需要任意连接集群中的一个可用节点即可
  - 槽位slot负责分配到各个物理服务节点，由对应的集群来负责维护节点、插槽和数据之间的关系



![image-20251221174559435](./assets/image-20251221174559435.png)



## 数据分布

### 数据分布理论

分布式数据库首先要解决把整个数据集按照分区规则映射到多个节点的问题，即把数据集划分到多个节点上，每个节点负责整体数据的一个子集。

分布式存储数据分区

![image-20251221161936248](./assets/image-20251221161936248.png)

需要重点关注的是数据分区规则。常见的分区规则有哈希分区和顺序分区两种

哈希分区和顺序分区对比

![image-20251221162026317](./assets/image-20251221162026317.png)

由于Redis Cluster采用哈希分区规则，这里我们重点讨论哈希分区，常见的哈希分区规则有几种，下面分别介绍。

#### 1.节点取余分区

- 原理：`slot=hash(key)%N`（N为节点数）
- 优点：实现简单，数据分布均匀。常用于数据库的分库分表规则，一般采用预分区的方式，提前根据数据量规划好分区数，比如划分为512或1024张表，保证可支撑未来一段时间的数据量，再根据负载情况将表迁移到其他数据库中。
- 缺点：节点数量变化时，如扩容或收缩节点，需**重新计算所有数据的槽位**，导致大规模数据迁移
- 优化方案：扩容时通常采用翻倍扩容，使数据迁移比例从80%降至50%，避免数据映射全部被打乱导致全量迁移的情况

![image-20251221162157051](./assets/image-20251221162157051.png)

#### 2.一致性哈希分区

- 原理：将节点和数据映射到**0～2^32-1的哈希环**上，数据顺时针查找最近的节点。实现思路是为系统中每个节点分配一个token，范围一般在0~2的32次方，这些token构成一个哈希环。数据读写执行节点查找操作时，先根据key计算hash值，然后顺时针找到第一个大于等于该哈希值的token节点
- **优点**：节点增减时**仅影响相邻节点**，数据迁移量小（如4节点扩容到5节点仅迁移20%数据）
- 缺点：
  - **数据分布不均**：节点在环上位置随机，可能导致负载倾斜
  - **少量节点时效果差**：2-3个节点时，节点变化影响范围大
  - **需配合虚拟节点**：为解决分布不均问题，需为每个物理节点创建多个虚拟节点1216

- 当我们需要存储一个kv健值对时,首先计算key的hash值, hash(key),将这个key使用相同的函数Hash计算出哈希值并确定此数据在环上的位置,从此位置沿环殿时针“行走”，第一台遇到的服务器就是其应该定位到的服务器，并将该键值对存储在该节点上。

- 如我们有Object A, Object B，Object C，Object D四个数据对象,经过哈希计算后,在环空间上的位置如下，根据一致性Hash算法，数据A会被定为到Node A上，B被定为到Node B上，C被定为到Node C上，D被定为到Node D上。

- 容错性：假设Node C宕机,可以看到此时对象A,B.D不会受到影响。一般的,在一致性Hash算法中,**如果一台服务器不可期,则受影响的数据仅仅是此服务器到其环空间中前一台服务器(即沿着逆时针方向行走遇到的第一台服务器)之间数据,**其它不会受到影响。简单说,就是C挂了,受到影响的只是B C之间的数据且这些数据会转移到D进行在存储

![image-20251221182644130](./assets/image-20251221182644130.png)



![image-20251221183031669](./assets/image-20251221183031669.png)



![image-20251221162248036](./assets/image-20251221162248036.png)



扩展性：数据量增加了，需要增加一台节点NodeX. X的位置在A和B之间，那收到影响的也就是A到X之间的数据，重新把A到X的数据录入到X上即可，不会导致hash取余全部数据重新洗牌.

![image-20251221183323150](./assets/image-20251221183323150.png)



这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响。但一致性哈希分区存在几个问题：

- 加减节点会造成哈希环中部分数据无法命中，需要手动处理或者忽略这部分数据，因此一致性哈希常用于缓存场景。

- 当使用少量节点时，节点变化将大范围影响哈希环中数据映射，因此这种方式不适合少量数据节点的分布式方案。一致性Hash算法在服务节点太少时，容易因为节点分布不均匀而造成数据倾斜（被缓存的对象大部分集中缓存在某一台服务器上）问题，例如系统中只有两台服务器

  ![image-20251221183541820](./assets/image-20251221183541820.png)

  

- 普通的一致性哈希分区在增减节点时需要增加一倍或减去一半节点才能保证数据和负载的均衡。

正因为一致性哈希分区的这些缺点，一些分布式系统采用虚拟槽对一致性哈希进行改进，比如Dynamo系统。

#### 3.虚拟槽分区

- 原理：使用**固定数量的槽**（16384）作为中间层，解耦数据与节点的直接映射
- 核心优势：
  - **解耦数据与节点关系**：新增/删除节点时只需迁移部分槽位，**无需停机**
  - **节点自主维护**：每个节点自身维护槽位映射，**无需客户端或代理维护元数据**
  - **支持动态伸缩**：通过`CLUSTER REBALANCE`命令自动调整槽分配

- 哈希槽实质就是一个数组，数组[0,2^14 -1]形成hash slot空间，

- 实例：当前集群有5个节点，每个节点平均大约负责3276个槽。由于采用高质量的哈希算法，每个槽所映射的数据通常比较均匀，将数据平均划分到5个节点进行数据分区。Redis Cluster就是采用虚拟槽分区，下面就介绍Redis数据分区方法。

![image-20251221162457521](./assets/image-20251221162457521.png)

### Redis数据分片

- 槽位分配与计算
  - **固定槽位总数**：Redis Cluster将数据空间划分为**16384个哈希槽**（0-16383）
  - **槽位计算公式**：`slot = CRC16(key) % 16384`，其中CRC16使用**CRC16-CCITT标准算法**（多项式0x1021）
  - **哈希标签支持**：键中包含`{}`时（如`{user:1000}:profile`），**仅对花括号内内容计算哈希**，确保相关键存储在同一槽位



![image-20251221180030966](./assets/image-20251221180030966.png)



![image-20251221184307100](./assets/image-20251221184307100.png)



![image-20251221162610194](./assets/image-20251221162610194.png)

- Redis虚拟槽分区的特点：
  - **解耦数据与节点关系**：新增/删除节点时只需迁移部分槽位，**无需停机**
  - **节点自主维护**：每个节点自身维护槽位映射，**无需客户端或代理维护元数据**
  - **支持动态伸缩**：通过`CLUSTER REBALANCE`命令自动调整槽分配
  - **数据迁移**：扩容时只需将部分槽位从旧节点迁移到新节点，**避免全量数据迁移**

### 为什么redis集群的最大槽数是16384个？

- 通信效率优化
  1. 心跳包大小控制
     - Redis节点通过**Gossip协议**定期交换集群状态信息，包括槽位分配情况
     - 槽位状态通过**位图(bitmap)**存储，16384个槽位仅需 **2KB** 内存（16384/8=2048字节）
     - 若使用65536个槽位，位图大小将增至 **8KB**（65536/8=8192字节）
     - 节点每秒需发送多个心跳包，**2KB比8KB节省75%的带宽**，显著降低网络负载4
  2. 传输效率提升
     - 位图在传输过程中会进行压缩，当节点数较少时，16384个槽位的位图**压缩率更高**
     - 位图压缩率公式：`压缩率 = slots / N`（N为节点数）
     - 节点数少时，65536个槽位会导致**压缩率过低**，传输效率下降
- 实际集群规模限制
  1. 官方建议的节点上限
     - Redis作者明确表示**不建议集群节点超过1000个**，这是基于大量实践经验的建议
     - 对于1000个节点的集群，16384个槽位可确保**每个节点平均负责16个槽位**
     - 若使用65536个槽位，每个节点仅负责约65个槽位，**槽位管理开销过大**5
  2. 网络拥堵预防
     - 节点数量增加时，心跳包中携带的数据量呈指数级增长
     - 超过1000个节点会导致**网络拥堵风险显著增加**
     - 16384个槽位在满足大多数业务场景的同时，**避免了不必要的网络负担**

- 数据分布与管理平衡
  1. 分布均匀性保障
     - 16384个槽位提供了**足够精细的数据分布粒度**
     - 通过`HASH_SLOT = CRC16(key) % 16384`计算，确保数据在节点间**均匀分布**
     - 槽位过少（如1024个）会导致**单个槽位数据量过大**，迁移成本高
  2. 扩容缩容灵活性
     - 新增节点时，只需将部分槽位从现有节点迁移到新节点
     - 16384个槽位提供了**合适的迁移粒度**，避免全量数据迁移
     - 槽位迁移过程**不会导致集群停机**，保证服务连续性11
  3. 计算效率优化
     - 16384是2^14，取模运算可通过**位运算高效实现**（`hash & 16383`）
     - 相比65536（2^16），在大多数机器上**计算速度更快**
     - CRC16算法本身输出16位值，16384个槽位**充分利用了算法特性**

CRC16算法产生的hash值有16bit.该算法可以产生2^16=65536个值. 换句话说值是分布在0~65535之间，有更大的65536不用为什么只用16384就够？ 作者在做modia算的时候,为什么不mod65536.而选择mod16384? HASH SLOT=CRC16(kev) mod 65536为什么没启用

正常的心跳数据包带有节点的完整配置，可以用幂等方式用旧的节点替换旧节点，以便更新旧的配置。 这意味着它们包含原始节点的插槽配置，该节点使用2k的空间和16k的插槽，但是会使用8k的空间（使用65k的插槽）。 同时，由于其他设计折衷，Redis集群不太可能扩展到1000个以上的主节点 

因此16k处于正确的范围内,以确保每个主机具有足够的插槽,最多可容纳1000个矩阵,但数量足够少,可以轻松地将插槽配置作为原始位图传播。请注意，在小型群集中，位图将难以压缩，因为当N较小时，位图将设置的slot/N位占设置位的很大百分比。

（1）如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大。 

在消息头中最占空间的是myslots[CLUSTER_SLOTS/8].当槽位为65536时，这块的大小是： 65536÷8÷1024=8kb]在消息头中最占空间的是myslots[CLUSTER_SLOTS/8].当槽位为16384，这块的大小是：16384÷8÷1024=2kb

因为每秒钟，redis节点需要发送一定数量的ping消息作为心跳包，如果槽位为65536，这个ping消息的消息头太大了，浪费带宽

(2)redis的集群主节点数量基本不可能超过1000个。 

集群节点越多,心跳包的消息体内携带的数据越多,如果节点过1000个,也会导致网络拥堵,因此redis作者不建议redis cluster节点数 超P,1000个，那么，对于节点数在1000以内的redis cluster集群，16384个槽位够用了，没有必要拓展到65536个。

(3)槽位越小,节点少的情况下,压缩比高,容易传输 

Redis主节点的配置信息中它所负责的哈希槽是通过一张bitmap的形式来保存的,在传输过程中会对bitmap进行压缩,但是如果bitmep的填充 slots/N很高的话(N表示节点数），bitmap的压缩率就很低。如果节点数很少，而哈希槽数量很多的话，bitmap的压缩率就很低。 

### 集群功能限制

Redis集群相对单机在功能上存在一些限制，需要开发人员提前了解，在使用时做好规避。限制如下：

1)key批量操作支持有限。如mset、mget，目前只支持具有相同slot值的key执行批量操作。对于映射为不同slot值的key由于执行mget、mget等操作可能存在于多个节点上因此不被支持。

2)key事务操作支持有限。同理只支持多key在同一节点上的事务操作，当多个key分布在不同的节点上时无法使用事务功能。

3)key作为数据分区的最小粒度，因此不能将一个大的键值对象如hash、list等映射到不同的节点。

4)不支持多数据库空间。单机下的Redis可以支持16个数据库，集群模式下只能使用一个数据库空间，即db0。

5)复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。

## 3主3从redis集群搭建

### 搭建步骤

例如：新建6个独立的redis示例服务器。Redis集群一般由多个节点组成，节点数量至少为6个才能保证组成完整高可用的集群。**三主三从架构是指3个主节点（Master）负责读写操作，3个从节点（Slave）负责数据备份和故障转移，形成高可用集群。**

![image-20251221191727897](./assets/image-20251221191727897.png)

1. 找3台真实虚拟机，各自新建  `/mkdir  -p  /myredis/cluster`

2. 第一台虚拟机   `IP：192.168.111.175+端口6381/端口6382`

   - `cluster-enabled yes`：开启集群模式
   - `cluster-node-timeout 15000`：节点超时时间，单位毫秒
   - `cluster-config-file "node-6379.conf"`：6379节点启动成功，第一次启动时如果没有集群配置文件，它会自动创建一份，文件名称采用cluster-config-file参数项控制，建议采用node-{port}.conf格式定义，通过使用端口号区分不同节点，防止同一机器下多个节点彼此覆盖，造成集群信息异常。如果启动时存在集群配置文件，节点会使用配置文件内容初始化集群信息。

   - 执行：
     - `vim /myredis/cluster/redisCluster6381.conf`
     - ``vim /myredis/cluster/redisCluster6382.conf`

![image-20251221191005990](./assets/image-20251221191005990.png)



![image-20251221191052947](./assets/image-20251221191052947.png)

3. 第二台虚拟机   `IP：192.168.111.172+端口6383/端口6384`
   - `cat redisCluster6383.conf`：可以检查集群配置文件是否正确

![image-20251221191311316](./assets/image-20251221191311316.png)

4. 第三台虚拟机  `IP：192.168.111.174+端口6385/端口6386`

5. redis-server启动

   ```
   # 依次启动6个节点
   redis-server /myredis/cluster/redisCluster6381.conf
   redis-server /myredis/cluster/redisCluster6382.conf
   redis-server /myredis/cluster/redisCluster6383.conf
   redis-server /myredis/cluster/redisCluster6384.conf
   redis-server /myredis/cluster/redisCluster6385.conf
   redis-server /myredis/cluster/redisCluster6386.conf
   
   # 验证节点是否启动成功
   ps -ef | grep redis
   netstat -tnulp | grep redis
   ```

6. 通过redis-cli命令为6台机器构建集群关系  构建主从关系命令 一切OK的话，3主3从搞定

   ```
   redis-cli --cluster create \
   192.168.1.101:7000 192.168.1.101:7001 192.168.1.101:7002 \
   192.168.1.102:7003 192.168.1.102:7004 192.168.1.102:7005 \
   --cluster-replicas 1 \
   -a YourStrongPassword
   ```

   - 命令解析：

     - `--cluster-replicas 1`：表示每个主节点配备1个从节点

     - `-a YourStrongPassword`：指定集群密码（如有设置）

     - **IP顺序**：前3个为候选主节点，后3个为从节点

   - **执行过程**：

     - 系统会自动分配哈希槽（0-5460, 5461-10922, 10923-16383）
     - 提示"Can I set the above configuration? (type 'yes' to accept)"时输入`yes`
     - 显示`[OK] All 16384 slots covered`表示集群创建成功
   
   - `redis-cli --cluster create` 命令（自动分配方式）
   
     1. 这是自动分配方式：
        - **完全正确**，该命令会**自动创建集群并分配槽位**，无需手动干预
        - Redis会**自动将16384个哈希槽平均分配**给各个主节点（3个主节点时，每个约5461个槽）
        - **自动建立主从关系**，根据`--cluster-replicas 1`参数为每个主节点分配1个从节点
        - 执行后会提示您确认配置（输入`yes`），然后**自动完成集群构建**23
     2. 关键参数说明：
        - `--cluster-replicas 1`：表示每个主节点配置1个从节点（生产环境**强烈建议至少配置1个从节点**）
        - `-a YourStrongPassword`：指定Redis访问密码（如有设置）
        - 命令中**前3个节点通常被自动识别为主节点**，后3个为从节点
   
   - `cluster addslots` 命令（手动分配方式）
   
     ```
     redis-cli -h 127.0.0.1 -p 6379 cluster addslots {0...5460}
     redis-cli -h 127.0.0.1 -p 6380 cluster addslots {5461...10922}
     redis-cli -h 127.0.0.1 -p 6381 cluster addslots {10923...16383}
     ```
   
     1. 这是手动分配方式：
        - **完全正确**，这些命令**手动为每个主节点分配特定范围的槽位**
        - `{0...5460}`表示分配0到5460的所有槽位（共5461个槽）
        - 三行命令**精确分配了全部16384个槽位**（0-5460、5461-10922、10923-16383）916
     2. 手动分配的适用场景：
        - 需要**自定义槽位分配策略**（如特定业务数据集中在特定节点）
        - **修复集群**时重新分配槽位（如某些槽位未分配导致`CLUSTERDOWN`错误）
        - **特殊迁移场景**下精确控制槽位分布
   
   - 两种方式对比与建议
   
     1. 自动分配 vs 手动分配：
        - **自动分配**：适合**常规集群搭建**，操作简单，Redis自动优化分配
        - **手动分配**：适合**特定需求场景**，需要精确控制槽位分布212
     2. 最佳实践建议：
        - **生产环境优先使用自动分配**：`redis-cli --cluster create`命令更可靠，避免人为错误
        - **手动分配需谨慎**：必须确保**16384个槽位全部分配完毕**，否则集群无法工作
        - **验证分配结果**：使用`redis-cli --cluster check`验证槽位分配
        - **集群创建后**：无需再次分配槽位，节点重启会自动加载集群配置212
     3. 常见问题预防：
        - **槽位未分配**：会导致`CLUSTERDOWN`错误，需确保所有槽位分配
        - **节点密码不一致**：会导致集群通信失败，确保所有节点使用**相同密码**
        - **防火墙限制**：需开放**集群通信端口**（通常为16379）1316
   
     > **重要提示**：在生产环境中，**建议优先使用自动分配方式**（`--cluster create`命令），因为它经过Redis官方优化，能确保槽位均匀分布和集群稳定性。手动分配方式仅在特殊需求下使用，并需严格验证槽位分配完整性

![image-20251221192129360](./assets/image-20251221192129360.png)

7. 链接进入6381作为切入点，查看节点状态

   - `cluster info`：查看集群基本信息
   - `cluster nodes`：查看节点详细信息

   

![image-20251221192617485](./assets/image-20251221192617485.png)

![image-20251221192816399](./assets/image-20251221192816399.png)

![image-20251221192755416](./assets/image-20251221192755416.png)

![image-20251221193025704](./assets/image-20251221193025704.png)

### 3主3从redi集群读写

1. 对6381新增两个key，看看效果如何？

![image-20251221193250194](./assets/image-20251221193250194.png)

2. 如何解决？带一个 -c 参数（-c 开启集群模式，支持自动重定向）

![image-20251221193944486](./assets/image-20251221193944486.png)

![image-20251221193608658](./assets/image-20251221193608658.png)

![image-20251221193643965](./assets/image-20251221193643965.png)

3. 查看某个key该属于对应的槽位值CLUSTER KEYSLOT键名称

![image-20251221193857931](./assets/image-20251221193857931.png)

4. 检查集群健康状态

   redis-cli --cluster check 192.168.10.101:6379

### 容错切换迁移

1. 集群不保证数据一致性100%OK，一定会有数据丢失情况：Redis集群不保证强一致性,这意味着在特定的条件下, Redis集群可能会丢掉一些被系统收到的写入请求命令

2. 现在来实现主机6381和从机6384切换，将6381主机停了，对应的真实从机上位

3. 从机6384上位成为新的master

![image-20251221203941654](./assets/image-20251221203941654.png)

![image-20251221204057643](./assets/image-20251221204057643.png)

4. 随后，6381原来的主机回来了，是否会上位？

5. 6381不会上位并以从节点形式回归

![image-20251221204226561](./assets/image-20251221204226561.png)

![image-20251221204246714](./assets/image-20251221204246714.png)

![image-20251221204433043](./assets/image-20251221204433043.png)

### 手动故障转移or节点从属调整该如何处理

1. 上面一换后6381、6384主从对调了，和原始设计图不一样了，该如何

2. 重写登陆6381

3. 执行CLUSTER FAILOVER

![image-20251221205050345](./assets/image-20251221205050345-1766321451835-1.png)

### 新增服务实例，动态扩容

![image-20251221205230583](./assets/image-20251221205230583.png)

新建6387，6388两个服务实例配置文件+新建后启动

1. 第四台虚拟机   IP：192.168.111.174+端口6387/端口6388
   - `vim /myredis/cluster/redisCluster6387.conf`
   - `vim /myredis/cluster/redisCluster6388.conf`

![image-20251221205614043](./assets/image-20251221205614043.png)

2. 启动，此时他们都是master
   - `redis-server /myredis/Eluster/redisCluster6387.conf`
   - `redis-server /myredis/Eluster/redisCluster6388.conf`

3. 第一种方式：使用add-node（自动识别主从）：`redis-cli -a 111111 --cluster add-node 192.168.111.174:6387 192.168.111.175:6381`

   第二种方式：使用meet命令添加（需手动指定主从关系）：

   redis-cli -c -p 6387 cluster meet 192.168.111.174

   redis-cli -c -p 6381 cluster meet 192.168.111.175

![image-20251221205903708](./assets/image-20251221205903708.png)

4. 验证节点加入：`redis-cli --cluster check 192.168.1.101:7000`
   - 能看到新节点出现在列表中，状态为`master`，但槽位范围为空

![image-20251221210746521](./assets/image-20251221210746521.png)

5. 重新分配槽号
   - **执行reshard命令**：`redis-cli --cluster reshard 192.168.1.111:6381`
   
   - 交互式操作流程：
   
     - 输入要分配的槽数量（如500）
     - 输入接收槽的新节点ID
     - 输入源节点ID（可从现有主节点中选择）
     - 确认分配计划
   
   - 自动化分配技巧：
   
     ````
     # 自动分配槽位（无需交互）
     redis-cli --cluster reshard 192.168.1.111:6381 \
     --cluster-from all \
     --cluster-to 新节点ID \
     --cluster-slots 500 \
     --cluster-yes
     ````

![image-20251221211001771](./assets/image-20251221211001771.png)

![image-20251221211105138](./assets/image-20251221211105138.png)

![image-20251221211237789](./assets/image-20251221211237789.png)



![image-20251221211557792](./assets/image-20251221211557792.png)

6. 槽号分派说明

![image-20251221211725156](./assets/image-20251221211725156.png)

7. 为主节点6387分配从节点6388
   - `redis-cli -a 111111 --cluster add-node 192.168.111.174:6388 192.168.111.174:6387  --cluster-slave  --cluster-master-id  6387的编号`

![image-20251221212051712](./assets/image-20251221212051712.png)



![image-20251221212435479](./assets/image-20251221212435479.png)



![image-20251221212502630](./assets/image-20251221212502630.png)

![image-20251221212657132](./assets/image-20251221212657132.png)



### 主从缩容案例，节点删除



![image-20251221212726357](./assets/image-20251221212726357.png)

1. 检查集群情况一次，先获得从节点6388的节点ID

![image-20251221213005666](./assets/image-20251221213005666.png)

2. 从集群中将4号从节点6388删除

![image-20251221213216177](./assets/image-20251221213216177.png)

3. 将6387的槽号，重新分配，本例将清出来的槽号都给6381

![image-20251221213503492](./assets/image-20251221213503492.png)

![image-20251221213653062](./assets/image-20251221213653062.png)

4. 第二次检查集群

![image-20251221213837969](./assets/image-20251221213837969.png)

5. 将6387删除

![image-20251221213959128](./assets/image-20251221213959128.png)

![image-20251221214052620](./assets/image-20251221214052620.png)

6. 不在同一个slot槽位下的多键操作支持不好，通识占位符登场

![image-20251222090229592](./assets/image-20251222090229592.png)



![image-20251222090344740](./assets/image-20251222090344740.png)

### CRC16源码

![image-20251222090621308](./assets/image-20251222090621308.png)

![image-20251222090637834](./assets/image-20251222090637834.png)



集群是否完整才能对外提供服务？

![image-20251222090937972](./assets/image-20251222090937972.png)

`CLUSTER COUNTKEYSINSLOT 槽位数字编号`：返回1表示该曹魏被占用，返回0表示该槽位没占用

`CLUSTER KEYSLOT 键名称`：该键在哪个槽位上



### 用redis-trib.rb搭建集群

手动搭建集群便于理解集群建立的流程和细节，不过读者也从中发现集群搭建需要很多步骤，当集群节点众多时，必然会加大搭建集群的复杂度和运维成本。因此Redis官方提供了redis-trib.rb工具方便我们快速搭建集群。

redis-trib.rb是采用Ruby实现的Redis集群管理工具。内部通过Cluster相关命令帮我们简化集群创建、检查、槽迁移和均衡等常见运维操作，使用之前需要安装Ruby依赖环境。下面介绍搭建集群的详细步骤。

#### 1.Ruby环境准备

安装Ruby：

```
-- 下载ruby
wget https:// cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
-- 安装ruby
tar xvf ruby-2.3.1.tar.gz
./configure -prefix=/usr/local/ruby
make
make install
cd /usr/local/ruby
sudo cp bin/ruby /usr/local/bin
sudo cp bin/gem /usr/local/bin
```

安装rubygem redis依赖：

```
wget http:// rubygems.org/downloads/redis-3.3.0.gem
gem install -l redis-3.3.0.gem
gem list --check redis gem
```

安装redis-trib.rb：

```
sudo cp /{redis_home}/src/redis-trib.rb /usr/local/bin
```

安装完Ruby环境后，执行redis-trib.rb命令确认环境是否正确，输出如下：

```
# redis-trib.rb
Usage: redis-trib <command> <options> <arguments …>
    create          host1:port1 … hostN:portN
                  --replicas <arg>
    check           host:port
    info            host:port
    fix              host:port
                  --timeout <arg>
    reshard         host:port
                  --from <arg>
                  --to <arg>
                  --slots <arg>
                  --yes
                  --timeout <arg>
                  --pipeline <arg>
    …忽略…
```

从redis-trib.rb的提示信息可以看出，它提供了集群创建、检查、修复、均衡等命令行工具。这里我们关注集群创建命令，使用redis-trib.rb create命令可快速搭建集群。

#### 2.准备节点

首先我们跟之前内容一样准备好节点配置并启动：

```
redis-server conf/redis-6481.conf
redis-server conf/redis-6482.conf
redis-server conf/redis-6483.conf
redis-server conf/redis-6484.conf
redis-server conf/redis-6485.conf
redis-server conf/redis-6486.conf
```

#### 3.创建集群

启动好6个节点之后，使用redis-trib.rb create命令完成节点握手和槽分配过程，命令如下：

```
redis-trib.rb create --replicas 1 127.0.0.1:6481 127.0.0.1:6482 127.0.0.1:6483
    127.0.0.1:6484 127.0.0.1:6485 127.0.0.1:6486
```

--replicas参数指定集群中每个主节点配备几个从节点，这里设置为1。我们出于测试目的使用本地IP地址127.0.0.1，如果部署节点使用不同的IP地址，redis-trib.rb会尽可能保证主从节点不分配在同一机器下，因此会重新排序节点列表顺序。节点列表顺序用于确定主从角色，先主节点之后是从节点。创建过程中首先会给出主从节点角色分配的计划，如下所示。

```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes…
Using 3 masters:
127.0.0.1:6481
127.0.0.1:6482
127.0.0.1:6483
Adding replica 127.0.0.1:6484 to 127.0.0.1:6481
Adding replica 127.0.0.1:6485 to 127.0.0.1:6482
Adding replica 127.0.0.1:6486 to 127.0.0.1:6483
M: 869de192169c4607bb886944588bc358d6045afa 127.0.0.1:6481
slots:0-5460 (5461 slots) master
M: 6f9f24923eb37f1e4dce1c88430f6fc23ad4a47b 127.0.0.1:6482
slots:5461-10922 (5462 slots) master
M: 6228a1adb6c26139b0adbe81828f43a4ec196271 127.0.0.1:6483
slots:10923-16383 (5461 slots) master
S: 22451ea81fac73fe7a91cf051cd50b2bf308c3f3 127.0.0.1:6484
replicates 869de192169c4607bb886944588bc358d6045afa
S: 89158df8e62958848134d632e75d1a8d2518f07b 127.0.0.1:6485
replicates 6f9f24923eb37f1e4dce1c88430f6fc23ad4a47b
S: bcb394c48d50941f235cd6988a40e469530137af 127.0.0.1:6486
replicates 6228a1adb6c26139b0adbe81828f43a4ec196271
Can I set the above configuration (type 'yes' to accept):
```

当我们同意这份计划之后输入yes，redis-trib.rb开始执行节点握手和槽分配操作，输出如下：

```
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 127.0.0.1:6481)
…忽略…
[OK] All nodes agree about slots configuration.
>>> Check for open slots…
>>> Check slots coverage…
[OK] All 16384 slots covered.
```

最后的输出报告说明：16384个槽全部被分配，集群创建成功。这里需要注意给redis-trib.rb的节点地址必须是不包含任何槽/数据的节点，否则会拒绝创建集群。

#### 4.集群完整性检查

集群完整性指所有的槽都分配到存活的主节点上，只要16384个槽中有一个没有分配给节点则表示集群不完整。可以使用redis-trib.rb check命令检测之前创建的两个集群是否成功，check命令只需要给出集群中任意一个节点地址就可以完成整个集群的检查工作，命令如下：

```
redis-trib.rb check 127.0.0.1:6379
redis-trib.rb check 127.0.0.1:6481
```

当最后输出如下信息，提示集群所有的槽都已分配到节点：

```
[OK] All nodes agree about slots configuration.
>>> Check for open slots…
>>> Check slots coverage…
[OK] All 16384 slots covered.
```

## 节点通信

### 通信流程

在分布式存储中需要提供维护节点元数据信息的机制，所谓元数据是指：节点负责哪些数据，是否出现故障等状态信息。常见的元数据维护方式分为：**集中式和P2P方式**。Redis集群采用**P2P的Gossip（流言）协议**，Gossip协议工作原理就是节点彼此不断通信交换信息，一段时间后所有的节点都会知道集群完整的信息，这种方式类似流言传播

节点彼此传播消息

![image-20251221165225059](./assets/image-20251221165225059.png)

通信过程说明：

1)集群中的每个节点都会单独开辟一个TCP通道，用于节点之间彼此通信，通信端口号在基础端口上加10000。

2)每个节点在固定周期内通过特定规则选择几个节点发送ping消息。

3)接收到ping消息的节点用pong消息作为响应。

集群中每个节点通过一定规则挑选要通信的节点，每个节点可能知道全部节点，也可能仅知道部分节点，只要这些节点彼此可以正常通信，最终它们会达到一致的状态。当节点出故障、新节点加入、主从角色变化、槽信息变更等事件发生时，通过不断的ping/pong消息通信，经过一段时间后所有的节点都会知道整个集群全部节点的最新状态，从而达到集群状态同步的目的。

### Gossip消息

Gossip协议的主要职责就是信息交换。信息交换的载体就是节点彼此发送的Gossip消息，了解这些消息有助于我们理解集群如何完成信息交换。常用的Gossip消息可分为：ping消息、pong消息、meet消息、fail消息等，它们的通信模式如图

不同消息通信模式

![image-20251221165452102](./assets/image-20251221165452102-1766307292601-1.png)



- meet消息：用于手动触发通知新节点加入集群。消息发送者通知接收者加入到当前集群，meet消息通信正常完成后，接收节点会加入到集群中并进行周期性的ping、pong消息交换。
- ping消息：集群内交换最频繁的消息，集群内每个节点每秒向多个其他节点发送ping消息，用于检测节点是否在线和交换彼此状态信息。ping消息发送封装了自身节点和部分其他节点的状态数据。
- pong消息：当接收到ping、meet消息时，作为响应消息回复给发送方确认消息正常通信。pong消息内部封装了自身状态数据。节点也可以向集群内广播自身的pong消息来通知整个集群对自身状态进行更新。
- fail消息：当节点判定集群内另一个节点下线时，会向集群内广播一个fail消息，其他节点接收到fail消息之后把对应节点更新为下线状态。

所有的消息格式划分为：消息头和消息体。消息头包含发送节点自身状态数据，接收节点根据消息头就可以获取到发送节点的相关数据，结构如下：

```
typedef struct {
char sig[4]; /* 信号标示 */
uint32_t totlen; /* 消息总长度 */
uint16_t ver; /* 协议版本*/
uint16_t type; /* 消息类型,用于区分meet,ping,pong等消息 */
uint16_t count; /* 消息体包含的节点数量，仅用于meet,ping,ping消息类型*/
uint64_t currentEpoch; /* 当前发送节点的配置纪元 */
uint64_t configEpoch; /* 主节点/从节点的主节点配置纪元 */
uint64_t offset; /* 复制偏移量 */
char sender[CLUSTER_NAMELEN]; /* 发送节点的nodeId */
unsigned char myslots[CLUSTER_SLOTS/8]; /* 发送节点负责的槽信息 */
char slaveof[CLUSTER_NAMELEN]; /* 如果发送节点是从节点，记录对应主节点的nodeId */
uint16_t port; /* 端口号 */
uint16_t flags; /* 发送节点标识,区分主从角色，是否下线等 */
unsigned char state; /* 发送节点所处的集群状态 */
unsigned char mflags[3]; /* 消息标识 */
union clusterMsgData data /* 消息正文 */;
} clusterMsg;
```

集群内所有的消息都采用相同的消息头结构clusterMsg，它包含了发送节点关键信息，如节点id、槽映射、节点标识（主从角色，是否下线）等。消息体在Redis内部采用clusterMsgData结构声明，结构如下：

```
union clusterMsgData {
    /* ping,meet,pong消息体*/
    struct {
        /* gossip消息结构数组 */
        clusterMsgDataGossip gossip[1];
    } ping;
/* FAIL 消息体 */
    struct {
        clusterMsgDataFail about;
    } fail;
// …
};
```

消息体clusterMsgData定义发送消息的数据，其中ping、meet、pong都采用cluster MsgDataGossip数组作为消息体数据，实际消息类型使用消息头的type属性区分。每个消息体包含该节点的多个clusterMsgDataGossip结构数据，用于信息交换，结构如下：

```
typedef struct {
char nodename[CLUSTER_NAMELEN]; /* 节点的nodeId */
uint32_t ping_sent; /* 最后一次向该节点发送ping消息时间 */
uint32_t pong_received; /* 最后一次接收该节点pong消息时间 */
    char ip[NET_IP_STR_LEN]; /* IP */
    uint16_t port; /* port*/
uint16_t flags; /* 该节点标识, */
} clusterMsgDataGossip;
```

当接收到ping、meet消息时，接收节点会解析消息内容并根据自身的识别情况做出相应处理，对应流程如图

消息解析流程

![image-20251221165711639](./assets/image-20251221165711639.png)

接收节点收到ping/meet消息时，执行解析消息头和消息体流程：

- 解析消息头过程：消息头包含了发送节点的信息，如果发送节点是新节点且消息是meet类型，则加入到本地节点列表；如果是已知节点，则尝试更新发送节点的状态，如槽映射关系、主从角色等状态。
- 解析消息体过程：如果消息体的clusterMsgDataGossip数组包含的节点是新节点，则尝试发起与新节点的meet握手流程；如果是已知节点，则根据cluster MsgDataGossip中的flags字段判断该节点是否下线，用于故障转移。

消息处理完后回复pong消息，内容同样包含消息头和消息体，发送节点接收到回复的pong消息后，采用类似的流程解析处理消息并更新与接收节点最后通信时间，完成一次消息通信。

### 节点选择

虽然Gossip协议的信息交换机制具有天然的分布式特性，但它是有成本的。由于内部需要频繁地进行节点信息交换，而ping/pong消息会携带当前节点和部分其他节点的状态数据，势必会加重带宽和计算的负担。Redis集群内节点通信采用固定频率（定时任务每秒执行10次）。因此节点每次选择需要通信的节点列表变得非常重要。通信节点选择过多虽然可以做到信息及时交换但成本过高。节点选择过少会降低集群内所有节点彼此信息交换频率，从而影响故障判定、新节点发现等需求的速度。因此Redis集群的Gossip协议需要兼顾信息交换实时性和成本开销，通信节点选择的规则如图

选择通信节点的规则和消息携带的数据量

![image-20251221165951040](./assets/image-20251221165951040.png)

根据通信节点选择的流程可以看出消息交换的成本主要体现在单位时间选择发送消息的节点数量和每个消息携带的数据量。

1.选择发送消息的节点数量

集群内每个节点维护定时任务默认每秒执行10次，每秒会随机选取5个节点找出最久没有通信的节点发送ping消息，用于保证Gossip信息交换的随机性。每100毫秒都会扫描本地节点列表，如果发现节点最近一次接受pong消息的时间大于cluster_node_timeout/2，则立刻发送ping消息，防止该节点信息太长时间未更新。根据以上规则得出每个节点每秒需要发送ping消息的数量=1+10*num(node.pong_received>cluster_node_timeout/2)，因此cluster_node_timeout参数对消息发送的节点数量影响非常大。当我们的带宽资源紧张时，可以适当调大这个参数，如从默认15秒改为30秒来降低带宽占用率。过度调大cluster_node_timeout会影响消息交换的频率从而影响故障转移、槽信息更新、新节点发现的速度。因此需要根据业务容忍度和资源消耗进行平衡。同时整个集群消息总交换量也跟节点数成正比。

2.消息数据量

每个ping消息的数据量体现在消息头和消息体中，其中消息头主要占用空间的字段是myslots[CLUSTER_SLOTS/8]，占用2KB，这块空间占用相对固定。消息体会携带一定数量的其他节点信息用于信息交换。具体数量见以下伪代码：

```
def get_wanted():
    int total_size = size(cluster.nodes)
# 默认包含节点总量的1/10
    int wanted = floor(total_size/10);
    if wanted < 3:
# 至少携带3个其他节点信息
        wanted = 3;
    if wanted > total_size -2 :
# 最多包含total_size - 2个
        wanted = total_size - 2;
  return wanted;
```

根据伪代码可以看出消息体携带数据量跟集群的节点数息息相关，更大的集群每次消息通信的成本也就更高，因此对于Redis集群来说并不是大而全的集群更好，对于集群规模控制的建议见之后10.7节“集群运维”。

## MOVED重定向

### 触发条件

1. **永久性槽位迁移**
   当槽位通过`CLUSTER SETSLOT {slot} NODE {newNodeId}`命令**完成迁移**，源节点确认该槽位不再由自己负责时触发。
2. **集群配置变更**
   包括节点扩容、缩容、故障转移后的新主节点接管等场景，只要**槽位归属关系发生永久性改变**即触发。
3. **关键区别**
   与ASK重定向不同，MOVED表示**迁移已完全完成**，而非迁移过程中（）

### 工作原理

1. 槽位计算流程
   - 客户端发送请求至任意节点
   - 节点计算`CRC16(key) % 16384`确定槽位
   - 检查本地`clusterState.slots`数组确认槽位归属
   - 若槽位属于当前节点，处理键命令
   - 若槽位不属当前节点，返回MOVED错误

2. 重定向响应格式

   ```
   (error) MOVED <slot> <target-node-ip>:<target-node-port>
   ```

   示例：`MOVED 3999 127.0.0.1:6381`表示槽3999已永久迁移到127.0.0.1:6381节点。

4. **Gossip协议支撑**
   节点通过Gossip协议传播槽位变更信息，配置纪元(config epoch)确保**变更顺序一致性**，避免旧信息覆盖新状态（）

5. MOVED重定向执行流程

![image-20251221173213178](./assets/image-20251221173213178.png)

### 案例

1. 例如，在之前搭建的集群上执行如下命令：

```
127.0.0.1:6379> set key:test:1 value-1
OK
```

2. 执行set命令成功，因为键key：test：1对应槽5191正好位于6379节点负责的槽范围内，可以借助**cluster keyslot{key}命令返回key所对应的槽**，如下所示：

```
127.0.0.1:6379> cluster keyslot key:test:1
(integer) 5191
127.0.0.1:6379> cluster nodes
cfb28ef1deee4e0fa78da86abe5d24566744411e 127.0.0.1:6379 myself,master - 0 0 10 connected 
    1366-4095 4097-5461 12288-13652
…
```

4. 再执行以下命令，由于键对应槽是9252，不属于6379节点，则回复MOVED{slot}{ip}{port}格式重定向信息：

```
127.0.0.1:6379> set key:test:2 value-2
(error) MOVED 9252 127.0.0.1:6380
127.0.0.1:6379> cluster keyslot key:test:2
(integer) 9252
```

5. 重定向信息包含了键所对应的槽以及负责该槽的节点地址，根据这些信息客户端就可以向正确的节点发起请求。在6380节点上成功执行之前的命令：

```
127.0.0.1:6380> set key:test:2 value-2
OK
```

6. **使用redis-cli命令时，可以加入-c参数支持自动重定向，简化手动发起重定向操作**，如下所示：

```
#redis-cli -p 6379 -c
127.0.0.1:6379> set key:test:2 value-2
-> Redirected to slot [9252] located at 127.0.0.1:6380
OK
```

7. redis-cli自动帮我们连接到正确的节点执行命令，这个过程是在redis-cli内部维护，实质上是client端接到MOVED信息之后再次发起请求，并不在Redis节点中完成请求转发

![image-20251221173403786](./assets/image-20251221173403786.png)

8. **节点对于不属于它的键命令只回复重定向响应，并不负责转发。**熟悉Cassandra的用户希望在这里做好区分，不要混淆。正因为集群模式下把解析发起重定向的过程放到客户端完成，所以集群客户端协议相对于单机有了很大的变化。

### 键命令执行步骤主要分两步：计算槽，查找槽所对应的节点。

1.计算槽

Redis首先需要计算键所对应的槽。根据键的有效部分使用CRC16函数计算出散列值，再取对16383的余数，使每个键都可以映射到0~16383槽范围内。伪代码如下：

```
def key_hash_slot(key):
    int keylen = key.length();
    for (s = 0; s < keylen; s++):
        if (key[s] == '{'):
        break;
    if (s == keylen) return crc16(key,keylen) & 16383;
    for (e = s+1; e < keylen; e++):
        if (key[e] == '}') break;
        if (e == keylen || e == s+1) return crc16(key,keylen) & 16383;
/* 使用{和}之间的有效部分计算槽 */
    return crc16(key+s+1,e-s-1) & 16383;
```

根据伪代码，**如果键内容包含{和}大括号字符，则计算槽的有效部分是括号内的内容；否则采用键的全内容计算槽。**cluster keyslot命令就是采用key_hash_slot函数实现的，例如：

```
127.0.0.1:6379> cluster keyslot key:test:111
(integer) 10050
127.0.0.1:6379> cluster keyslot key:{hash_tag}:111
(integer) 2515
127.0.0.1:6379> cluster keyslot key:{hash_tag}:222
(integer) 2515
```

其中键内部使用大括号包含的内容又叫做hash_tag，它提供不同的键可以具备相同slot的功能，常用于Redis IO优化。例如在集群模式下使用mget等命令优化批量调用时，键列表必须具有相同的slot，否则会报错。这时可以利用hash_tag让不同的键具有相同的slot达到优化的目的。命令如下：

```
127.0.0.1:6385> mget user:10086:frends user:10086:videos
(error) CROSSSLOT Keys in request don't hash to the same slot
127.0.0.1:6385> mget user:{10086}:friends user:{10086}:videos
1) "friends"
2) "videos"
```

开发提示

Pipeline同样可以受益于hash_tag，由于Pipeline只能向一个节点批量发送执行命令，而相同slot必然会对应到唯一的节点，降低了集群使用Pipeline的门槛。

2.槽节点查找

Redis计算得到键对应的槽后，需要查找槽所对应的节点。**集群内通过消息交换每个节点都会知道所有节点的槽信息，内部保存在clusterState结构中**，结构所示：

```
typedef struct clusterState {
clusterNode *myself; /* 自身节点,clusterNode代表节点结构体 */
    clusterNode *slots[CLUSTER_SLOTS]; /* 16384个槽和节点映射数组，数组下标代表对应的槽 */
…
} clusterState;
```

**slots数组表示槽和节点对应关系**，实现请求重定向伪代码如下：

```
def execute_or_redirect(key):
    int slot = key_hash_slot(key);
    ClusterNode node = slots[slot];
    if(node == clusterState.myself):
        return executeCommand(key);
    else:
        return '(error) MOVED {slot} {node.ip}:{node.port}';
```

根据伪代码看出节点对于判定键命令是执行还是MOVED重定向，都是借助slots[CLUSTER_SLOTS]数组实现。**根据MOVED重定向机制，客户端可以随机连接集群内任一Redis获取键所在节点，这种客户端又叫Dummy（傀儡）客户端，它优点是代码实现简单，对客户端协议影响较小，只需要根据重定向信息再次发送请求即可。但是它的弊端很明显，每次执行键命令前都要到Redis上进行重定向才能找到要执行命令的节点，额外增加了IO开销，这不是Redis集群高效的使用方式。正因为如此通常集群客户端都采用另一种实现：Smart（智能）客户端。**

| 特性       | Dummy客户端      | Smart客户端          |
| ---------- | ---------------- | -------------------- |
| 连接方式   | 随机连接任一节点 | 维护与多个节点的连接 |
| 槽位查询   | 每次请求需重定向 | 本地缓存槽位映射     |
| 性能       | 较低，额外IO开销 | 较高，减少重定向次数 |
| 实现复杂度 | 简单             | 较复杂               |

**关键优势**：Smart客户端使Redis Cluster的性能与单节点部署处于**同级别**，这是Redis集群能支撑高并发场景的关键

### Smart客户端

#### Smart客户端原理

1. 初始化流程
   1. **连接种子节点**：客户端连接任意集群节点
   2. **获取槽位信息**：发送`CLUSTER SLOTS`命令获取完整槽位分配
   3. **构建本地映射**：解析响应并**为每个节点创建连接池**，建立slot→node映射缓存
   4. **准备服务**：初始化完成后可直接处理客户端请求

2. 请求处理流程

   ```
   graph TD
       A[接收客户端请求] --> B{是否为集群模式}
       B -->|是| C[计算key的slot]
       C --> D[查询本地slot映射]
       D --> E{是否为当前节点}
       E -->|是| F[直接执行命令]
       E -->|否| G[连接目标节点]
       G --> H[执行命令并返回结果]
       B -->|否| I[单机模式处理]
   ```

#### Smart客户端高级特性

1. 哈希标签（Hash Tag）支持
   - **原理**：仅对`{}`内内容计算哈希值，使不同键映射到相同slot
   - **实现**：`CLUSTER KEYSLOT user:{1000}:profile`与`user:{1000}:address`返回相同slot值
   - 应用场景：
     - **多键操作**：`MGET user:{1000}:profile user:{1000}:address`可正常执行
     - **Pipeline优化**：相同slot的键可批量发送到同一节点

2. 连接池管理
   - **多节点连接**：为每个集群节点**独立维护连接池**，避免单点瓶颈
   - **连接复用**：对同一节点的请求**复用连接**，降低建立连接的开销
   - **动态调整**：根据负载自动调整连接池大小，优化资源使用

3. 容错与恢复机制
   - **故障检测**：通过节点间Gossip协议感知节点状态变化
   - **自动重试**：遇到MOVED错误后**自动更新映射并重试**，最多5次
   - **超时控制**：设置合理超时参数，避免因单点故障导致整个服务阻塞

#### 以java的Jedis为例，说明Smart客户端操作集群的流程

客户端如何选择见：http://redis.io/clients

Smart客户端通过在内部维护slot→node的映射关系，本地就可实现键到节点的查找，从而保证IO效率的最大化，而MOVED重定向负责协助Smart客户端更新slot→node映射。

1)首先在JedisCluster初始化时会选择一个运行节点，初始化槽和节点映射关系，使用cluster slots命令完成，如下所示：

```
127.0.0.1:6379> cluster slots
1) 1) (integer) 0 // 开始槽范围
2) (integer) 1365 // 结束槽范围
3) 1) "127.0.0.1" // 主节点ip
2) (integer) 6385 // 主节点地址
4) 1) "127.0.0.1" // 从节点ip
2) (integer) 6386 // 从节点端口
2) 1) (integer) 5462
   2) (integer) 6826
   3) 1) "127.0.0.1"
      2) (integer) 6385
   4) 1) "127.0.0.1"
      2) (integer) 6386
…
```

2)JedisCluster解析cluster slots结果缓存在本地，并为每个节点创建唯一的JedisPool连接池。映射关系在JedisClusterInfoCache类中，如下所示：

```
public class JedisClusterInfoCache {
    private Map<String, JedisPool> nodes = new HashMap<String, JedisPool>();
    private Map<Integer, JedisPool> slots = new HashMap<Integer, JedisPool>();
    …
}
```

3)JedisCluster执行键命令的过程有些复杂，但是理解这个过程对于开发人员分析定位问题非常有帮助

键命令执行流程：

1. 计算slot并根据slots缓存获取目标节点连接，发送命令。

2. 如果出现连接错误，使用随机连接重新执行键命令，每次命令重试对redi-rections参数减1。

3. 捕获到MOVED重定向错误，使用cluster slots命令更新slots缓存（renewSlotCache方法）。

4. 重复执行1)~3)步，直到命令执行成功，或者当redirections<=0时抛出Jedis ClusterMaxRedirectionsException异常。

5. Jedis客户端命令执行流程

   ![image-20260105112822038](./assets/image-20260105112822038.png)



部分代码如下：

```
public abstract class JedisClusterCommand<T> {
// 集群节点连接处理器
    private JedisClusterConnectionHandler connectionHandler;
// 重试次数，默认5次
    private int redirections;
// 模板回调方法
    public abstract T execute(Jedis connection);
    public T run(String key) {
        if (key == null) {
             throw new JedisClusterException("No way to dispatch this command to 
                 Redis Cluster.");
        }
        return runWithRetries(SafeEncoder.encode(key), this.redirections, false, 
            false);
    }
// 利用重试机制运行键命令
    private T runWithRetries(byte[] key, int redirections, boolean tryRandomNode, 
        boolean asking) {
        if (redirections <= 0) {
            throw new JedisClusterMaxRedirectionsException("Too many Cluster redi
                rections");
        }
        Jedis connection = null;
        try {
        if (tryRandomNode) {
// 随机获取活跃节点连接
            connection = connectionHandler.getConnection();
        } else {
// 使用slot缓存获取目标节点连接
            connection = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.
                 getSlot(key));
        }
        return execute(connection);
        } catch (JedisConnectionException jce) {
// 出现连接错误使用随机连接重试
            return runWithRetries(key, redirections - 1, true/*开启随机连接*/, asking);
        } catch (JedisRedirectionException jre) {
            if (jre instanceof JedisMovedDataException) {
// 如果出现MOVED重定向错误,在连接上执行cluster slots命令重新初始化slot缓存
                this.connectionHandler.renewSlotCache(connection);
            }
            // slot初始化后重试执行命令
            return runWithRetries(key, redirections - 1, false, asking);
        } finally {
            releaseConnection(connection);
        }
    }
}
```

从命令执行流程中发现，客户端需要结合异常和重试机制时刻保证跟Redis集群的slots同步，因此Smart客户端相比单机客户端有了很大的变化和实现难度。了解命令执行流程后，下面我们对Smart客户端成本和可能存在的问题进行分析：

1)客户端内部维护slots缓存表，并且针对每个节点维护连接池，当集群规模非常大时，客户端会维护非常多的连接并消耗更多的内存。

2)使用Jedis操作集群时最常见的错误是：

```
throw new JedisClusterMaxRedirectionsException("Too many Cluster redirections");
```

这经常会引起开发人员的疑惑，它隐藏了内部错误细节，原因是节点宕机或请求超时都会抛出JedisConnectionException，导致触发了随机重试，当重试次数耗尽抛出这个错误。

3)当出现JedisConnectionException时，Jedis认为可能是集群节点故障需要随机重试来更新slots缓存，因此了解哪些异常将抛出JedisConnectionException变得非常重要，有如下几种情况会抛出JedisConnectionException：

- Jedis连接节点发生socket错误时抛出。
- 所有命令/Lua脚本读写超时抛出。
- JedisPool连接池获取可用Jedis对象超时抛出。

前两点都可能是节点故障需要通过JedisConnectionException来更新slots缓存，但是第三点没有必要，因此Jedis2.8.1版本之后对于连接池的超时抛出Jedis Exception，从而避免触发随机重试机制。

4)Redis集群支持自动故障转移，但是从故障发现到完成转移需要一定的时间，节点宕机期间所有指向这个节点的命令都会触发随机重试，每次收到MOVED重定向后会调用JedisClusterInfoCache类的renewSlotCache方法。部分代码如下：

```
private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
private final Lock r = rwl.readLock();
private final Lock w = rwl.writeLock();
public void renewSlotCache(Jedis jedis) {
    try {
        cache.discoverClusterSlots(jedis);
    } catch (JedisConnectionException e) {
        renewSlotCache();
    }
}
public void discoverClusterSlots(Jedis jedis) {
// 获取写锁
    w.lock();
    try {
        this.slots.clear();
// 执行cluster slots
        List<Object> slots = jedis.clusterSlots();
        for (Object slotInfoObj : slots) {
// 初始化slots缓存代码,忽略细节…
        }
    } finally {
        w.unlock();
    }
}
public JedisPool getSlotPool(int slot) {
// 获取读锁
    r.lock();
    try {
// 返回slot对应的jedisPool
        return slots.get(slot);
    } finally {
        r.unlock();
    }
}
```

根据代码看出，只有当重试次数到最后1次或者出现MovedDataException时才更新slots操作，降低了cluster slots命令调用次数。·当更新slots缓存时，不再使用ping命令检测节点活跃度，并且使用redis covering变量保证同一时刻只有一个线程更新slots缓存，其他线程忽略，优化了写锁阻塞和cluster slots调用次数。伪代码如下：

```
def renewSlotCache(Jedis jedis) :
    //使用rediscovering变量保证当有一个线程正在初始化slots时，其他线程直接忽略。
    if (!rediscovering):
        try :
            w.lock();
            rediscovering = true;
            if (jedis != null) :
                try :
// 更新本地缓存
                    discoverClusterSlots(jedis);
                    return;
                except JedisException,e:
// 忽略异常，使用随机查找更新slots
// 使用随机节点更新slots
            for (JedisPool jp : getShuffledNodesPool()) :
                try :
// 不再使用ping命令检测节点
                    jedis = jp.getResource();
                    discoverClusterSlots(jedis);
                    return;
                except JedisConnectionException,e:
                    // try next nodes
                finally :
                    if (jedis != null) :
                        jedis.close();
        finally :
// 释放锁和rediscovering变量
            rediscovering = false;
            w.unlock();
```

综上所述，Jedis2.8.2之后的版本，当出现JedisConnectionException时，命令发送次数变为5次：4次重试命令+1次cluster slots命令，同时避免了cluster slots不必要的并发调用。

开发提示建议升级到Jedis2.8.2以上版本防止cluster slots风暴和写锁阻塞问题，但是笔者认为还可以进一步优化，如下所示：·执行cluster slots的过程不需要加入任何读写锁，因为cluster slots命令执行不需要做并发控制，只有修改本地slots时才需要控制并发，这样降低了写锁持有时间。·当获取新的slots映射后使用读锁跟老slots比对，只有新老slots不一致时再加入写锁进行更新。防止集群slots映射没有变化时进行不必要的加写锁行为。这里我们用大量篇幅介绍了Smart客户端Jedis与集群交互的细节，主要原因是针对于高并发的场景，这里是绝对的热点代码。集群协议通过Smart客户端全面高效的支持需要一个过程，因此用户在选择Smart客户端时要重点审核集群交互代码，防止线上踩坑。必要时可以自行优化修改客户端源码。

#### Smart客户端-JedisCluster

(1)JedisCluster的定义

Jedis为Redis Cluster提供了Smart客户端，对应的类是JedisCluster，它的初始化方法如下

```\
public JedisCluster(Set<HostAndPort> jedisClusterNode, int connectionTimeout, int
    soTimeout, int maxAttempts, final GenericObjectPoolConfig poolConfig) {
    …
}
```

其中包含了5个参数：·Set<HostAndPort>jedisClusterNode：所有Redis Cluster节点信息（也可以是一部分，因为客户端可以通过cluster slots自动发现）。·int connectionTimeout：连接超时。·int soTimeout：读写超时。·int maxAttempts：重试次数。·GenericObjectPoolConfig poolConfig：连接池参数，JedisCluster会为Redis Cluster的每个节点创建连接池，有关连接池的详细说明参见第4章。

例如下面代码展示了一次JedisCluster的初始化过程。

```
// 初始化所有节点(例如6个节点)
Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
jedisClusterNode.add(new HostAndPort("10.10.xx.1", 6379));
jedisClusterNode.add(new HostAndPort("10.10.xx.2", 6379));
jedisClusterNode.add(new HostAndPort("10.10.xx.3", 6379));
jedisClusterNode.add(new HostAndPort("10.10.xx.4", 6379));
jedisClusterNode.add(new HostAndPort("10.10.xx.5", 6379));
jedisClusterNode.add(new HostAndPort("10.10.xx.6", 6379));
// 初始化commnon-pool连接池，并设置相关参数
GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
// 初始化JedisCluster
JedisCluster jedisCluster = new JedisCluster(jedisClusterNode, 1000, 1000, 5, poolConfig);
```

JedisCluster可以实现命令的调用，如下所示。

```
jedisCluster.set("hello", "world");
jedisCluster.get("key");
```

对于JedisCluster的使用需要注意以下几点：·JedisCluster包含了所有节点的连接池(JedisPool)，所以建议JedisCluster使用单例。·JedisCluster每次操作完成后，不需要管理连接池的借还，它在内部已经完成。·JedisCluster一般不要执行close()操作，它会将所有JedisPool执行destroy操作。

(2)多节点命令和操作

Redis Cluster虽然提供了分布式的特性，但是有些命令或者操作，诸如keys、flushall、删除指定模式的键，需要遍历所有节点才可以完成。下面代码实现了从Redis Cluster删除指定模式键的功能：

```
// 从RedisCluster批量删除指定pattern的数据
public void delRedisClusterByPattern(JedisCluster jedisCluster, String pattern, 
    int scanCounter) {
// 获取所有节点的JedisPool
    Map<String, JedisPool> jedisPoolMap = jedisCluster.getClusterNodes();
    for (Entry<String, JedisPool> entry : jedisPoolMap.entrySet()) {
// 获取每个节点的Jedis连接
        Jedis jedis = entry.getValue().getResource();
// 只删除主节点数据
        if (!isMaster(jedis)) {
            continue;
        }
// 使用Pipeline每次删除指定前缀的数据
        Pipeline pipeline = jedis.pipelined();
// 使用scan扫描指定前缀的数据
        String cursor = "0";
// 指定扫描参数：每次扫描个数和pattern
        ScanParams params = new ScanParams().count(scanCounter).match(pattern);
        while (true) {
// 执行扫描
            ScanResult<String> scanResult = jedis.scan(cursor, params);
// 删除的key列表
            List<String> keyList = scanResult.getResult();
            if (keyList != null && keyList.size() > 0) {
                for (String key : keyList) {
                    pipeline.del(key);
                }
// 批量删除
                pipeline.syncAndReturnAll();
            }
            cursor = scanResult.getStringCursor();
// 如果游标变为0，说明扫描完毕
            if ("0".equals(cursor)) {
                break;
            }
        }
    }
}
// 判断当前Redis是否为master节点
private boolean isMaster(Jedis jedis) {
    String[] data = jedis.info("Replication").split("\r\n");
    for (String line : data) {
        if ("role:master".equals(line.trim())) {
            return true;
        }
    }
    return false;
}
```

具体分为如下几个步骤：1)通过jedisCluster.getClusterNodes()获取所有节点的连接池。2)使用info replication筛选1)中的主节点。3)遍历主节点，使用scan命令找到指定模式的key，使用Pipeline机制删除。例如下面操作每次遍历1000个key，将Redis Cluster中以user开头的key全部删除。

```
String pattern = "user*";
int scanCounter = 1000;
delRedisClusterByPattern(jedisCluster, pattern, scanCounter);
```

所以对于keys、flushall等需要遍历所有节点的命令，同样可以参照上面的方法进行相应功能的实现。

(3)批量操作的方法

Redis Cluster中，由于key分布到各个节点上，会造成无法实现mget、mset等功能。但是可以利用CRC16算法计算出key对应的slot，以及Smart客户端保存了slot和节点对应关系的特性，将属于同一个Redis节点的key进行归档，然后分别对每个节点对应的子key列表执行mget或者pipeline操作，具体使用方法可以参考11.5节“无底洞优化”。

(4)使用Lua、事务等特性的方法

Lua和事务需要所操作的key，必须在一个节点上，不过Redis Cluster提供了hashtag，如果开发人员确实要使用Lua或者事务，可以将所要操作的key使用一个hashtag，如下所示：

```
// hashtag
String hastag = "{user}";
// 用户A的关注表
String userAFollowKey = hastag + ":a:follow";
// 用户B的粉丝表
String userBFanKey = hastag + ":b:fans";
// 计算hashtag对应的slot
int slot = JedisClusterCRC16.getSlot(hastag);
// 获取指定slot的JedisPool
JedisPool jedisPool = jedisCluster.getConnectionHandler().getJedisPoolFromSlot(slot);
// 在当个节点上执行事务
Jedis jedis = null;
try {
    jedis = jedisPool.getResource();
// 用户A的关注表加入用户B，用户B的粉丝列表加入用户A
    Transaction transaction = jedis.multi();
    transaction.sadd(userAFollowKey, "user:b");
    transaction.sadd(userBFanKey, "user:a");
    transaction.exec();
} catch (Exception e) {
    logger.error(e.getMessage(), e);
} finally {
    if (jedis!= null)
        jedis.close();
}
```

具体步骤如下：1)将事务中所有的key添加hashtag。2)使用CRC16计算hashtag对应的slot。3)获取指定slot对应的节点连接池JedisPool。4)在JedisPool上执行事务。

### 调试技巧

```
# 查看特定键的槽位
redis-cli -c -p 6379 CLUSTER KEYSLOT user:1000

# 查看槽位归属
redis-cli -c -p 6379 CLUSTER SLOTS | grep 1000

# 检查集群状态
redis-cli -c -p 6379 CLUSTER INFO | grep slots_assigned
```

### ASK重定向

#### 定义与触发场景

- **本质**：一种**临时性重定向**机制，仅在Redis集群进行**数据迁移（resharding）** 时触发
- **触发条件**：当客户端请求的键**正在迁移过程中**，且该键已从源节点迁移到目标节点
- 典型场景：
  - 集群扩容/缩容时的槽位迁移
  - 负载均衡导致的槽位重新分配
  - 故障恢复后的数据迁移

| 特性     | MOVED重定向                  | ASK重定向                  |
| -------- | ---------------------------- | -------------------------- |
| 性质     | 永久性重定向                 | 临时性重定向               |
| 触发时机 | 槽位已完全转移至新节点       | 槽位正在迁移过程中         |
| 缓存更新 | 更新客户端槽位缓存           | 不更新客户端槽位缓存       |
| 后续请求 | 直接发送至新节点             | 仍需先尝试源节点           |
| 错误格式 | `MOVED <slot> <target-node>` | `ASK <slot> <target-node>` |

**关键区别**：MOVED表示槽位归属已**永久变更**，而ASK仅表示当前请求需**临时转向**

#### 流程

```
graph TD
    A[客户端向源节点发送请求] --> B{源节点检查键是否存在}
    B -->|键存在| C[直接返回结果]
    B -->|键不存在| D{检查槽位迁移状态}
    D -->|槽位未迁移| E[返回MOVED重定向]
    D -->|槽位正在迁移| F[返回ASK重定向]
    F --> G[客户端向目标节点发送ASKING命令]
    G --> H[客户端向目标节点发送原始命令]
    H --> I[目标节点处理请求并返回结果]
```



1. 源节点检查：
   - 客户端向源节点（如节点A）发送请求
   - 源节点检查请求的键是否存在于当前节点

2. ASK重定向触发：
   - 若键不存在且槽位处于**MIGRATING状态**（正在迁移）
   - 源节点返回格式化错误：`(error) ASK <slot> <target-node-ip>:<target-node-port>`
   - 示例：`(error) ASK 16330 172.17.18.2:6379`
3. 客户端处理：
   - 客户端收到ASK错误后，**不更新本地槽位缓存**
   - 向目标节点（如节点B）先发送`ASKING`命令
   - 再发送原始请求命令（如`GET key`）
4. 目标节点响应：
   - 目标节点收到带`ASKING`标记的请求后，**临时处理**该槽位的请求
   - 处理完成后返回结果，**不改变槽位归属状态**

#### 实例1

- **场景**：向6节点集群添加第7个节点，触发槽位迁移

- 过程：

  1. 源节点（节点1）将槽位5461-10922的部分数据迁移到新节点（节点7）

  2. 客户端请求键`user:1001`（槽位8192），该键已迁移到节点7

  3. 节点1返回：`(error) ASK 8192 192.168.1.7:6379`

  4. 客户端向节点7发送：

     ```
     ASKING
     GET user:1001
     ```

  5. 节点7返回键值，客户端获取数据

| 场景       | MOVED重定向示例                        | ASK重定向示例                        |
| ---------- | -------------------------------------- | ------------------------------------ |
| 错误响应   | `(error) MOVED 5000 192.168.1.20:6379` | `(error) ASK 5000 192.168.1.20:6379` |
| 客户端行为 | 更新缓存，后续请求直接发往新节点       | 不更新缓存，仅本次请求转向           |
| 节点状态   | 槽位5000已完全转移                     | 槽位5000正在迁移过程中               |
| 后续请求   | 直接访问新节点                         | 仍先尝试源节点，可能再次触发ASK      |

#### 示例2

为了支持ASK重定向，源节点和目标节点在内部的clusterState结构中维护当前正在迁移的槽信息，用于识别槽迁移情况，结构如下：

```
typedef struct clusterState {
clusterNode *myself;                                /* 自身节点 /
clusterNode *slots[CLUSTER_SLOTS];          /* 槽和节点映射数组 */
clusterNode *migrating_slots_to[CLUSTER_SLOTS];/* 正在迁出的槽节点数组 */
clusterNode *importing_slots_from[CLUSTER_SLOTS];/* 正在迁入的槽节点数组*/
    …
} clusterState;
```

节点每次接收到键命令时，都会根据clusterState内的迁移属性进行命令处理，如下所示：

- 如果键所在的槽由当前节点负责，但键不存在则查找migrating_slots_to数组查看槽是否正在迁出，如果是返回ASK重定向。
- 如果客户端发送asking命令打开了CLIENT_ASKING标识，则该客户端下次发送键命令时查找importing_slots_from数组获取clusterNode，如果指向自身则执行命令。
- 需要注意的是，asking命令是一次性命令，每次执行完后客户端标识都会修改回原状态，因此每次客户端接收到ASK重定向后都需要发送asking命令。
- 批量操作。ASK重定向对单键命令支持得很完善，但是，在开发中我们经常使用批量操作，如mget或pipeline。当槽处于迁移状态时，批量操作会受到影响。

例如，手动使用迁移命令让槽4096处于迁移状态，并且数据各自分散在目标节点和源节点，如下所示：

```
#6379节点准备导入槽4096数据
127.0.0.1:6379>cluster setslot 4096 importing 1a205dd8b2819a00dd1e8b6be40a8e2abe77b756
OK
#6385节点准备导出槽4096数据
127.0.0.1:6379>cluster setslot 4096 migrating cfb28ef1deee4e0fa78da86abe5d24566744411e
OK
# 查看槽4096下的数据
127.0.0.1:6385> cluster getkeysinslot 4096 100
1) "key:test:5028"
2) "key:test:68253"
3) "key:test:79212"
# 迁移键key:test:68253和key:test:79212到6379节点
127.0.0.1:6385>migrate 127.0.0.1 6379 "" 0 5000 keys key:test:68253 key:test:79212
OK
```

现在槽4096下3个键数据分别位于6379和6380两个节点，使用Jedis客户端执行批量操作。mget代码如下：

```
@Test
public void mgetOnAskTest() {
    JedisCluster jedisCluster = new JedisCluster(new HostAndPort("127.0.0.1", 6379));
    List<String> results = jedisCluster.mget("key:test:68253", "key:test:79212");
    System.out.println(results);
    results = jedisCluster.mget("key:test:5028", "key:test:68253", "key:test:79212");
    System.out.println(results);
}
```

运行mget测试结果如下：

```
[value:68253, value:79212]
redis.clients.jedis.exceptions.JedisDataException: TRYAGAIN Multiple keys request 
    during rehashing of slot
at redis.clients.jedis.Protocol.processError(Protocol.java:127)
…
```

测试结果分析：

- 第1个mget运行成功，这是因为键key：test：68253，key：test：79212已经迁移到目标节点，当mget键列表都处于源节点/目标节点时，运行成功。
- 第2个mget抛出异常，当键列表中任何键不存在于源节点时，抛出异常。

综上所处，当在集群环境下使用mget、mset等批量操作时，slot迁移数据期间由于键列表无法保证在同一节点，会导致大量错误。

Pipeline代码如下：

```
@Test
public void pipelineOnAskTest() {
    JedisSlotBasedConnectionHandler connectionHandler = new JedisCluster(new 
        Host AndPort ("127.0.0.1", 6379)) {
        public JedisSlotBasedConnectionHandler getConnectionHandler() {
            return (JedisSlotBasedConnectionHandler) super.connectionHandler;
        }
    }.getConnectionHandler();
    List<String> keys = Arrays.asList("key:test:68253", "key:test:79212", "key:test:
        5028");
    Jedis jedis = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.
        get Slot(keys.get(2)));
    try {
        Pipeline pipelined = jedis.pipelined();
        for (String key : keys) {
            pipelined.get(key);
        }
        List<Object> results = pipelined.syncAndReturnAll();
        for (Object result : results) {
            System.out.println(result);
        }
    } finally {
        jedis.close();
    }
}
```

Pipeline的代码中，由于Jedis没有开放slot到Jedis的查询，使用了匿名内部类暴露JedisSlotBasedConnectionHandler。通过Jedis获取Pipeline对象组合3条get命令一次发送。运行结果如下：

```
redis.clients.jedis.exceptions.JedisAskDataException: ASK 4096 127.0.0.1:6379
redis.clients.jedis.exceptions.JedisAskDataException: ASK 4096 127.0.0.1:6379
value:5028
```

结果分析：返回结果并没有直接抛出异常，而是把ASK异常JedisAskDataException包含在结果集中。但是使用Pipeline的批量操作也无法支持由于slot迁移导致的键列表跨节点问题。得益于Pipeline并没有直接抛出异常，可以借助于JedisAskDataException内返回的目标节点信息，手动重定向请求给目标节点，修改后的程序如下：

```
@Test
public void pipelineOnAskTestV2() {
    JedisSlotBasedConnectionHandler connectionHandler = new JedisCluster(new Host
        AndPort("127.0.0.1", 6379)) {
        public JedisSlotBasedConnectionHandler getConnectionHandler() {
            return (JedisSlotBasedConnectionHandler) super.connectionHandler;
        }
    }.getConnectionHandler();
    List<String> keys = Arrays.asList("key:test:68253", "key:test:79212", "key:
        test:5028");
    Jedis jedis = connectionHandler.getConnectionFromSlot(JedisClusterCRC16.get
        Slot(keys.get(2)));
    try {
        Pipeline pipelined = jedis.pipelined();
        for (String key : keys) {
            pipelined.get(key);
        }
        List<Object> results = pipelined.syncAndReturnAll();
        for (int i = 0; i < keys.size(); i++) {
// 键顺序和结果顺序一致
            Object result = results.get(i);
            if (result != null && result instanceof JedisAskDataException) {
                JedisAskDataException askException = (JedisAskDataException) result;
                HostAndPort targetNode = askException.getTargetNode();
                Jedis targetJedis = connectionHandler.getConnectionFromNode(tar
                    getNode);
                try {
// 执行asking
                    targetJedis.asking();
// 获取key并执行
                    String key = keys.get(i);
                    String targetResult = targetJedis.get(key);
                    System.out.println(targetResult);
                } finally {
                    targetJedis.close();
                }
            } else {
                System.out.println(result);
            }
        }
    } finally {
        jedis.close();
    }
}
```

修改后的Pipeline运行结果以下：

```
value:68253
value:79212
value:5028
```

根据结果，我们成功获取到了3个键的数据。以上测试能够成功的前提是：1)Pipeline严格按照键发送的顺序返回结果，即使出现异常也是如此（更多细节见3.3节“Pipeline”）。2)理解ASK重定向之后，可以手动发起ASK流程保证Pipeline的结果正确性。综上所处，使用smart客户端批量操作集群时，需要评估mget/mset、Pipeline等方式在slot迁移场景下的容错性，防止集群迁移造成大量错误和数据丢失的情况。开发提示集群环境下对于使用批量操作的场景，建议优先使用Pipeline方式，在客户端实现对ASK重定向的正确处理，这样既可以受益于批量操作的IO优化，又可以兼容slot迁移场景。

# 故障转移

## 故障发现

- 主观下线（PFAIL）- 单节点初步判断
  - 触发条件：节点A向节点B发送PING消息后，**在cluster-node-timeout时间内(默认15秒)**未收到PONG响应
  - 流程说明
    - 1)节点a发送ping消息给节点b，如果通信正常将接收到pong消息，节点a更新最近一次与节点b的通信时间。
    - 2)如果节点a与节点b通信出现问题则断开连接，下次会进行重连。如果一直通信失败，则节点a记录的与节点b最后通信时间将无法更新。
    - 3)节点a内的定时任务检测到与节点b最后通信时间超高cluster-node-timeout时，更新本地对节点b的状态为主观下线(pfail)。

- 客观下线（FAIL）- 集群共识确认
  - **触发条件**：**超过半数的主节点**都认为某节点处于PFAIL状态
  - 确认流程：
    - 节点A收集到其他节点对节点B的PFAIL报告
    - 当**有效PFAIL报告数量 > 持有槽的主节点总数/2**时
    - 节点A将节点B标记为**FAIL状态**(确认故障)
    - 节点A向集群中所有节点广播FAIL消息

- 关键参数

  | 参数                              | 默认值  | 作用               | 调优建议                              |
  | --------------------------------- | ------- | ------------------ | ------------------------------------- |
  | `cluster-node-timeout`            | 15000ms | 节点通信超时阈值   | 高延迟网络→30000ms                    |
  | `cluster-slave-validity-factor`   | 10      | 从节点资格验证因子 | 高可用优先→5，数据优先→15             |
  | `cluster-replica-validity-factor` | 0       | 从节点有效性验证   | 与`cluster-slave-validity-factor`相同 |

## 故障恢复

当主节点被标记为客观下线后，Redis集群通过**严格的资格筛选、优先级排序和投票机制**确保故障转移的准确性和高效性，最终选出**数据最新、网络状态最佳**的从节点作为新主节点。

1. 资格检查-筛选合格候选者
   - **关键判断条件**：
     `断线时间 > cluster-node-timeout × cluster-slave-validity-factor`
     默认`cluster-node-timeout=15秒`，`cluster-slave-validity-factor=10`，即**断线超过150秒的从节点自动失去资格**

2. 优先级排序-确定选举顺序

   - **复制偏移量（offset）**：**从节点按以下三重标准排序**（优先级依次降低）：
     偏移量越大 → **数据越新** → 优先级越高

     复制偏移量越大说明从节点延迟越低，那么它应该具有更高的优先级来替换故障主节点。主节点b进入客观下线后，它的三个从节点根据自身复制偏移量设置延迟选举时间，如复制偏移量最大的节点slave b-1延迟1秒执行，保证复制延迟低的从节点优先发起选举。

     ![image-20260105165205309](./assets/image-20260105165205309.png)

   - **运行ID（runid）**：
     当偏移量相同时，**runid越小** → 优先级越高
     *（runid是节点启动时生成的唯一标识）*

   - **配置纪元（configEpoch）**：
     作为最终决胜标准，**纪元值越小** → 优先级越高

3. 发起选举

   当从节点定时任务检测**故障选举时间(failover_auth_time)**到达后，发起选举流程如下：

   - （1）更新配置纪元

     配置纪元是一个只增不减的整数，每个主节点自身维护一个配置纪元(clusterNode.configEpoch)标示当前主节点的版本，所有主节点的配置纪元都不相等，从节点会复制主节点的配置纪元。整个集群又维护一个全局的配置纪元(clusterState.current Epoch)，用于记录集群内所有主节点配置纪元的最大版本。执行cluster info命令可以查看配置纪元信息：

     ```
     127.0.0.1:6379> cluster info
     …
     cluster_current_epoch:15                        // 整个集群最大配置纪元
     cluster_my_epoch:13                     // 当前主节点配置纪元
     ```

     配置纪元会跟随ping/pong消息在集群内传播，当发送方与接收方都是主节点且配置纪元相等时代表出现了冲突，nodeId更大的一方会递增全局配置纪元并赋值给当前节点来区分冲突，伪代码如下：

     ```
     def clusterHandleConfigEpochCollision(clusterNode sender) :
         if (sender.configEpoch != myself.configEpoch || !nodeIsMaster(sender) || !nodeIsMaster
             (myself)) :
         return;
     // 发送节点的nodeId小于自身节点nodeId时忽略
         if (sender.nodeId <= myself.nodeId):
             return
     // 更新全局和自身配置纪元
         server.cluster.currentEpoch++;
         myself.configEpoch = server.cluster.currentEpoch;
     ```

     配置纪元的主要作用：

     - 标示集群内每个主节点的不同版本和当前集群最大的版本。
     - 每次集群发生重要事件时，这里的重要事件指出现新的主节点（新加入的或者由从节点转换而来），从节点竞争选举。都会递增集群全局的配置纪元并赋值给相关主节点，用于记录这一关键事件。
     - 主节点具有更大的配置纪元代表了更新的集群状态，因此当节点间进行ping/pong消息交换时，如出现slots等关键信息不一致时，以配置纪元更大的一方为准，防止过时的消息状态污染集群。

   - 配置纪元的应用场景有：

     - 新节点加入。
     - 槽节点映射冲突检测。
     - 从节点投票选举冲突检测。

   - （2）广播选举消息

     在集群内广播选举消息(FAILOVER_AUTH_REQUEST)，并记录已发送过消息的状态，保证该从节点在一个配置纪元内只能发起一次选举。消息内容如同ping消息只是将type类型变为FAILOVER_AUTH_REQUEST。

4. 选举投票

   - 从节点检测到主节点故障后，**递增配置纪元**并广播`FAILOVER_AUTH_REQUEST`消息。**只有持有槽的主节点才会处理故障选举消息(FAILOVER_AUTH_REQUEST)**，因为每个持有槽的节点在一个配置纪元内都有**唯一的一张选票**，当接到第一个请求投票的从节点消息时**回复FAILOVER_AUTH_ACK**消息作为投票，之后相同配置纪元内其他从节点的选举消息将忽略。

   - 投票过程其实是一个领导者选举的过程，如集群内有N个持有槽的主节点代表有N张选票。由于在每个配置纪元内持有槽的主节点只能投票给一个从节点，因此只能有一个从节点获得N/2+1的选票，保证能够找出唯一的从节点。

   - Redis集群没有直接使用从节点进行领导者选举，主要因为从节点数必须大于等于3个才能保证凑够N/2+1个节点，将导致从节点资源浪费。使用集群内所有持有槽的主节点进行领导者选举，即使只有一个从节点也可以完成选举过程。

   - **当从节点收集到N/2+1个持有槽的主节点投票时，从节点可以执行替换主节点操作**，例如集群内有5个持有槽的主节点，主节点b故障后还有4个，当其中一个从节点收集到3张投票时代表获得了足够的选票可以进行替换主节点操作

   - 从节点slave b-1成功获得3张选票

     ![image-20260105172935167](./assets/image-20260105172935167.png)

     运维提示

     故障主节点也算在投票数内，假设集群内节点规模是3主3从，其中有2个主节点部署在一台机器上，当这台机器宕机时，由于从节点无法收集到3/2+1个主节点选票将导致故障转移失败。这个问题也适用于故障发现环节。因此部署集群时所有主节点最少需要部署在3台物理机上才能避免单点问题。投票作废：每个配置纪元代表了一次选举周期，如果在开始投票之后的cluster-node-timeout*2时间内从节点没有获取足够数量的投票，则本次选举作废。从节点对配置纪元自增并发起下一轮投票，直到选举成功为止。

5. 替换主节点

   当从节点收集到足够的选票之后，触发替换主节点操作：

   1)当前从节点取消复制变为主节点。

   2)执行clusterDelSlot操作撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽委派给自己。

   3)向集群广播自己的pong消息，通知集群内所有的节点当前从节点变为主节点并接管了故障主节点的槽信息。

## 故障转移时间

在介绍完故障发现和恢复的流程后，这时我们可以估算出故障转移时间：

1)主观下线(pfail)识别时间=cluster-node-timeout。

2)主观下线状态消息传播时间<=cluster-node-timeout/2。消息通信机制对超过cluster-node-timeout/2未通信节点会发起ping消息，消息体在选择包含哪些节点时会优先选取下线状态节点，所以通常这段时间内能够收集到半数以上主节点的pfail报告从而完成故障发现。

3)从节点转移时间<=1000毫秒。由于存在延迟发起选举机制，偏移量最大的从节点会最多延迟1秒发起选举。通常第一次选举就会成功，所以从节点执行转移时间在1秒以内。

根据以上分析可以预估出故障转移时间，如下：

```
failover-time(毫秒) ≤ cluster-node-timeout + cluster-node-timeout/2 + 1000
```

因此，故障转移时间跟cluster-node-timeout参数息息相关，默认15秒。配置时可以根据业务容忍度做出适当调整，但不是越小越好，下一节的带宽消耗部分会进一步说明。



​        

