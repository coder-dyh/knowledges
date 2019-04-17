# 安装

## macos

1. 安装

```shell
brew install nginx
brew install certbot
# certbot 是为了让 nginx 更好的支持 ssl（ certbot 是一个能够给 https 添加证书
# 软件）
```

certbot 参考：https://certbot.eff.org/lets-encrypt/osx-nginx

https://www.jianshu.com/p/50777176e214

2. 启动服务

```shell
brew services start nginx
```

3. 文件存放路径

```shell
# nginx安装文件目录
/usr/local/Cellar/nginx
# nginx配置文件目录
/usr/local/etc/nginx
# config文件目录　
/usr/local/etc/nginx/nginx.conf
# 系统hosts位置
/private/etc/hosts
```

## centos

### 准备工作

1. **安装 zib**

```shell
# 下载 
sudo wget http://www.zlib.net/zlib-1.2.11.tar.gz
# 解压
sudo tar -zxvf zlib-1.2.11.tar.gz
# 进入目录
cd zlib-1.2.11
# 安装
sudo ./configure
sudo make && make install
```

2. **安装 nginx**

```shell
wget -c https://nginx.org/download/nginx-1.12.0.tar.gz -P 存放路径
tar -zxvf nginx-1.12.0.tar.gz
cd nginx-1.12.0
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-pcre=../pcre-8.41
make
make install
# 查找安装路径
whereis nginx
# 启动 nginx
cd /usr/local/nginx/sbin/
./nginx
```

```shell
# 1.报错：
./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using –without-http_gzip_module
option, or install the zlib library into the system, or build the zlib 
library
statically from the source with nginx by using –with-zlib=<path> option
# 解决办法：yum install -y zlib-devel
# 2.报错：
./configure --prefix=/usr/local/pcre
# 解决办法：yum install -y gcc gcc-c++
# 3. 报错：
```

**注意：最好从原来的电脑上拷贝，用 wget 下载的 nxginx 可能不完整；编译 nginx 时它会显示安装路径**

## 常用命令

1. 启动 Nginx

```shell
nginx
```

2. 停止 Nginx

```shell
#快速停止nginx
nginx -s quit
# 安全关闭nginx
nginx -s quit
```

3. 重启

```shell
# 重新加载配置
nginx -s reload
# 重启 Nginx
nginx -s reopen
```

## 代理

### 正向代理

### 反向代理

在 `location`节点添加如下配置

```
# 代理 http://127.0.0.1:8080 的服务
proxy_pass http://127.0.0.1:8080
# 设置真实ip
proxy_set_header real_ip $remote_addr; # real_ip 设置变量名，可以通过web端获取
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_cookie_path / /;
proxy_set_header Cookie $http_cookie;
client_max_body_size 50m;

```



## 负载均衡

在 http 节点下新增以下内容：

```shell
# 负载均衡配置
    upstream balance {
           # weight 值越大,负载权重越大,请求次数越多             
           # max_fails 允许请求失败的次数，超过失败次数后，转发到下一个服务器，当有max_fails个请求失败，就表示
           # 后端的服务器不可用，默认为1，将其设置为0可以关闭检查   
           # fail_timeout 
           # 指定时间内无响应则失败,在以后的fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
           # down 表示当前server不参与负载
           # backup 其他非backup server都忙的时候，backup server作为备用服务器，将请求转发到backup服务器
           server 192.168.58.149:8080 weight=1 max_fails=2 fail_timeout=30s;
           server 192.168.58.150:8081 weight=1 max_fails=2 fail_timeout=30s;
           server 192.168.58.151:8082 down;
           server 192.168.58.152:8083 backup;
    }
```

## 动静分离

在 server 节点下新增以下内容，其中匹配模式可自己变更，支持正则

```shell
# 动态资源
         location ~ \.(jsp|jspx|do|action)(\/.*)?$ { # 动态请求转发到tomcat服务器，匹配方式可自定义
                   # 设置真实ip
                   proxy_set_header real_ip $remote_addr;  # real_ip 设置变量名，可以通过web端获取
                   proxy_pass http://127.0.0.1:8080;
       }
        # 静态资源 
        location ~ .*\.(js|css|htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$ {
         # 静态资源到nginx服务器下static（具体目录自定义）获取
           root ./static; # 注：前后端分离项目时前端项目一般放在此处
       }
```

参考：https://cloud.tencent.com/developer/article/1119373



## Nginx 核心配置 nginx.conf

### 配置文件节点关系

```json
http {
    server {
        location {
            error_log ...
        }
        upstream {
            
        }
    }
}
	
```

### location 写法

```shell
 = 
 # 开头表示精确匹配
 /
 #但是正则和最长字符串会优先匹配
 /documents/
 #匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索，只有后面的正# 则表达式没有匹配到时，这一条才会采用这一条（换句话说要是后面匹配到了，后面的会覆 # 盖前面的，即后面的优先级高）
 ~ /documents/Abc
# 匹配任何以 /documents/Abc 开头的地址，匹配符合以后，还要继续往下搜索，只有后面# # 的正则表达式没有匹配到时，这一条才会采用这一条（换句话说要是后面匹配到了，后面的# # 会覆盖前面的，即后面的优先级高）
 ^~ /images/
# 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条（通 # 常用于动静分离）
~* \.(gif|jpg|jpeg)$
# 匹配所有以 gif,jpg或jpeg 结尾的请求

```

#### 实际使用

```
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}
# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器
# 非静态文件请求就默认是动态请求，自己根据实际把握
# 毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了
location / {
    proxy_pass http://tomcat:8080/
}
```

### Rewrite 规则

rewrite功能就是，使用`nginx`提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。`rewrite`只能放在`server{},location{},if{}`中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如 `http://seanlook.com/a/we/index.php?id=1&u=str` 只对`/a/we/index.php`重写。语法`rewrite regex replacement [flag];`

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理

### Rewrite 和 location 区别

主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里。它们的执行顺序是：

1. 执行server块的rewrite指令
2. 执行location匹配
3. 执行选定的location中的rewrite指令

如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件。

参考：http://seanlook.com/2015/05/17/nginx-location-rewrite/



## 正则表达式

在正则表达式中，如果直接给出字符，就是精确匹配。

### 简单匹配

- 用`\d`可以匹配一个数字，`\w`可以匹配一个字母或数字，`.`可以匹配任意字符。例如：`\d{3}\s+\d{3,8}`能匹配 010 123456`
- 使用特殊符号如 `/ `，`-`等是需要转义的，使用转义字符`\`转义，即`\/`，`\-`

**注意：对 Java 中的字符串中含`\\`转义时，需要使用`\\\\`，因为在 Java 字符串中的`\`也需要转义，变成`\\`，在正则中又需要对每个`/`转义，所以变成了`\\\\`**

### 高级匹配

要做更精确地匹配，可以用`[]`表示范围，比如：

- `[0-9a-zA-Z\_]`可以匹配一个数字、字母或者下划线；
- `[0-9a-zA-Z\_]+`可以匹配至少由一个数字、字母或者下划线组成的字符串，比如`'a100'`，`'0_Z'`，`'Py3000'`等等；
- `[a-zA-Z\_][0-9a-zA-Z\_]*`可以匹配由字母或下划线开头，后接任意个由一个数字、字母或者下划线组成的字符串，也就是Python合法的变量；
- `[a-zA-Z\_][0-9a-zA-Z\_]{0, 19}`更精确地限制了变量的长度是1-20个字符（前面1个字符+后面最多19个字符）
- 使用`(pdf|jpg|png)`表示从 `pdf、jpg、png`选择一个，使用 `$1`可以引用第一个括号中定义的规则，`$2`为第二个括号中定义的规则，以此类推
- **`^`表示行的开头，`^\d`表示必须以数字开头**
- **`$`表示行的结束，`\d$`表示必须以数字结束**

参考：https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143193331387014ccd1040c814dee8b2164bb4f064cff000



## Keepalive 实现 nginx 高可用

### keepalive 安装

参考：https://www.linuxidc.com/Linux/2017-03/141593p2.htm

### 高可用实现

参考：http://seanlook.com/2015/05/18/nginx-keepalived-ha/



## Http 和 Https 区别

https 协议 = http 协议 + ssl 协议

![image-20181224151948729](/Users/dyh/Library/Application Support/typora-user-images/image-20181224151948729.png)



注意：证书传输过程到客户端时解析是通过客户端的TLS来完成的

参考：https://juejin.im/entry/58d7635e5c497d0057fae036