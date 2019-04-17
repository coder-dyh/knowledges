# Linux常用命令

## 文件操作

1.文件拷贝：

```shell
cp -Riv hello.txt ~/test/
```

2.查看文件列表：

```shell
ls -Al ./
```

3.创建文件夹或者文件：

```shell
mkdir -p ~/test/gitdemo/demo1
```

4.移动文件或者文件夹/文件重命名（注意重命名文件夹时后面需要加上/）：

```shell
mv /usr/men/* .
mv ./demo1 ch01
mv 源文件地址 新地址
```

5.删除文件或者文件夹：

```shell
rm -rfv ../gitdemo/
```

6.文件解压缩：

```shell
# 仅打包，不压缩
tar -cvf log.tar log2012.log
# 打包后，以 gzip 压缩 
tar -zcvf log.tar.gz log2012.log
# 打包后，以 bzip2 压缩 
tar -jcvf log.tar.bz2 log2012.log
# 查阅上述tar包内有哪些文件
tar -ztvf log.tar.gz
# 将tar包解压缩
tar -zxvf /opt/soft/test/log.tar.gz
```

7.文件查找（find）

1).按文件名匹配	2).按文件类型匹配		3).基于目录深度匹配	4).按时间戳搜索匹配	5).按文件大小匹配

8. 文件权限

- 查看文件权限（`ll`命令）

第一位代表文件类型，2-4位代表所有者user的权限说明，5-7位代表组群group的权限说明，8-10位代表其他人other的权限说明。

**注意：r代表可读权限，w代表可写权限，x代表可执行权限**

- 修改权限

```shell
chmod o hello.txt # 表示给其他人授予写 hello.txt 这个文件的权限
chmod go-rw hello.txt 
#表示删除 hello.txt 中组群和其他人的读和写的权限
	# u 代表所有者（user）
	# g 代表所有者所在的组群（group）
	# o 代表其他人，但不是u和g （other）
	# a 代表全部的人，也就是包括u，g和o 
# r、w、x也有对应的数字：
	# r—4 
	# w—2 
	# x—1
# eg：给“/var/www”这个目录赋予所有人可读可写可执行权限，4+2+1=7。
sudo chmod  -R 777 /var/www 
# 或者
sudo chmod a+rwx hello.txt 
# 注意：+ 表示添加权限，- 表示取消权限


```

参考：https://blog.csdn.net/Axela30W/article/details/78981749

## 文件传输

scp: 

下载到本地：

```shell
scp -r root@10.10.10.10:/opt/soft/mongodb /opt/soft/
# 注意：前者为服务器地址，后者为本地地址
```

上传到远程：

```shell
scp -r /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/scptest
```

ftp:

## 系统设置

1.重启：reboot -n

2.关机：poweroff

3.切换到超级管理员：sudo su

4. 查看系统配置：cat /proc/cpuinfo  查看CPU信息

   参考：https://www.jianshu.com/p/7234592d9e03

5. 

### 环境变量配置

系统启动加载首先加载的配置文件（系统级别）：

```shell
/etc/profile
/etc/paths
```

用户级别的环境变量（只对当前用户有效）：

```shell
~/.bash_profile
~/.bashrc（当打开shell时加载）
```

设置环境变量：

```shell
# 设置PATH: 
export PATH=/usr/localmysql/bin:$PATH
# 使修改后的配置文件立即生效：
source ~/.bash_profile
```

参考：

macos修改环境变量：https://www.jianshu.com/p/e6396fab1879



### 系统服务

#### 添加系统服务

写一个`shell`脚本，然后把它放到`/etc/init.d`这个目录下，再用 `service + 脚本名字`运行即可

**注意：service这个命令往往是即时生效，不用开关机，但是重启后服务会回到默认状态；chkconfig是用于把服务加到开机自动启动列表里，只要启动它，就能自动启动，重启后永久生效即：**

```shell
chkconfig --add COMMAND
chkconfig COMMAND on/off    # 重启后永久生效
```

参考：https://blog.csdn.net/u013554213/article/details/78792686