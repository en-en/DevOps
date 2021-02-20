## 哨兵模式

![Cloud Native Core target](../images/redis.png)
# 实现哨兵模式

当主宕机了从接替主成为新的主，宕机的主启动后自动变成了从，其实它和Mysql的双主模式是一样的互为主从；redis哨兵需要用到redis-sentinel程序和sentinel.conf配置文件。

mkdir -p /usr/local/redis
mkdir -p /usr/local/redis/6379
mkdir -p /usr/local/redis/6380
mkdir -p /usr/local/redis/redis_cluster

主配置

 vim redis_6379.conf

复制代码
daemonize yes
pidfile /usr/local/redis/6379/redis_6379.pid
port 6379
tcp-backlog 128
timeout 0
tcp-keepalive 0
loglevel notice
logfile ""
databases 16
save 900 1    ###save
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb   ###dbfile
dir "/usr/local/redis/6379"
masterauth "123456"
requirepass "123456"
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes



 vim sentinel_1.conf

 哨兵文件配置

port 6000
dir "/usr/local/redis/sentinel"
## 守护进程模式
daemonize yesprotected-mode nologfile "/usr/local/sentinel/sentinel.log"


从配置
 vim redis_6380.conf


 daemonize yes
pidfile "/usr/local/redis/6380/redis_6380.pid" 
port 6380
tcp-backlog 128
timeout 0
tcp-keepalive 0
loglevel notice
logfile ""
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/usr/local/redis/6380"
masterauth "123456"
requirepass "123456"
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes

vim sentinel_2.conf
#sentinel端口
port 6000
#工作路径，注意路径不要和主重复
dir "/usr/local/sentinel"
# 守护进程模式
daemonize yesprotected-mode no
# 指明日志文件名
logfile "/usr/local/sentinel/sentinel.log"


注意：

1.应用程序连接到哨兵端口，通过指定不同的master名称连接到具体的主副本。

2.哨兵配置文件中只需要配置主从复制中的主副本ip和端口即可，当主从进行切换时哨兵会自动修改哨兵配置文件中的主副本ip为新在主副本ip。

3.一个哨兵配置文件中可以同时配置监控多个主从复制。

4.单个哨兵就可以用来进行主从故障监控，但是如果只有一个sentinel进程，如果这个进程运行出错，或者是网络堵塞，那么将无法实现redis集群的主备切换（单点问题）;<quorum>这个2代表投票数，当2个sentinel认为一个master已经不可用了以后，将会触发failover，才能真正认为该master已经不可用了。（sentinel集群中各个sentinel也有互相通信，通过gossip协议）;所以合理的配置应该是同时启动多个哨兵进程,并且最好是在不同的服务器中启动。

5.注意mymaster的需要在整个网络环境都是唯一的，哨兵之间会自动通过mastername去建立关联关系只要网络环境是相通的。


启动redis

1.主从都要启动

src/redis-server redis.conf
2.登入到6380建立主从关系

redis-cli -p 6380
slaveof 192.168.137.40 6379

配置哨兵

主从两个哨兵都要启动，还可以通过redis-server方式启动，例如“redis-server sentinel.conf --sentinel”

1.启动哨兵

src/redis-sentinel sentinel.conf
2.登入哨兵(两台哨兵都需要登入执行)，添加主从监控信息

redis-cli -p 6000

sentinel monitor mymaster 192.168.137.40 6379 2
sentinel set mymaster down-after-milliseconds 5000
sentinel set mymaster failover-timeout 15000
sentinel set mymaster auth-pass 123456

启动报错处理

错误1：

WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

两个解决方法(overcommit_memory)

1.  echo "vm.overcommit_memory=1" > /etc/sysctl.conf  或 vi /etcsysctl.conf , 然后reboot重启机器

2.  echo 1 > /proc/sys/vm/overcommit_memory  不需要启机器就生效
overcommit_memory参数说明：
设置内存分配策略（可选，根据服务器的实际情况进行设置）
/proc/sys/vm/overcommit_memory
可选值：0、1、2。
0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
2， 表示内核允许分配超过所有物理内存和交换空间总和的内存

注意：redis在dump数据的时候，会fork出一个子进程，理论上child进程所占用的内存和parent是一样的，比如parent占用 的内存为8G，这个时候也要同样分配8G的内存给child,如果内存无法负担，往往会造成redis服务器的down机或者IO负载过高，效率下降。所 以这里比较优化的内存分配策略应该设置为 1（表示内核允许分配所有的物理内存，而不管当前的内存状态如何）。

这里又涉及到Overcommit和OOM。

什么是Overcommit和OOM
在Unix中，当一个用户进程使用malloc()函数申请内存时，假如返回值是NULL，则这个进程知道当前没有可用内存空间，就会做相应的处理工作。许多进程会打印错误信息并退出。

Linux使用另外一种处理方式，它对大部分申请内存的请求都回复"yes"，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。这种技术叫做Overcommit。
当内存不足时，会发生OOM killer(OOM=out-of-memory)。它会选择杀死一些进程(用户态进程，不是内核线程)，以便释放内存。

Overcommit的策略
Linux下overcommit有三种策略(Documentation/vm/overcommit-accounting)：
0. 启发式策略。合理的overcommit会被接受，不合理的overcommit会被拒绝。
1. 任何overcommit都会被接受。
2. 当系统分配的内存超过swap+N%*物理RAM(N%由vm.overcommit_ratio决定)时，会拒绝commit。
overcommit的策略通过vm.overcommit_memory设置。
overcommit的百分比由vm.overcommit_ratio设置。

## echo 2 > /proc/sys/vm/overcommit_memory
## echo 80 > /proc/sys/vm/overcommit_ratio

当oom-killer发生时，linux会选择杀死哪些进程
选择进程的函数是oom_badness函数(在mm/oom_kill.c中)，该函数会计算每个进程的点数(0~1000)。
点数越高，这个进程越有可能被杀死。
每个进程的点数跟oom_score_adj有关，而且oom_score_adj可以被设置(-1000最低，1000最高)。
错误2：
WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.

echo 511 > /proc/sys/net/core/somaxconn
错误3：

16433:X 12 Jun 14:52:37.734 * Increased maximum number of open files to 10032 (it was originally set to 1024).


新装的linux默认只有1024，当负载较大时，会经常出现error: too many open files

ulimit -a：使用可以查看当前系统的所有限制值

vim /etc/security/limits.conf

在文件的末尾加上

* soft nofile 65535
* hard nofile 65535
执行su或者重新关闭连接用户再执行ulimit -a就可以查看修改后的结果。 

故障切换机制

1. 启动群集后，群集程序默认会在从库的redis文件中加入连接主的配置

## Generated by CONFIG REWRITE
slaveof 192.168.137.40 6379
2.启动群集之后，群集程序默认会在主从的sentinel.conf文件中加入群集信息

主：

port 26379
dir "/usr/local/redis-6379"
## 守护进程模式
daemonize yes
## 指明日志文件名
logfile "./sentinel.log"
sentinel monitor mymaster 192.168.137.40 6379 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 18000
sentinel auth-pass mymaster 123456
## Generated by CONFIG REWRITE
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 1
sentinel known-slave mymaster 192.168.137.40 6380
sentinel known-sentinel mymaster 192.168.137.40 26380 c77c5f64aaad0137a228875e531c7127ceeb5c3f
sentinel current-epoch 1

从：

#sentinel端口
port 26380
#工作路径
dir "/usr/local/redis-6380"
## 守护进程模式
daemonize yes
## 指明日志文件名
logfile "./sentinel.log"
#哨兵监控的master，主从配置一样，在进行主从切换时6379会变成当前的master端口，
sentinel monitor mymaster 192.168.137.40 6379 1
## master或slave多长时间（默认30秒）不能使用后标记为s_down状态。
sentinel down-after-milliseconds mymaster 5000
#若sentinel在该配置值内未能完成failover操作（即故障时master/slave自动切换），则认为本次failover失败。
sentinel failover-timeout mymaster 18000
#设置master和slaves验证密码
sentinel auth-pass mymaster 123456
#哨兵程序自动添加的部分


## Generated by CONFIG REWRITE]
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 1
###指明了当前群集的从库的ip和端口，在主从切换时该值会改变
sentinel known-slave mymaster 192.168.137.40 6380
###除了当前的哨兵还有哪些监控的哨兵
sentinel known-sentinel mymaster 192.168.137.40 26379 7a88891a6147e202a53601ca16a3d438e9d55c9d
sentinel current-epoch 1



