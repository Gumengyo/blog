title: Docker 配置 redis 集群3主3从
author: 顾梦
tags:
  - Redis
  - Docker
categories:
  - 服务配置
index_img: 'https://cdn.jishuqin.cn//202304201547698.png'
date: 2023-04-20 15:33:00
---
## Docker 配置 redis 集群3主3从

![](https://cdn.jishuqin.cn//202304201540834.png)

### 1. 启动docker服务

<p class="note note-warning">注意开放端口，或者关闭防火墙，否则从机连接不上主机。</p>

```
systemctl start docker
```

### 2. 创建6个docker容器redis实例

#### 1）创建实例 redis-node-1

```
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
```

#### 2）创建实例 redis-node-2

```
docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382
```

#### 3）创建实例 redis-node-3

```
docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383 
```

#### 4）创建实例 redis-node-4

```
docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384
```

#### 5）创建实例 redis-node-4

```
docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385
```

#### 6）创建实例 redis-node-6

```
docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
```

命令解释：

- `docker run` ：创建并运行docker容器实例
- `--name redis-node-6`：容器名字
- `--net host` ：使用宿主机的IP和端口，默认
- `--privileged=true`：获取宿主机root用户权限
- `-v /data/redis/share/redis-node-6:/data`：容器卷，宿主机地址:dockerp内部地址
- `redis:6.0.8`：redis镜像和版本号
- `--cluster-enabled ye`：开启redis集群
- `--appendonly yes`：开启持久化
- `--port 6386`：redis端口号

<p class="note note-success">运行成功，效果如下</p>

![](https://cdn.jishuqin.cn//image-20230420160742483.png)

### 3. 进入容器构建集群关系

进入容器 redis-noe-1

```
docker exec -it redis-node-1 /bin/bash
```

<p class="note note-warning">注意：进入docker容器后才能执行一下命令，且注意自己的真实IP地址</p>

**构建主从关系**

`--cluster-replicas 1` 表示为每个master创建一个slave节点

```
redis-cli --cluster create 192.168.111.147:6381 192.168.111.147:6382 192.168.111.147:6383 192.168.111.147:6384 192.168.111.147:6385 192.168.111.147:6386 --cluster-replicas 1
```

![](https://cdn.jishuqin.cn//image-20230420161646479.png)

![](https://cdn.jishuqin.cn//image-20230420161721416.png)

![](https://cdn.jishuqin.cn/s0yfnhqj.png)

### 4. 查看集群状态

链接进入6381作为切入点，查看节点状态

```
redis-cli -p 6381

cluster info

cluster nodes
```

![](https://cdn.jishuqin.cn//image-20230420162502394.png)

<p class="note note-success">若集群状态跟上图一样，则说明3主3从集群搭建成功啦~👌</p>