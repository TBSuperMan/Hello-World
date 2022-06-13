# 补充

## 服务器之间互相之间如何检测是否宕机（SpringBoot健康检查与监控？）




# 基础知识

## CAP 定理

C：Consistency一致性：用户访问分布式系统中任一节点，得到的数据具有一致性
A：Availability可用性：用户访问集群中的任意健康节点，必须得到响应，而不是超时或拒绝
P：Partition tolerance分区容错性：大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区，分布式系统遇到**任何**节点或网络分区故障时，分区之间可能无法通信，**此时整个系统仍然能够对外提供满足一致性与可用性的服务**，即**部分故障不影响整体使用**

CAP保证同一数据在分布式系统中，不同服务器上的拷贝是相同的，是一种逻辑保证 ，分布式系统一定会出现分区，因此CAP无法同时满足

- CA：优先保证一致性与可用性，放弃分区容错，即放弃系统的扩展性，违背了分布式的初衷
- CP：优先保证一致性与分区容错性，放弃可用性。
- AP：优先保证可用性与分区容错性，放弃一致性


## zk是cp还是ap，对最终一致性的理解，zk如何保证强一致性

Zookeeper是一个CP系统
> Zookeeper不能保证每次服务请求的可用性（极端情况下，zk会丢一一些请求，消费者程序需要重新请求才能得到结果）
> 进行Leader选举时，集群是不可用的

- 最终一致性：系统中的所有分散在不同节点的数据，经过一定时间后，最终能够达到符合业务定义的一致的状态。
- 弱一致性：系统中的某个数据被更新后，后续对该数据的读取操作可能得到更新后的值，也可能是更改前的值。但**经过“不一致时间窗口”这段时间后，后续对该数据的读取都是更新后的值**；

## ZK的特点

- 顺序一致性： 从同一客户端发起的事务请求，最终将会**严格地按照顺序**被应用到 ZooKeeper 中去。
- 原子性： **所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的**，也就是说，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。
- 单一系统映像 ： 无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。
- 可靠性： 一旦一次更改请求被应用，**更改的结果就会被持久化**，直到被下一次更改覆盖。

## ZK节点类型

- 持久节点：一旦创建就一直存在（即使 ZooKeeper 集群宕机也保留），直到将其删除
  - 客户端和服务端断开连接口，创建的节点不删除
- 短暂节点：**临时节点的生命周期是与 客户端会话（session） 绑定的，会话消失则节点消失** 。并且，**临时节点只能做叶子节点 ，不能创建子节点**
  - **客户端和服务端断开连接口，创建的节点自己删除**
- 持久顺序节点：除了持久节点的特性之外，ZK额外给该节点名称进行顺序编号
- 临时顺序节点：除了临时节点的特性之外，ZK额外给该节点名称进行顺序编号

创建ZNode时设置顺序表示，ZNode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护
> 在分布式系统中，顺序号可以被用于所有时间的全局排序，这样一来，客户端可以通过顺序号推断事件的顺序

## 权限控制

ZooKeeper 采用 ACL（AccessControlLists）策略来进行权限控制，类似于 UNIX 文件系统的权限控制。

对于 znode 操作的权限，ZooKeeper 提供了以下 5 种：

CREATE : 能创建**子节点**
READ ：能获取节点数据和列出其子节点
WRITE : 能设置/更新节点数据
DELETE : 能删除**子节点**
ADMIN : 能设置节点 ACL 的权限

其中尤其需要注意的是，CREATE 和 DELETE 这两种权限都是针对 **子节点** 的权限控制。

## ZK事件监听器

Watcher（事件监听器），是 ZooKeeper 中的一个很重要的特性。
ZK 允许用户在指定节点上注册一些 Watcher，并且**在一些特定事件触发的时候，ZK服务端会将事件通知到感兴趣的客户端上去**，该机制是 ZooKeeper 实现分布式协调服务的重要特性。

## ZK怎么实现消息持久化



## ZK集群角色有哪些？

ZK集群采用的是`Master/Slave模式`，在这种模式中，**Master节点负责写服务，Slave 服务器通过**异步复制**的方式获取 `Master` 服务器最新的数据，并提供读服务**

但是，ZK并没有采用传统的角色划分，而是将节点划分为`Leader`、`Follower`和`Observer`三类：
- Leader：为客户端提供**读和写**的服务，负责投票的发起和决议，更新系统状态。
- Follower：为客户端提供**读**服务，如果是写服务则转发给 Leader。参与选举过程中的投票。
- Observer：为客户端提供**读**服务，如果是写服务则转发给 Leader。
  - 不参与选举过程中的投票，也不参与“过半写成功”策略。在不影响写性能的情况下提升集群的读性能。

## ZAB协议原理

http://www.jasongj.com/zookeeper/fastleaderelection/

ZK在解决分布式数据一致性问题时，没有直接使用paxos，而是使用了 **支持崩溃恢复** 的 **原子广播协议ZAB**
  
==在Zookeeper中主要依赖Zab协议来实现数据一致性==，基于该协议，zk实现了一种主从模型（即Leader和Follower模型）的系统架构来保证集群中各个副本之间数据的一致性。

### 关键属性

- myid：集群中节点的编号
- **zxid（ZooKeeper Transaction Id）**：标识该节点的状态被修改的id，是一个全局递增的id，可以用于标识对ZK节点的修改顺序，越大则说明数据越新
  - 也可以作为Leader发出的事务Proposal的递增全局ID
- peerEpoch：Leader的任职周期，每新选出一名Leader则+1
- electionEpoch：用于记录当前选举的轮次数，每次Leader选举则+1，不论是否选举成功

### ZAB协议的基本模式：崩溃恢复与消息广播

- 崩溃恢复：当**整个ZK集群正在启动**，或**Leader宕机、与Leader保持连接机器不超过半数**，ZAB协议将进入崩溃恢复模式，选举新的Leader。
  - 当选举得到新Leader，且集群中==过半==的机器与Leader**完成状态同步**，ZAB则退出崩溃恢复模式
    - 状态同步：即数据同步，用来保证集群中存在过半的机器能够和 Leader 服务器的数据状态保持一致。

- 消息广播：ZAB完成崩溃恢复后，即进入消息广播模式。

### 消息广播

首先介绍消息广播

由于ZK集群中，只有Leader能处理写请求，它需要在写之后让Follower与Observer同步更新数据，而**数据副本的传递策略就是采用消息广播模式**。zookeeper中数据副本的同步方式与2PC相似，但是却又不同。
  > 2PC要求事务协调者必须等到**所有的参与者**全部反馈ACK确认消息后，再发送`commit`消息。
  > 要求所有的参与者要么全部成功，要么全部失败。阻塞同步的2PC会产生严重的阻塞问题

Zab协议中 Leader 等待 Follower 的ACK反馈消息是指**只要半数以上**的Follower成功反馈即可，不需要收到全部Follower反馈

![](./../images/zookeeper/zab-消息广播.png)

具体步骤：
1. 客户端发起写请求
2. `Leader`将写请求转化为`事务Proposal`并提出，同时**为每个Proposal分配一个递增的全局ID`zxid`**
3. `Leader`为每个Follower分配==一个FIFO队列==，然后将需要广播的`Proposal`依次放到队列中
   - FIFO队列也是为了保证数据的一致性，**防止写请求以不同的顺序进入队列**
   - 由于ZAB是通过`TCP`来进行网络通信的，就**保证了消息的有序性**
4. `Follower`接收到`Proposal`后，首先==将其以事务日志的方式写入本地磁盘==，写入成功后向Leader反馈一个`ACK响应`
5. Leader接收到**半数以上的ACK**后，即认为消息发送成功，可以发送`commit`消息
   - 注意，这里的半数以上仅限FOLLOWER，不考虑LEADER和OBSERVER 
6. Leader向所有Follower**广播commit消息**，同时自身完成事务提交。**Follower接收到commit消息后，会将上一条事务提交**

> Zookeeper采用ZAB协议的核心，即**只要有一台服务器提交了Proposal，就要确保所有服务器最终能正确提交Proposal**。
> 这也是CAP/BASE实现**最终一致性**的一个体现

### Leader选举

介绍崩溃恢复前，首先需要介绍Leader选举的流程，因为一旦崩溃的是Leader，或能与Leader连接的节点少于总节点的一半，就需要重新进行Leader选举

集群选举场景：
- 集群启动时，进行初次选举
- Follower重启，或者发生网络分区后找不到Leader，会进入LOOKING状态并发起新的一轮投票
  - 如果发现了已有Leader，就成为其Follower
- Leader重启，Follower发现Leader宕机了，会进入LOOKING状态

Zookeeper节点的4种状态：
- `LEADING`：说明此节点已经是leader节点，只有leader才有写权限
- `FOLLOWING`：跟随者，说明当前集群的leader已经选举得到了。该节点负责以下功能：
  - 向leader发送请求（PING、REQUEST、ACK、REVALIDATE）
  - 接收leader消息并进行处理
  - 接收client发送的请求，如果是写请求，则发送给Leader进行投票处理，然后把结果返回给client
- `LOOKING`：**选举中，正在寻找leader**
- `OBSERVING`：类似FOLLOWING，但不参加投票和选举

原则：
- 同一任职周期（epoch）内，节点的zxid越大，表示该节点的数据越新，zxid最大的节点优先获得选票。
- 节点的zxid相同，则myid最大的节点，优先获得选票。

每个ZK服务器节点收到选票提议之后，只有两个选择：
- 接受选票提议，认可提议中推荐的服务器作为候选Leader
- 不接受选票提议，推荐自己**上一次推荐的服务器**作为Leader候选人。
  - 选举开始是总是推荐自己作为候选人，选举中会根据收到的选票信息决定是否更换推荐候选人

ZAB选举算法：
1. **状态转换**
   - 节点刚启动时，为Following状态，尝试与Leader通过心跳机制感应彼此，在发现没有Leader后，切换为`LOOKING`状态
2. **自增选举轮次**
   - ZK规定，所有有效的选票应当处于同一选举轮次，因此，每个进入`LOOKING`状态的节点，首先自增服务器中的`logicalClock`
3. **发送初始化选票**
   - 每个ZK服务器都维护了一个票箱`recvset`，记录其收到的每一个投票者的最后一票。在广播选票前，需要清空票箱。
   - 最开始时，**异步地广播选举自己为Leader的票**，内容为`<选举轮次electionEpoch，被投节点的zxid，被投节点的myid>`
4. **接收外部选票**
   - 服务器尝试接收其他服务器发来的选票，并记录入自己的票箱内。
   - 如果无法获得任何外部选票，则检查自己与集群中的其他服务器是否保持连接。
     - 是则再次发送选票，否则重新与其建立连接 
5. **判断选举轮次**
   - 节点收到外部投票后，首先比较`选票的electionEpoch`与`自己的logicalClock`，判断是不是最新一轮的选票
     - 外部的`electionEpoch`大于自己的`logicalClock`，说明当前服务器落后于其他服务器的选举轮次，立即清空票箱，并更新`logicalClock`，然后进行选票PK
     - 相等，立刻进行选票PK
     - 小于自己的`logicalClock`，直接忽略该选票
6. **选票PK**
   - 基于选票的`vote_zxid, vote_myid`与自己的`zxid, myid`进行对比
     - 若外部选票的`vote_zxid`更大，则将自己选票中的`zxid, myid`修改为`vote_zxid, vote_myid`并广播出去，相当于与外部选票投给一样的服务器。另外，还需要将接受的票与自己的票更新到票箱中。
     - 若`zxid`一致，则比较`myid`，同样是大的优先，其他同上
7. **统计选票**
   - **当服务器节点发现自己获得的票超过半数**，该服务器的状态转为`LEADING`，并成为**Leader**

> ZK中的ZAB算法与论文中的ZAB算法有出入
> 这是因为在ZK的实现中，使用的是fast leader election，会选举拥有**最新事务Proposal**的Follower节点作为Leader

所以，成为Leader的条件为：
- 拥有**最大的electionEpoch**
- 若epoch相等，则选取`zxid`最大的
- 若epoch和zxid相等，则选取`myid`最大的

### 崩溃恢复

崩溃恢复的问题，也就是当集群中有机器挂了，我们整个集群如何保证数据一致性

Follower挂了，一般问题不大（除非剩余的低于半数+1），但如果Leader挂了，就需要重新选举

![](./../images/zookeeper/崩溃恢复-1.png)

如图所示：
- 挂掉的旧Leader收到了三个写请求：P1、P2、P3，但只commit了其中两个：C1、C2
- 进行Leader选举后，具有最大zxid的B当选为Leader

这时候，需要确保：
1. **已经被Leader提交的提案最终能够被所有的Follower提交**
   - 在上例中，C、D和E成为Follower且以B为Leader后，会**主动将自己最大的zxid发送给B**，**B会将Follower的zxid与自身zxid间的所有被Commit过的消息同步给Follower**
     - 由于B被选中，是因为它拥有最大的`zxid`，旧Leader提交过的数据，在B中一定也是提交了的，即其数据是与`旧Leader`同步且最新的。
     - 所以能够保证：**已经被Leader提交的提案最终能够被所有的Follower提交**
   - 在完成数据同步后，Leader节点B会向Follower发送`NEWLEADER`命令，并等待大多数服务器的ACK，然后向所有服务器广播`UPTODATE`命令，即可对外服务

2. **丢弃未commit的proposal**（这里的commit是指Leader在收到大多数ACK后的commit）
   - 在上例中，P3未被`旧Leader`提交过（因为没有过半的服务器返回ACK），因此B也未Commit P3，所以它不会将P3广播出去
   - 具体做法：
     1. B在成为Leader后，先判断**自身未Commit的消息（本例中即P3）是否存在于大多数服务器中**，从而决定是否要将其Commit。
     2. 然后B可得出自身所包含的被Commit过的消息中的最小zxid（记为`min_zxid`）与最大zxid（记为`max_zxid`）
     3. C、D和E向B发送**自身Commit过的最大消息zxid（记为max_zxid）以及未被Commit过的所有消息（记为zxid_set）**。B根据这些信息作出如下操作：
        - Follower的`max_zxid`与Leader的`max_zxid`相等，说明该Follower与Leader**完全同步，无须同步任何数据**
        - Follower的`max_zxid`在Leader的`(min_zxid，max_zxid)`范围内，Leader会通过`TRUNC`命令通知Follower将其`zxid_set`中**大于Leader的max_zxid（如果有）的所有消息全部删除**
    > 上述操作保证了未被Commit过的消息不会被Commit从而对外不可见。


![](./../images/zookeeper/崩溃恢复-2.png)

## ZAB协议的作用

1. 高可用：**当Leader节点出现异常时，整个zk集群仍能正常工作**
2. 数据一致性（共识性）：**使用一个单一的主进程（leader）来接受并处理客户端的事务请求（写请求）**，并采用了Zab的原子广播协议，将服务器数据的状态变更以 **事务proposal （事务提议）**的形式**广播**到所有的副本（Follower）进程上去
3. **保证一个全局的变更序列被顺序引用**
  > Zookeeper是类似Unix的树形结构，很多操作需要保证父节点存在才能执行

## ZAB协议核心

核心：**定义了事务请求的处理方式**

1. **所有的事务请求**必须由一个**全局唯一的服务器**来协调处理，这样的服务器被叫做 **Leader服务器**。其他剩余的服务器则是 Follower服务器
2. Leader服务器 负责将一个客户端事务请求，转换成一个 **事务Proposal**，并将该 Proposal 分发给集群中所有的 Follower 服务器，也就是**向所有 Follower 节点发送数据广播请求（或数据复制）**
3. 分发之后Leader服务器需要**等待所有Follower服务器的反馈（Ack请求）**，在Zab协议中，只要超过半数的Follower服务器进行了正确的反馈后（也就是收到半数以上的Follower的Ack请求），那么 Leader 就会**再次向所有的 Follower服务器发送 Commit 消息**，要求其将上一个 事务proposal 进行提交。

![](./../images/zookeeper/zk-事务请求处理.png)

## ZAB协议如何保证数据一致性

假设两种异常：
1. 假设在Leader提出Proposal之后挂了
   - 其他Follower通过Proposal，并将其持久化到本地磁盘，返回ACK给Leader 
2. 一个事务在Leader上提交了，且过半Follower响应ACK，**但Leader在发出commit之前挂了**
 
为了在以上情况满足一致性，ZAB协议的**崩溃恢复算法**必须满足以下要求：
1. 确保**Leader提交的Proposal**，必须**最终被所有Follower服务器提交**
2. 确保**丢弃没有被成功提交的Proposal**

根据上述要求，ZAB协议需要保证选举出来的Leader具有以下条件：
1. 新选举的Leader**不能包含未提交的Proposal**。
  > 即新选举的Leader必须都是**已经提交了Proposal的Follower**
2. 新选举的Leader中含有**最大的zxid**
  > 保证新Leader中的数据时最新的，可以避免Leader检查Proposal的提交和丢弃

## ZAB数据同步过程中，如何处理需要丢弃的Proposal

当一台机器加入集群，以Follower的身份连接Leader，Leader会根据**自己服务器上最后提交的Proposal的zxid，即max_zxid**与Follower的**Proposal的max_zxid**进行对比
如果这台机器包含了**上一个Leader周期中尚未提交过的事务Proposal**，基于zxid的设计策略，**Leader会要求Follower进行回退**，从而保证最新Proposal确实被集群中的过半机器Commit

## ZAB协议如何保证消息的有序性

1. Leader为**每个Follower分配了一个FIFO队列，用于存储写请求**
2. **ZAB通过TCP进行连接**，保证了数据的有序性

由于Zab协议需要保证每一个消息的严格的顺序关系，因此**必须将每一个proposal按照其zxid的先后顺序进行排序和处理**。

## zxid的设计

在ZAB的事务编号`zxid`设计中，zxid是一个**64位的数字**，其中：
- 低32位可以看成一个简单的递增计数器，**针对客户端的每一个事务请求，Leader在产生新的Proposal事务时，都会对该计数器加一**。
- 高32位则代表了**Leader周期的epoch编号**
> epoch 编号可以理解为**当前集群所处的年代**，或者选举周期
> **每次Leader变更之后都会在 epoch 的基础上加1**，这样一来，旧的 Leader 崩溃恢复之后，**其他Follower也不会听它的了**，因为 Follower 只服从epoch最高的 Leader 命令。

每当选举产生一个新的Leader，就会从这个Leader服务器上取出本地日志中**具有最大编号的事务Proposal的zxid**，并**从zxid中解析得到对应的epoch编号，对其加一**，作为新的epoch值，并**将低32位归零**，重新生成zxid


## ZAB协议如何完成数据同步

1. 完成Leader选举后，在正式工作开始之前（接受新的事务请求），Leader服务器会**首先确认事务日志中的所有Proposal是否已经被集群中的过半的服务器Commit**
2. Leader服务器需要确保**所有的Follower能够接受每一条事务的Proposal**，并且**能将所有已经提交的事务Proposal应用到内存数据中**。
3. 等到Follower**将所有尚未同步的事务Proposal都从Leader服务器上同步过来**，并应用到内存数据后，Leader才会把该Follower加入可用Follower列表中


## ZAB为啥设置奇数个节点

假如ZAB集群中有三个节点，挂了一个还能正常工作，但挂了两个就不能正常工作了（**已经没有超过半数的节点数了，所以无法进行投票等操作了**）。
而假设现在有四个节点，挂了一个也能工作，但是**挂了两个也不能正常工作了**，**多出来的那个节点并没有带来额外的可用性收益**，所以 Zookeeper 推荐奇数个 server

但是，更加重要的一点是：使用2N+1个节点，可以很好地避免==脑裂问题==：
**当网络分裂后，==始终有一个集群的节点数量过半数，而另一个节点数量小于N+1==, 因为选举Leader需要过半数的节点同意，所以不会分裂为两个Leader集群**

## ZK的脑裂问题

脑裂问题发生在ZK的旧版本上，由于新版本采用的是fast leader election算法，不会产生脑裂
> 假如原来有7台服务器，其中3台因为网络原因，无法与Ledaer正常连接，但它们之间是可以正常连接的，于是这3个节点就会发起Leader选举。
> 在旧版本就会导致出现2个Leader集群：4台的和3台的，出现脑裂
> 但在新版本，由于要求了**想要完成Leader选举，就必须得到半数以上的选票**，因此，只有3台机器的集群是无法满足这一要求的



## Paxos算法

Paxos算法是基于**消息传递**且具有**高度容错特性**的一致性算法，是目前公认的解决分布式一致性问题最有效的算法之一，要优于2PC与3PC。

> Paxos算法的背景
> >在常见的分布式系统中，总会发生诸如机器宕机或网络异常（包括消息的延迟、丢失、重复、乱序，还有网络分区）等情况。Paxos算法需要解决的问题就是**如何在一个可能发生上述异常的分布式系统中，快速且正确地在集群内部对某个数据的值达成一致，并且保证不论发生以上任何异常，都不会破坏整个系统的一致性**
> > > 这里的“某个数据的值”并不是狭义的某个数，可以是一条命令或一条日志
 
### 相关概念

在Paxos算法中，有三种角色：1. Proposer； 2. Acceptor； 3. Learners
一个进程可以有多种角色

另一个重要概念：提案（Proposal），提案包含了最终要达成一致的value
> 提案 = 提案编号 + value

Paxos算法与2PC一样，包含`Prepare`与`accept`两个阶段

> Proposer可以提出（propose）提案；
> Acceptor可以接受（accept）提案；
> 如果某个提案被选定（chosen），那么该提案里的value就被选定了。

> 对某个数据的值达成一致，是指Proposer、Acceptor、Learner都认为同一个value被选定（chosen）
> > Proposer：只要Proposer发送的提案proposal被Acceptor接受，Proposer就认为该提案里的value被选定
> > Acceptor：只要Acceptor接受了某个提案，Acceptor就认为该提案里的value被选定
> > Learner：由Acceptor告知的结果，决定哪个value被选定

### 推导过程

在多个Proposer与多个Acceptor的场景下，需要一个一致性算法，保证在Proposer提出的value中，最终只有一个被chosen。

如果我们希望即使只有一个Proposer提出了一个value，该value也最终被选定。于是得到约束P1

> 约束P1：一个Acceptor必须接受它收到的第一个proposal；

但此时，如果每个Proposer分别提出不同的value，发给不同的Acceptor。根据P1，Acceptor分别接受自己收到的value，就导致不同的value被选定。出现了不一致。

![](./../images/zookeeper/paxos-p2.png)

所以需要规定：

> 一个proposal被选定，需要被半数以上的Acceptor接受
> 同时令 提案 = 提案编号 + value

虽然允许多个提案被选定，但必须保证所有被选定的提案都具有相同的value值。否则又会出现不一致。于是有了约束P2

> 约束P2：如果某个value为v的提案被选定了，那么每个编号更高的被选定提案的value必须也是v
>  > 同时，可以将P2改写成对 Acceptor的约束P2a
> 
> 约束P2a：如果某个value为v的提案被选定了，那么 每个编号更高的被Acceptor接受的提案的value必须也是v

但在下图的情况中，Acceptor1宕机，且其余Acceptor接受了Proposer2提出的`[M1, V1]`。
当Acceptor1恢复后，收到Proposer1提出的`[M2, V2]`提案，这是它第一个收到的提案，必须接受，同时Acceptor1认为V2被选定，就出现了不一致问题

![](./../images/zookeeper/paxos-p2a.png)

所以需要对 约束P2a 进一步强化

>约束P2b：如果某个value为v的提案被选定了，那么之后任何Proposer提出的编号更高的提案的value必须也是v。
> >约束P2b是对Proposer的约束
> >可以通过P2b推导得到P2a，进一步得到P2
> >而 约束P2b 由下述 约束P2c 保证
> 
> 约束P2c：对于任意的N和V，如果提案[N, V]被提出，那么存在一个半数以上的Acceptor组成的集合S，满足以下两个条件中的任意一个：
> - S中每个Acceptor都没有接受过编号小于N的提案
> - S中Acceptor接受过的最大编号的提案的value为V。

为了满足约束P2b，还有一个重要思想：Proposer生成提案前，应当先==学习==已经chosen或可能被chosen的value，然后以该value为自己提出的proposal的value。如果没有value被选定，Proposer才可以自己决定value的值。
这个过程是通过一个==Prepare请求==实现的


### paxos算法详述
**prepare阶段（Proposer生成提案）**

1. `Proposer`选择一个**具有全局唯一性的、递增的提案编号N**，然后向某个Acceptor集合（半数以上）发送`prepare`请求，将提案编号N发给Acceptor集合，要求该集合中的每个Acceptor做出如下响应（response）
  a. 保证**不再接受编号小于N的提案**（即不再接受编号小于N的Accept请求）
  b. 保证**不再接受编号小于等于N的Prepare请求**
  > 如果Acceptor已经接受过提案，那么就向Proposer响应**已经接受过的编号小于N的最大编号的提案maxN**，没有就返回空值

**accept阶段**

2. 如果Proposer收到了**半数及以上的Acceptor的响应**，则它会向所有Acceptor发送完整的proposal与提案编号
  Acceptor收到proposal后，将其与自己批准过的最大的提案编号maxN进行比较：
     - 如果该提案编号 >= 已批准的最大编号，则accept该提案（==此时执行提案内容但不提交==），随后将情况返回给proposer
     - 如果该提案编号 < 已批准的最大编号，不回应或返回NO

3. 当Proposer收到超过半数的`accept`，它会向所有acceptor发送提案的`commit`请求
    > 因为**超过半数的 acceptor 批准执行了该提案内容，其他没有批准的并没有执行该提案内容**，所以这个时候需要**向未批准的 acceptor 发送提案内容和提案编号并让它无条件执行和提交**，
    而对于前面已经批准过该提案的 acceptor 来说，仅仅需要发送该提案的编号 ，让 acceptor 执行提交就行了。

  如果Proposer没有收到超过半数的`accept`，它会**递增该proposal的编号，然后重新来一次**

**Learn阶段**

![](./../images/zookeeper/paxos-算法演示.jpg)

### Learner学习被选定的value

![](./../images/zookeeper/paxos-learner.png)

## Paxos算法的死循环问题

![](./../images/zookeeper/paxos-活性.png)

基于这种思路，就得到了ZAB协议


## 共识算法

**共识**是可容错系统中的一个基本问题：**即使面对故障，服务器也可以在共享状态上达成一致。**

**共识算法允许一组节点像一个整体一样一起工作，即使其中的一些节点出现故障也能够继续工作下去**，其正确性主要是源于**复制状态机的性质**：**一组Server的状态机计算相同状态的副本**，即使有一部分的Server宕机了它们仍然能够继续运行。

一般通过使用**复制日志**来实现复制状态机
每个Server存储着一份==包括命令序列的日志文==*，状态机会按顺序执行这些命令。
因为**每个日志包含相同的命令，并且顺序也相同**，所以**每个状态机处理相同的命令序列**。
由于状态机是确定性的，所以处理相同的状态，得到相同的输出。

因此共识算法的工作就是==保持复制日志的一致性==

服务器上的共识模块从客户端接收命令并将它们添加到日志中，它与其他服务器上的共识模块通信，以确保即使某些服务器发生故障，每个日志最终也包含相同顺序的请求。
一旦命令被正确地复制，它们就被称为已提交，每个服务器的状态机按照日志顺序处理已提交的命令，并将输出返回给客户端，因此，这些服务器形成了一个**单一的、高度可靠的状态机**

共识算法的特性：
- **安全**
  - 确保在非拜占庭条件下的安全性，包括网络延迟、分区、包丢失、复制和重新排序。
- **高可用**
  - 只要大多数服务器都是可操作的，并且可以相互通信，也可以与客户端进行通信，那么这些服务器就可以看作完全功能可用的。
- **一致性不依赖时序**
  - 在最坏的情况（错误的时钟和极端的消息延迟）下也只会造成可用性问题，而**不会产生一致性问题**。
- 在集群中，**只要有大多数机器响应，就可以完成命令**，不会被少数缓慢的服务器拖慢

## Raft算法

一个Raft集群中包括若干服务器，以典型的5服务器为例，在任意时间，每个服务器一定处于以下三种状态之一：
- Leader：负责发起心跳，响应客户端，创建日志，同步日志。
- Follower：接受 Leader 的心跳和日志同步数据，投票给 Candidate。
- Candidate：选举过程中的临时角色，由 Follower 转化而来，**发起投票参与竞选**。

在Raft集群中，Follower不会发送任何请求，只是响应来自 Leader 和 Candidate 的请求。

**任期**：每次Leader选举开启一个任期，任期用连续的数字表示，看作当前 term 号。如果没有选出 Leader，将会开启另一个任期，并立刻开始下一次选举。
raft 算法保证在给定的一个任期最少要有一个 Leader。

每个节点都会存储当前的 `term` 号，当服务器之间进行通信时会交换当前的 term 号；:
- 如果有服务器发现自己的 term 号比其他人小，那么他会更新到较大的 term 值。
- 如果一个 Candidate 或者 Leader 发现自己的 term 过期了，他会**立即退回成 Follower**。
- 如果一台服务器收到的请求的 term 号是过期的，那么它会拒绝此次请求

### Leader选举

Raft通过心跳机制触发Leader选举：

- Leader向所有Follower周期性发送心跳，保证自己的Leader地位。
- 
- 如果一台服务器能够收到来自 `Leader` 或者 `Candidate` 的有效信息，那么它会一直保持为 `Follower` 状态，并且刷新自己的 `electionElapsed`，重新计时。
- 
- 如果一个 `Follower` 在一个周期内没有收到心跳信息，就叫做选举超时，它就会认为此时没有可用的 `Leader`，并且**开始进行一次选举以选出一个新的 Leader**。
  - 为了开始新的选举，`Follower` 会**自增自己的 term 号，并且转换状态为 Candidate**。然后他会向所有节点发起 `RequestVoteRPC` 请求， Candidate 的状态会持续到以下情况发生：
      1. 该节点赢得选举 
      2. 其他节点赢得选举
      3. 一轮选举后，无人胜出

> 赢得选举的条件是：一个 Candidate 在一个任期内收到了来自集群内的多数选票（N/2+1），就可以成为 Leader。

在 Candidate 等待选票的时候，它可能收到其他节点声明自己是 Leader 的心跳，此时有两种情况：

- 收到的 term 号**大于等于自己**的 term 号，说明对方已经成为 Leader，则自己回退为 Follower。
- 收到的 term 号**小于自己**的 term 号，那么会拒绝该请求并让该节点更新 term。

因此，如果在同一时刻出现了多个`Candidate`，导致没有 Candidate 获得大多数选票。
Raft使用了随机的选举超时时间来避免上述情况：
- Raft为每一个节点**随机分配一个新的选举超时时间**，即**从Follower变为Candidate的等待时间**，这种机制使得出现多个节点同时选举的概率较低

### 日志复制

一旦选出了 Leader，它就开始接受客户端的请求。每一个客户端的请求都包含一条需要被复制状态机执行的命令。

Leader 收到客户端请求后，会生成一个 entry，包含`<index, term, cmd>`，在将这个 entry 添加到自己的日志末尾后，向所有的节点广播该 entry，要求其他服务器复制这条 entry。
如果Follower接受该Entry，则将Entry添加至自己的日志后面，同时返回ACK给Leader
在收到超过半数的ACK响应后，Leader将这个Entry应用到自己的状态机中，并令该Entry为`committed`，并向客户端返回结果

RAFT保证以下两个性质：
- 在两个日志里，有两个 entry 拥有相同的 index 和 term，那么它们一定有相同的 cmd
- 在两个日志里，有两个 entry 拥有相同的 index 和 term，那么它们前面的 entry 也一定相同

Follower通过维护一个`nextIndex`来找到该Follower的下一条日志的索引。

如果Leader与Follower的日志不一致，Leader会强制 Follower 复制自己的日志来处理日志的不一致。
在Leader发送心跳消息时，如果Follower的日志与Leader的日志不一致，心跳会返回失败，Follower的`nextIndex`需要递减，直到回到一个`Leader`与`Follower`日志一致的地方，从而在Follower中添补缺失的日志。

### 安全性

- 选举限制：Leader 需要保证自己存储全部已经提交的日志条目，所以使得日志只会从Leader流向Follower。
  - 每个Candidate在选举时，会带上最后一个Entry的信息，节点收到时将该Entry与自己的进行比较，如果自己的更新，则拒绝投票给它
  > 通过日志的term比较，如果term相同，则更长的index更新 
- 节点崩溃
  - Leader崩溃，会进行选举，选举期间集群不可用
  - Follower或Candidate崩溃，则发给它的RequestVoteRPC 和 AppendEntriesRPC 会失败，而由于**RAFT的请求是幂等的**，失败的话会无限重试，等到恢复后即可选择追加还是拒绝Entry
- 时间与可用性
  - Raft的安全性不依赖于时间，系统不能仅仅因为一些事件发生的比预想的快一些或者慢一些就产生错误。
  - 因此，需要满足：`broadcastTime << electionTimeout << MTBF`
    - broadcastTime（0.5~20ms）：向其他节点并发发送消息的平均响应时间
    - electionTimeout（10~500ms）：选举超时时间
    - MTBF（mean time between failures，一般为一两个月）：单台机器的平均健康时间
  - broadcastTime应该比electionTimeout小一个数量级，为的是使Leader能够持续发送心跳信息（heartbeat）来阻止Follower开始选举；
  - electionTimeout也要比MTBF小几个数量级，为的是使得系统稳定运行。当Leader崩溃时，大约会在整个electionTimeout的时间内不可用；我们希望这种情况仅占全部时间的很小一部分。




# 注册中心

## 要成为注册中心，至少需要具备哪些条件

1. 高可用
2. 高并发
3. 服务注册功能
   - 就是将**提供某个服务的模块的信息**(通常是这个服务的ip和端口)注册到公共的组件上去 
4. 服务发现功能
   - 新注册的这个服务模块能够及时的被其他调用者发现。不管是服务新增和服务删减都能实现自动发现。 
5. 服务健康检查功能
   - 能够自动检测服务提供方是否正常运行 

## ZK是怎样实现注册中心的功能的？

简单来讲，zookeeper是一个分布式文件系统，可以充当一个**服务注册表（Service Registry）**，每当一个服务提供者部署后，都要将自己的服务注册到ZK的某一路径上：`/{service}/{version}/{ip:port}`，从而让多个服务提供者形成一个集群，**让服务消费者通过服务注册表获取具体的服务访问地址（IP+端口）去访问具体的服务提供者**

在zookeeper中，进行服务注册，实际上就是在zookeeper中创建了一个znode节点，该节点存储了该服务的IP、端口、调用方式(协议、序列化方式)等，以供服务消费者获取节点中的信息，从而定位到服务提供者真正网络拓扑位置以及得知如何调用。

## ZK是如何发现服务下线的

zookeeper提供了**心跳检测**功能，它会**定时向各个服务提供者发送一个请求（实际上建立的是一个 socket 长连接）**，如果长期没有响应，服务中心就认为该服务提供者已经“挂了”，并将其剔除。

服务消费方会监听zookeeper相应的路径，一旦路径上的数据有任何的变化，zookeeper就会将新的服务列表发送给消费方，消费方在获取到数据后，刷新本地缓存的列表。

## 每一个请求过来都会打到注册中心吗？（会话缓存？）

不一定，实际上，消费者在从ZK从获得服务调用方的调用路径之后，就可以将其缓存在本地，每次需要调用这个服务时，直接从缓存中取即可。
当然，这样做的必要条件是：**ZooKeeper的ZNODE的监听机制**，服务消费者会去监听相应路径（`/HelloWorldService/1.0.0/{ip:port}`），**一旦路径上的数据有任务变化，zookeeper都会通知服务消费方、服务提供者地址列表已经发生改变，从而进行更新**


## 为什么不用redis作为注册中心

Zookeeper能保证CP，Eureka能保证AP，而Redis集群两种情况都没有很好的解决

## Nacos和ZK的区别，如何选择


