## 启动 zookeeper

1. 修改 conf 文件夹下面的 `zoo.example.conf`文件名为`zoo.cfg`

2. 编辑配置：

   ```shell
   # 修改快照保存位置
   dataDir = /usr/local/tools/zkSnapshot
   ```

   ```shell
   # 修改日志保存位置：编辑 bin/zkEnv.sh 文件 
   ZOO_LOG_DIR="/usr/local/tools/zkLog"
   ```

## 集群搭建

```shell
server.1=127.0.0.1 :2888:3888
server.2=127.0.0.1 :2889:3889
server.3=127.0.0.1 :2890:3890
# 必须满足格式：server.<id> = <ip>:<port1>:<port2>
```

**注意**：

-  必须在 `dataDir`下创建一个 `myid`文件，内容为 id 编号，集群中每一个节点都必须设置；
- portl：表示 Leader 节点与Follower 节点进行心跳检测与数据同步时所使用的端口
- port2：表示迸行领导选举过程中，用于投票通信的端口。

**添加修改好配置后依次启动各个节点则集群就启动成功了，然后用 `zkServer.sh status`就能查看各个节点状态**