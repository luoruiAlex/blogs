## 数据模型
### 文件系统
- Zookeeper 会维护一个具有层次关系的数据结构，它非常类似于一个标准的文件系统。
- 树形结构的每个节点都被称作 Znode。Znode通过路径引用，如同Unix中的文件路径。路径必须是绝对路径，因此他们必须由斜杠字符来开头。除此以外，它们必须是唯一的
### Znode
- stat 状态信息, 描述该Znode的版本, 权限等信息
- data 与该Znode关联的数据，只管理调度数据，不存储大数据或大量数据。最大1M
- children 该Znode下的子节点
### 节点类型
- 创建时即确定节点类型，不能改变
- 根据是否依赖会话(session）生成区分。客户端和ZooKeeper服务器的一次连接称为一次会话。
- 临时节点(Ephemeral Node)
  - 生命周期依赖于创建它们的会话。一旦会话结束，临时节点将被自动删除，当然也可以手动删除。
  - 虽然每个临时的Znode都会绑定到一个客户端会话，但他们对所有的客户端还是可见的。
  - ZooKeeper的**临时节点不允许拥有子节点**
- 永久节点(Persistent Node)
  - 生命周期不依赖于会话，并且只有在客户端显式地执行删除操作的时候，他们才能被删除
- 顺序节点
  - 当创建Znode的时候，用户可以请求在ZooKeeper的路径结尾添加一个递增的计数。这个计数对于此节点的父节点来说是唯一的，当客户端请求创建这个节点A后，ZooKeeper会根据父节点的zxid状态，为这个A节点编写一个全目录唯一的编号（这个编号只会一直增长）。这样的节点称为顺序节点
  - 临时节点与永久节点都可以成为顺序节点
    - PERSISTENT-持久化目录节点
    - PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点
    - EPHEMERAL-临时目录节点
    - EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点
### 数据访问
- 每个节点存储的数据要被**原子性的操作**
- 每一个节点都拥有自己的ACL，这个列表规定了用户的权限，即限定了特定用户对目标节点可以执行的操作：CREATE READ(读取节点数据和子节点列表) WRITE DELETE ADMIN(设置节点的ACL)
### Zxid
- 致使ZooKeeper节点状态改变的每一个操作都将使节点接收到一个Zxid格式的时间戳，并且这个时间戳全局有序
- Zxid是一个64为的数字，它高32位是epoch用来标识Leader关系是否改变，每次一个Leader被选出来，它都会有一个新的epoch。低32位是个递增计数
- ZooKeeper的每个节点维护者两个Zxid值
  - cZxid 节点的创建时间所对应的Zxid格式时间戳
  - mZxid 是节点的修改时间所对应的Zxid格式时间戳
### 版本号
- 版本号是用来记录节点数据或者是节点的子节点列表或者是权限信息的修改次数
- 对节点的每一个操作都将致使这个节点的版本号增加
- 每个节点维护着三个版本号
  - version：节点数据版本号
  - cversion：子节点版本号
  - aversion：节点所拥有的ACL版本号
### watcher机制
- ZooKeeper允许用户在指定节点上注册一些Watcher，当数据节点发生变化的时候，ZooKeeper服务器会把这个变化的通知发送给感兴趣的客户端
- 一般来说，ZooKeeper会向客户端发送且仅发送一条通知。

## 系统架构
- Leader 负责投票的发起和决议，更新系统状态
  - **只能有一台机器充当Leader的角色，只有Leader才有权力发起更改数据的操作**，Follower如果接收到了更改数据的请求，需要转交给Leader来处理
- Learner
  - Follower 接收客户请求并向客户端返回结果，在选举过程中参与投票
  - Observer 接收客户端链接并将请求转发给leader。不参与投票，只同步Leader的状态。Observer的目的是为了扩展系统，提高读取速度
- Client 发起请求
- 流程
  - Leader接受到一个更改数据的请求后，会广播消息通知对某个节点进行某项指令
  - Follower按照Leader的要求执行完任务之后会发送完成消息给Leader
  - Leader收到了半数以上的Follower的确认消息，就判定该操作已生效，广播消息通知指令生效
  
## Zab协议
- 基于Paxos算法基础上扩展改造而来
### 两种模式
- 恢复模式(选主)
  - ，大多数Server完成了和 Leader的状态同步以后，恢复模式就结束了
- 广播模式(同步)
  - Leader节点与Follower节点使用心跳检测来感知对方的存在
### QuorumCnxManager
- 每台服务器在启动的过程中，会启动一个QuorumPeerManager，负责各台服务器之间的底层Leader选举过程中的网络通信
- 消息队列
  - recvQueue：消息接收队列，用于存放那些从其他服务器接收到的消息
  - queueSendMap：消息发送队列，用于保存那些待发送的消息，按照SID进行分组
  - senderWorkerMap：发送器集合，每个SenderWorker消息发送器，都对应一台远程Zookeeper服务器，负责消息的发送，也按照SID进行分组
  - lastMessageSent：最近发送过的消息，为每个SID保留最近发送过的一个消息
- 建立连接
  - Zookeeper集群中的所有机器都需要两两建立起网络连接
  - 为了避免两台机器之间重复地创建TCP连接，Zookeeper**只允许SID大的服务器主动和其他机器建立连接**，否则断开连接
  - 一旦连接建立，就会根据远程服务器的SID来创建相应的消息发送器SendWorker和消息接收器RecvWorker，并启动
- 消息接收与发送
  - 在SendWorker中，一旦Zookeeper发现针对当前服务器的消息发送队列为空，那么此时需要从lastMessageSent中取出一个最近发送过的消息来进行再次发送，这是为了解决接收方在消息接收前或者接收到消息后服务器挂了，导致消息尚未被正确处理。同时，Zookeeper能够保证接收方在处理消息时，会对重复消息进行正确的处理

## 选举机制
- 服务器ID：每个服务器有个编号，编号越大在选择算法中的权重越大
- 数据ID：服务器中存放的最大数据ID，值越大说明数据越新，权重越大
- 逻辑时钟(投票次数)：同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断
- 选举状态
  - LOOKING 竞选状态
  - FOLLOWING 随从状态，同步leader状态，参与投票
  - OBSERVING 观察状态，同步leader状态，不参与投票
  - LEADING 领导者状态
- 选举消息
  - id 被选举的leader的SID
  - zxid 被选举的leader的事务ID
  - electionEpoch 逻辑时钟，用来判断多个投票是否在同一轮选举周期中
  - peerEpoch 被选举的leader的epoch
  - state 当前服务器的状态
  - 选举状态
- 启动时期的leader选举
  - 每个Server将做起作为leader来发起投票，每次投票会包含所推举的server的myid和ZXID，使用(myid, ZXID)来表示，发给集群中的其他server
  - 接收来自各个server的投票，收到后，先判断投票的有效性(是否为本轮投票，是否来自LOOKING状态的服务器)
  - 处理投票，将自己的投票和别人的投票进行PK：ZXID较大的服务器优先作为leader(ZXID越大，数据越新，成为Leader的可能性越大，也就越能够保证数据的恢复)，ZXID相同的选择myid大的作为leader。比较完后重新投票。
  - 统计投票，判断是否有过半的的机器收到相同的投票信息，如果有，则认为选出了leader
  - 改变server状态，Follwer改为FOLLOWING，Leader改为LEADING
- 运行时期的leader选举
  - 变更状态，Leader挂机后，剩余的非Observer服务器将自己的状态改为LOOKING，然后进入leader选举过程
  - 剩余基本与启动时期一致。启动时期的ZXID都是0，主要比较myid，而运行时期的ZXID可能不同