## PostgreSQL主备切换

## 安装数据库

一主两从操作步骤： 
在三台机器分别按照步骤1-4安装pg数据包 
1、 安装 
./configure –prefix=/usr/pgsql9.3.4 –with-perl –with-openssl –with-pam –without-ldap –with-libxml –with-libxslt –enable-thread-safety –with-wal-blocksize=16 –with-blocksize=16 
make world 
make install-world

2、添加用户 
groupadd postgres 
useradd -g postgres postgres 
passwd postgres

3、修改内核参数 
vim /etc/sysctl.conf

#Kernel paramaters required by PostgreSql 
kernel.shmmni = 4096 
kernel.sem = 250 32000 100 128 
fs.file-max = 65536 
net.ipv4.ip_local_port_range = 1024 65000 
net.core.rmem_default = 1048576 
net.core.rmem_max = 1048576 
net.core.wmem_default = 262144 
net.core.wmem_max = 262144

sysctl -p

vim /etc/security/limits.conf

postgres soft nproc 16384 
postgres hard nproc 16384 
postgres soft nofile 65536 
postgres hard nofile 65536

4、增加.bash_profile环境变量 
export PGHOME=/usr/pgsql9.3.4 
export PATH=PGHOME/bin:PATH:. 
export MANPATH=PGHOME/share/man:MANPATH

 

## 配置主数据库

主：172.18.18.99 
su - postgres 
123 
1》mkdir /data/pgdata/pg_primary 
2》mkdir /data/pgdata/pg_primary_data 
3》/usr/pgsql9.3.4/bin/initdb -D /data/pgdata/pg_primary/ -E UTF8 –locale=C -U postgres -W

4》数据库参数 
postgresql.conf//修改port等等（直接拿99开发机配置库配置文件替换) 
pg_hba.conf //修改如下

  修改pg_hba.conf：（METHOD 和 最后一项ADDRESS）
  # TYPE  DATABASE        USER            ADDRESS                 METHOD  

  # "local" is for Unix domain socket connections only
  local   all             all                                     md5
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  # IPv6 local connections:
  #host    all             all             ::1/128                 trust

  host    all             all              172.18.18.0/24          md5
  host    replication     replica     172.18.18.101/32                 md5
  host    replication     replica     172.18.18.100/32                 md5
  注:后两行作用->增加replica用户，进行同步

  psql -p 3021 -U postgres -d postgres （主，99机器）
  postgres# CREATE ROLE replica login replication encrypted password 'replica'


  另需要修改postgresql.conf：
  port = 3021
  wal_level = hot_standby  # 这个是设置主为wal的主机
  max_wal_senders = 32 # 这个设置了可以最多有几个流复制连接，差不多有几个从，就设置几个
  wal_keep_segments = 256 ＃ 设置流复制保留的最多的xlog数目
  wal_sender_timeout = 60s ＃ 设置流复制主机发送数据的超时时间     
  max_connections = 100 # 这个设置要注意下，从库的max_connections必须要大于主库的          

  启动主：
  /usr/pgsql9.3.4/bin/pg_ctl restart -D /data/pgdata/pg_primary 

  如若要创建表空间及索引空间，参考《pg数据库安装手册.txt》步骤7
6、从库：172.18.18.101 
su - postgres 
123 
1》mkdir /data/pgdata/pg_stand_by 
2》mkdir /data/pgdata/pg_stand_by_data 
3》/usr/pgsql9.3.4/bin/pg_basebackup -F p –progress -D /data/pgdata/pg_stand_by -h 172.18.18.99 -p 3021 -U replica –password 
Password: 
replica 
成功之后，就可以看到这个目录中有文件了

4》cp  /usr/pgsql9.3.4/share/recovery.conf.sample  /data/pgdata/pg_stand_by/recovery.conf 
修改recovery.conf: 
standby_mode = on # 这个说明这台机器为从库 
primary_conninfo = ‘host=172.18.18.99 port=3021 user=replica password=replica’ # 这个说明这台机器对应主库的信息 
recovery_target_timeline = ‘latest’ # 这个说明这个流复制同步到最新的数据

5》配置文件修改 
postgresql.conf//修改port等等（直接拿99开发机配置库配置文件替换） 
另需要修改： 
max_connections = 1000 ＃ 一般查多于写的应用从库的最大连接数要比较大 
hot_standby = on ＃ 说明这台机器不仅仅是用于数据归档，也用于数据查询 
max_standby_streaming_delay = 30s # 数据流备份的最大延迟时间 
wal_receiver_status_interval = 1s # 多久向主报告一次从的状态，当然从每次数据复制都会向主报告状态，这里只是设置最长的间隔时间 
hot_standby_feedback = on # 如果有错误的数据复制，是否向主进行反馈

  pg_hba.conf //修改
   TYPE  DATABASE        USER            ADDRESS                 METHOD  

"local" is for Unix domain socket connections only
  local   all             all                                     md5
   IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  # IPv6 local connections:
  #host    all             all             ::1/128                 trust

  host    all             all              172.18.18.0/24          md5

  启动从库：
  chmod 700 /data/pgdata/pg_stand_by
  /usr/pgsql9.3.4/bin/pg_ctl start -D /data/pgdata/pg_stand_by
 

7、从库：172.18.18.100

　　　　操作同6

 

8.查看进程： 
99机器： 
postgres 21374 21359 0 Aug06 ? 00:00:03 postgres: wal sender process replica 172.18.18.101(43399) streaming 0/5008F38 
postgres 22136 21359 0 Aug06 ? 00:00:03 postgres: wal sender process replica 172.18.18.100(16065) streaming 0/5008F38

100机器： 
ps -ef |grep postgres 
postgres 23111 23107 0 Aug06 ? 00:00:13 postgres: wal receiver process streaming 0/5008F38

101机器： 
postgres 14229 13892 0 Aug06 ? 00:00:10 postgres: wal receiver process streaming 0/5008FD8

postgres=# select * from pg_stat_replication; 
pid | usesysid | usename | application_name | client_addr | client_hostname | client_port | backend_start | state | sent_location | write_location | flush_location | replay_location | sync_priority | sync_state 
——-+———-+———+——————+—————+—————–+————-+——————————-+———–+—————+—————-+—————-+—————–+—————+———— 
21374 | 16384 | replica | walreceiver | 172.18.18.101 | | 43399 | 2015-08-06 16:46:34.428745+08 | streaming | 0/5008FD8 | 0/5008FD8 | 0/5008FD8 | 0/5008FD8 | 0 | async 
22136 | 16384 | replica | walreceiver | 172.18.18.100 | | 16065 | 2015-08-06 17:08:42.798029+08 | streaming | 0/5008FD8 | 0/5008FD8 | 0/5008FD8 | 0/5008FD8 | 0 | async

 

主备区分： 
1》通过自带的函数，是备机则是true 
postgres=# select pg_is_in_recovery(); 
pg_is_in_recovery 
——————- 
f 
(1 row)

2》pg_controldata命令 
[postgres@KFJK pg_primary]$ /usr/pgsql9.3.4/bin/pg_controldata /data/pgdata/pg_primary 
pg_control version number: 937 
Catalog version number: 201306121 
Database system identifier: 6179727005669384190 
Database cluster state: in production

[postgres@localhost pg_stand_by]$ /usr/pgsql9.3.4/bin/pg_controldata /data/pgdata/pg_stand_by
pg_control version number:            937
Catalog version number:               201306121
Database system identifier:           6179727005669384190
Database cluster state:               in archive recovery


登录方式：
psql -p 3021 -U postgres -d postgres （主，99机器）
psql -p 3121 -U postgres -d postgres （从，101机器）
psql -p 3221 -U postgres -d postgres （从，100机器）
 

 

主从切换操作：

1》主库宕机或者测试主备切换情况下停掉主库：/usr/pgsql9.3.4/bin/pg_ctl stop -D /data/pgdata/pg_primary -m fast  
从库会报日志错误信息：
[postgres@localhost pg_log]$ tail -100 postgresql-2015-08-07_000000.csv
2015-08-07 16:55:10.588 CST,,,23894,,55c4726e.5d56,1,,2015-08-07 16:55:10 CST,,0,FATAL,XX000,"could not connect to the primary server: could not connect to server: Connection refused
        Is the server running on host ""172.18.18.99"" and accepting
        TCP/IP connections on port 3021?
",,,,,,,,"libpqrcv_connect, libpqwalreceiver.c:106",""
2》原从库操作（原主库宕机情况下将其作为主库操作）： 
在之前备机上的recovery.conf中配置trigger_file = ‘/data/pgdata/pg_stand_by/trigger.unl’ 
touch /data/pgdata/pg_stand_by/trigger.unl 
修改 pg_hba.conf： 
增加 
host replication replica 172.18.18.99/32 md5 
host replication replica 172.18.18.100/32 md5 
重启从库: /usr/pgsql9.3.4/bin/pg_ctl restart -D /data/pgdata/pg_stand_by 
查看是否切换成功：/usr/pgsql9.3.4/bin/pg_controldata /data/pgdata/pg_stand_by -》Database cluster state: in production 表示是主库 
recovery.conf文件名字变成了recovery.done

3》原主库操作（恢复原主库为从库）： 
cp /usr/pgsql9.3.4/share/recovery.conf.sample /data/pgdata/pg_primary/recovery.conf 
修改recovery.conf： 
recovery_target_timeline = ‘latest’ 
standby_mode = on 
primary_conninfo = ‘host=172.18.18.101 port=3121 user=replica password=replica’ 
修改postgresql.conf文件： 
hot_standby = on 
启动原主库（当前从库）：/usr/pgsql9.3.4/bin/pg_ctl start -D /data/pgdata/pg_primary 
4》修改100机器从库对应的主库信息： 
修改recovery.conf ： 
primary_conninfo = ‘host=172.18.18.101 port=3121 user=replica password=replica’ 
重启从库：/usr/pgsql9.3.4/bin/pg_ctl restart -D /data/pgdata/pg_stand_by -m fast 
5》检查主从是否切换成功： 
在新的主库上执行： 
postgres=# select * from pg_stat_replication; 
pid | usesysid | usename | application_name | client_addr | client_hostname | client_port | backend_start | state | sent_location | write_loca 
tion | flush_location | replay_location | sync_priority | sync_state 
——-+———-+———+——————+—————+—————–+————-+——————————-+———–+—————+———– 
—–+—————-+—————–+—————+———— 
32162 | 16384 | replica | walreceiver | 172.18.18.99 | | 47980 | 2015-08-11 15:16:12.925255+08 | streaming | 0/7002F38 | 0/7    002F38 
| 0/7002F38 | 0/7002F38 | 0 | async 
32181 | 16384 | replica | walreceiver | 172.18.18.100 | | 13258 | 2015-08-11 15:18:28.106803+08 | streaming | 0/7002F38 | 0/7002F38 
| 0/7002F38 | 0/7002F38 | 0 | async 
(2 rows)

  表明切换成功