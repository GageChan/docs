# 1.节点说明

| 节点       | 说明                                                |
| ---------- | --------------------------------------------------- |
| /dubbo     | 服务注册节点(服务内部通信rpc框架选用的apache dubbo) |
| /master    | 主节点，存放数据节点中的key所在node                 |
| /DataNode1 | 数据节点1，存放数据                                 |
| /DataNode2 | 数据节点2，存放数据                                 |
| /lock      | 用作分布式锁                                        |

# 2.数据走向

### put and delete

![image-20200628234916536](/Users/gagechan/Library/Application Support/typora-user-images/image-20200628234916536.png)

### read

![image-20200628235155014](/Users/gagechan/Library/Application Support/typora-user-images/image-20200628235155014.png)

# 3.分布式锁说明

在操作某个key之前判断/lock下是否存在该节点，如若存在，则说明正有其他节点正在操作该key，则熔断。如若不存在该key, 则为该key上锁，执行完对应的操作后，释放该key所持有的锁。

分布式锁的定义:common->com.wx.lock.Lock

# 4.特别说明

* 如果data主节点挂掉，自定义负载均衡就会找到对应数据节点的备份节点详见                                                 client->com.wx.loadbalance.CustomLoadBalance#select
* DataNode1,DataNode2,DataNodeBack1,DataNodeBack2,master启动时都会校验zk上是否存在自身节点所对应的数据分区，如果没有，则创建; master会多一个操作: 查看是否存在/lock，如若不存在则创建。
* 当服务启动完成会监听自身连接状态的变化，用于维护zk连接的session长存，断线重连。
