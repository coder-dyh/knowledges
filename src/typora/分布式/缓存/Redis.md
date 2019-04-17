# Redis

## 集群部署

### 修改配置文件

```
port  7000                                        //端口     
bind 192.168.186.91                                     //默认ip为127.0.0.1 需要改为其他节点机器可访问的ip 否则创建集群时无法访问对应的端口，无法创建集群
daemonize    yes                               //redis后台运行
pidfile  ./redis_7000.pid          //pidfile文件对应7000,7001,7002,7003,7004,7005 
cluster-enabled  yes                           //开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf   //集群的配置，配置文件首次启动自动生成 7000,7001,7002,7003,7004,7005 
cluster-node-timeout  15000                //请求超时  默认15秒，可自行设置
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志（开启性能极低）　
```

### 安装 Ruby

**macos**：`brew install ruby`

**centos**：`yum -y install ruby ruby-devel rubygems rpm-build`

**注意**：如果还是不行，则尝试安装 `Ruby Redis 接口`：`gem install redis`

### 启动集群

```properties
redis-trib.rb  create  --replicas  1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

**注意**：启动集群时可能会出现如下问题

```properties
>>> Creating cluster
[ERR] Node 192.168.186.91:7000 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
# 原因：Redis 内有节点没有清空所有数据
# 解决：在Redis中执行以下命令
flushall
cluster reset
```

```properties
/usr/local/rvm/gems/ruby-2.3.0/gems/redis-4.0.1/lib/redis/client.rb:119:in `call': ERR Slot 9189 is already busy (Redis::CommandError)
# 可能原因：之前搭建过集群，集群的配置文件cluster-config-file已经存在
# 解决：重新解压一个Redis安装包或者删除指定cluster-config-file文件，文件存放路
# 径在配置文件 dir 项下的路径
```

