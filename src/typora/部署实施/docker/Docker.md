## 常用命令

1. 删除容器：

   ```dockerfile
   批量删除所有容器：docker rm $(docker ps -a -q)
   删除单个容器：docker rm 容器名/容器id
   ```

2. 删除镜像：

   ```dockerfile
   批量删除镜像：docker image rm $(docker image ls -a -q)
   删除单个镜像：docker rmi 镜像名/镜像id
   ```

   **注意：删除镜像时可能会报错：**

   - ```shell
     Error: No such image: dyh/java
     ```

     可能原因：该删除的 docker 镜像有正在使用的容器

     解决办法：关闭容器重新删除

     ```shell
     Error response from daemon: conflict: unable to delete 1f320a251c9f (must be forced) - image is referenced in multiple repositories
     ```

     可能原因：容器之间存在互联

     解决办法：

     切换到超级管理员用户：sudo su

     然后强制删除：docker rmi 镜像名:标签

3. 重命名 docker 镜像：

```dockerfile
docker tag IMAGEID(镜像id) REPOSITORY:TAG（仓库：标签）
```

4. 查看服务启动日志: `docker logs 容器名`
5. 

### 容器相关

1. 启动镜像容器：docker run -i -t -v ~/software:/mnt/software 镜像名/镜像ID  /bin/base
2. 进入容器：docker attach 容器ID
3. 像运行中的容器执行命令：docker exec -i -t 容器ID ls -l
4. 停止容器：docker stop  容器ID
5. 启动已停止的容器：docker start  容器ID
6. 重启容器：docker restart  容器ID 
7. 退出时自动删除容器：docker run -ti --rm centos /bin/bash

## 镜像构建

### 手动构建

```shell
# 将运行中的容器提交为一个镜像
docker commit 6f10e1e023cc basic_centos:1.0.2
docker commit 镜像ID [取的镜像名]
```



### DockerFile 自动构建镜像

![image-20181214180042858](/Users/dyh/Library/Application Support/typora-user-images/image-20181214180042858.png)

```dockerfile
# 使用命令通过 docker 构建镜像：
docker build -t centos_basic .
```



### 搭建 Docker Registry

```shell
docker run -d -p 50000:5000 -v ~/docker/DockerHub:/tmp/registry registry
```



## 构建基础镜像环境

### 基于 centos

1. 安装` wget`

```shell
yum -y install wget
yum -y install setup 
yum -y install perl
```

2. 安装 `gcc`

```shell
yum install gcc make
```

以上参考：https://www.imooc.com/article/17781

3. 安装` vim`工具

```shell
yum -y install vim*
```

4. 安装 `Redis`

```shell
# 下载并解压 Redis 安装包
wget http://download.redis.io/releases/redis-3.2.3.tar.gz
tar -zxvf redis-3.2.3.tar.gz -C /usr/local
# 进入 Redis 目录安装 Redis
cd /usr/localredis-3.2.3/
make && make instal
```

5. 配置`jdk`

6. ```shell
   vim /etc/profile
   # 将下面三行放到配置文件最底部
   export JAVA_HOME=/usr/lib/jvm/java1.8.0_181
   export PATH=$JAVA_HOME/bin:$PATH
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   source /etc/profile
   ```

   

7. 安装 docker

8. ```shell
   # 如果之前安装过 docker 需要先卸载
   sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-selinux \
                     docker-engine-selinux \
                     docker-engine
   # 在新主机上首次安装Docker CE之前，需要设置Docker存储库。之后，您可以从存储
   # 库安装和更新Docker
   sudo yum install -y yum-utils \
     device-mapper-persistent-data \
     lvm2
   sudo yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
       
   # 安装 Docker
   sudo yum install docker-ce
   
   # 启动 Docker 后台服务
   service docker start
   
   # 测试运行 hello-world,由于本地没有hello-world这个镜像，所以会下载一个hello
   # - world的镜像，并在容器内运行。
   docker run hello-world
   
   ```

   添加阿里云官方镜像加速地址教程（找到docker运行的操作系统傻瓜式的复制命令，注意你的加速地址是不能包括<>的，即替换掉<your accelerate address>）：https://help.aliyun.com/document_detail/60750.html?spm=5176.10695662.1996646101.searchclickresult.27e273d4EZZAnE

   从此处获取个人的加速地址：https://cr.console.aliyun.com/cn-hangzhou/mirrors

   安装参考官网教程：https://docs.docker.com/install/linux/docker-ce/centos/

   8. 开启 Docker Remote API

   ```shell
   # 编辑 docker.service 文件，在 [Service] 节点新增 如下内容:
   sudo vim /usr/lib/systemd/system/docker.service
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
   # ExecStart=/usr/bin/dockerd -H fd://
   # 同时注释原来的 ExecStart=/usr/bin/dockerd -H fd://
   
   # 重新读取配置文件
   systemctl daemon-reload
   # 重启 docker 服务
   systemctl restart docker
   
   # 出现如下结果说明成功了：
   ps -ef | grep docker
   root     26208     1  0 23:51 ?        00:00:00 /usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
   ```

   参考：https://blog.csdn.net/farYang/article/details/75949611

## docker 部署

### Registry 搭建

```shell
docker run -d -p 5555:5000 -v ~/docker/tools/registry:/tmp/registry registry
```

### Harbor 搭建

```shell
# 安装 Python
sudo yum install -y python
# 安装docker-compose（harbor的部分组件需要用到它做容器管理）
sudo yum install docker-compose
# 下载 Harbor 的压缩包(tar.gz的安装包，推荐 1.4.0 版本)
https://github.com/goharbor/harbor/
```

参考：

包含 harbor 的简介之类的：

​	http://www.ywnds.com/?p=7958

安装过程中包管理器找不多合适的源安装可参考：

​	 https://blog.csdn.net/qq12547345/article/details/79482468 

现成的教程（推荐）：

​	https://www.wencst.com/archives/1061



### jenkins 安装

```shell
# 下载运行 Jenkins 容器
sudo docker run -d --name jenkins -p 9090:8080 -v ~/dyh/docker:/var/jenkins_home jenkinsci/jenkins:lts
# 查看日志获取初始化密码 
docker logs <container_id>
# 进入 Jenkins 主页面
localhost:9090
# jenkins dind 参考：https://www.wencst.com/archives/733 （ jenkinsci 是通
# 过该地址构建的镜像）
docker run -d -p 8080:8080 -p 50000:50000 -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock --name jenkins jenkinsci
# 新建 Jenkins 用户（如果已经存在则删除）
userdel -r jenkins
useradd -u 1003 -m jenkins # -u 表示用户ID，-m 表示用户名
# 改变 docker.sock 的所有者
chown 1003:1003 /var/run/docker.sock
# 将 docker.sock 给所有用户赋权
chmod 777 /var/run/docker.sock
# 进入运行的 Jenkins 容器看看是否可以执行 docker 命令
docker exec -it jenkins /bin/bash
docker ps 

```

Docker 结合 Jenkins 实现 CI 的 Jenkins 执行 shell 脚本参考：

```shell
# 定义变量
API_NAME="springboot-demo"
API_VERSION="1.0.8"
API_PORT=8080
IMAGE_NAME="springboot-docker-demo"

cd $WORKSPACE/target
cp classes/Dockerfile .
docker build -t $IMAGE_NAME .
# docker push $IMAGE_NAME

cid=$(docker ps | grep "$CONTAINER_NAME" | awk '{print $1}')
if [ "$cid" != "" ]; then
docker rm -f $cid
fi

docker run -d -p $API_PORT:8080 --name $CONTAINER_NAME $IMAGE_NAME

rm -f Dockerfile
```

### Gitlab 安装

```shell
# 下载安装 Gitlab
sudo docker run --detach \
--hostname localhost \
--publish 443:443 --publish 88:80 --publish 66:22 \
--name gitlab \
--restart always \
--volume ~/dyh/gitlab/config:/etc/gitlab \
--volume ~/dyh/gitlab/logs:/var/log/gitlab \
--volume ~/dyh/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```

参考：https://www.jianshu.com/p/24959481340e

### 日志收集系统 ELK

#### 搭建 elasticsearch

```shell
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \ 
elasticsearch -Ecluster.name=es_cluster -Enode.name=es_node1
```

#### 搭建 Logstash

```shell
docker run -it -d --rm logstash:2.4.1 logstash \
-e 'input { stdin {} } output { stdout { } }'
```

注意：上面的 `-e ` 选项表示 Logstash 启动时需要执行的命令（后面的是 logstash 的日志），其中 `input`表示输入，`output` 表示输出，还可以通过配置文件的方式启动：

```
input {
    stdin {
        
    }
}
output {
    stdout {
        
    }
}

```

注意：stdin 表示以标准输入设备的日志为输入源， stdout 表示以终端输出为标准输出源，input 和 output 表示 Logstash 的输入和输出源

使用配置文件启动 Logstash:

```shell
docker run -it --rm -v ~/logstash/logstash.conf:/etc/logstash.conf \
--name logstash logstash -f /etc/logstash.conf
```

重新构建 Logstash 镜像，使其读取指定位置的配置文件（下面为构建的 Dockerfile ）：

```dockerfile
FROM logstash:2.4.1
COPY logstash.conf /etc/
CMD ["logstash", "-f", "/etc/logstash.conf"]
```

#### 搭建 Kibana

```shell
docker run --rm -p 5601:5601 --link elasticsearch:elasticsearch \
-e ELASTICSEARCH_URL=http://elasticsearch:9200 --name kibana kibana
```

#### 搭建 ELK 日志中心

思路：将所有服务的日志全部通过 Linux 环境的 syslog 服务转发到 Logstash，然后存到 

Elasticsearch 中，最后通过 Kibana 展示出来

1. 启动 Elasticsearch（使用以下内容）

```shell
docker run -d -p 9200:9200 -v ~/elasticsearch/data:/usr/share/elasticsearch/data \
--name elasticsearch elasticsearch
```



2. 启动 syslog ，修改配置开起服务并设置日志转发的 tcp 端口，然后重启 syslog 服务

即：`systemctl restart syslog`

```shell
sudo vim /etc/rsyslog.conf
$ModLoad imtcp
$InputTCPServerRun 514
*.* @@localhost:4560
# 注意：前面两个已经有了，取消注释即可，后面一个需要添加，其中 4560 为 Logstash
# 输入组件的端口
```

3. 启动 Logstash ，以下面的配置文件启动

```
input {
    syslog {
        type => "rsyslog"
        port => 4560
    }
}
output {
    elasticsearch {
        hosts => [ "elasticsearch:9200" ]
    }
}
```

```shell
# 启动容器的命令
docker run -d -p 4560:4560 -v ~/logstash/logstash.conf:/etc/logstash.conf \
--link elasticsearch:elasticsearch --name logstash logstash \
logstash -f /etc/logstash.conf
```

4. 启动 Kibana

```shell
docker run -d -p 5601:5601 --link elasticsearch:elasticsearch -e \
ELASTICSEARCH_URL=http://elasticsearch:9200 \
--name kibana kibana
```

注意： 这里的 -e 选项是为了 Kibana 能够连接到 elasticsearch 并获取日志而给 Kibana 容器启动时传入的环境变量，该 url 即是连接到 elasticsearch 的 url

5. 模拟生产日志的服务端（nginx）

```shell
docker run -d -p 8888:80 --log-driver syslog --log-opt syslog-address=tcp://localhost:514 --log-opt tag="nginx" --name nginx nginx
```

注意：此处将 `docker` 容器的日志驱动方式设置为 `syslog`，即使用 `--log-driver` 指定，同时使用`--log-opt tag`指定日志的标签，有助于在 `Logstash`中识别是哪个应用的日志

