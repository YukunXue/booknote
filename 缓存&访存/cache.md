# Cache 学习笔记

## 1.存储系统的速度影响设计者对Cache数据块容量的判断

存储延迟越短 -> cache数据块容量小 <- 因为可以及时从存储单元中更新数据信息

存储带宽越高 -> cache数据块容量大 <- 因为一次性可以写很多数据到cache中 （因为使用大容量的数据块会降低失效率，减少cache中与数据存储相关的标签存储），通常加大DRAM带宽来避免性能损失-->加宽主存和交叉访问

## 2.失效率降低和相联度关系

相连度每增加1倍，cache组数将减少1/2

--> 索引的长度每减少1位，标签的长度就增加1位

### 2.1 使用多级cache减少失效代价

一级cache更关心命中时间，二级cache更关心失效率

```
增加相联系度--> 降低失效率 --> 但是增加了代价和访问时间

随着cache容量增加，相联度的提高对性能改进作用小---> 因为大容量的cache总失效率低，改善失效率的机会减少 

```

### 2.2 实现一致性的基本方案

在一个支持cache一致性的多核处理器中，cache为每个共享数据项提供迁移和复制：

* 1.迁移 ： 数据项可以移动到本地cache并以透明的方式被使用。

* 2.复制 ： 当同时读取共享数据，cache会在本地cache中创建数据项的副本

**实现Cache一致性协议的关键是追踪每一个共享数据块的状态**


## Summary:
[refer](https://www.gatevidyalay.com/cache-line-cache-line-size-cache-memory/)

Keep the cache size constant

## 1. The larger the block size, better will be the spatial locality

`Effect of Changing Block Size on Spatial Locality`

## 2. In direct mapped cache, block size does not affect the cache tag anyhow.

`Effect of Changing Block Size On Cache Tag in Direct Mapped Cache`

## 3. In fully associative cache, on decreasing block size, cache tag is reduced and vice versa.

`Effect of Changing Block Size On Cache Tag in Fully Associative Cache`

## 4. In set associative cache, block size does not affect cache tag anyhow.

`Effect of Changing Block Size On Cache Tag in Set Associative Cache`

## 5. A smaller cache block incurs a lower cache miss penalty.
`A smaller cache block incurs a lower cache miss penalty.`

## 6. A smaller cache tag ensures a lower cache hit time.
`Effect of Cache Tag On Cache Hit Time`

##  Cache Coherence  策略：

* Snoopying-based

    每个cache block 对应着一个共享的状态，系统中所有的cache控制器通过共享总线互联，每个cache控制器都监控着共享总线上的操作，根据共享总线上的命令来更新自己的共享状态


* Directory-based
    
    在某个单一的位置（directory）对某个cache line的共享状态进行跟踪


Notebook:

《量化》：
存储器： 附录B，第二章，附录D，附录M

Snooping protocol: 【A cache controller initiates a request for a block by broadcasting a request message to all other coherence controllers.】 The coherence controllers 【collectivel “do the right thing】,” e.g., sending data in response to another core’s request if they are the owner. Snooping protocols rely on the interconnection network to deliver the 【broadcast messages in a consistent order to all cores】. Most snooping protocols assume that requests arrive in a total order, e.g., via a shared-wire bus, but more advanced interconnection networks and relaxed orders are possible.

Directory protocol: 【A cache controller initiates a request for a block by unicasting it to the memory controller that is the home for that block.】 The memory controller maintains a directory that holds state about each block in the LLC/memory, such as the identity of the current owner or the identities of current sharers. When a request for a block reaches the home, the memory controller looks up this block’s directory state. For example, if the request is a GetS, the memory controller looks up the directory state to determine the owner. If the LLC/memory is the owner, the memory controller completes the transaction by sending a data response to the requestor. If a cache controller is the owner, the memory controller forwards the request to the owner cache; 【when the owner cache receives the forwarded request, it completes the transaction by sending a data response to the requestor.】

The choice of snooping vs. directory involves making tradeoffs. Snooping protocols are logically simple, but they do not scale to large numbers of cores because broadcasting does not scale. Directory protocols are scalable because they unicast, but many transactions take more time because they require an extra message to be sent when the home is not the owner. In addition, the choice of protocol affects the interconnection network (e.g., classical snooping protocols require a total order for request messages).

## Cache Slice

divide the LLC into slices with the purpose of reducing the bandwidth bottlenecks when more than one core attempts to retrieve data from the LLC at the same time

