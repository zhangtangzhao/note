---
sort: 4
---

# zookeeper

## CAP理论
C：一致性 A：可用性  P：分区容错性
一个分布式系统不可能同时满足这三个基本需要，最多只能同时满足其中两项

| 被抛弃的谁    | 说明  |
| ------- | -------- |
| 放弃P (满足AC) | 将数据和服务都放在一个节点上，避免因网络引起的负面影响，充分保证系统的可用性和一致性。但放弃P意味着放弃了系统的可扩展性 |
| 放弃A (满足PC) | 当节点故障或者网络故障时，受到影响到的服务需要等待一定的时间，因此在等待时间里，系统无法对外提供正常服务，因此是不可用 |
| 放弃C (满足AP) | 系统无法保证数据的实时一致性，但是承诺数据最终会保证一致性。因此存在数据不一致的窗口期，至于窗口期的长短取决于系统的设计 |

TPS：不可能把所有应用全部放到一个节点上，因此需要花在怎样根据业务场景在A和C直接寻求平衡

## BASE理论
即使无法做到强一致性，但分布式系统可以根据自己的业务特点，采用适当的方式来使系统达到最终一致性

Basically Avaliable 基本可用：当分布式系统出现不可预见的故障时，允许损失部分可用性，保障系统的"基本可用"；体现在"时间上的损失"和"功能上的损失"；
Soft state 软状态：允许系统中的数据存在中间状态，即系统的不同节点的数据副本之间的数据同步过程存在延迟，并认为这种延时不会影响系统的可用性
Eventually consistent 最终一致性：所有的数据在经过一段时间的数据同步后，最终能够达到一个一致的状态



## zookeeper的配置参数

| 参数名     | 说明  |
| ---------- | -------- |
| clientPort | 客户端连接server的端口，即对外服务端口，一般为2181 |
| dataDir    | 存储快照文件snapshot的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir，事务日志的写性能直接影响zk的性能 |
| tickTime   | zk中的一个时间单元。zk中所有时间都是以这个时间单元为基础，进行整数倍配置的。例如，session的最小超时时间是2*tickTime |
| dataLogDir   | 事务日志传输目录。尽量给事务日志的输出配置单独的磁盘或是挂载点，这将极大的提升zk性能 |
| globalOutstandingLimit   | 最大请求堆积数。默认1000.zk运行时候，尽管server已经没有空闲来处理更多的客户端请求了，但是还是允许客户端将请求提交到服务器上来，以提高吞吐性能。当然，为了防止server内存溢出，这个请求堆积数还是需要限制下 |
| preAllocSize   | 预先开辟磁盘空间，用于后续写入事务日志。默认是64M，每个事务日志大小就是64M。如果zk的快照频率较大的话，建议适当减小这个参数 |
| snapCount   | 每进行snapCount次事务日志输出后，触发一次快照(snapshot)，此时，zk会生成一个napshot.*文件，同时创建一个新的事务日志文件log.*。默认是100000.(真正的代码实现中，会进行一定的随机数处理，以避免所有服务器在同一时间进行快照而影响性能) |
| traceFile   | 用于记录所有请求的Log,一般调试过程中可以使用，但是生产环境不建议使用，会严重影响性能 |
| maxClientCnxns   | 单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制。请注意这个限制的使用范围，仅仅是单台客户端机器与单台zk服务器之间的连接数的限制，不是针对指定客户端ip，也不是zk集群的连接数限制，也不是单台zk对所有客户端的连接数限制 |
| clientPortAddress   | 对于多网卡的机器，可以为每个ip指定不同的监听端口。默认情况是有有Ip都监听clientPort指定的端口 |
| minSessionTimeoutmaxSessionTimeout   | Session超时时间限制，如果客户端设置的超时时间不在这个范围，那么会被强制设置为最大或最小时间，默认的Session超时时间在2*tickTime ~ 20*tickTime |
| fsync.warningthresholdns   | 事务日志输出时，如果调用fsync方法超过指定的超时时间，那么会在日志中输出警告信息。默认是1000ms |
| autopurge.purgenlnterval   | 在上文中已经提到，3.4.0及之后版本，ZK提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整 数，默认是0，表示不开启自动清理功能，但可以运行bin/zkCleanup.sh来手动清理zk日志。 |
| autopurge.snapRetainCount   | 这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个 |
| electionAlg   | 在 之前的版本中， 这个参数配置是允许我们选择leader选举算法，但是由于在以后的版本中，只会留下一种“TCP-based version of fast leader   election”算法，所以这个参数目前看来没有用了，这里也不详细展开说了。(No Java system property) |
| initLimit   | Follower 在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。Leader允许Follower在initLimit时间内完 成这个工作。通常情况下，我们不用太在意这个参数的设置。如果ZK集群的数据量确实很大了，Follower在启动的时候，从Leader上同步数据的时 间也会相应变长，因此在这种情况下，有必要适当调大这个参数了。默认值为10，即10 * tickTime  (No Java system property) |
| syncLimit   | 在 运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。如果Leader发出心跳包在 syncLimit之后，还没有从Follower那里收到响应，那么就认为这个Follower已经不在线了。注意：不要把这个参数设置得过大，否则可 能会掩盖一些问题，设置大小依赖与网络延迟和吞吐情况。默认为5，即5 * tickTime (No Java system   property) |
| leaderServes   | 默 认情况下，Leader是会接受客户端连接，并提供正常的读写服务。但是，如果你想让Leader专注于集群中机器的协调，那么可以将这个参数设置为 no，这样一来，会大大提高写操作的性能。默认为yes(Java system property:   zookeeper.leaderServes)。 |
| server.x=[hostname]:n:n   | 这 里的x是一个数字，与myid文件中的id是一致的，用来标识这个zk server，大小为1-255。右边可以配置两个端口，第一个端口用于Follower和Leader之间的数据同步和其它通信，第二个端口用于 Leader选举过程中投票通信。Zk启动时，会读取myid中的值，从而得到server.x的配置为本机配置，并且也可以通过这个id找到和其他zk 通信的地址和端口。hostname为机器ip，第一个端口n为事务发送的通信端口，第二个n为leader选举的通信端口，默认为2888:3888。 (No Java system property) |
| group.x=nnnnn[:nnnnn] weight.x=nnnnn   | 对机器分组和权重设置 |
| cnxTimeout   | Leader选举过程中，打开一次连接（选举的server互相通信建立连接）的超时时间，默认是5s。 |
| zookeeper.DigestAuthenticationProvider.superDigest   | ZK权限设置相关 |
| skipACL   | 对所有客户端请求都不作ACL检查。如果之前节点上设置有权限限制，一旦服务器上打开这个开头，那么也将失效。 |
| forceSync   | 这个参数确定了是否需要在事务日志提交的时候调用FileChannel.force来保证数据完全同步到磁盘。 |
| jute.maxbuffer   | 每个节点最大数据量，是默认是1M。这个限制必须在server和client端都进行设置才会生效。 |
| server.x:hostname:n:n:observer   | 配置observer，表示本机是一个观察者（观察者不参与事务和选举，但会转发更新请求给leader）。比如：server.4:localhost:2181:3181:observer |
| peerType=observer   | 结合上面一条配置，表示这个zookeeper为观察者 |

## zokeeper 节点状态属性
| 属性  | 数据结构 | 描述 |
| ----- | -------- | ---- |
| czxid | long | 节点被创建的zxid值 |
| mzxid | long | 节点被修改的zxid值 |
| pzxid | long |  子节点最有一次被修改时的事务ID |
| ctime | long | 节点被创建的时间 |
| mtime | long | 节点被修改的时间 |
| version | long | 节点被修改的版本号 |
| cversion | long | 节点所拥有的子节点被修改的版本号 |
| aversion | long | 节点的ACL被修改的版本号  |
| emphemeralOwner | long | 如果此节点为临时节点，那么它的值为这个节点拥有者的会话ID；否则为0 |
| dataLength | Int | 节点数据域的长度 |
| numChildren | int | 节点拥有的子节点个数 |

## zookeeper 的 ACL
ACL机制：表示scheme:id:permissions，第一个字段表示采用哪一种机制，第二个ID表示用户，permissions表示相关权限(如只读，读写，管理等)
scheme的取值：
world:它下面只有一个Id，叫anyone,world:anyone代表任何人，zookeeper中对所有人有权限的节点就是属于world:anyone的
auth: 它不需要id, 只要是通过authentication 的user 都有权限（zookeeper 支持通过kerberos来进行authencation, 也支持username/password 形式的authentication)
digest: 它对应的id 为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
ip: 它对应的id 为客户机的IP 地址，设置的时候可以设置一个ip 段，比如ip:192.168.1.0/16,表示匹配前16 个bit 的IP 段

## zookeeper客户端常用命令
1. 显示根目录下、文件： ls / 使用ls 命令来查看当前ZooKeeper 中所包含的内容
2. 显示根目录下、文件： ls2 / 查看当前节点数据并能看到更新次数等数据
3. 创建文件，并设置初始内容： create /zk "test" 创建一个新的znode 节点“ zk ”以及与它关联的字符串[-e] [-s] 【-e 零时节点】【-s 顺序节点】
4. 获取文件内容：get /zk 确认znode 是否包含我们所创建的字符串 [watch【] watch 监听】
5. 修改文件内容： set /zk "zkbak" 对zk 所关联的字符串进行设置
6. 删除文件： delete /zk 将刚才创建的znode 删除，如果存在子节点删除失败
7. 递归删除：rmr /zk 将刚才创建的znode 删除，子节点同时删除
8. 退出客户端： quit
9. 帮助命令： help

## zookeeper 集群的角色
Leader 集群工作机制中的核心
事务请求的唯一调度和处理者，保证集群事务处理的顺序性
集群内部服务器的调度者(管理follower，数据同步)

Follower 集群工作机制中的跟随者
处理非事务请求，转发事务请求给Leader
参与事务请求proposal投票
参与leader选举投票


Observer 3.30以上版本提供，和follower功能相同，但不参与任何形式投票
处理非事务请求，转发事务请求给Leader
提高集群非事务处理能力


## zookeeper 消息广播模式

为了保证分区容错性，zookeeper 是要让每个节点副本必须是一致的

zookeeper 中消息广播的具体步骤如下：
1. 客户端发起一个写操作请求
2. Leader 服务器将客户端的request 请求转化为事物proposql 提案，同时为每个proposal 分
配一个全局唯一的ID，即ZXID。
3. leader 服务器与每个follower 之间都有一个队列，leader 将消息发送到该队列
4. follower 机器从队列中取出消息处理完(写入本地事物日志中)毕后，向leader 服务器发送
ACK 确认。
5. leader 服务器收到半数以上的follower 的ACK 后，即认为可以发送commit
6. leader 向所有的follower 服务器发送commit 消息。

zookeeper 采用ZAB 协议的核心就是只要有一台服务器提交了proposal，就要确保所有的服
务器最终都能正确提交proposal。这也是CAP/BASE 最终实现一致性的一个体现。




