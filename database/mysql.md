## mysql 双主架构
（1）两台 mysql 都可读写，互为主备，默认只使用一台（masterA）负责数据的写入，另一台（masterB）备用；

（2）masterA 是 masterB 的主库，masterB 又是 masterA 的主库，它们互为主从；

（3）两台主库之间做高可用，采用 keepalived 方案（使用 VIP 对外提供服务）；
## 实现

1、master机器上的配置

（1）vim /etc/my.cnf   #修改mysql的配置文件

（2）配置bind-address （配置成master-ip）和 server-id （改成ip后三位即可，也可以改为其他，比如5）以及开启bin-log（默认是已经开启），如下：
bind-address = 0.0.0.0
server-id = 105
（3）创建用于slave机器获取master机器上binlog文件的账号（从机器复制用户）

grant replication slave on *.* to 'xiaobudiu'@'%' identified by 'xiaobudiu123';

（4） flush privileges;  #刷新数据库

（5）show master status; #查看master状态，并记录下binlog日志文件名以及position

2、slave机器上的配置

（1）在两台从机器上分别配置mysql配置文件中的bind-address 和 server-id

192.168.0.106机器上：bind-address = 0.0.0.0
server-id = 106

192.168.0.107机器上：

bind-address = 0.0.0.0
server-id = 107
（2）分别在两台从库上操作
change master to master_host='192.168.0.105', master_port=3306, master_user='xiaobudiu', master_password='xiaobudiu123', master_log_file='mysql-bin.000045', master_log_pos=402;
（3）分别开启两台从库
start slave
（4）验证从库状态
当从库的 IO线程和SQL线程的状态都是yes时，说明主从模式配置成功。

