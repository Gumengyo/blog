title: Docker 安装 mysql 主从复制
author: 顾梦
tags:
  - Docker
  - Mysql
categories:
  - 服务配置
index_img: 'https://cdn.jishuqin.cn//202304192055832.png'
date: 2023-04-19 20:53:00
---
## 安装 mysql 主从复制

### 1. 新建主服务容器实例 3307

```
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

### 2. 新建 my.cnf 文件

进入/mydata/mysql--master/conf目录下新建 my.cnf，粘贴以下内容

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

### 3. 修改好配置后重启 master 实例

```
docker restart mysql-master
```

### 4. 进入mysql-master 容器

```
docker exec -it mysql-master /bin/bash
mysql -uroot -proot
```

### 5. master 容器实例内创建数据同步用户

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

### 6. 查看同步用户权限

通过指令"**select user,host from user;**"查看备份账号IP访问权限是否为'%'，通过"**show grants for slave**"查看复制账户是否拥有REPLICATION CLIENT、REPLICATION SLAVE、SUPER、RELOAD权限；

```
use mysql;

select user,host from user;

show grants for slave;
```

![](https://cdn.jishuqin.cn//image-20230416232437814.png)

如果没有的话使用命令授权

```
grant REPLICATION CLIENT ON *.* TO slave;
grant REPLICATION SLAVE ON *.* TO slave;
grant SUPER ON *.* TO slave;
grant reload on *.* to slave;
FLUSH PRIVILEGES;
```

### 7. 新建从服务容器实例 3308

```
docker run -p 3308:3306 --name mysql-slave \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

### 8. 新建 my.cnf 文件

进入/mydata/mysql-slave/conf目录下新建my.cnf

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

### 9. 修改完配置后重启slave实例

```
docker restart mysql-slave
```

### 10. 在主数据库中查看主从同步状态

```
show master status;
```

![](https://cdn.jishuqin.cn//image-20230416233143501.png)

### 11. 进入mysql--slave容器

```
docker exec -it mysql-slave /bin/bash

mysql -uroot -proot
```

### 12. 在从数据库中配置主从复制

<p class="note note-warning">注意开放端口，或者关闭防火墙，否则从机连接不上主机。</p>

```
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
```

参数说明：

- master_host：主数据库的IP地址；
- master_port：主数据库的运行端口；
- master_user：在主数据库创建的用于同步数据的用户账号；
- master_password：在主数据库创建的用于同步数据的用户密码；
- master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
- master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
- master_connect_retry：连接失败重试的时间间隔，单位为秒。

### 13. 在从数据库中开启主从同步

```
start slave;
```

### 14. 查看从数据库主从同步状态

```
show slave status \G;
```

![](https://cdn.jishuqin.cn//image-20230416233720733.png)

`Slave_IO_Running`和`Slave_SQL_Running`状态为Yes则配置成功。

### 15. 主从复制测试

主机新建库-使用库-新建表-插入数据

![](https://cdn.jishuqin.cn//image-20230416234054073.png)

从机使用库-查看记录

![](https://cdn.jishuqin.cn//image-20230416234203024.png)

<p class="note note-success">主机写入数据，从机能查询到数据，则说明主从复制搭建成功~👌</p>