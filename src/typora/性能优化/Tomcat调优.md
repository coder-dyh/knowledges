# Tomcat调优 

## tomcat运行机制

### server.xml配置简介

**server**

- port 指定一个端口，这个端口负责监听关闭tomcat的请求

- shutdown 指定向端口发送的命令字符串

**service**

- name 指定service的名字

**Connector**

表示客户端和service之间的连接

- port                           指定服务器端要创建的端口号，并在这个断口监听来自客户端的请求

- minSpareThreads    最小备用线程数，tomcat启动时的初始化的线程数。

- maxSpareThreads  如果空闲状态的线程数多于设置的数目，则将这些线程中止，减少这个池中的线程总数。

- enableLookups        如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址

- redirectPort               指定服务器正在处理http请求时收到了一个SSL传输请求后重定向的端口号

- acceptCount              指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理

- connectionTimeout 指定超时的时间数(以毫秒为单位) 

- executor                    连接器使用的线程池名称

- acceptorThreadCount   用于接收连接的线程的数量，默认值是1。一般这个值需要改动的时候是因为该服务器是一个多核CPU，如果是多核 CPU 一般配置为 2

- 1).compression="on" 打开压缩功能

  2).compressionMinSize="2048" 启用压缩的输出内容大小，这里面默认为2KB

  3).noCompressionUserAgents="gozilla, traviata" 对于以下的浏览器，不启用压缩

  4).compressableMimeType="text/html,text/xml"　压缩类型

**Engine**

表示指定service中的请求处理机，接收和处理来自Connector的请求

- defaultHost 指定默认的处理请求的主机名，它至少与其中的一个host元素的name属性值是一样的

**Context**

表示一个web应用程序

- docBase 应用程序的路径或者是WAR文件存放的路径
- path 表示此web应用程序的url的前缀，这样请求的url为http://localhost:8080/path/
- reloadable 这个属性非常重要，如果为true，则tomcat会自动检测应用程序的/WEB-INF/lib 和/WEB-INF/classes目录的变化，自动装载新的应用程序，我们可以在不重起tomcat的情况下改变应用程序

**host**

表示一个虚拟主机

- name 指定主机名(IP)
- appBase 应用程序基本目录，即存放应用程序的目录(默认webapps)
- unpackWARs 如果为true，则tomcat会自动将WAR文件解压，否则不解压，直接从WAR文件中运行应用程序

**Logger**

表示日志，调试和错误信息

- className 指定logger使用的类名，此类必须实现org.apache.catalina.Logger 接口
- prefix 指定log文件的前缀
- suffix 指定log文件的后缀
- timestamp 如果为true，则log文件名中要加入时间，如下例:localhost_log.2001-10-04.txt

**Realm**

表示存放用户名，密码及role的数据库

- className 指定Realm使用的类名，此类必须实现org.apache.catalina.Realm接口

**Valve**

功能与Logger差不多，其prefix和suffix属性解释和Logger 中的一样

- className 指定Valve使用的类名，如用org.apache.catalina.valves.AccessLogValve类可以记录应用程序的访问信息
- directory 指定日志存放的地址，默认为logs文件夹

**directory**

指定log文件存放的位置

- pattern 有两个值，common方式记录远程主机名或ip地址，用户名，日期，第一行请求的字符串，HTTP响应代码，发送的字节数。combined方式比common方式记录的值更多

XML配置关系：

```xml
<Service>
	<Connector />
    <Engine>
        <Host>
            <Valve></Valve>
        </Host>
    </Engine>
</Service>

```



### Tomcat Server处理http请求过程

假设来自客户的请求为：http://localhost:8080/wsota/wsota_index.jsp

1. 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
2. Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应
3. Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机Host
4.  Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
5. localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有Context
6. Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为""的Context去处理）
7. path="/wsota"的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的servlet
8.  Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
9. 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
10. Context把执行完了之后的HttpServletResponse对象返回给Host
11. Host把HttpServletResponse对象返回给Engine
12. Engine把HttpServletResponse对象返回给Connector
13. Connector把HttpServletResponse对象返回给客户browser

请求处理所经过的路径：

![image-20180905043255151](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180905043255151.png)



### 管理

#### 用户配置

在进行具体Tomcat管理之前，先给tomcat添加一个用户，使这个用户有权限来进行管理。 打开conf目录下的tomcat-users.xml文件，在相应的位置添加下面一行： 
​    <user name=”user”password=”user” roles=”standard,manager”/> 
然后重起tomcat，在浏览器中输入http://localhost:8080/manager/，会弹出对话框，输入上面的用户名和密码即可。



## Tomcat线程模型

### Tomcat支持的三种请求处理方式

#### BIO模式

阻塞式I/O操作，表示Tomcat使用的是传统Java I/O操作(即Java.io包及其子包)。Tomcat7以下版本默认情况下是以bio模式运行的，由于每个请求都要创建一个线程来处理，线程开销较大，不能处理高并发的场景，在三种模式中性能也最低。启动tomcat看到如下日志，表示使用的是BIO模式：

![image-20180905050849337](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180905050849337.png)



#### NIO模式

是java SE 1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，它拥有比传统I/O操作(bio)更好的并发运行性能。在tomcat 8之前要让Tomcat以nio模式来运行比较简单，只需要在Tomcat安装目录/conf/server.xml文件中将如下配置

```
<Connector port="8080" protocol="HTTP/1.1"connectionTimeout="20000"redirectPort="8443" />
```

修改成

```
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol"connectionTimeout="20000"redirectPort="8443" />
```

Tomcat8以上版本，默认使用的就是NIO模式，不需要额外修改 



#### apr模式

简单理解，就是从操作系统级别解决异步IO问题，大幅度的提高服务器的处理和响应性能， 也是Tomcat运行高并发应用的首选模式。启用这种模式稍微麻烦一些，需要安装一些依赖库。



### tomcat的NioEndpoint

我们先来简单回顾下目前一般的NIO服务器端的大致实现，借鉴infoq上的一篇文章Netty系列之Netty线程模型中的一张图 

![image-20180905051906815](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180905051906815.png)

一个或多个Acceptor线程，每个线程都有自己的Selector，Acceptor只负责accept新的连接，一旦连接建立之后就将连接注册到其他Worker线程中。 
多个Worker线程，有时候也叫IO线程，就是专门负责IO读写的。一种实现方式就是像Netty一样，每个Worker线程都有自己的Selector，可以负责多个连接的IO读写事件，每个连接归属于某个线程。另一种方式实现方式就是有专门的线程负责IO事件监听，这些线程有自己的Selector，一旦监听到有IO读写事件，并不是像第一种实现方式那样（自己去执行IO操作），而是将IO操作封装成一个Runnable交给Worker线程池来执行，这种情况每个连接可能会被多个线程同时操作，相比第一种并发性提高了，但是也可能引来多线程问题，在处理上要更加谨慎些。tomcat的NIO模型就是第二种。

![image-20180905052059760](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180905052059760.png)



这张图勾画出了NioEndpoint的大致执行流程图，worker线程并没有体现出来，它是作为一个线程池不断的执行IO读写事件即SocketProcessor（一个Runnable），即这里的Poller仅仅监听Socket的IO事件，然后封装成一个个的SocketProcessor交给worker线程池来处理。下面我们来详细的介绍下NioEndpoint中的Acceptor、Poller、SocketProcessor。 
它们处理客户端连接的主要流程如图所示：

![image-20180905052121929](/var/folders/3m/lg512m_94_1d9ll7dh437_q00000gn/T/abnerworks.Typora/image-20180905052121929.png)



图中Acceptor及Worker分别是以线程池形式存在，Poller是一个单线程。注意，与BIO的实现一样，缺省状态下，在server.xml中没有配置<Executor>，则以Worker线程池运行，如果配置了<Executor>，则以基于java concurrent 系列的java.util.concurrent.ThreadPoolExecutor线程池运行。

#### Acceptor

接收socket线程，这里虽然是基于NIO的connector，但是在接收socket方面还是传统的serverSocket.accept()方式，获得SocketChannel对象，然后封装在一个tomcat的实现类org.apache.tomcat.util.net.NioChannel对象中。然后将NioChannel对象封装在一个PollerEvent对象中，并将PollerEvent对象压入events queue里。这里是个典型的生产者-消费者模式，Acceptor与Poller线程之间通过queue通信，Acceptor是events queue的生产者，Poller是events queue的消费者。

#### Poller

Poller线程中维护了一个Selector对象，NIO就是基于Selector来完成逻辑的。在connector中并不止一个Selector，在socket的读写数据时，为了控制timeout也有一个Selector，在后面的BlockSelector中介绍。可以先把Poller线程中维护的这个Selector标为主Selector。 Poller是NIO实现的主要线程。首先作为events queue的消费者，从queue中取出PollerEvent对象，然后将此对象中的channel以OP_READ事件注册到主Selector中，然后主Selector执行select操作，遍历出可以读数据的socket，并从Worker线程池中拿到可用的Worker线程，然后将socket传递给Worker。整个过程是典型的NIO实现。

#### Worker

Worker线程拿到Poller传过来的socket后，将socket封装在SocketProcessor对象中。然后从Http11ConnectionHandler中取出Http11NioProcessor对象，从Http11NioProcessor中调用CoyoteAdapter的逻辑，跟BIO实现一样。在Worker线程中，会完成从socket中读取http request，解析成HttpServletRequest对象，分派到相应的servlet并完成逻辑，然后将response通过socket发回client。在从socket中读数据和往socket中写数据的过程，并没有像典型的非阻塞的NIO的那样，注册OP_READ或OP_WRITE事件到主Selector，而是直接通过socket完成读写，这时是阻塞完成的，但是在timeout控制上，使用了NIO的Selector机制，但是这个Selector并不是Poller线程维护的主Selector，而是BlockPoller线程中维护的Selector，称之为辅Selector。



#### **tomcat8的并发参数控制**

##### acceptCount

请求在被请求处理线程处理之前暂存的队列，acceptCount就是这个队列的最大长度，超过这个长度后面的请求将被拒绝。这个队列主要的作用缓冲请求，提高并发度

##### acceptorThreadCount 

Acceptor线程只负责从上述队列中取出已经建立连接的请求。在启动的时候使用一个ServerSocketChannel监听一个连接端口如8080，可以有多个Acceptor线程并发不断调用上述ServerSocketChannel的accept方法来获取新的连接。参数acceptorThreadCount其实就是使用的Acceptor线程的个数。

##### maxConnections

这里就是tomcat对于连接数的一个控制，即最大连接数限制。一旦发现当前连接数已经超过了一定的数量（NIO默认是10000），上述的Acceptor线程就被阻塞了，即不再执行ServerSocketChannel的accept方法从队列中获取已经建立的连接。但是它并不阻止新的连接的建立，新的连接的建立过程不是Acceptor控制的，Acceptor仅仅是从队列中获取新建立的连接。所以当连接数已经超过maxConnections后，仍然是可以建立新的连接的，存放在上述acceptCount大小的队列中，这个队列里面的连接没有被Acceptor获取，就处于连接建立了但是不被处理的状态。当连接数低于maxConnections之后，Acceptor线程就不再阻塞，继续调用ServerSocketChannel的accept方法从acceptCount大小的队列中继续获取新的连接，之后就开始处理这些新的连接的IO事件了。

##### maxThreads

这个简单理解就算是上述worker的线程数。他们专门用于处理IO事件，默认是200。



## tomcat线程池管理（优化）

使用线程池，用较少的线程处理较多的访问，可以提高tomcat处理请求的能力。使用方式： 
首先。打开/conf/server.xml，增加 

```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="500" minSpareThreads="20" maxIdleTime="60000" prestartminSpareThreads="true" maxQueueSize="100" />
```

- name                  线程名称
- namePrefix        线程前缀
- maxThreads      最大并发连接数（表示最多同时处理X个连接），不配置时默认200，一般建议设置500~ 800 ，要根据自己的硬件设施条件和实际业务需求而定。
- minSpareThreadsTomcat                     启动初始化的线程数，默认值25个
- prestartminSpareThreads                    是否在tomcat初始化的时候就初始化minSpareThreads的值
- minSpareThreadsmaxQueueSize       最大的等待队列数，超过则拒绝请求
- minSpareThreads                                   线程最大空闲时间60秒

然后，修改<connector></connector> 节点，增加executor属性，属性值为线程池的名称。









参考博客：

线程模型原理：https://blog.csdn.net/zz_99/article/details/1596385

https://blog.csdn.net/qq_16681169/article/details/75003640

线程池优化：https://blog.csdn.net/younger_z/article/details/70308044

https://zhidao.baidu.com/question/940756464855990572.html

tomcat调优总结：https://www.cnblogs.com/ddcoder/articles/8284073.html



