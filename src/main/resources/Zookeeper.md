##### 乐观锁删除：

znode的元数据里有dataVersion信息，dataVersion为节点的操作提供了乐观锁的功能。

delete -v 1 /node

> 该命令表示 /node节点的dataVersion为1时才会被删除。

##### 命令：

ls -r /node:读取/node下面的所有节点，-r 递归 recur

create -e -s /node data

-e:临时节点 -s：序列节点

delete /node

set /node newData

###### acl权限（只允许auth:root/password的session有对该节点cdwra的权限）：

###### 创建节点并设置权限：

create /node abc auth:root:password:cdwra

为当前session设置权限：addauth degest root:password

一个znode包含的信息：

- 业务数据：data
- 元数据：mateData
    - dataVersion:节点内数据的版本，每更新一次数据，version+1.
    - aclVersion:此节点的权限版本。
    - dataLength：节点内数据长度。
    - numChildren：该节点的子节点个数。（注意：不包含子节点的子节点个数）
    - ephemeralOwner：如果该节点是临时节点，该值是当前节点所有者的session id，如果不是临时节点，该值为0.
- 子节点：childNode
- acl权限：
    - c : create 创建权限，允许在该节点下创建子节点。
    - w : write 更新权限，允许更新该节点的业务数据。
    - r :  read 读取权限，允许读取该节点的内容及子节点的列表信息。
    - d : delete 删除权限， 允许删除该节点的子节点。
    - a ：admin 管理者权限，允许对该节点进行acl权限设置。

#### 最常使用的zookeeper客户端：curetor

#### zookeeper通过临时序列节点实现分布式锁：

zxid：一个session的每一步操作都会记录当前zxid。

- 加读锁时会判断比当前序列号小的节点是否为读锁，是则加读锁成功，不是则监听节点，陷入阻塞装填。
- 加写锁时会判断当前序列号是否为最小序列号节点，是则加写锁成功，不是则监听小序列号节点，并陷入阻塞。

必须是临时节点，使得有释放锁的可能性。

#### zookeeper监听机制：

get -w /znode：

>  A session对znode节点进行监听，如果此时有一个B session对znode节点进行操作，则zookeeper服务端会向A session发送通知它所监听的节点/zode发生了改变，注意，只会发送简单的通知如：nodeDelete/nodeChanged，并不会通知具体改变的内容。
>
>  此时Asession就要根据通知进行相关的业务处理。
>
>  注意：Asession对/znode的监听被通知过一次之后就会被取消，所以说如果被通知过之后还要继续对该节点保持监听就应该在业务处理逻辑后继续保持对该节点的监听的操作，如：get -w /znode
>
>  注意：如果Bsession对/znode增加了一个子节点如/znode/sub,则Asession不会收到通知

zookeeper的leader节点作用读写，follower节点和observer节点提供读，会将客户端的写请求转发到leader节点。observer与follower节点的区别是observer不参与leader的选举过程，也不参与写操作的过半写成功策略，observer的目的是为了拓展系统，提高读取速度。

zookeeper的leader选举是通过ZAB协议，当ZAB(Zookeeper atomic broadcast)协议选举了新的leader服务器，并且集群中过半的follower机器与leader完成数据同步之后，ZAB就会退出恢复模式。

ZAB协议的leader election选举为了容忍f个replicas的失败需要有2f+1个replicas，需要过多服务器，使得集群吞吐量下降，所以该算法只适合于zookeeper这种共享集群配置的系统中而很少在需要存储大量数据的系统中使用的原因。

kafka的leader election算法通过在zookeeper动态维护一个ISR（in-sync replicas)set，set中的每个replica都与leader完成了数据的同步，只有set中的replica能被选为leader。在这种模式下，对于f+1个replicas，一个kafka topic能在保证不丢失已commit的消息的前提下容忍f个replica失败。set中有1个replica就能选举成功且不丢失commit的消息。

事实上，为了容忍f个replica的失败，majority vote 和 ISR在commit前等待follower同步的replica的数量是一样的，但是ISR需要的总的replica的数量几乎是majority vote的一半。

redis的leader选举过程是通过哨兵机制来决定的。

### Zookeeper写数据

写数据到主节点之后，主节点会先将数据写入内存，接着会把数据发给从节点，当从节点收到数据写进内存之后会发送一个ack消息给主节点(主节点将数据写进自身内存时也会给自己发送一个ack消息)，当主节点收到的ack消息超过居群节点的一半数量以上时，就会发送commit命令给从节点，从节点收到commit命令之后将存在内存中的数据持久化到硬盘之中。此时这个数据才算真正的写入到集群成功。

##### 两阶段提交：

