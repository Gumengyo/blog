title: Redis消息队列
tags:
  - Redis
categories:
  - 学习笔记
index_img: 'https://cdn.jishuqin.cn//lk956kjc.png'
author: 顾梦
date: 2023-04-05 12:49:00

---

## Redis消息队列

### 1.认识消息队列

**消息队列**（**M**essage **Q**ueue），字面意思就是存放消息的队列。

最简单的消息队列模型包括3个角色：

* 消息队列：存储和管理消息，也被称为消息代理（Message Broker）

* 生产者：发送消息到消息队列

* 消费者：从消息队列获取消息并处理消息

![](https://cdn.jishuqin.cn//202304051317741.png)

Redis提供了三种不同的方式来实现消息队列：

- list结构：基于List结构模拟消息队列

- PubSub：基本的点对点消息模型

- Stream：比较完善的消息队列模型

### 2.基于List实现消息队列

Redis的list数据结构是一个双向链表，很容易模拟出队列效果。

队列是入口和出口不在一边，因此我们可以利用：LPUSH 结合 RPOP、或者 RPUSH 结合 LPOP来实现。

不过要注意的是，当队列中没有消息时RPOP或LPOP操作会返回null，并不像JVM的阻塞队列那样会阻塞并等待消息。因此这里应该使用**BRPOP**或者**BLPOP**来实现阻塞效果。

命令：

`LPUSH key value1 [value2]`		将一个或多个值插入到列表头部。

` RPOP key`		移出并获取列表的最后一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

`BRPOP key1 [key2] timeout`：移除并获取列表最后一个元素。

`RPUSH key value1 [value2]`：在列表中添加一个或多个值。

` BLPOP key1 [key2] timeout`：移出并获取列表的第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

![](https://cdn.jishuqin.cn//1653575176451.png)

优点：

- 利用Redis存储，不受限于JVM内存上限

- 基于Redis的持久化机制，数据安全性有保证

- 可以满足消息有序性

缺点：

- 无法避免消息丢失

- 只支持单消费者

### 3.基于PubSub的消息队列

**PubSub（发布订阅）**是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。

命令：

` SUBSCRIBE channel [channel] `：订阅一个或多个频道
 `PUBLISH channel msg` ：向一个频道发送消息
` PSUBSCRIBE pattern[pattern] `：订阅与pattern格式匹配的所有频道

![](https://cdn.jishuqin.cn/1653575506373.png)

优点：

* 采用发布订阅模型，支持多生产、多消费

缺点：

* 不支持数据持久化
* 无法避免消息丢失
* 消息堆积有上限，超出时数据丢失

### 4.基于Stream的消息队列

Stream 是 Redis 5.0 引入的一种新数据类型，可以实现一个功能非常完善的消息队列。

它支持消息的持久化、支持自动生成全局唯一D、支持ack确认消息的模式、支持消费组模式等，让消息队列更加的稳定和可靠。

##### Stream结构

![](https://cdn.jishuqin.cn//202304051338733.png)

| 名称            | 含义                                                         |
| --------------- | :----------------------------------------------------------- |
| Message Content | 消息内容                                                     |
| Consumer group  | 消费组，通过XGROUP CREATE 命令创建，同一个消费组可以有多个消费者 |
| Consumer        | 游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。 |
| Consumer        | 消费者，消费组中的消费者                                     |
| Pending_ids     | 消费者，消费组中的消费者                                     |

##### 队列相关指令

| 指令名称  | 指令作用                                   |
| --------- | ------------------------------------------ |
| XADD      | 添加消息到队列末尾                         |
| XTRIM     | 限制Stream的长度,如果已经超长会进行截取    |
| XDEL      | 删除消息                                   |
| XLEN      | 获取Stream中的消息长度                     |
| XRANGE    | 获取消息列表(可指定范围),忽略删除的信息    |
| XREVRANGE | 和XRANGE相比区别在于反向获取,ID从大到小    |
| XREAD     | 获取消息(阻塞/非阻塞),返回大于指定ID的消息 |

STREAM类型消息队列的XREAD命令特点：

* 消息可回溯
* 一个消息可以被多个消费者读取
* 可以阻塞读取
* 有消息漏读的风险

##### 消费组相关指令

| 指令名称           | 指令作用                                                     |
| ------------------ | ------------------------------------------------------------ |
| XGROUP CREATE      | 创建消费者组                                                 |
| XREADGROUP GROUP   | 读取消费者组中的消息                                         |
| XACK               | ack消息，消息被标记为"已处理                                 |
| XGROUP SETID       | 设置消费者组最后递送消息的D                                  |
| XGROUP DELCONSUMER | 删除消费者组                                                 |
| XPENDING           | 打印待处理消息的详细信息                                     |
| XCLAIM             | 转移消息的归属权（长期未被处理/无法处理的消息，转交给其他消费者组进行处理） |
| XINFO              | 打印Stream\Consumer八Group的详细信息                         |
| XINFO GROUPS       | 打印消费者组的详细信息                                       |
| XINFO STREAM       | 打印Streaml的详细信息                                        |

STREAM类型消息队列的XREADGROUP命令特点：

* 消息可回溯
* 可以多消费者争抢消息，加快消费速度
* 可以阻塞读取
* 没有消息漏读的风险
* 有消息确认机制，保证消息至少被消费一次

##### 四个特殊符号

1. `- +` ：最小和最大可能出现的id。

2. `$` ：$表示只消费新的消息，当前流中最大的d,可用于将要到来的信息。

3. `>` ：用于XREADGROUP命令，表示迄今还没有发送给组中使用者的信息，会更新消费者组的最后ID。

4. `*` ：用于XADD命令中，让系统自动生成id。
---
#### 对比:

![](https://cdn.jishuqin.cn//202304051356489.png)







