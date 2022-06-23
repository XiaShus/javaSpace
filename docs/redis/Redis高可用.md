# Redis 高可用

## 主从架构

### 概念

主从架构是指在多个机器中运行 Redis 实例，并且指定主从关系，主库负责写以及将数据同步到从库。从库负责读取数据，从库多的情况下可以有效地降低读写压力，将压力分摊到各个服务器上，来保证 Redis 的高可用。

### 操作步骤

准备三台主机 这里有 192.168.137.128、192.168.137.129、192.168.137.130

分别在三台机器上装上 redis，不会的可以看另一篇文章 [安装Redis](docs/redis/安装Redis.md)



在 128、129主机打开 redis 客户端

**前置**

```bash
# 启动时需要指定这个选项
--protected-mode no
```



**选择主库**

```bash
# 将指定主机当做本机的 master 主库
slaveof 192.168.137.130 6379
# 不需要同步时可以使用该命令关闭主从
slaveof no one
```

**执行命令查看同步情况**

```bash
192.168.137.128:6379> info replication
# Replication
role:slave
master_host:192.168.137.130
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:247534
slave_repl_offset:247534
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:98c84bfb65686cbf560317985645f4122e16b8a0
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:247534
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:34193
repl_backlog_histlen:213342

```



```bash
192.168.137.129:6379> info replication
# Replication
role:slave
master_host:192.168.137.130
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:238053
slave_repl_offset:238053
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:98c84bfb65686cbf560317985645f4122e16b8a0
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:238053
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:34193
repl_backlog_histlen:203861

```



```bash
192.168.137.130:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.137.129,port=6379,state=online,offset=252216,lag=0
slave1:ip=192.168.137.128,port=6379,state=online,offset=252071,lag=1
master_failover_state:no-failover
master_replid:98c84bfb65686cbf560317985645f4122e16b8a0
master_replid2:08f980d0195624d11cd8472ab75780c594cac5df
master_repl_offset:252216
second_repl_offset:33735
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:28018
repl_backlog_histlen:224199
```



**测试主从库同步**

```bash
192.168.137.130:6379> set key1 keyval
OK

192.168.137.128:6379> get key1
"keyval"

192.168.137.129:6379> get key1
"keyval"
```



通常建议设置该属性，来保证从库只能读而不能写

```bash
192.168.137.128:6379> config set slave-read-only yes


192.168.137.128:6379> set key2 val2
(error) READONLY You can't write against a read only replica.
```





## 哨兵模式

### 概念

哨兵是一个单独运行的 redis 实例，哨兵主要是用来监控主从库在线状态，实现判断主从库是否下线，实现主从切换等操作。

### 操作步骤

分别修改 128、129、130上的 sentinel.config 文件

```
sentinel monitor mymaster 192.168.137.130 6379 2
daemonize yes
logfile "./sentinel_log.log"
```

保存之后运行即可

```
./redis-server sentinel.conf --sentinel
```

然后可以测试 哨兵模式下的主从库切换

关闭 130 主库下的服务端

```bash
# 查询 redis 相关进程
[root@xiashu3 redis-6.2.6]# ps -ef|grep redis
root       9594   2213  0 11:24 pts/0    00:00:10 ./src/redis-server *:6379
root       9698      1  0 11:27 ?        00:00:37 ./src/redis-server *:26379 [sentinel]
root      11254   2213  0 13:26 pts/0    00:00:00 grep --color=auto redis

# 此时模拟的是 主库redis 进程崩溃，通常关闭 redis 建议调用命令自行关闭
[root@xiashu3 redis-6.2.6]# kill -9 9594
```

**执行命令查看同步情况**

```bash
192.168.137.128:6379> info replication
# 提示服务器关闭了连接，这个是因为当前实例信息被修改了，需要重新获取。
Error: Server closed the connection
192.168.137.128:6379> info replication
# Replication
# 当前实例角色已经变为主库
role:master
connected_slaves:1
slave0:ip=192.168.137.129,port=6379,state=online,offset=1567812,lag=0
master_failover_state:no-failover
master_replid:dc4aa537b0056700a68a32d94834d47fea0a574e
master_replid2:98c84bfb65686cbf560317985645f4122e16b8a0
master_repl_offset:1567812
second_repl_offset:1565456
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:519237
repl_backlog_histlen:1048576

```

```bash
192.168.137.129:6379> info replication
Error: Server closed the connection
192.168.137.129:6379> info replication
# Replication
role:slave
# 主库已变为 128
master_host:192.168.137.128
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:1565913
slave_repl_offset:1565913
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:dc4aa537b0056700a68a32d94834d47fea0a574e
master_replid2:98c84bfb65686cbf560317985645f4122e16b8a0
master_repl_offset:1565913
second_repl_offset:1565456
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:517338
repl_backlog_histlen:1048576

```



可以再查看一下主从的数据同步，这里就不再赘述了。



**哨兵集群下的主从切换**

PUB/SUB 频道

```bash
192.168.137.130:26379> PSUBSCRIBE *
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "*"
3) (integer) 1
1) "pmessage"
2) "*"
3) "+reboot"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "-role-change"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379 new reported role is master"
1) "pmessage"
2) "*"
3) "+convert-to-slave"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+role-change"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379 new reported role is slave"
1) "pmessage"
2) "*"
3) "+reboot"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+reboot"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "-role-change"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379 new reported role is master"
1) "pmessage"
2) "*"
3) "+role-change"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379 new reported role is slave"
1) "pmessage"
2) "*"
3) "+sdown"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+odown"
4) "master mymaster 192.168.137.128 6379 #quorum 2/2"
1) "pmessage"
2) "*"
3) "+new-epoch"
4) "21"
1) "pmessage"
2) "*"
3) "+try-failover"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+vote-for-leader"
4) "0222f3630408225ebb681d4c4ba55f7a571f3c35 21"
1) "pmessage"
2) "*"
3) "+elected-leader"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+failover-state-select-slave"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+selected-slave"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+failover-state-send-slaveof-noone"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+failover-state-wait-promotion"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "-role-change"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379 new reported role is master"
1) "pmessage"
2) "*"
3) "+promoted-slave"
4) "slave 192.168.137.130:6379 192.168.137.130 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+failover-state-reconf-slaves"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-sent"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-inprog"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+slave-reconf-done"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+failover-end"
4) "master mymaster 192.168.137.128 6379"
1) "pmessage"
2) "*"
3) "+switch-master"
4) "mymaster 192.168.137.128 6379 192.168.137.130 6379"
1) "pmessage"
2) "*"
3) "+slave"
4) "slave 192.168.137.129:6379 192.168.137.129 6379 @ mymaster 192.168.137.130 6379"
1) "pmessage"
2) "*"
3) "+slave"
4) "slave 192.168.137.128:6379 192.168.137.128 6379 @ mymaster 192.168.137.130 6379"
1) "pmessage"
2) "*"
3) "-role-change"
4) "slave 192.168.137.128:6379 192.168.137.128 6379 @ mymaster 192.168.137.130 6379 new reported role is master"
1) "pmessage"
2) "*"
3) "+role-change"
4) "slave 192.168.137.128:6379 192.168.137.128 6379 @ mymaster 192.168.137.130 6379 new reported role is slave"
```



**Java Demo**

```java
@Slf4j
public class RedisMsgPubSubListener extends JedisPubSub {

    @Override
    public void unsubscribe() {
        super.unsubscribe();
    }

    @Override
    public void unsubscribe(String... channels) {
        super.unsubscribe(channels);
    }

    @Override
    public void subscribe(String... channels) {
        super.subscribe(channels);
    }

    @Override
    public void psubscribe(String... patterns) {
        super.psubscribe(patterns);
    }

    @Override
    public void punsubscribe() {
        super.punsubscribe();
    }

    @Override
    public void punsubscribe(String... patterns) {
        super.punsubscribe(patterns);
    }

    @Override
    public void onMessage(String channel, String message) {
        log.info("onMessage: channel[{}], message[{}]", channel, message);
    }

    @Override
    public void onPMessage(String pattern, String channel, String message) {
        log.info("onPMessage: pattern[{}], channel[{}], message[{}]", pattern, channel, message);
    }

    @Override
    public void onSubscribe(String channel, int subscribedChannels) {
        log.info("onSubscribe: channel[{}], subscribedChannels[{}]", channel, subscribedChannels);
    }

    @Override
    public void onPUnsubscribe(String pattern, int subscribedChannels) {
        log.info("onPUnsubscribe: pattern[{}], subscribedChannels[{}]", pattern, subscribedChannels);
    }

    @Override
    public void onPSubscribe(String pattern, int subscribedChannels) {
        log.info("onPSubscribe: pattern[{}], subscribedChannels[{}]", pattern, subscribedChannels);
    }

    @Override
    public void onUnsubscribe(String channel, int subscribedChannels) {
        log.info("channel:{} is been subscribed:{}", channel, subscribedChannels);
    }

    public static void main(String[] args) {
        Jedis jr = null;
        try {
            // redis服务地址和端口号
            jr = new Jedis("192.168.137.130", 26379, 0);
            RedisMsgPubSubListener sp = new RedisMsgPubSubListener();
            // jr客户端配置监听 channel
            jr.subscribe(sp, "+slave", "+reboot", "-role-change", "+convert-to-slave", "+role-change");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (jr != null) {
                jr.disconnect();
            }
        }
    }
}
```