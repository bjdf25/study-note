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

先写进内存，再写进硬盘。写进内存的ack过半时才commit，这样就保证了过半的机器能持久化数据，防止出现同步机器中一台服务器有数据，一台没有的情况。