+ 点对点消息传递：一条消息只能被消费一次，消息被消费一次就从队列中删除。生产者发送一条消息到queue，**只有一个消费者能收到。**
+ 发布-订阅消息传递：消息被持久化到一个topic中，消费者可以订阅一个或者多个topic，消费者可以消费该topic中所有的数据，**同一条数据可以被多个消费者消费，数据被消费后不会立马删除。**
+ Kafka读取消息时间复杂度为O(1)

###### 异步通信：

很多时候消费者不需要立即处理消息，异步机制提供了想向队列中放多少消息就放多少，然后在需要的时候再去处理它们。

###### broker：

Kafka集群包含一个或多个服务器，服务器节点称为broker。**broker存储topic的数据**，如果某topic有N个partition，集群有N个broker，每个broker存储该topic 的一个partition；如果某topic有N个partition，集群有（N+M）个broker，那么有N个broker存储该topic的一个partition，剩下的M个broker不存储该topic的partition数据；如果某topic有N个partition，集群中broker数目少于N个，那么一个broker存储该topic的一个或多个partition。在实际生产环境中尽量避免这种情况的发生，这种情况会容易导致Kafka集群数据不均衡。

###### topic:

每条发布到Kafka集群的消息都有一个类别，这个类别被称为topic。(物理上不同topic的消息分开存储，逻辑上一个topic的消息虽然存于一个或多个broker上但用户只需要指定消息的topic即可生产或消费数据而不必关心数据存于何处)

###### partition：

+ topic的数据分割为一个或多个partition，每个topic至少有一个partition。每个partition的数据使用多个segment文件存储。**partition中的数据是有序的,不同partition间的数据就丢失了数据的顺序，如果topic有多个partition，消费数据时就不能保证数据的顺序，在需要严格保证消息的消费顺序的场景下，需要将partition数目设置为1。**每个partition有多个副本，只有一个作为leader。
+ 引入Replication之后，同一个Partition可能会有多个Replica，而这时需要在这些Replication之间选出一个Leader，Producer和Consumer只与这个Leader交互，其它Replica作为Follower从Leader中复制数据。

![](images\Kafka架构图.png)

> kafka把数据写到partition的时候是属于顺序写磁盘，因此效率非常高（顺序写磁盘效率比随机写内存还要高）

Consumer在从broker读取消息后，可以选择commit，该操作会在Zookeeper中保存该Consumer在该Partition中读取的消息的offset。该Consumer下一次再读该Partition时会从下一条开始读取。如未commit，下一次读取的开始位置会跟上一次commit之后的开始位置相同。当然可以将Consumer设置为autocommit，即Consumer一旦读到数据立即自动commit。

delivery guarantee：

> At most once 　　消息可能会丢，但绝不会重复传输
>
> At least one 　　 消息绝不会丢，但可能会重复传输
>
> Exactly once 　　 每条消息肯定会被传输一次且仅传输一次，很多时候这是用户所想要的。

**Kafka默认保证At least once**，并且允许通过设置Producer异步提交来实现At most once。而Exactly once要求与外部存储系统协作，幸运的是Kafka提供的offset可以非常直接非常容易得使用这种方式。

**Kafka的一个partition只能绑定一个consumer group中的一个consumer，一个consumer可以绑定多个partition，这取决于consumer的消费能力，partion数量一定要大于等于consumer数量，如果consumer数量大于了partition，多出的consumer将消费不到任何消息。**

Kafka不保证total order，只保证了local order，对于total order有需求的AP系统不适用于kafka。

C：一致性 A：可用性 P：

kafka的读写全部作用于leader分区，follower分区仅作为备份存在。

zookeeper的leader节点作用读写，follower节点和observer节点提供读，会将客户端的写请求转发到leader节点。observer与follower节点的区别是observer不参与leader的选举过程，也不参与写操作的过半写成功策略，observer的目的是为了拓展系统，提高读取速度。

zookeeper的leader选举是通过ZAB协议，当ZAB(Zookeeper atomic broadcast)协议选举了新的leader服务器，并且集群中过半的follower机器与leader完成数据同步之后，ZAB就会退出恢复模式。

ZAB协议的leader election选举为了容忍f个replicas的失败需要有2f+1个replicas，需要过多服务器，使得集群吞吐量下降，所以该算法只适合于zookeeper这种共享集群配置的系统中而很少在需要存储大量数据的系统中使用的原因。

kafka的leader election算法通过在zookeeper动态维护一个ISR（in-sync replicas)set，set中的每个replica都与leader完成了数据的同步，只有set中的replica能被选为leader。在这种模式下，对于f+1个replicas，一个kafka topic能在保证不丢失已commit的消息的前提下容忍f个replica失败。set中有1个replica就能选举成功且不丢失commit的消息。

事实上，为了容忍f个replica的失败，majority vote 和 ISR在commit前等待follower同步的replica的数量是一样的，但是ISR需要的总的replica的数量几乎是majority vote的一半。

redis的leader选举过程是通过哨兵机制来决定的。

### Zookeeper写数据

写数据到主节点之后，主节点会先将数据写入内存，接着会把数据发给从节点，当从节点收到数据写进内存之后会发送一个ack消息给主节点(主节点将数据写进自身内存时也会给自己发送一个ack消息)，当主节点收到的ack消息超过居群节点的一半数量以上时，就会发送commit命令给从节点，从节点收到commit命令之后将存在内存中的数据持久化到硬盘之中。此时这个数据才算真正的写入到集群成功。

##### 两阶段提交：

先写进内存，再写进硬盘。写进内存的ack过半时才commit，这样就保证了过半的机器能持久化数据，防止出现同步机器中一台服务器有数据，一台没有的情况。