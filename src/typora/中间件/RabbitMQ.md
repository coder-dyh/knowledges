# RabbitMQ 使用

## 基础

### **安装**

### **运行**

#### 启动

1. 启动服务：在RabbitMQ安装目录下找到./sbin目录，运行

```shell
./rabbitmq-server -detached
```

注意：-detached 表示后台启动

2. 启动后台：

```shell
rabbitmq-plugins enable rabbitmq_management
```

默认端口 ：15672

### **停止**

这种方法会通知RabbitMQ干净地关闭，并保护好那些持久化队列，运行：

```shell
./sbin/rabbitmqctl stop
```

启动管控台参考：https://blog.csdn.net/spyiu/article/details/24697221

运行和管理参考：https://www.jianshu.com/p/a46b3fe2e9f1



