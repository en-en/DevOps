## mysql 双主架构
（1）两台 mysql 都可读写，互为主备，默认只使用一台（masterA）负责数据的写入，另一台（masterB）备用；

（2）masterA 是 masterB 的主库，masterB 又是 masterA 的主库，它们互为主从；

（3）两台主库之间做高可用，采用 keepalived 方案（使用 VIP 对外提供服务）；
